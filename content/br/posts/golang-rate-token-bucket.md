+++
title = 'Explorando o pacote Rate e o algoritmo de Token Bucket'
date = 2025-05-14T19:16:23-03:00
draft = false
+++

No post sobre [Circuit Breaker]({{< relref golang-circuit-breaker >}}), citei que atualmente é comum que a sua aplicação tenha que se comunicar com outras e com isso, estratégias de controle de tráfego se tornam essenciais. Recentemente descobri o [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket), uma estratégia baseada em tokens para controlar o tráfego.

Imagine que você tenha 5 ingressos para um brinquedo e que a cada hora você ganhe um novo ingresso, mas nunca podendo exceder o limite de 5. A cada ida ao brinquedo, um ticket é usado. Dessa forma, ao utilizar todos seus tickets, você não pode ir ao brinquedo até ganhar um novo ticket. É um algoritmo bem interessante, utilizado por exemplo pelo PIX, para controlar a busca de chaves e evitando que um malfeitor pegue dados de usuários.

![Token bucket](/img/posts/golang-token-bucket/token_bucket.png)

Traduzindo para código, vamos propor um cenário em que 10 `goroutines` sejam inicializadas e vão executar uma ação qualquer.

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

O código acima inicializa diversas `goroutines` para executar uma ação qualquer e espera pelo processamento delas. Caso você ainda não conheça sobre [goroutines e concorrência]({{< relref introduction-concurrency-go >}}), recomendo o meu post sobre o tema. Pelo `.gif` abaixo, pode-se notar que as `goroutines` foram executadas sem nenhum controle maior.

![goroutine example](/img/posts/golang-token-bucket/something.gif)

O algoritmo de Token Bucket vem da necessidade de controlar essa execução e, como sempre, a própia linguagem e/ou a comunidade Go nos fornecem uma solução, neste caso o pacote [rate](https://pkg.go.dev/golang.org/x/time/rate). Seu uso é bem simples, primeiro é necessário inicializar um novo `Limitter`, que recebe a quantidade máxima de tokens disponível e um `Limit`, que é a taxa com quem novos tokens são gerados por segundo. 

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

O legal do pacote `rate` é que ele possui três estrátégias diferentes. A primeira delas é a estratégia de `allow`. 

```golang
func doSomethingWithAllow(l *rate.Limiter, x int, c chan int) {
  if l.Allow() {
    fmt.Printf("Allowing %d to run\n", x)
  }

  c <- x
}
```

Nesta estratégia a execução será permitida se existir um token para ser consumido naquele momento e, caso não exista, nada será executado.  

![Allow strategy](/img/posts/golang-token-bucket/allow.gif)

A segunda estratégia é chamada de `wait` e é provavelmente a estratégia mais comum de ser utilizada, segundo o autor do pacote.

```golang
func doSomethingWithWait(l *rate.Limiter, x int, c chan int) {
  err := l.Wait(context.Background())
  if err != nil {
    fmt.Printf("Error waiting for %d: %v\n", x, err)
    c <- x
    return
  }

  fmt.Printf("Allowing %d to run\n", x)
  c <- x
}
```

Nesta estratégia, a `goroutine` vai esperar até existir um token disponível para ser usado.

![Wait strategy](/img/posts/golang-token-bucket/wait.gif)

Por fim, temos a estratégia de `reserve`. Como o nome diz, você reservar o próximo ticket disponível, e aguarda até o momento dele ser emitido.

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

Esta estratégia é bem parecida com a de `wait`, contudo nos permite controlar com mais detalhes e saber quando é possível executar novamente.

![Reserve strategy](/img/posts/golang-token-bucket/reserve.gif)

Além disso, todas essas estratégias permitem você queimar mais de um ticket, caso seja necessário. Por exemplo, a função `Allow()` vira `AllowN(t time.Time, n int)`, onde `t` é o tempo para acontecer `n` eventos.

Outra funcionalidade que achei bem legal, foi a possiblidade de se fazer um Circuit Breaker simples utilizando esse pacote e o tipo `Sometimes`. Nele é possível configurar os seguintes parâmetros:
- `First`: as primeiras N chamadas serão executadas.
- `Every`: A cada N chamadas executadas, uma será bloqueada.
- `Inverval`: o limite de chamadas no tempo

Segundo a documentação:
```golang
func main() {
  s := rate.Sometimes{Every: 2}
  s.Do(func() { fmt.Println("1") })
  s.Do(func() { fmt.Println("2") })
  s.Do(func() { fmt.Println("3") })
}

// -> Output:
// -> 1
// -> 3
```

## Conclusão

O token bucket é um algoritmo de controle de tráfego extremamente interessante, já que ele permite um barrar código excedente e agendar execuções futuras. A implementação em Go é simples de se utilizar e é seguro ao se utilizar em múltiplas `goroutines`.

O pacote rate ainda provê uma simples implementação de circuit breaker, evitando assim a adição de mais bibliotecas de terceiros em seu código, caso você deseje utilizar os dois métodos em sua aplicação.

Para ver todos os exemplos, acesse este [repositório.](https://github.com/mfbmina/poc_golang_rate)

## Links úteis

- [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket)
- [Pacote Rate](https://pkg.go.dev/golang.org/x/time/rate)
- [Introdução a concorrência em Go]({{< relref introduction-concurrency-go >}})
- [Circuit Breaker]({{< relref golang-circuit-breaker >}})
