+++
title = 'Go 1.24: Benchmark Tests'
date = 2025-03-24T20:07:57-03:00
draft = false
tags = ["go", "testing", "1.24", "news"]
+++

One of my favorite features in Go is the possibility of writing benchmark tests. At Go 1.24, this feature has a new look, making it easier to use. 

To demonstrate these changes, let's suppose a function that calculates the factorial recursively and one that calculates it through loops.

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

Previously, to write a benchmark, it was necessary to write down the whole execution loop of the test. When done, we need to run the command `$ go test -bench .`

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

However, the compiler tends to optimize the tests, making some functions some faster than they should. In order to avoid this behavior,  it is necessary to make changes:
- Set the function return to a variable.
- Set this variable to a global and public variable.

If you do this, the compiler cannot predict the behavior, and therefore it doesn't make optimizations.

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

The new Go version solved these *issues*, and, with a simpler syntax, we have trustworthy benchmarks without premature compiler optimizations.

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

This is a small improvement, but it makes the developer experience much finer. It also reinforces the trust at the benchmark tests by ensuring less interference at the code being checked. As we can see from the results, the older version with optimizations tends to be better than it actually is.

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

Let me know in the comments section what you think about this change and what news from Go 1.24 that you liked the most!
