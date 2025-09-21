+++
title = "Destrinchando o pacote sync do Go"
subtitle = ""
date = 2025-09-19T11:17:23-03:00
lastmod = 2025-09-19T11:17:23-03:00
+++ 

Na minha opinião, Go fornece um excelente suporte a se trabalhar concorrentemente não só pelas goroutines, mas pelo ecossistema da linguagem. Um grande exemplo disso é o pacote [sync](https://pkg.go.dev/sync), que auxilia na sincronização das rotinas concorrentes. Neste post, vamos aprofundar em tudo que este pacote pode nos oferecer.

## Waitgroups
Waitgroups são utilizados para coordenar a execução de diversas rotinas. Ele facilita a criação e a garantia de que todas as sub-rotinas serão finalizadas antes de finalizar a rotina principal. No post sobre [waitgroups]({{< ref waitgroups >}}) explico melhor como elas funcionam e o que mudou com a versão 1.25 do Go.

## Mutex
Mutex é o mesmo que mutual exclusion locker, ou em português, tranca de exclusão mútua. A sua função é travar o acesso a um recurso enquanto uma operação é executada, evitando que outras rotinas tentem escrever nesse recurso ao mesmo tempo. Por exemplo, qual o retorno da função a seguir?

```go
func count() int {
	counter := 0
	wg := sync.WaitGroup{}

	for i := 0; i < 1000; i++ {
		wg.Go(func() {
			counter++
		})
	}

	wg.Wait()
	return counter
}
```

Se a resposta foi 1000, existe alguma chance de você ter acertado, mas é bem improvável. Isso acontece, pois como as rotinas são executadas concorrentemente, elas podem tentar escrever no recurso ao mesmo tempo. Para garantir que isso não aconteça, basta adicionar uma mutex e travar o acesso àquele recurso.

```go
func count() int {
	counter := 0
	mu := sync.Mutex{}
	wg := sync.WaitGroup{}

	for i := 0; i < 1000; i++ {
		wg.Go(func() {
			mu.Lock()
			counter++
			mu.Unlock()
		})
	}

	wg.Wait()
	return counter
}
```

O uso é bem simples, para travar o acesso a um registro você usa a função `Lock` e, ao finalizar, é só utilizar o `Unlock`. Só é necessária atenção para não cair em [deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science)). Também existe a função `TryLock`, que valida se existe uma trava ativa ou não, porém seu caso de uso é mais raro. 

## RW Mutex

O RW Mutex é uma evolução da mutex onde existem travas específicas para escrita e leitura. Essa distinção é bastante útil quando uma ou mais rotinas precisam acessar um recurso só para leitura, mas não querem que o objeto seja modificado durante sua execução. Contudo, é importante citar que a escrita tem mais prioridade que a leitura e, dessa forma, Go evita o [starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science)).

```go
var numbers []int
var mu sync.RWMutex

func store(x int) {
	mu.Lock()
	numbers = append(numbers, x)
	mu.Unlock()
}

func avg() float64 {
	mu.RLock()
	defer mu.RUnlock()

	size := len(numbers)
	sum := 0
	for _, n := range numbers {
		sum += n
	}

	return float64(sum) / float64(size)
}
```

No exemplo acima, podemos ter diversas rotinas chamando o `avg` para pegar a média da lista de inteiros, contudo, se uma rotina decidir inserir mais um valor, todos vão ter que esperar essa escrita finalizar.

## Atomic

