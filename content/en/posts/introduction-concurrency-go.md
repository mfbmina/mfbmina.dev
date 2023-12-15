+++
title = 'Introduction to Concurrency in Go'
date = 2022-11-01
draft = false
tags = ['go','concurrency','introduction']
+++

One of the best Go features is how easy we can use concurrency. The language gives us `goroutines`, which are like lightweight threads managed by the Go runtime. It help us to run several functions at the same instant and is very helpful if you wish to improve the performance of your application.

Using this feature is easy as adding the `go` keyword before any function call. This will make the function run concurrently. To make it simpler, let's show you the code. Here I've written the `SleepSort` algorithm, which uses the `sleep` method to sort each element in its place.

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

In this code, we have the `sort` function which sleeps for x seconds and prints the value. When every function finishes its execution, we will have the array in the correct order, right? Unfortunately, the answer is no!


![Response at the beggining](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t5lxza9xm8x7f5zjfgu8.png)

As the functions run concurrent to the main process, we only see what the main has to return. One way to fix this issue is to make all processes talk somehow with the main process. For that, Go give us the `channel` concept which is like a pipe/conduit responsible for send data between processes. Let's update our code to use this feature!

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

And finally, we have a response:

![Response after introducing channels](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m3i1ne5hhsdyul34eoh9.png)

We can notice that `channels` have a special operator `<-`. To understand how it works we have to think about it as an arrow. The data will always flow to where the arrow is pointing out. For example, inside the function `sort`, data is sent through the channel by using `c <- x` and the main process receives it and assigns to a new variable by using `x := <-c`.

It is also important to say that we had to change the `sort` function signature was updated to accept a `chan`. It is initialized in the main function and then given to the `goroutines`.

As we can see, starting using concurrency in Go is quite simple. The language also supports more advanced options but we can talk about it in the next blog post. If you like the subject, you can also find me on **[Twitter](https://twitter.com/mfbmina)**, **[GitHub](https://github.com/mfbmina)**, or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
