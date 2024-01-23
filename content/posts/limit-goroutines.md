+++ 
date = 2024-01-23T09:14:31+05:30
title = "Limiting number of goroutines"
description = "Techniques to achieve bounded concurrency"
tags = ["go", "concurrency"]
+++

Recently I was working on a feature to add a flag to the `oc` command line tool. `oc` has a sub-command that helps an OpenShift administrator collect "must-gather" from their OpenShift cluster. `must-gather` is a command that gathers information that might help solve the problem causing an OpenShift cluster to function erratically. It is like `sosreport` but focused on a Kubernetes cluster instead of an individual host.

The flag I was adding was `--all-images` which would replace a command like below with a simple `oc adm must-gather --all-images`:

```shell
$ oc adm must-gather \
--image=quay.io/myrepo/image1 \
--image=quay.io/myrepo/image2 \
--image=registry.redhat.io/container-native-virtualization/cnv-must-gather-rhel9:v4.14.2

$ oc adm must-gather --all-images
```

To know more details about `--all-images` flag, take a look at this [enhancement proposal](https://github.com/openshift/enhancements/pull/1487).

As mentioned in the proposal, we want to limit the number of Pods started to collect the cluster's must-gather to four. Otherwise, `--image` or `--all-images` flag could potentially overwhelm the cluster and cause more problem on an already erratic cluster.

### Semaphore pattern

I'm not well-versed with Design Patterns, Concurrency Patterns, or any kind of Computer Science patterns. So, when I started out, I wrote code that I knew best to limit the number of goroutines. And I don't know how I knew that approach, but it's either because I keep reading [StackOverflow Q&A with goroutines tag](https://stackoverflow.com/search?q=%5Bgoroutine%5D%2C%5Bgo%5D) or the [golang subreddit](https://www.reddit.com/r/golang/).

Anyway, a simple Go program to limit number of goroutines based on semaphore pattern would look like below:

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

// number of concurrent goroutines we want to run
const workers = 4

func main() {
	var wg sync.WaitGroup
    // channel to be populated and depopulated at the start and end of each goroutine
	w := make(chan struct{}, workers)

	for i := 0; i < 10; i++ {
		w <- struct{}{}
		wg.Add(1)

		go func() {
			defer func() {
				<-w
				wg.Done()
			}()

			fmt.Println("# goroutines: ", runtime.NumGoroutine())
			time.Sleep(5 * time.Second)
		}()
	}
	wg.Wait()
}
```

What's this code doing?
1. First, we create a buffered channel of the size of maximum number of goroutines we want to start concurrently. In our example it's 4. It helps ensure that our code doesn't start any more than four goroutines.
2. A `for` loop that will, in total, start 10 goroutines.
3. First statement in the `for` loop puts a `struct{}{}` on the channel. If the channel is full, this operation will wait till there is space available on it. Hence, it prevents further goroutines from spinning up.
4. Once we put a `struct{}{}` on the channel, we start a goroutine that prints number of running goroutines (using `runtime.NumGoroutine()`) and sleeps for 2 seconds before reading a `struct{}{}` from the channel and decrementing the WaitGroup counter by 1 in its `defer` call.
5. A `wg.Wait()` call is required to make sure that the `main` goroutine doesn't exit before all of the goroutines created in `for` loop are done processing. Try removing it from the code and see if the `# goroutines` is printed 10 times or less.

This code spins up a total of 10 goroutines. Since creation and deletion of goroutines is not very expensive, we don't mind it. However, there are cases where it's better idea to create less goroutines.

Enter worker pool pattern.

### Worker Pool pattern

When I opened a PR that solved the problem using Semaphore pattern, I got a review from the maintainer to try and use a queue of Pods. The code [they referenced](https://github.com/openshift/oc/pull/1633/#discussion_r1442802841) used `RateLimitingInterface` interface from the Kubernetes client-go library. I spent some time looking at it, but it made no sense, and after a brief discussion on the PR, I started learning and then implementing Worker Pool pattern to solve the problem.

A simple Go program to limit number of goroutines based on Worker Pool pattern would look like below:
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var concurrentMG = 4

func main() {
	var wg sync.WaitGroup
	podChan := make(chan int)

	go func(){
    	for i := 0; i < 10; i++ {
            // put the task to process on podChan
            podChan <- i
	    }
        close(podChan)
    }()

	// spin up concurrentMG number of workers
	wg.Add(concurrentMG)
    for i := 0; i < concurrentMG; i++ {
		fmt.Printf("Spinning up a goroutine...\n")
		go func() {
			// read till podChan is empty
			for i := range podChan {
				fmt.Printf("%d\n", i)
				time.Sleep(1 * time.Second)
			}

			// no more pods to process
			wg.Done()
		}()
	}

	wg.Wait()
}
```

This code achieves the same goal of running only a certain number of goroutines concurrently, but it does so by spinning up those goroutines right away and reading the data to be processed through an unbuffered channel. Let's look at what this code does:
1. Create an unbuffered channel where we will put the data to be processed.
2. A goroutine that puts data onto the `podChan` and closes the `podChan` when it's done. This code is invoked in a goroutine since we are putting data onto an unbuffered channel which blocks the execution unless something reads from the channel.
3. The `wg.Add(concurrentMG)` statement is indicating that we will be starting `concurrentMG` number of goroutines.
4. The `for` loop in this code is starting the goroutines. The `for` loop within each goroutine will read from the `podChan`. This `for` loop exits when `podChan` is closed. At this point, we call `wg.Done()`.

For the same task, Semaphore pattern spins up 10 goroutines while Worker Pool pattern spins up only 4 goroutines. There might be use cases where latter is more performant over the former when the total number of goroutines is larger.

### Queue of tasks

Note that this is not a pattern. After I implemented the Worker Pool pattern, the maintainers [asked me to use the queue approach](https://github.com/openshift/oc/pull/1633/#discussion_r1451979825) they recommended earlier because the larger `oc` code uses that at a bunch of places which made maintaining things easier.

A simple program using queue approach is available on [this repository](https://github.com/dharmit/queueofpods). It uses the queue implementation from Kubernetes code. To me, this approach is akin to using the Worker Pool pattern without the channels. It differs from the Worker Pool pattern in that `queue.Add` and `queue.Get()` statemets ensure that tasks are performed in a FIFO order, whereas with the Worker Pool pattern covered above doesn't gaurantee order.

An excellent in-depth explanation of this rate limiting queue available in Kubernetes code can be found [in this blog](https://danielmangum.com/posts/controller-runtime-client-go-rate-limiting/).

### That's it

Initially I despised the back and forth that happened in the process of getting this PR approved. But, in the end, I am glad that it helped me try and learn different approaches. If you have any feedback/suggestions, please let me know via [Twitter](https://twitter.com/dharm1t).