O [atomic](https://pkg.go.dev/sync/atomic) é um subpacote do pacote sync que implementa suporte à concorrência em tipos primitivos. Atualmente, ele suporta os seguintes tipos: `bool`, `int32`, `int64`, `pointer`, `uint32`, `uint64`, `uintpointer` e `value`. Com ele, podemos simplificar o exemplo utilizado na mutex:

```go
func countWithAtomic() atomic.Int32 {
	var counter atomic.Int32
	wg := sync.WaitGroup{}

  counter.Add(1)
	for i := 0; i < 1000; i++ {
		wg.Go(func() {
			v, ok := counter.Load

		})
	}

	wg.Wait()
	return counter.Load()
}
```

É necessário notar que as operações básicas, como adição, foram reimplementadas para garantir que as rotinas não concorram pelo recurso.

## Map

O `Map` é como qualquer outro `map` normal. Ele fornece funções para comparar, trocar, atribuir ou recuperar os valores, com a diferença de ser seguro para a concorrência. 

```go
func mapExample() int {
	var m sync.Map
	wg := sync.WaitGroup{}

	for i := 0; i < 1000; i++ {
		wg.Go(func() {
			m.LoadOrStore(i, i*i)
		})
	}

	wg.Wait()

	v, _ := m.Load(0)
	return v.(int)
}
```

A própria documentação sugere que ele deve ser utilizado em dois casos:
1. Quando uma chave é escrita somente uma vez, mas lida diversas vezes. Um exemplo é um cache que só cresce. 
2. Quando múltiplas goroutines leem e escrevem grupos distintos de chaves. 

Qualquer outro caso é melhor utilizar o map tradicional com mutexes.

## Once

O tipo `Once` garante que algo será executado uma única vez, mesmo que diversas rotinas tentem executar. Um exemplo disso poderia ser a inicialização de recursos, como é demonstrado pela documentação do Go.

```go
func doSomething() int {
	wg := sync.WaitGroup{}
	o := sync.Once{}
	result := 0

	for i := 0; i < 10; i++ {
		wg.Go(func() {
			o.Do(func() {
				result++
			})
		})
	}

	wg.Wait()
	return result
}
```

É importante se atentar que, caso a função entre em pânico, ela não será reexecutada.

## Cond

Como o próprio nome diz, o `Cond` funciona a partir de uma condicional, ou seja, quando algo acontece, ele libera a execução de uma rotina. Essa execução pode ser liberada uma a uma utilizando a função `signal` ou ativando todas de uma só vez com o `broadcast`.

```go
func condExample() {
	mu := sync.Mutex{}
	cond := sync.NewCond(&mu)
	wg := sync.WaitGroup{}
	active := false

	for i := 0; i < 1000; i++ {
		wg.Go(func() {
			cond.L.Lock()
			defer cond.L.Unlock()

			for !active {
				cond.Wait()
			}

			fmt.Println("Active is true, printing: ", i)
		})
	}

	// Activate all goroutines after some time
	time.Sleep(time.Second * 5)
	fmt.Println("Setting Active to true...")
	active = true
	fmt.Println("Wake up one goroutine...")
	cond.Signal()

	time.Sleep(time.Second * 5)
	fmt.Println("Wake up all goroutines...")
	cond.Broadcast()

	wg.Wait()
	fmt.Println("All goroutines finished.")
}
```

Primeiro, iniciamos um `cond` com algum `Locker`, uma interface que implementa as funções `Lock` e `Unlock`, no exemplo, utilizamos um mutex. Ao inicializar cada goroutine, é necessário garantir o lock e então a colocamos em estado de espera com o `Wait`, que retira o lock, permitindo que as novas rotinas sejam iniciadas. Quando uma rotina é liberada com o `Signal` ou `Broadcast`, o `Wait` pega o `Lock` novamente e libera a execução do código. A documentação do Go recomenda que o `Wait` aconteça dentro de um laço esperando uma condição, pois o `Cond` sozinho não consegue dizer se algo aconteceu ou não, mas isso não é estritamente necessário. O fluxo geral então é: 

`goroutine garante o lock` &rarr; `wait libera o lock` &rarr; `wait aguarda um sinal` &rarr; `wait recebe um sinal` &rarr; `wait garante novo lock` &rarr; `wait libera a execução` &rarr; `goroutine realiza o trabalho` &rarr; `goroutine libera o lock`

## Pool

O `Pool` fornece uma maneira de se lidar com objetos de curta duração na memória. Isso ajuda a aliviar a pressão no GC, pois o espaço de memória é sempre reutilizado. A documentação oficial cita como exemplo o pacote `fmt`, que usa pools como buffers temporários de saída que ajustam seu tamanho conforme a necessidade.

```go
type Message struct {
	Text string
}

var p = sync.Pool{
	New: func() any { return new(Message) },
}

func poolExample() {
	v := p.Get().(*Message)
	defer p.Put(v)

	v.Text = "hello guys"
}
```

Para inicializar uma `Pool` precisamos definir a sua função de inicialização. Ao utilizar o `Get` recuperamos o que está salvo na memória e com o `Put` escrevemos um novo valor nela. O `New` só é utilizado se não existir nada alocado na memória.

## Conclusão

O pacote `sync` fornece diversas funcionalidades que são extremamente úteis ao se trabalhar com múltiplas goroutines. É possível controlar a execução com os tipos `Cond` e `Once`. `Waitgroups` garantem que tudo será executado. `Mutex`, `RWMutex` e os tipos atômicos evitam a concorrência pelos recursos. Por fim, o `Pool` alivia o trabalho do GC quando é possível trabalhar com objetos de curta duração na memória. Sem sombra de dúvidas, esse pacote é crucial para quem trabalha com goroutines. Caso você queira entender os detalhes de implementação deste pacote, recomendo a palestra apresentada na Gophercon UK 2025, [Deep dive into the sync package](https://www.youtube.com/watch?v=DOj1G7CMT-I), apresentada pelo Jesus Hawthorn. Diga nos comentários se já utilizou este pacote e se de alguma forma ele te ajudou. Se ainda não utilizou, comente o que achou do post.

## Links Extras
- [sync](https://pkg.go.dev/sync)
- [atomic](https://pkg.go.dev/sync/atomic)
- [Introdução à concorrência em Go]({{< ref introduction-concurrency-go >}})
- [Waitgroups: o que são, como usar e o que mudou com o Go 1.25]({{< ref waitgroups >}})
- [Deep dive into the sync package - Gophercon UK](https://www.youtube.com/watch?v=DOj1G7CMT-I)
- [Go Goroutine Synchronization: a Practical Guide](https://medium.com/@Realblank/go-goroutine-synchronization-a-practical-guide-49705a499fd7)
- [Deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science))
- [Starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science))
- [Exemplos de código](https://github.com/mfbmina/poc_sync_package)
