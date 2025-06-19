+++
title = 'Eliminating dead code in Go projects'
date = 2025-06-19T10:39:57-03:00
draft = false
+++

As the software we work on grows, the code tends to undergo various changes and refactorings. During this process, we might simply forget pieces of code that were once used but no longer make sense in the project, the infamous dead code. A very common example is when an API is deactivated, and only the `handler` is removed, but all the business logic remains, unused.

Dead code can be defined as a function that exists within your codebase, is syntactically valid, but is not used by any other part of your code. In other words, it's an unreachable function. Dead code brings indirect problems to a project, such as outdated libraries, legacy code, code bloat, security vulnerabilities, and so on. If it's still not clear what dead code is, see the example below:

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

In this code, we have the private functions `unreachable` and `reachable`. By default, [gopls](https://pkg.go.dev/golang.org/x/tools/gopls#section-readme) will tell you that the `unreachable` function is not being used and can be removed. However, this doesn't prevent the project from compiling. `gopls` is a language server used by editors to enable features like code completion, syntax corrections, etc. But if the function becomes public, this error won't be flagged because it can theoretically be used by other packages.

This problem expands when dealing with packages, as unused packages are also not reported. Imagine a package with private and public functions that isn't used in the project:

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

The Go team then provided a solution to this problem with the `deadcode` tool. It's worth mentioning that the tool should always be run from `main`, as it searches for dead code based on what would be executed in production. When you run this tool, you finally get all unused functions:

```sh
$ go tool deadcode ./...
> main.go:11:6: unreachable func: unreachable
> main.go:19:6: unreachable func: Public
> unused/unused.go:5:6: unreachable func: UnusedFunction
> unused/unused.go:11:6: unreachable func: indirectUnreachable
```

This way, we can easily find dead code in our project. To install the tool, simply run the command:

```sh
$ go get -tool golang.org/x/tools/cmd/deadcode@latest
```

This tool is very useful to run after project refactorings and has helped me a lot to keep the code lean and containing only what truly matters to the project. If you're interested and want to know more, I recommend reading the [official post](https://go.dev/blog/deadcode). Tell me in the comments what you think of the tool, and if you want to see the full code, access it [here](https://github.com/mfbmina/poc_deadcode).
