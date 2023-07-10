+++ 
date = 2023-07-10T21:26:34+05:30
title = "Go's sync.Once"
description = "sync.Once runs any function only once"
tags = ["go"]
+++

In my observation, when it comes to Go's concurrency primitives, the most talked about things are goroutines, 
channels and mutexes. What I have never seen discussed is standard library's `sync.Once` type.

From [Go's documentation](https://pkg.go.dev/sync#Once):
> Once is an object that will perform exactly one action.

Take a look at a very simple example to demonstrate its functionality:

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	f := func() {
		fmt.Println("hello, world")
	}
	var once sync.Once

	for i := 0; i < 10; i++ {
		once.Do(f)
	}
}
```

When you run above program, Go prints `hello, world` exactly one time. This is because we wrapped it inside a `once.
Do` call. While I don't have any objection to the documentation, I would add to to it that:
> Once runs a piece of code exactly one time and no more than that.

This could be useful when initializaing a database or a network connection that's being used by multiple goroutines. 
No matter how many number of goroutines call the initialization function, it gets executed exactly once if it's 
wrapped inside a `once.Do` call.

Where I have seen this being used is in `controller-runtime` library provided by Kubernetes. It makes sure that a 
[controller is started exactly once](https://github.com/kubernetes-sigs/controller-runtime/blob/87487d3539d7ae1abf2857f2c8623637eb25266e/pkg/manager/runnable_group.go#L128)
and that it's subsequent calls have no effect whatsoever:

```go

// Start starts the group and waits for all
// initially registered runnables to start.
// It can only be called once, subsequent calls have no effect.
func (r *runnableGroup) Start(ctx context.Context) error {
	var retErr error

	r.startOnce.Do(func() {
		defer close(r.startReadyCh)
```