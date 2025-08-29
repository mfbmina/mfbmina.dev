+++
title = 'Enriching requests with Traefik'
date = 2022-10-13T09:22:18-03:00
draft = false
+++

Currently, a large part of authentication flows is based on generating a token, which can, for example, use the JWT standard. The frontend then makes requests informing the backend who the user is that is actually making the call. You can see this when the frontend sends the `Authorization` header in its requests.

## Extracting information

It's common for this token to contain user information, like their ID, for example. So when it receives the request, the backend decodes the token to extract this information and then link it to a user in the database. With the user in hand, we execute the desired action. Below, I'll give you an example of a Go service that does exactly this.

```go
package service

import (
  "encoding/json"
  "errors"
  "fmt"
  "net/http"
  "strings"

  "github.com/golang-jwt/jwt"
)

type User struct {
  ID    string `json:"id"`
  Name  string `json:"name"`
  Email string `json:"email"`
}

func main() {
  http.HandleFunc("/", getEmail)

  fmt.Println("Listening on :8081")
  err := http.ListenAndServe(":8081", nil)
  if err != nil {
    fmt.Println(err)
  }
}

func getEmail(w http.ResponseWriter, r *http.Request) {
  // example from https://pkg.go.dev/github.com/golang-jwt/jwt/v4@v4.4.2
  authHeader := r.Header.Get("Authorization")
  tokenStr, err := getToken(authHeader)
  if err != nil {
    fmt.Println(err)
  }

  token, err := jwt.Parse(tokenStr, func(token *jwt.Token) (interface{}, error) {
    if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
      return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
    }

    return []byte("my_secret_key"), nil
  })

  claims, ok := token.Claims.(jwt.MapClaims)
  if !ok || !token.Valid {
    fmt.Println(err)
  }

  // get the user from the ID
  // user := users.GetUserById(claims["user_id"])
  userID := claims["user_id"].(string)
  user := User{ID: userID, Name: "Matheus Mina", Email: "mfbmina@gmail.com"}
  userJSON, _ := json.Marshal(user)
  w.Write(userJSON)
}

func getToken(tokenStr string) (string, error) {
  authHeaderParts := strings.Fields(tokenStr)
  if len(authHeaderParts) != 2 || strings.ToLower(authHeaderParts[0]) != "bearer" {
    return "", errors.New("Not valid")
  }

  return authHeaderParts[1], nil
}
```

This flow works great for monoliths, but not so much for micro-services. The problem is that when you move to a micro-services environment, this logic for opening the token has to be replicated for each new service. If the token format changes, all the micro-services will have to be updated to follow the new token standard.

## Enriching requests

