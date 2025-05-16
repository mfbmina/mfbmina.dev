+++
title = 'Exploring the Rate package and the Token Bucket algorithm'
date = 2025-05-14T19:16:23-03:00
draft = false
+++

At my [Circuit Breaker]({{< relref golang-circuit-breaker >}}) post, I mentioned that nowadays it is common that the application has to communicate with other ones, and with that, traffic control strategies become essential. Recently I've discovered the [Token Bucket,](https://en.wikipedia.org/wiki/Token_bucket) a strategy based on tokens used to control the traffic.

Imagine that you have 5 tickets to a ride, and every new hour you get a new ticket, but you can never exceed the limit of 5. Every time you ride, a ticket is used. So, if you use all your tickets, you can't ride anymore until you get a new one. It is a very interesting algorithm, used by the PIX (a Brazilian instant payment method), for not allowing an attacker to scrape all users' data.

![Token bucket](/img/posts/golang-token-bucket/token_bucket.png)

Translating to code, let's propose a scenario where 10 `goroutines` are initialized and execute something. If you don't know yet about [goroutines and concurrency,]({{< relref introduction-concurrency-go >}}) I recommend my post about it.

```golang
package main

import	"golang.org/x/time/rate"

func main() {
  c := make(chan int)

  for range 10 {
    go doSomething(i, c)
  }

  for range 10 {
    <-c
  }
}

func doSomething(x int, c chan int) {
  fmt.Printf("goroutine %d did something\n", x)

  c <- x
}
```

While running the code above, it is noticeable that the `goroutines` were executed without any control.

![goroutine example](/img/posts/golang-token-bucket/something.gif)

The Token Bucket algorithm comes from the necessity of controlling the execution, and, as always, Go or its community gives us a solution. In this case, it is the [rate](https://pkg.go.dev/golang.org/x/time/rate) package. Using it is quite simple. At first a new `Limitter` is initialized with the maximum number of tokens and a `Limit`, which defines how many tokens are created per second.

```golang
package main

import	"golang.org/x/time/rate"

func main() {
  c := make(chan int)
  r := rate.Limit(1.0) // refreshes 1 token per second
  l := rate.NewLimiter(r, 5) // allows 5 tokens max

  ... // do more stuff
}
```

The coolest thing from the `rate` package is it has three different strategies. The first strategy is called `allow`.

```golang
func doSomethingWithAllow(l *rate.Limiter, x int, c chan int) {
  if l.Allow() {
    fmt.Printf("Allowing %d to run\n", x)
  }

  c <- x
}
```

At this strategy, the execution is allowed if there is a token to be consumed at that moment, and, if not, nothing will happen.

![Allow strategy](/img/posts/golang-token-bucket/allow.gif)

The second one is called `wait` and by the package author, it is probably the most common one of being used.

```golang
func doSomethingWithWait(ctx context.Context, l *rate.Limiter, x int, c chan int) {
  err := l.Wait(ctx)
  if err != nil {
    fmt.Printf("Error waiting for %d: %v\n", x, err)
    c <- x
    return
  }

  fmt.Printf("Allowing %d to run\n", x)
  c <- x
}
```

With `wait`, the `goroutine` will be held until there is a token available for use.

![Wait strategy](/img/posts/golang-token-bucket/wait.gif)

At last, we have the `reserve` strategy. As the name says, you reserve the next available ticket and wait until it is created.

```golang
func doSomethingWithReserve(l *rate.Limiter, x int, c chan int) {
  r := l.Reserve()
  if !r.OK() {
    return
  }

  fmt.Printf("Reserving %d to run\n", x)
  d := r.Delay()
  time.Sleep(d)
  fmt.Printf("Allowing %d to run\n", x)

  c <- x
}
```

This strategy is similar to the `wait`, however, it allows us to control it with more details and know when it is possible to execute again.

![Reserve strategy](/img/posts/golang-token-bucket/reserve.gif)

Besides all that, all these strategies can burn more than one ticket if it is necessary. For example, the `Allow()` function turns into `AllowN(t time.Time, n int)`, where `t` is the time allowed to happen `n` events.

Another cool feature was the possibility of making a simple Circuit Breaker with this package and the type `Sometimes`. You can configure the following parameters:
`First`: first Nth calls will be executed.
`Every`: every Nth call will be executed.
`Inverval`: calls within the time interval.

```golang
package main

import (
  "fmt"

  "golang.org/x/time/rate"
)

func main() {
  s := rate.Sometimes{Every: 2}

  for i := range 10 {
    s.Do(func() { fmt.Printf("Allowing %d to run!\n", i) })
  }
}
```

As we can see, at every two calls, one call is blocked.

![Circuit Breaker](/img/posts/golang-token-bucket/cb.gif)

## Conclusion

The Token Bucket is a fascinating traffic control algorithm since it allows us to block exceeding executions and book future executions. The Go implementation is simple to use and safe to use with multiple `goroutines`.

The `rate` package also provides a simple circuit breaker implementation, preventing us from adding more third-party libs to the code if it is needed to use both methods in our application.

But it is important to say that this package is still experimental and can change the contract between versions, or be discontinued, or even be added to the `stdlib`.

To sell all the examples, access this [repository.](https://github.com/mfbmina/poc_golang_rate)

## Useful links

- [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket)
- [Rate package](https://pkg.go.dev/golang.org/x/time/rate)
- [Introduction to Concurrency in Go]({{< relref introduction-concurrency-go >}})
- [Circuit Breaker]({{< relref golang-circuit-breaker >}})
- [Request timit at PIX (in Portuguese)](https://www.bcb.gov.br/content/estabilidadefinanceira/pix/API-DICT.html#section/Seguranca/Limitacao-de-requisicoes)
