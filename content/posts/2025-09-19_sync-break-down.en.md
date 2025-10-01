+++
title = "Breaking down Go's sync package"
subtitle = ""
date = 2025-09-19T11:17:23-03:00
lastmod = 2025-09-19T11:17:23-03:00
+++

In my opinion, Go provides excellent support for concurrent work, not only due to goroutines but also because of the language's ecosystem. A great example of this is the [sync](https://pkg.go.dev/sync) package, which helps synchronize concurrent routines. In this post, we'll dive into everything this package has to offer.

## Waitgroups

Waitgroups are used to coordinate the execution of multiple routines. They make it easy to create and ensure that all sub-routines will finish before the main routine ends. In the post about [waitgroups]({{< ref waitgroups >}}) I explain better how they work and what changed with Go version 1.25.

## Mutex

Mutex stands for mutual exclusion locker. Its function is to lock access to a resource while an operation is being executed, preventing other routines from trying to write to that resource at the same time. For example, what is the return of the following function?

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

If your answer was 1000, there's a chance you might have gotten it right, but it's unlikely. This happens because, as routines are executed concurrently, they might try to write to the resource at the same time. To ensure this doesn't happen, simply add a mutex and lock access to that resource.

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

Usage is quite simple: to lock access to a record you use the `Lock` function, and when finished, just use `Unlock`. You just need to be careful not to fall into [deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science)). There's also the `TryLock` function, which validates whether an active lock exists or not, but its use case is rarer.

## RW Mutex

The RW Mutex is an evolution of the mutex where there are specific locks for writing and reading. This distinction is quite useful when one or more routines need to access a resource for reading only, but don't want the object to be modified during its execution. However, it's important to mention that writing has higher priority than reading, and thus, Go avoids [starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science)).

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

In the example above, we can have several routines calling `avg` to get the average of the integer list. However, if a routine decides to insert another value, everyone will have to wait for that write to finish.

## Atomic

The [atomic](https://pkg.go.dev/sync/atomic) is a subpackage of the sync package that implements concurrency support for primitive types. Currently, it supports the following types: `bool`, `int32`, `int64`, `pointer`, `uint32`, `uint64`, `uintpointer`, and `value`. With it, we can simplify the example used in the mutex:

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

It's necessary to note that basic operations, such as addition, have been re-implemented to ensure that routines do not contend for the resource.

## Map

The `Map` is like any other normal `map`. It provides functions to compare, swap, assign or retrieve values, with the difference that it is safe for concurrency.

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

The documentation itself suggests it should be used in two cases:

1.  When a key is written only once, but read multiple times. An example is a cache that only grows.
2.  When multiple goroutines read and write distinct groups of keys.

Any other case is better to use the traditional map with mutexes.

## Once

The `Once` type guarantees that something will be executed only once, even if multiple routines try to execute it. An example of this could be resource initialization, as demonstrated by the Go documentation.

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

It's important to note that if the function panics, it will not be re-executed.

## Cond

As the name suggests, `Cond` works based on a conditional, meaning that when something happens, it releases the execution of a routine. This execution can be released one by one using the `signal` function or activating all at once with `broadcast`.

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

First, we initialize a `cond` with some `Locker`, an interface that implements the `Lock` and `Unlock` functions. In the example, we use a mutex. When initializing each goroutine, it's necessary to ensure the lock and then we put it in a waiting state with `Wait`, which releases the lock, allowing new routines to be started. When a routine is released with `Signal` or `Broadcast`, `Wait` acquires the `Lock` again and releases the code execution. Go's documentation recommends that `Wait` happens inside a loop waiting for a condition, because `Cond` alone can't tell if something has happened or not, but this is not strictly necessary. The general flow then is:

`goroutine acquires the lock` → `wait releases the lock` → `wait waits for a signal` → `wait receives a signal` → `wait acquires a new lock` → `wait releases execution` → `goroutine performs work` → `goroutine releases the lock`

## Pool

The `Pool` provides a way to deal with short-lived objects in memory. This helps relieve pressure on the GC, as memory space is always reused. The official documentation cites the `fmt` package as an example, which uses pools as temporary output buffers that adjust their size as needed.

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

To initialize a `Pool` we need to define its initialization function. When using `Get` we retrieve what is saved in memory and with `Put` we write a new value to it. `New` is only used if there is nothing allocated into the memory.

## Conclusion

The `sync` package provides several functionalities that are extremely useful when working with multiple goroutines. It's possible to control execution with `Cond` and `Once` types. `Waitgroups` ensure everything will be executed. `Mutex`, `RWMutex`, and atomic types prevent resource contention. Finally, `Pool` relieves the GC's work when it's possible to work with short-lived objects in memory. Without a doubt, this package is crucial for anyone working with goroutines. If you wish to understand the implementation details of this package, I recommend watching the talk presented at Gophercon UK 2025, [Deep dive into the sync package](https://www.youtube.com/watch?v=DOj1G7CMT-I) by Jesus Hawthorn. There is also a [talk]({{< ref easing_concurrency_with_the_sync_package >}}) that I presented at Golang SP about it. Tell me in the comments if you have already used this package and if it helped you in any way. If you haven't used it yet, comment on what you thought of the post.

## Extra Links
- [sync](https://pkg.go.dev/sync)
- [atomic](https://pkg.go.dev/sync/atomic)
- [Introduction to concurrency in Go]({{< ref introduction-concurrency-go >}})
- [Waitgroups: what they are, how to use them, and what changed with Go 1.25]({{< ref waitgroups >}})
- [Presentation at Golang SP]({{< ref easing_concurrency_with_the_sync_package >}})
- [Deep dive into the sync package - Gophercon UK](https://www.youtube.com/watch?v=DOj1G7CMT-I)
- [Go Goroutine Synchronization: a Practical Guide](https://medium.com/@Realblank/go-goroutine-synchronization-a-practical-guide-49705a499fd7)
- [Deadlock](https://en.wikipedia.org/wiki/Deadlock_(computer_science))
- [Starvation](https://en.wikipedia.org/wiki/Starvation_(computer_science))
- [Code examples](https://github.com/mfbmina/poc_sync_package)
