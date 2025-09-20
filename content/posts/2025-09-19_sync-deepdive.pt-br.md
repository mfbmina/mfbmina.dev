+++
title = "Destrinchando o pacote sync do Go"
subtitle = ""
date = 2025-09-19T11:17:23-03:00
lastmod = 2025-09-19T11:17:23-03:00
+++ 

Na minha opinião, Go fornece um excelente suporte a se trabalhar de forma concorrente não só por causa das goroutinas, mas por causa do ecossistema da linguagem. Um grande exemplo disso é o pacote sync, que auxilia na sincronização das rotinas concorrentes. Neste post vamos aprofundar em tudo que este pacote pode nos oferecer.

## Waitgroups

Waitgroups são utilizados para coordenação a execução de diversas rotinas. Ele facilita a criação e a garantia de que todas as subrotinas serão finalizadas antes de finalizar a rotina principal. No post sobre [waitgroups]({{< ref waitgroups >}}) eu explico melhor como elas funcionam e o que mudou com a versão 1.25 do Go.

## Mutex

Mutex é o mesmo que mutual exclusion locker, ou em português, tranca de exclusão mutua. A sua função é travar ao acesso a um recurso, enquanto você executa uma operação para evitar que outras rotinas tentem escrever nesse recurso ao mesmo tempo. Por exemplo, qual o retorno da função a seguir?

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

Se a resposta foi 1000, existe alguma chance de você ter acertado, mas é bem improvável. Isso acontece, pois como as rotinas são executadas de forma concorrente, elas podem tentar escrever no recurso ao mesmo tempo. Para garantir que isso não aconteça, basta adicionar uma mutex, e travar o acesso a aquele recurso.

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

O uso é bem simples, para travar o acesso a um registro você usa a função `Lock` e ao finalizar é só utilizar o `Unlock`. Também existe a função `TryLock`, que valida se existe um lock ativo ou não, porém seu caso de uso é mais raro. Só é necessário atenção para não cair em [deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science)).

## RW Mutex

O RW Mutex é uma evolução da mutex onde existem locks especificos para escrita e leitura. Essa distinção é bastante útil quando uma ou mais rotinas precisam acessar um recurso só para leitura, mas não querem que o objeto seja modificado durante sua execução. Contudo é importante citar pontuar que o escrita tem mais prioridade que a leitura e, dessa forma, Go evita o [starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science)).

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

O [atomic](https://pkg.go.dev/sync/atomic) é um subpacote do pacote sync que implementa suporte a concorrência em tipos primitivos. Atualmente ele suporta operações os tipos: bool, int32, int64, pointer, uint32, uint64, uintpointer e value. Com ele, podemos simplificar o exemplo utilizado na mutex:

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

O map é funciona como qualquer outro map normal. Ele fornece funções que para comparar, trocar, atribuir ou recuperar os valores, com a diferença de ser seguro para concorrência. 

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

A própia documentação sugere que ele deve ser utilizado em dois casos:
1. quando a uma key é escrita apenas uma vez, mas lida diversas vezes. Um exemplo é um cache que só cresce. 
2. quando múltiplas goroutinas lêem e escrevem grupos distintos de chaves. 

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

É importante se atentar que caso a função dê `panic`, ela não será reexecutada.

## Pool

## Cond

## Conclusão

## Links Extras
- [sync](https://pkg.go.dev/sync)
- [atomic](https://pkg.go.dev/sync/atomic)
- [Introdução à concorrência em Go]({{< ref introduction-concurrency-go >}})
- [Waitgroups: o que são, como usar e o que mudou com o Go 1.25]({{< ref waitgroups >}})
- [Deep dive into the sync package - Gophercon UK](https://www.youtube.com/watch?v=DOj1G7CMT-I)
- [Go Goroutine Synchronization: a Practical Guide](https://medium.com/@Realblank/go-goroutine-synchronization-a-practical-guide-49705a499fd7)
- [Deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science))
- [Starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science))
