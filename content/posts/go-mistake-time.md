+++ 
date = 2023-03-23T17:28:15+05:30
title = "A Go Mistake with time.Second"
tags = ["go", "programming"]
+++

Recently, while working on a [post about Kubernetes Operators](../operator-part-1), I faced 
[an issue](https://github.com/operator-framework/operator-sdk/issues/6366) with "time multiplication" and couldn't 
understand why Go did what it did. If you have some background with Kubernetes Operators, I recommend checking out 
the issue itself. If you are focussed on just Go, here's the code:
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	d := 5 * time.Second
	fmt.Println(d)
	fmt.Println(d * time.Second)
}
```
and here's the output:
```shell
$ go run ./main.go
5s
1388888h53m20s
```

I expected the second `fmt.Println` output to print `5s` as well. `1388888h53m20s` made no sense to me! I ended up 
pinging on the Gophers Slack and got [an answer](https://gophers.slack.com/archives/C029RQSEE/p1679129285130729) 
which surprised me.

The reason for this output is how `time.Second` is declared in the standard library's
[`time` package](https://github.com/golang/go/blob/0aa14fca8c639c9ceba264dbf0d82bd53306aeaa/src/time/time.go#L631-L638)
\- it's an integer with the value `10^9`. So the expression inside second `fmt.Println` would translate to 
[`5 * 10^9 * 10^9`](https://convertlive.com/u/convert/nanoseconds/to/hours#5000000000000000000) and the result of 
that would be printed by the
[`String`](https://github.com/golang/go/blob/0aa14fca8c639c9ceba264dbf0d82bd53306aeaa/src/time/time.go#L644) method.

### That's it

I had a hunch while posting the GitHub issue on operator-sdk repository, but I couldn't think of a possible cause 
till I saw the first response to my query. That was a pretty neat thing I recently learned about Go.