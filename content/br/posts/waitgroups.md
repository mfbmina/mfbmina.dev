+++
title = 'Waitgroups: o que são, como usar e o que mudou com o Go 1.25'
date = 2025-08-23T10:11:15-03:00
draft = false
+++

Imagine o seguinte problema: é necessário processar centenas de registros e gerar uma única saída. Uma forma de resolver é processar cada registro sequencialmente e só no fim unificar a saída. Contudo isso pode ser extremamente lento, dependendo do tempo gasto para processar cada registro. Outra forma é fazer o processamento de forma concorrente, acelerando o tempo gasto para o processamento. No meu post sobre [introdução à concorrência]({{< relref introduction-concurrency-go >}}) falei um pouco sobre `goroutines` e `channels` e agora decidi falar sobre os `waitgroups`, uma forma de simplificar a gestão de múltiplas `goroutines`.

`Waitgroups` são parte do pacote [sync](https://pkg.go.dev/sync#WaitGroup) e seu uso é relativamente simples. Para cada goroutine disparada, é necessário adicionar 1 ao contador do `sync` e depois, deve-se esperar que todas `goroutines` terminem seus trabalhos. É necessário que cada goroutine reduza 1 do contador para indicar seu fim. Exemplificando:

```go
func Before1_25() {
	r := rand.IntN(10)
	wg := sync.WaitGroup{}
	wg.Add(1) // Wait for 1 more goroutine to process
	go doSomethingTheOldWay(&wg, r) // Doing something async
	wg.Wait() // Wait until all goroutines finishes
}

func doSomethingTheOldWay(wg *sync.WaitGroup, sleep int) {
	defer wg.Done() // Tell the waitgroup you're done. It the same as wg.Add(-1)

	time.Sleep(time.Duration(sleep) * time.Second)
}
```

Apesar de o uso ser fácil, é necessário se preocupar em sempre ter o número correto de `goroutines` no `waitgroup`, ou seja, para cada `goroutine` adicionada com `wg.Add` é necessário ter um `wg.Done`. Se isso não ocorrer, pode-se causar com um [deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science)) durante o `wg.Wait`. Isso acontece, por exemplo, se adicionarmos uma goroutine ao `waitgroup` e nunca finalizarmos com o `wg.Done`. O inverso deste problema é finalizar mais `goroutines` do que foram adicionadas ao grupo de espera. Neste caso, um `panic` é gerado durante a execução do seu programa com a mensagem `panic: sync: negative WaitGroup counter`. No entanto esse problema é intermitente, pois o processo principal pode finalizar antes da goroutine. Para evitar estes casos, a biblioteca [goleak](https://github.com/uber-go/goleak) implementa um validador de vazamento de `goroutines`. 

```go
func TestCases(t *testing.T) {
	t.Run("Before1_25", func(t *testing.T) {
		defer goleak.VerifyNone(t)

		Before1_25()
	})
}
```

A partir da versão 1.25 do Go, tudo mudou e nossa API ficou ainda mais simples e sem esses problemas! Em vez de manualmente ser necessário controlar quais `goroutines` foram adicionadas e sinalizar o seu fim, podemos simplesmente utilizar a nova função que já faz isso automaticamente com o `wg.Go`.

```go
func After1_25() {
	r := rand.IntN(10)
	wg := sync.WaitGroup{}
	wg.Go(func() { doSomethingTheNewWay(r) })
	wg.Wait()
}

func doSomethingTheNewWay(sleep int) {
	time.Sleep(time.Duration(sleep) * time.Second)
}
```

Como essa função recebe um `func ()` como argumento, é necessário encapsular nossa função dentro de outra, caso seja necessário fornecer argumentos. Como o incremento e decremento passaram a ser feitos de forma automática, os problemas citados acima não existem mais e, de fato, ficou ainda mais simples de se trabalhar de forma concorrente. 

Apesar de não ser o foco desta postagem, quero citar outra melhoria a partir desta versão da linguagem. Antes da versão 1.25, se sua aplicação executava em Kubernetes, era necessário utilizar alguma biblioteca como [automaxprocs](https://github.com/uber-go/automaxprocs) para obter um valor de CPU válido para as `goroutines` e agora isso é feito para nós automaticamente. Para quem tiver interesse, recomendo a leitura do artigo [Container-aware GOMAXPROCS](https://go.dev/blog/container-aware-gomaxprocs).

No próximo post, quero destrinchar mais sobre o pacote `sync` e como podemos utilizar suas outras funcionalidades para gerenciar o trabalho concorrente de forma mais simples. Os exemplos estão neste [repositório](https://github.com/mfbmina/poc_waitgroups). Comente abaixo o que achou do post e das novidades do Go 1.25!
