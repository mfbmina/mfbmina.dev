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

## Pool

## Once

## Cond

## Map

## Conclusão

## Links Extras
- [sync](https://pkg.go.dev/sync)
- [Introdução à concorrência em Go]({{< ref introduction-concurrency-go >}})
- [Waitgroups: o que são, como usar e o que mudou com o Go 1.25]({{< ref waitgroups >}})
- [Deep dive into the sync package - Gophercon UK](https://www.youtube.com/watch?v=DOj1G7CMT-I)
- [Go Goroutine Synchronization: a Practical Guide](https://medium.com/@Realblank/go-goroutine-synchronization-a-practical-guide-49705a499fd7)
- [Deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science))
- [Starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science))
