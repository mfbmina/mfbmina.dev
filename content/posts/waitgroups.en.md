+++
title = 'Waitgroups: what they are, how to use them and what changed with Go 1.25'
date = 2025-08-23T10:11:15-03:00
draft = false
+++

Imagine the following problem: you need to process hundreds of records and generate a single output. One way to solve this is to process each record sequentially and unify the output only at the end. However, this can be extremely slow, depending on the time spent processing each record. Another way is to process them concurrently, speeding up the overall time. In my post about [introduction to concurrency]({{< relref introduction-concurrency-go >}}), I talked a bit about `goroutines` and `channels`. Now, I've decided to talk about `waitgroups`, a way to simplify the management of multiple `goroutines`.

## Before 1.25
`Waitgroups` are part of the [sync](https://pkg.go.dev/sync#WaitGroup) package, and their use is relatively simple. For each `goroutine` you start, you must add 1 to the `sync` counter, and then you have to wait for all `goroutines` to finish their work. Each `goroutine` must reduce the counter by 1 to indicate its completion. For example:

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

Although they are easy to use, you must always ensure the correct number of `goroutines` in the `waitgroup`. In other words, for every `goroutine` added with `wg.Add`, you must have a `wg.Done`. If this doesn't happen, it can cause a [deadlock](https://en.wikipedia.org/wiki/Deadlock_\(computer_science\)) during `wg.Wait`. This occurs, for example, if we add a `goroutine` to the `waitgroup` and never call `wg.Done`. The reverse of this problem is finishing more `goroutines` than were added to the waitgroup, which generates a `panic` with the message `panic: sync: negative WaitGroup counter`. However, this problem is intermittent, as the main process might finish before the `goroutine`. To avoid these cases, the [goleak](https://github.com/uber-go/goleak) library implements a `goroutine` leak validator.

```go
func TestCases(t *testing.T) {
  t.Run("Before1_25", func(t *testing.T) {
    defer goleak.VerifyNone(t)

    Before1_25()
  })
}
```

## After 1.25
Starting with Go version 1.25, everything changed, and our API became even simpler and free of these problems\! Instead of manually having to control which `goroutines` were added and signal their end, we can simply use the new function `wg.Go`, which does this automatically.

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

Since this function accepts a `func()` as an argument, you need to wrap your function inside another one if you need to pass arguments. As the increment and decrement are now handled automatically, the problems mentioned above no longer exist, and it has indeed become even simpler to work concurrently.

## Conclusion
Although it is not the focus of this post, I want to mention another improvement in this version of the language. Before version 1.25, if your application ran on Kubernetes, you needed to use a library like [automaxprocs](https://github.com/uber-go/automaxprocs) to get a valid CPU value for `goroutines`. Now, this is done for us automatically. For those interested, I recommend reading the article [Container-aware GOMAXPROCS](https://go.dev/blog/container-aware-gomaxprocs).

In the next post, I want to explore more about the `sync` package and how we can use its other functionalities to manage concurrent work more simply. The examples are in this [repository](https://github.com/mfbmina/poc_waitgroups). Comment below what you thought of the post and the new features in Go 1.25!
