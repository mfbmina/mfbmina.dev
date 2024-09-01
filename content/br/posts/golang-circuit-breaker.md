+++
title = 'Circuit Breaker em aplicações Go'
date = 2024-08-26T20:07:07-03:00
draft = false
tags = ["go", "resiliencia", "performance"]
+++

Nos dias de hoje, é bem comum que nossa aplicação dependa de outras, principalmente se estamos trabalhando em um ambiente de microsserviços. É bem comum que nossa aplicação comece a reportar erros, que ao se investigar, notamos que alguma API de uma equipe parceira ou fornecedor está fora do ar. 

Uma boa prática para aumentar a resiliência da nossa aplicação, é cortar a comunicação com essas aplicações que estão em estado depreciados. Observando outras áreas, absorvermos da Engenharia Elétrica o conceito de Circuit Breaker. Nele é colocado um equipamento, ou disjuntor, que se desliga automaticamente caso alguma falha aconteça. Isso é muito comum em nossas casas, que possuem disjuntores que se desligam sozinhos caso a rede elétrica comece a ficar instável.

Já na computação, o nosso Circuit Breaker é um pouco mais complexo, uma vez que definimos também um estado intermediário. O desenho abaixo explica melhor o funcionamento de um Circuit Breaker:

![Circuit Breaker](/img/posts/circuit_breaker.png)

Por fim, os estados são:
- `open`: não há comunicação entre as aplicações. Ao atingir este estado, um temporizador se inicia para dar tempo do serviço de reestabeler. Ao fim do temporizador, transitamos para `half-open`. 
- `closed`: há comunicação entre as aplicações. A cada requisição feita com falhas, um contador é atualizado. Se for atingido o limite de falhas, transitamos para o circuito para `open`. 
- `half-open`: estado de recuperação até a comunicação poder fluir completamente. Nele um contador de sucessos é atualizado a cada requisição. Se for atingido o número ideal de sucessos, transitamos o circuito para `closed`. Se as requisições falharem, transitamos de volta para `open`.

Bem legal, né? Mas para exemplificar melhor o conceito, que tal fazermos na prática?

Primeiro, vamos construir nosso serviço A. Ele vai ser responsável por receber as requisições, ou seja, ele vai ser o serviço que nossa aplicação depende, o serviço do fornecedor, ou etc. Para facilitar, vamos expor dois endpoints, um `/success` que vai retornar sempre 200 e um `/failure` que vai retornar sempre 500.

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

Já o serviço B vai ser responsável por chamar o serviço A. É ele quem vai construir o nosso circuit breaker. Para nossa sorte, a comunidade de Go já tem a biblioteca [gobreaker](https://github.com/sony/gobreaker) que implementa o padrão! Primeiro, definimos as propriedades do nosso breaker:

```go
var st gobreaker.Settings
st.Name = "Circuit Breaker PoC"
st.Timeout = time.Second * 5
st.MaxRequests = 2
st.ReadyToTrip = func(counts gobreaker.Counts) bool {
	return counts.ConsecutiveFailures >= 1
}
```

Apesar da biblioteca nos permitir customizar mais coisas, vamos focar em três:
- `Timeout`: o tempo que o circuito vai ficar no estado `open`. No nosso caso, foi definido o tempo de 5 segundos.
- `MaxRequests`: quantidade de requisições bem sucedidas antes de ir para `closed`. No nosso exemplo, definimos em 2 requisições.
- `ReadyToTrip`: define a condição para transitar de `closed` para `open`. Para facilitar, vamos dizer que uma falha é suficiente.

Depois já podemos inicializar o breaker e realizar requisições:

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

Podemos notar que o `gobreaker` funciona como um wrapper em uma função. Se a função retornar um erro, ele aumenta a quantidade de erros, se não, aumenta a quantidade de sucessos. Vamos então definir essa função:

```go
func Get(url string) (int, error) {
	r, _ := http.Get(url)

	if r.StatusCode != http.StatusOK {
		return r.StatusCode, fmt.Errorf("failed to get %s", url)
	}

	return r.StatusCode, nil
}
```

E temos nosso serviço Go usando um circuit breaker! Ao utilizar esse padrão, você consegue aumentar a resiliência e a tolerância a falhas dos seus serviços. Podemos notar que ao utilizar a biblioteca, a complexidade foi toda abstraida, tornando muito simples o processo de integramos isso em nosso dia a dia. Se quiser ver o código todo da prova de conceito é só acessar [aqui.](https://github.com/mfbmina/poc_circuit_breaker)

Se tiver curiosidade para conhecer outros padrões de resiliência, o Elton Minetto publicou um ótimo post sobre o [tema](https://eltonminetto.dev/post/2024-08-24-resilience-in-communication-between-microservices-using-the-failsafe-go-lib/)!

Me diga o que você achou dessa postagem nos comentários e fica uma pergunta: você já utilizou circuit breakers antes?
