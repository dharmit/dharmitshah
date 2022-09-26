+++ 
date = 2022-09-26T10:20:33+05:30
title = "Stack, Heap, Go"
tags = ["go", "tech"]
+++

Last week, a colleague asked me what I knew about the stack and heap for a process. My answer was a disappointing, "I don't know anything about it". The truth is, I just knew that there's a stack and heap to a process, but I didn't know any of the details. We went on to have other discussion, but I made a note to look it up when I was by myself.

### Stack and Heap in general

Whatever I looked up about stack and heap had mostly got examples with C programs. It was interesting to learn that every program has five segments of memory:
* stack
* heap
* uninitialized data
* initialized data
* text

I think I knew about the stack, heap and text only.

Stack is the part of the memory that stores data local to the function - this could be variables and function calls. Stack is of a smaller size compared to a heap. For a *nix system, you can find the default stack size by doing:

```sh
$ ulimit -a | grep stack
```

Stack memory is organized as a stack data structure with contagious blocks of memory being used for it. This is what makes it fast. It follows a LIFO (Last In First Out) order so that the last function to be put on to the stack is the first one that gets removed. This makes sense because the `main` function is usually the last function we want to exit from.

For every function in the program, a stack frame is created on the stack memory which holds information related to that function. These stack frames are what stack memory, as a whole, is comprised of.

Heap is the part of the memory that is usually obtained from the computer's RAM based on the requirement/need of the function/program. Now stack is also using computer's RAM, but the key difference is that stack has contagious blocks of memory, whereas heap doesn't. This is why accessing data from stack is faster than heap.

In your C program, when you make a call to `malloc`, and ask for a sufficiently large amount of memory to be allocated (I think larger than what above `ulimit` output shows on your system), it's put on to the heap memory instead of stack.

Another key thing to remember, for C programs, is that when we return from a function, its stack frame is removed from the stack memory, and hence all the data local to that function is invalidated. This is why segmentation fault could occur when a function is returning a pointer in C if static variable isn't used. For example, below C code would not compile:

```c
#include <stdio.h>

int* f() {
    int a;
    a = 42;
    return &a;
}

void main()
{
    printf("%d", *f());
}
```
```sh
$ gcc prog.c
prog.c: In function ‘f’:
prog.c:6:12: warning: function returns address of local variable [-Wreturn-local-addr]
    6 |     return &a;
      |            ^~
```

### Stack and Heap in Go

Now, Go is a little different in terms of invalidating the data on the stack frame. Go compiler uses something called "escape analysis" to determine what goes on to the stack and what goes to heap. The Go FAQ says that:
> if the compiler cannot prove that the variable is not referenced after the function returns, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors.

This is what causes a similar program in Go to not fail:

```go
package main

import (
    "fmt"
)

func f() *int {
    a := 42
    return &a
}

func main() {
    fmt.Printf("%d", *f())
}
```

```sh
$ go run main.go
42
```

Doing a `go tool compile -m main.go` gives some interesting output. But this post is already getting too long. :wink:

### Go is memory safe
While talking with this colleague of mine, I also said that, "I know Go is memory safe, but I won't be able to explain what part makes it memory safe." While reading up about stack and heap in C and Go, I felt that this right here seems like the thing that makes Go memory safe. To confirm my understanding, I did a [quick search](https://duckduckgo.com/?q=+why+is+go+memory+safe&t=newext&atb=v307-1&ia=web) and came across the article by Dave Cheney which confirmed that my understanding was right.

This also explains why I mostly found stack/heap examples and explanations for C programs, because, as a C programmer, one has to be very cautious when it comes to memory management and avoiding leaks.

### That's it
I'm frequently surprised by how I am at loss when it comes to explain, remember and recall what certain technical jargon means. But when I am reading/listening about it, my mind connects the dots and I have an "Aha" moment! This certainly doesn't help me when I am having conversations with fellow enginers (or imagine flunking such questions in an interview, only to realize I did understand the concept, but couldn't recall it at the right time), but the silver lining is that I can connect the dots.

Finally, I would highly recommend watching [this talk](https://www.youtube.com/watch?v=ZMZpH4yT7M0) to learn more about allocations in Go.

### References

1. [Stack vs Heap. What’s the Difference and Why Should I Care?](https://www.linux.com/training-tutorials/stack-vs-heap-whats-difference-and-why-should-i-care/) 
1. [Understanding Allocations: the Stack and the Heap - GopherCon SG 2019](https://www.youtube.com/watch?v=ZMZpH4yT7M0)
1. [Escape Analysis in Go: Stack vs. Heap](https://christensen.codes/Escape-Analysis-in-Go:-Stack-vs.-Heap) (Used program examples from this site)
1. [Go FAQ on stack and heap](https://go.dev/doc/faq#stack_or_heap)
1. [Why Go?](https://dave.cheney.net/2017/03/20/why-go)