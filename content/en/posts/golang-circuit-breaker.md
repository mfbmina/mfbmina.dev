+++
title = 'Circuit Breaker in Go apps'
date = 2024-08-26T20:07:07-03:00
draft = false
tags = ["go", "resilience", "performance"]
+++

Today it is common for our applications to have a couple of dependencies, especially when working in a microservice environment. It isn't rare when our app reports errors, we find out that one dependency is down.

One good practice for improving our resilience is to shut down the communication with those apps that are not behaving well. Looking into other fields, we learned the concept of circuit breakers from electrical engineering, where a switch turns off when a failure happens. In Brazil, all houses have these switches that automatically shut down if our electric network becomes unstable.

In computer science, our circuit breaker is a bit more complex because it also has an intermediary state. The drawing below explains more about how it works:

![Circuit Breaker](/img/posts/circuit_breaker.png)

In short, the possible states are:
- `open`: there is no communication between apps. When reaching this state, a timer starts, allowing   the dependency to re-establish itself. When the timer ends, we move to `half-open`.
- `closed`: there is communication between apps. At each request done with failures, a counter is updated. If we reach the failure threshold, we move the circuit to `open`. 
- `half-open`: it is a healing state until we can work as usual. While on it, if we reach the success threshold we move to `closed`. If the requests keep failing, we move back to `open`.


Pretty cool, right? To explain the concept better, why not create one?

First, let's build our service A. It will be responsible for receiving all requests, in other words, it will be the service that our main app depends on. To simplify, we will expose two endpoints: a `/success` that will always respond with 200 and the `/failure` that will always respond with 500.

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/success", func(w http.ResponseWriter, r *http.Request) { 
    w.WriteHeader(http.StatusOK) })
	http.HandleFunc("/failure", func(w http.ResponseWriter, r *http.Request) { 
    w.WriteHeader(http.StatusInternalServerError) })

	fmt.Println("Server is running at http://localhost:8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Our service B will be responsible for calling service A and will build our circuit breaker. The Go community already has the lib [gobreaker](https://github.com/sony/gobreaker) that already implements the pattern. First of all, we define our breaker properties:

```go
var st gobreaker.Settings
st.Name = "Circuit Breaker PoC"
st.Timeout = time.Second * 5
st.MaxRequests = 2
st.ReadyToTrip = func(counts gobreaker.Counts) bool {
	return counts.ConsecutiveFailures >= 1
}
```

Even tho the lib allows us to customize more properties, we will focus on only three:
- `Timeout`: how long it will be in the `open` state. In this example, we choose five seconds.
- `MaxRequests`: how many successful requests before it goes to `closed`. In this example, we decided on two requests.
- `ReadyToTrip`: defines the condition to move from `closed` to `open`. Simplifying, one failure will be enough.

Now we just init the breaker and send the requests:

```go
cb := gobreaker.NewCircuitBreaker[int](st)

url := "http://localhost:8080/success"
cb.Execute(func() (int, error) { return Get(url) })
fmt.Println("Circuit Breaker state:", cb.State()) // closed!

url = "http://localhost:8080/failure"
cb.Execute(func() (int, error) { return Get(url) })
fmt.Println("Circuit Breaker state:", cb.State()) // open!

time.Sleep(time.Second * 6)
url = "http://localhost:8080/success"
cb.Execute(func() (int, error) { return Get(url) })
fmt.Println("Circuit Breaker state:", cb.State()) // half-open!

url = "http://localhost:8080/success"
cb.Execute(func() (int, error) { return Get(url) })
fmt.Println("Circuit Breaker state:", cb.State()) // closed!
```

We can notice that `gobreaker` works like a wrapper around a function. If the function returns an error, it will increase the failure counter, and if not, it will increase the success counter. Let's define that function:
```go
func Get(url string) (int, error) {
	r, _ := http.Get(url)

	if r.StatusCode != http.StatusOK {
		return r.StatusCode, fmt.Errorf("failed to get %s", url)
	}

	return r.StatusCode, nil
}
```

And that is how we can have a Go app with a circuit breaker! When using this pattern, you can increase the resilience of your app by making it more tolerant of failures from your dependencies. Also, using this lib removed most of the complexity, making it easier to adopt the pattern in our day-to-day apps. If you want to see the code of this proof of concept, check it [here.](https://github.com/mfbmina/poc_circuit_breaker)

If you are still curious about other resilience patterns, Elton Minetto also published a great blog post [about it!](https://eltonminetto.dev/en/post/2024-08-24-resilience-in-communication-between-microservices-using-the-failsafe-go-lib/)

Tell me what you think of this blog post in the comments and one question: have you ever used circuit breakers before?
