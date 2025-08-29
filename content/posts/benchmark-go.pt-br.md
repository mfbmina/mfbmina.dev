+++
title = 'Go 1.24: Testes de Benchmark'
date = 2025-03-24T20:07:57-03:00
draft = false
+++

Uma das minhas funcionalidades favoritas em Go é a possibilidade de se escrever testes de benchmark. Agora na versão 1.24, essa funcionalidade ganhou uma cara nova, se tornando ainda mais fácil de ser utilizada.

Para demonstrar estas mudanças, suponha uma função que calcule o fatorial de forma recursiva e uma atráves de laços.

```golang
func FatorialRecursive(n int) int {
	if n == 0 {
		return 1
	}

	return n * FatorialRecursive(n-1)
}

func FatorialLoop(n int) int {
	aux := 1
	for i := 1; i <= n; i++ {
		aux *= i
	}

	return aux
}
```

Anteriormente, para escrever um teste de benchmark, precisávamos informar o laço de execução do teste. Com o teste escrito, era só rodar o comando `$ go test -bench .`

```golang
func Benchmark_FatorialLoop(b *testing.B) {
	for i := 0; i < b.N; i++ {
		FatorialLoop(100)
	}
}

func Benchmark_FatorialRecursive(b *testing.B) {
	for i := 0; i < b.N; i++ {
		FatorialRecursive(100)
	}
}
```

Contudo, o compilador tende a otimizar os testes de benchmark, levando a alguns tempos de execução irreais e mais rápidos do que deveriam ser. Para evitar esse comportamento, é necessário realizar algumas mudanças:
- Setar o retorno em uma variável.
- Setar o valor dessa variável de forma global e pública.

Dessa forma, o compilador não consegue predizer o comportamento da função e não faz otimizações.

```golang
var X int

func Benchmark_FatorialLoopWithoutCompilerImprovements(b *testing.B) {
	x := 0
	for i := 0; i < b.N; i++ {
		x = FatorialLoop(100)
	}

	X = x
}

func Benchmark_FatorialRecursiveWithoutCompilerImprovements(b *testing.B) {
	x := 0
	for i := 0; i < b.N; i++ {
		x = FatorialRecursive(100)
	}

	X = x
}
```

A nova versão corrigiu esses *problemas*. Agora, com uma sintaxe simples e direta, conseguimos ter testes de benchmark confiáveis e sem otimizações precoces do compilador. 

```golang
func Benchmark_FatorialLoop_1_24(b *testing.B) {
	for b.Loop() {
		FatorialLoop(100)
	}
}

func Benchmark_FatorialRecursive_1_24(b *testing.B) {
	for b.Loop() {
		FatorialRecursive(100)
	}
}
```

Essa é uma melhoria pequena, mas que torna a experiência de desenvolvimento ainda mais agradável. Além disso, reforça a confiança nos testes de benchmark ao garantir menos interferências no que vai ser validado. Como podemos ver nos resultados, a versão antiga, com o código otimizado, apresentou resultados relativamente melhores.

```
goos: darwin
goarch: arm64
pkg: github.com/mfbmina/poc
cpu: Apple M2
Benchmark_FatorialLoop-8                                   	39609032	        30.50 ns/op
Benchmark_FatorialLoopWithoutCompilerImprovements-8        	22448245	        53.18 ns/op
Benchmark_FatorialRecursive-8                              	 1870860	       574.9 ns/op
Benchmark_FatorialRecursiveWithoutCompilerImprovements-8   	 1984813	       560.1 ns/op
Benchmark_FatorialLoop_1_24-8                              	22177114	        53.88 ns/op
Benchmark_FatorialRecursive_1_24-8                         	 2151256	       556.8 ns/op
PASS
```

Diz nos comentários o que você achou dessa mudança e qual foi a novidade do Go 1.24 de que você mais gostou!