To avoid this problem, we can do something called **request enrichment**. This involves adding more information to the original request, giving more context and information to the backend. A service that does this, for example, is [Cloudflare](https://developers.cloudflare.com/fundamentals/get-started/reference/http-request-headers/), which adds some headers to your request. To perform this enrichment, we can place an intermediary application to open the token and put the request into the headers of the other responses.

A very simple way to do this is to use the middleware mechanisms of [Traefik](https://doc.traefik.io/traefik/). It's a reverse proxy and load balancer that allows us to easily route between our micro-services. Plus, it's open-source and written in Go. Using the middleware idea with Traefik, our architecture would look like this:

Pretty cool, right? To sum it all up, the request cycle will work like this:

1.  The user makes a request to the backend.
2.  Traefik receives the request and holds the original request.
3.  Traefik makes a new request to the middleware.
4.  Traefik gets the response from the middleware and adds the configured header to the original request.
5.  Traefik forwards the original request to the backend service.
6.  The service responds to the user.

## Building the solution

Getting our hands dirty, let's configure our service in Traefik to receive and forward calls to our service.

```yaml
entryPoints:
  web:
    # Listen on port 8081 for incoming requests
    address: :8081

providers:
  # Enable the file provider to define routers / middlewares / services in file
  file:
    directory: /path/to/dynamic/conf

# dynamic config below
http:
  routers:
    # Define a connection between requests and services
    user-service:
      rule: "Path(`/users`)"
      service: user-service

  services:
    # Define how to reach an existing service on our infrastructure
    user-service:
      loadBalancer:
        servers:
        - url: http://private/user-service
```

With this configuration, every request to `/users` will go to our UserService. The idea here is to add a middleware in between, so that it's transparent to the user that the token is being opened. To do this, we'll create another micro-service whose sole responsibility is to open this token and enrich this request.

```go
package middleware

import (
  "errors"
  "fmt"
  "log"
  "net/http"
  "strings"

  "github.com/golang-jwt/jwt"
)

func main() {
  http.HandleFunc("/", setHeaderExample)

  fmt.Println("Listening on :8082")
  err := http.ListenAndServe(":8082", nil)
  if err != nil {
    log.Fatal(err)
  }
}

func setHeaderExample(w http.ResponseWriter, r *http.Request) {
  // example from https://pkg.go.dev/github.com/golang-jwt/jwt/v4@v4.4.2
  authHeader := r.Header.Get("Authorization")
  tokenStr, err := getToken(authHeader)
  if err != nil {
    fmt.Println(err)
  }

  token, err := jwt.Parse(tokenStr, func(token *jwt.Token) (interface{}, error) {
    if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
      return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
    }

    return []byte("my_secret_key"), nil
  })

  claims, ok := token.Claims.(jwt.MapClaims)
  if !ok || !token.Valid {
    fmt.Println(err)
  }

  // Setting the header X-User-Id"
  userID := claims["user_id"].(string)
  w.Header().Add("X-User-Id", userID)
  w.Write([]byte("This response has the X-User-Id header"))
}

func getToken(tokenStr string) (string, error) {
  authHeaderParts := strings.Fields(tokenStr)
  if len(authHeaderParts) != 2 || strings.ToLower(authHeaderParts[0]) != "bearer" {
    return "", errors.New("Not valid")
  }

  return authHeaderParts[1], nil
}
```

Now, we need to add it as a middleware in Traefik:

```yaml
http:
  services:
    # Define how to reach an existing service on our infrastructure
    user-middleware:
      loadBalancer:
        servers:
        - url: http://private/user-middleware

  middlewares:
    user-middleware:
      forwardAuth:
        address: "http://private/user-middleware"
        authResponseHeaders:
          - "X-User-ID"
```

We also need to tell the UserService to use the middleware:

```yaml
http:
  routers:
    # Define a connection between requests and services
    user-service:
      rule: "Path(`/users`)"
      service: user-service
      middlewares:
      - user-middleware
```

The full configuration looks like this:

```yaml
entryPoints:
  web:
    # Listen on port 8081 for incoming requests
    address: :8081

providers:
  # Enable the file provider to define routers / middlewares / services in file
  file:
    directory: /path/to/dynamic/conf

# dynamic config below
http:
  routers:
    # Define a connection between requests and services
    user-service:
      rule: "Path(`/users`)"
      service: user-service
      middlewares:
      - user-middleware
  middlewares:
    user-middleware:
      forwardAuth:
        address: "http://private/user-middleware"
        authResponseHeaders:
          - "X-User-ID"
  services:
    # Define how to reach an existing service on our infrastructure
    user-service:
      loadBalancer:
        servers:
        - url: http://private/user-service
    user-middleware:
      loadBalancer:
        servers:
        - url: http://private/user-middleware
```

And that's it\! The magic is working\! All requests to the UserService will now have the `X-User-Id` header. To finish up, we just need to remove the code that "opens" the token and start reading the information from the header instead. Our handler would look like this:

```go
func getEmailFinal(w http.ResponseWriter, r *http.Request) {
  // get the user from the ID
  // user := users.GetUserById(claims["user_id"])
  userID := r.Header.Get("X-User-Id")
  user := User{ID: userID, Name: "Matheus Mina", Email: "mfbmina@gmail.com"}
  userJSON, _ := json.Marshal(user)
  w.Write(userJSON)
}
```

## Conclusion

By enriching the request, we can simplify our service code, transparently passing useful information to the backend. As you can see, our service's handler is much cleaner, focusing only on what it should actually be doing.

If you liked this post, you can also find me on **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)**, or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
