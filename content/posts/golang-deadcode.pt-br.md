+++
title = 'Acabando com código morto nos projetos Go'
date = 2025-06-19T10:39:57-03:00
draft = false
+++

Conforme o software que trabalhamos vai crescendo, a tendência é do código passar por diversas mudanças e refatorações. Nesse processo, podemos simplesmente esquecer pedaços de código que um dia foram utilizados e que agora não fazem mais sentido no projeto, os famosos códigos mortos. Um exemplo muito comum é quando uma API é desativada e só o `handler` é removido, porém, toda a lógica de negócio continua ali, mas sem ser utilizada. 

## O que é código morto?
Pode-se dizer que o código morto é basicamente uma função que existe dentro da sua base de código que é sintaticamente válida, porém não é utilizada por nenhuma outra parte do seu código, ou seja, é uma função inalcançável. Códigos mortos trazem problemas indiretos para o projeto, como bibliotecas desatualizadas, códigos legados, inchaço da base de código, falhas de segurança e por aí vai. Se ainda não ficou claro o que é um código morto, veja o exemplo abaixo:

```golang
package main

import "fmt"

func main() {
  fmt.Println("Hello, World!")

  reachable()
}

func unreachable() {
  fmt.Println("This function will never be called")
}

func reachable() {
  fmt.Println("This function is reachable")
}

func Public() {
  fmt.Println("This is a public function but it is unused")
}
```

Neste código temos as funções privadas `unreachable` e `reachable`. Por padrão, [gopls](https://pkg.go.dev/golang.org/x/tools/gopls#section-readme) vai dizer que a função `unreachable` não está sendo utilizada e que pode ser removida, entretanto, isso não impede a compilação do projeto. O `gopls` é um `language server` utilizado pelos editores para habilitar funcionalidades como completamento de código, correções de sintaxe, etc. Porém, se a função se tornar pública, este erro não vai ser apontado, pois ele teoricamente pode ser utilizado por outros pacotes.

Esse problema se amplia ao lidarmos com pacotes, pois pacotes não utilizados também não são reportados. Suponha então um pacote com funções privadas e públicas, porém que não é utilizado no projeto.

```golang
package unused

import "fmt"

func UnusedFunction() {
  fmt.Println("This function is unused")

  indirectUnreachable()
}

func indirectUnreachable() {
  fmt.Println("This function is unreachable")
}
``` 

## Pacote deadcode
A equipe do Go trouxe então uma solução para este problema, a ferramenta [deadcode](https://pkg.go.dev/golang.org/x/tools/cmd/deadcode). Vale a pena mencionar que a ferramenta sempre deve ser executada a partir da `main`, pois ela procura por código morto a partir do que seria executado em produção. Ao rodar essa ferramenta, temos finalmente o resultado de todas as funções não utilizadas.

```sh
$ go tool deadcode ./...
> main.go:11:6: unreachable func: unreachable
> main.go:19:6: unreachable func: Public
> unused/unused.go:5:6: unreachable func: UnusedFunction
> unused/unused.go:11:6: unreachable func: indirectUnreachable
```

Assim, podemos facilmente encontrar código morto em nosso projeto. Para instalar a ferramenta, é basicamente rodar o comando:

```sh
$ go get -tool golang.org/x/tools/cmd/deadcode@latest
```

## Conclusão
Essa ferramenta é bem útil para ser executada após refatorações no projeto e tem me ajudado bastante a manter o código enxuto e somente com o que de fato importa para o projeto. Se você ficou interessado e quer saber mais, recomendo a leitura do [post oficial.](https://go.dev/blog/deadcode) Me diz nos comentários o que você achou da ferramenta e, se quiser ver o código todo, acesse [aqui.](https://github.com/mfbmina/poc_deadcode)
