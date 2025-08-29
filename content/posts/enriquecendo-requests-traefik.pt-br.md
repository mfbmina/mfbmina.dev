+++
title = 'Enriquecendo requests com Traefik'
date = 2022-10-13T09:22:18-03:00
draft = false
+++

Atualmente grande parte dos fluxos de autenticação se baseia em gerar um token, que pode por exemplo usar o padrão JWT, e o frontend faz as requisições informando ao backend quem é o usuário que está de fato realizando a chamada. Isso pode ser observado com as requests do frontend enviando o header `Authorization` nas requests.

![Exemplo do Header Authorization](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/40k9kehupm6ck21njuih.png)

## Extraindo informações
É comum que esse token contenha informações do usuário, como por exemplo o id. Então ao receber a requisição, o backend decodifica esse token para extrair essas informações e assim relacionar com algum usuário do banco de dados. Com o usuário em mãos, executamos a ação desejada. Logo abaixou vou dar um exemplo de um serviço em Go que faz exatamente isso.

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

Esse fluxo funciona muito bem para monolitos, mas para micro-serviços não. O problema é que ao ir para um ambiente de micro-serviços, essa lógica responsável por abrir o token tem que ser replicada para cada um dos serviços novos. Se por acaso o formato do token mudar, todos os micro-serviços vão ter que se atualizar para seguir o padrão novo de token.

## Enriquecendo requisições
Para evitar esse problema, podemos fazer algo chamado de enriquecimento de requests. Isso consiste em adicionar mais informações a request original, dando mais contexto e informações ao backend. Um serviço que faz isso, por exemplo, é o [Cloudflare](https://developers.cloudflare.com/fundamentals/get-started/reference/http-request-headers/) que adiciona alguns headers na sua requisição. Para fazer esse enriquecimento, podemos colocar uma aplicação intermediaria para fazer essa abertura de token e colocar a requisição no header das demais respostas.

Uma maneira bem simples de fazer isso é utilizar os mecanismos de middleware do [Traefik](https://doc.traefik.io/traefik/). Ele é um proxy reverso e load balancer que nos permite de maneira simples fazer roteamento entre os nossos microserviços. Além disso, ele é open-source e escrito em Go. Utilizando a idéia do middleware com o Traefik, a nossa arquitetura ficaria assim:

![Fluxo da solução final](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f0kjbfjlmvm54mdivplq.png)

Bem legal, né? Para resumir tudo, o ciclo da requisição vai funcionar assim:

1. O usuário faz a requisição ao backend
2. O Traefik recebe a requisição e segura a requisição original
3. O Traefik faz uma nova requisição ao middleware
4. O Traefik pega a resposta do middleware e adiciona o header configurado na request original.
5. O Traefik encaminha a request original ao serviço backend
6. O serviço responde o usuário

## Construindo a solução
Colocando a mão na massa, vamos configurar nosso serviço no Traefik receber e encaminhar as chamadas para o nosso serviço.

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

Com essa configuração, todo request para `/users` vai ir para o nosso UserService.  A idéia aqui é adicionar um middleware no meio, de forma que seja transparente para o usuário que o token está sendo aberto. Para isso vamos criar um outro microserviço cuja responsabilidade seja só abrir esse token e enriquecer essa request.

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

Dessa forma, é necessário adicionar o mesmo como um middleware no Traefik:

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

Também vamos dizer para o UserService utilizar o middleware:

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

A configuração completa fica da seguinte forma:

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

E pronto! Mágica funcionando! Todas as requests pro UserService vão ter o header `X-User-Id`. Para finalizar, é só a gente remover o código que "abre" o token e passar a ler a informação vinda do header. Nosso handler ficaria assim:

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

## Conclusão
Ao enriquecer a request podemos simplificar o código dos nossos serviços, repassando informações utéis ao backend de forma transparente. Podemos ver que o handler do nosso serviço ficou bem mais limpo, focando somente no que ele de fato deveria fazer.

Se curtiu o post, você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
