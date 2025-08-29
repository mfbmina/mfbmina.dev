+++
title = 'Introdução à concorrência em Go'
date = 2022-11-01
draft = false
+++

Uma das funcionalidades mais legais de Go é a facilidade de se utilizar concorrência. A linguagem nos fornece as chamadas `goroutines` que são *lighweight threads* gerenciadas pelo própio *runtime* do Go. Concorrência nos permite rodar diversas funções ao mesmo tempo. Isso é extremamente útil caso você queira melhorar a performance de sua aplicação.

## Keyword go
Para utilizar essa funcionalidade, basta colocar a palavra-chave `go` antes de qualquer função e ela automaticamente vai estar rodando de forma concorrente. Para ficar mais fácil de entender, vamos ver um pouco de código. Aqui está implementado o `SleepSort`, é um algoritmo de ordenação que utiliza um `sleep` para poder colocar cada elemento no seu lugar.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	arr := []int{3, 1, 5, 6, 9, 2, 0}

	fmt.Println("Sorting", arr)
	for _, v := range arr {
		go Sort(v)
	}
}

func Sort(x int) {
	time.Sleep(time.Duration(x) * time.Second)
	fmt.Printf("%d ", x)
}
```

Nesse código, temos a função `sort` que dorme por x segundos e imprime o valor desejado.  Assim, ao final da execução de todo mundo, vamos ter impresso o array na ordem certa, correto? Infelizmente a resposta é: não!

![Resposta inicial do código](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/upvgnz9mmdahtamxgm04.png)

## Channels
Como a função roda de forma concorrente ao processo principal, só vemos na saída o que este processo retorna. Uma maneira de solucionar este problema é fazermos os processos concorrentes se comunicarem de alguma forma com o processo principal. Para isso, a linguagem nos fornece o conceito de `channel`. Podemos entender este conceito como um canal, ou conduinte, responsável por enviar dados entre os processos. Vamos atualizar nosso código então para utilizar `channels`.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	arr := []int{3, 1, 5, 6, 9, 2, 0}
	c := make(chan int)

	fmt.Println("Sorting", arr)
	for _, v := range arr {
		go sort(v, c)
	}

	fmt.Println("Sorted:")
	for range arr {
		x := <-c
		fmt.Printf("%d ", x)
	}
}

func sort(x int, c chan int) {
	time.Sleep(time.Duration(x) * time.Second)
	c <- x
}
```

E finalmente temos como resposta:

![Resposta após utilizarmos channels](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w68flkg7z5g7x0lx1g4x.png)

Como podemos notar, existe um operador especial para os `channels` que é o `<-`. Para entender como ele funciona, é só pensar que o dado vai seguir o sentido da seta. Por exemplo, dentro da função `sort` o dado é enviado através do canal usando `c <- x` e na função principal recebemos um valor do canal e atribuímos em uma nova variável utilizando `x := <-c`.

Também é importante dizer que a assinatura da função `sort` foi alterada para receber um `chan`. Esse `chan` então é inicializado na função principal e passado para as `goroutines`.

## Conclusão
Como podemos ver, começar a utilizar concorrência em Go é simples. A linguagem também suporta opções mais avançadas, mas isso fica para um próximo blog post. Se curtiu o assunto, você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).** Você também pode achar esse texto em **[Inglês](https://dev.to/mfbmina/introduction-to-concurrency-in-go-2bg7)**.
