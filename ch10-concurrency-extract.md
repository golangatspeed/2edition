# Chapter 10 - Concurrency - Complete Extract

**Source**: Original HTML book (Go 1.19 content)  
**Status**: Needs updating for Go 1.20-1.25 concurrency features

## Chapter Introduction

One of the most attractive features of Go is its support for *concurrency* which provides the ability to run multiple tasks at once. 

In this chapter, we're going to explore Go's concurrency features, including *goroutines*, *waitgroups*, and *channels*. We'll learn how to use them, why they work well together, and how *context* also contributes to writing safer concurrent programs.

We'll also look at another more traditional style of concurrent programming, one where we share memory but use *mutexes* to lock memory for use by a single process at a time. 

By the end of this chapter, we'll have the ability to identify opportunities for using concurrency in our programs. We'll know how to use the appropriate tools and, understand the risks associated with introducing asynchronous code blocks into our codebase.

## Chapter Structure

### 10.1 Goroutines
### 10.2 Context
### 10.3 Blocking execution with waitgroups
### 10.4 Sharing variables using mutexes
### 10.5 Communicating with channels
### 10.6 Summary

---

## 10.1 Goroutines

Goroutines are the foundations of asynchronous programming in Go. They allow multiple functions to run concurrently within a single program by scheduling them as lightweight operations, on top of threads. Each goroutine is created with an initial stack size of 2KB which can be grown as required. 

Goroutines are *non-blocking*. When a goroutine is started it returns immediately, leaving the function to be handled asynchronously by the Go scheduler.

The Go scheduler starts and stops goroutines based on several factors, which include:

- Goroutine priority
- Channel communication
- System load
- Available processors
- Goroutine stack size
- Number of existing goroutines.

Regarding *priority*, the Go scheduler dynamically assigns priority based on the goroutine's state, how long it has been blocked, and how many other goroutines are blocked and waiting to run.

*Channel communication* will often block goroutines, so the scheduler may start a goroutine while another is blocked, waiting to send or receive on the channel.

Another consideration for the Go scheduler is *stack size*. Goroutines with a large stack size may put pressure on system resources so *may* be scheduled less frequently.

We don't usually care about goroutine scheduling which takes place in the background - with possibly one exception. By default, the Go scheduler will schedule goroutines on all available CPU cores. 

In most situations this is appropriate, but we may find our Go application is overutilising resources, limiting memory and CPU available to other applications on the same hardware. In such circumstances we can limit the CPU cores the scheduler can utilise using `GOMAXPROCS`. 

This value can be set either as an environment variable of the same name or within the program itself using the *runtime* package, which means the CPU cores available to the program can be restricted for specific execution paths.

**Example 125 - Setting GOMAXPROCS during runtime**

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Printf("Setting GOMAXPROCS to 5, previously was %d\n", runtime.GOMAXPROCS(5))
	fmt.Printf("Setting GOMAXPROCS to 2, previously was %d\n", runtime.GOMAXPROCS(2))
}

// Go Playground: https://go.dev/play/p/ptFRl7NAoT_6
```

Output:
```
Setting GOMAXPROCS to 5, previous was 8
Setting GOMAXPROCS to 2, previous was 5

Program exited.
```

> **Note**: The return value of `runtime.GOMAXPROCS()` is the previously set value, not the value just set. We ran the function twice to demonstrate that.

### 10.1.1 The *go* keyword

Although goroutines are very easy to create, it's rare that we can simply create a goroutine and forget about it. However, writing code that runs asynchronously in a goroutine is very simple, only requiring that we prefix a function invocation with the `go` keyword.

```go
// calling a named function as a goroutine
go sendEmail(...)

// wrapping code in an anonymous concurrent function
go func(){
    ...
}()
```

In practice, we usually at least want to be sure the function completes and using goroutines alone as in the previous example isn't sufficient to guarantee that. To illustrate this, check out the gotcha below, in which the expected output never prints.

**Example 126 - Where is the output?**

```go
package main

import (
	"fmt"
)

func main() {
	go func() {
		fmt.Println("Will this print?")
	}()

	//time.Sleep(1 * time.Second)
}

// Go Playground: https://go.dev/play/p/nd5nwv5b3ok
```

Because the goroutine returns immediately, program execution continues. In this case, the program completes and exits before the body of the goroutine is scheduled and executed.

Uncomment line 12 which pauses the program, preventing it from terminating, for long enough that we see the expected output.

As well as completion, we often want to know the result of an operation performed by a goroutine, and, if anything went wrong. But since goroutines return immediately we can't use traditional function return values. Even if we could, it would be unwise since it could easily lead to race conditions and other synchronization issues.

So, usually, we don't use goroutines in isolation. Instead, we combine them with *waitgroups* and *channels* which we'll cover shortly.

---

## 10.2 Context

Go introduced *Context* in version 1.7 in 2016. Context solves one important problem for us: *it provides a way to maintain control across processes*. 

Its application is not limited to use with goroutines, but it fits well with our discussion of concurrency, and the problems of communication and control we may encounter when writing asynchronous code.

We've already seen how simple it is to run something concurrently using a goroutine and we'll look at the different approaches we can employ for signalling and messaging, **from** goroutines, back to our main program, later in this chapter.

But what about the other way? How does our main program communicate with a goroutine once it has started? How does it cancel a goroutine, for example, if the program must exit, allowing the goroutine the opportunity to clean up after itself and end gracefully?

We also need to manage goroutines during execution and not just when closing down our program. For example, if we are blocking while waiting for goroutines to finish their work, a single goroutine that can't complete its work could result in allocated memory which we can't reclaim and a slow or unresponsive program.

So, while it's easy to run a function concurrently by starting it in a new goroutine, managing that goroutine once it has started, is not as straightforward.

The context package solves this problem. Using a context, we can communicate *timeouts*, *deadlines*, *cancellations*, and other request-scoped values across API boundaries, processes and goroutines.

We still can't ensure a goroutine finishes its work - many factors will be beyond our control - but the context package allows us to limit the execution window for that work.

### 10.2.1 Conventions

There are two conventions we need to be aware of. We should use *ctx* as the idiomatic name for a variable which holds a context and, any function or receiver which accepts a context value, should take it as the first parameter.

### 10.2.2 The context.Background() empty context

The function `context.Background()` returns an empty context or `emptyCtx`. Think of it as the parent or root context, it has no cancellation rules, and no values or deadline.

Since context can be chained: a child can inherit a parent context and can add to the context, so cancellation rules, values, and deadlines can be added later.

Although its application is normally limited to `main()`, `context.Background()` is often used in testing of functions which require a context.

### 10.2.3 The context.TODO() empty context

This is essentially a placeholder. In fact, `context.TODO()` returns the same empty context as `context.Background()`, but the intended purpose of `context.TODO()` is different. 

Since not all Go code uses context and, when developing or refactoring our code to use context it may not be immediately obvious how the context will be made available, we have `TODO`, a placeholder. We're saying we know this function will accept a context, but we don't know much more than that at this time.

`context.TODO()` should be used in function arguments in preference to `context.Background()`, the compiler differentiates between them when checking for correct context propagation.

### 10.2.4 Context with cancellation

Cancellation allows us to manually cancel those goroutines which have been written to handle and honour the context contract. 

We have to write code in the goroutine to accomplish this, it isn't an automatic process. Goroutines which don't honour the contract will not terminate, despite a cancellation signal.

To implement cancellation on the current `context` we use the `context.WithCancel()` function. This returns a *cancelFunc* type, a function, which we can defer, or call explicitly in response to an event such as a control signal to terminate our program.

In the example below, we call the `cancel()` function after a short delay. One goroutine has been written to handle the context and returns on the cancellation signal. The second goroutine doesn't handle context, so carries on regardless, or at least until the timer unblocks and the program exits. 

**Example 127 - Using context.WithCancel()**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)

	// wait 5 seconds then call the cancelFunc
	// which will stop any running goroutines which honour context
	go func() {
		time.Sleep(5 * time.Second)
		cancel()
	}()

	// no context
	go func() {
		for {
			time.Sleep(2 * time.Second)
			fmt.Println("I'm Dave Blogs and I AM unstoppable")
		}
	}()

	// with context
	go func() {
		for {
			select {
			case <-time.After(1 * time.Second):
				fmt.Println("I'm Joe Blogs I'm unstoppable!")
			case <-ctx.Done():
				err := ctx.Err()
				fmt.Printf("I SPOKE TOO SOON... '%v' OH NO!\n", err)
				fmt.Println("Cleanup and graceful shutdown type stuff performed here...")
				return
			}
		}
	}()

	// exit after 20 seconds
	time.Sleep(20 * time.Second)
}

// Go Playground: https://go.dev/play/p/aAbmie_rtVV
```

See how we extended the parent context, building on the empty context returned by `context.Background()` and adding the cancellation to it?

Note also how one of the goroutines has been written to honour context? We're using a select statement and listening for signals on both the `time.After()` and `ctx.Done()` channels. When we receive on the latter channel, we have the opportunity to stop what we are doing gracefully and generate logs before returning.

Finally, observe how `ctx.Err()` provides specific information about why the context ended, in this case its value was "context canceled".

### 10.2.5 Context with timeout

In the previous example, we created a context which could send cancellation signals. Although we cheated by running the cancellation function after a short delay, we could have called it conditionally.

In this section, we're going to create a context which has a *timeout*. Timeout specifies a duration of time. When we create a context with a timeout we are stating how long we are prepared to wait for the results of the goroutine. The context will expire when that duration of time has elapsed.

> **Note**: In this example and the examples which follow, the way we handle the context contract inside our goroutines is identical to the way we did it for cancellation, it's only the way we set up the context which differs. 

In the example below, we've simplified the program so it only includes one goroutine, which handles context.

---

## 10.3 Blocking execution with waitgroups

Waitgroups provide a simple way to wait for multiple goroutines to complete, before continuing program execution. The alternative would be a naive pause for some arbitrary amount of time as we saw in the previous examples, **which is not recommended**, or implementing more complex communication over channels, purely to signal completion.  

Waitgroups are most useful where there is no requirement to communicate results or data between goroutines, something which would mandate the use of channels.

Goroutines managed by a waitgroup are capable of updating simple shared state, for example, a counter, or an array of known size.

In the next example, we use a waitgroup to hold execution until goroutines have completed, each populating an element in the `results` array. The example demonstrates how to orchestrate a waitgroup, and how safe state updates are possible in some circumstances.

> **Note**: Beyond a demonstration, this would be a poor application for concurrency. The code itself is very simple, and our inference *should* be that is unlikely to be faster than it's synchronous counterpart. In my testing it was between two and three times slower to schedule goroutines, than to populate the array directly.

**Example 131 - WaitGroup updating shared state safely**

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	t := time.Now()
	var results [30]int

	var wg sync.WaitGroup
	//wg.Add(29)

	for i := 1; i <= 30; i++ {
		wg.Add(1)
		go func(i int) {
			results[i-1] = i * 10
			wg.Done()
		}(i)
	}

	wg.Wait()
	fmt.Println(results)
	fmt.Println("Completed in", time.Since(t))
}

// Go Playground: https://go.dev/play/p/Z3Rh0ufgiVr
```

Running through the code. The `wg` variable is of type *sync.WaitGroup*. We're making use of closures so not providing the `wg` argument as a parameter to the anonymous function. When a *sync.Waitgroup* typed variable is passed as an argument to a named function it should be passed by reference and not by value.

We add `1` to the waitgroup every time we invoke a new goroutine with `wg.Add(1)` and when each function has completed its work we call the `wg.Done()` receiver.

`wg.Wait()` blocks execution until all goroutines report they have completed via `wg.Done()`.

> **Note**: Observe `wg.Add(29)` on line 14. It is generally better to add goroutines to the waitgroup at the point of use - for example in the loop as we do. If there is a mismatch between the number of goroutines we add, and the number which call `wg.Done()` our program will panic at runtime. By calling `wg.Add(1)` in the loop which invokes the goroutine we prevent this possibility. Uncomment line 14, and comment out line 17 to experiment for yourself.

To summarise, waitgroups are simple to reason about. The waitgroup understands how many completion notifications it should expect, and each goroutine informs the waitgroup when it has completed its work. When all goroutines are accounted for, the waitgroup unblocks and program execution continues.

---

## 10.4 Sharing variables using mutexes

In the previous example, we used goroutines to populate the elements of an array which had a predefined size and capacity. It's safe to do this, because, as the code is written, each goroutine mutates just one element in the array. We are sharing the array, but we're not sharing the elements which comprise the array. 

But few values can be safely shared in this way. Usually, unguarded concurrent access is unsafe, potentially giving rise to a race condition. Certainly any concurrent operation which reads a shared value and then changes that value is unsafe.

Consider the following example, which creates 10,000 goroutines, each incrementing the `myCounter` variable by a value of `1`. There is a waitgroup in place, so the program blocks until all goroutines have completed. 

We print the `myCounter` variable once the work is done and expect the printed output to be `10000`.

**Example 132 - WaitGroup updating shared state unsafely**

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var myCounter int

	var wg sync.WaitGroup
	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func() {
			myCounter += 1
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Printf("Counter total is: %d\n", myCounter)
}

// Go Playground: https://go.dev/play/p/dW1kdCrqmCu
```

Output: 
```
Counter total is: 9764

Program exited.
```

But it isn't 10,000. In fact every time we run the code, we get a different result, and it's never 10,000. Our code has a problem since there is unguarded shared access to the `myCounter` variable. Multiple goroutines can read the value in that memory and increment it, simultaneously.

The actual value printed for `myCounter` depends on how many goroutines manage to read and write to `myCounter` atomically before another goroutine reads the variable. There's no way we can predict that access which is why we get a different result each time. 

In short, we have a *race condition* or *data race*. 

### 10.4.1 Implementing mutually exclusive access

Mutexes are guard mechanisms which guarantee exclusive access to a piece of memory. They are part of the `sync` package along with waitgroups. To declare a variable (or struct field) which holds a value of type `sync.Mutex` we generally use the naming convention `mu`.

We're going to improve the previous code and remove the race condition. We'll use a *mutex* in the example which follows, but in section 10.3 we'll solve the same problem using a *channel*.

**Example 133 - Mutex to guarantee exclusivity of access**

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var myCounter int
	var wg sync.WaitGroup

	var mu sync.Mutex

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func() {
			mu.Lock()
			myCounter += 1
			mu.Unlock()
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Printf("Counter total is: %d\n", myCounter)
}

// Go Playground: https://go.dev/play/p/o_M7JXKFzHs
```

[Note: The example appears to be cut off in the source, but the key pattern is shown with mu.Lock() and mu.Unlock() protecting the critical section.]

---

## 10.5 Communicating with channels

We discussed channels briefly in Chapter 7 and we've already seen channel mechanics employed earlier in this chapter when we considered *context*, which uses channels in its implementation to send cancellation signals.

For the remainder of this chapter, we'll look at how channels provide a mechanism for messaging and signalling between concurrent goroutines. There's a lot to cover, but we'll progress slowly, and hopefully logically. By the end of this chapter, we'll have channels down. 

Let's get started!

### 10.5.1 Signalling versus messaging

Messaging over channels speaks for itself. We pass messages, or data, which could include work to be performed, results of work and possibly errors too, between goroutines, or from goroutines back to our *main* execution.

But what about *signalling*? When signalling over channels, we may still send data, but the data is *signal* rather than *information*. Channels for signalling, provide a way for goroutines to inform each other - and main - of their status. 

The idiomatic way to create a signalling channel is to use a channel of type empty struct. Although we'll often see types of *bool* or *int* used too, the empty struct is better for a few reasons.

An empty struct is 0 bytes in size, so it occupies no storage. It **cannot** carry any data, so it is clear the channel is for signalling only. 

If we use *bool* or *int* types, the intent is less clear. Is the channel being used to share state which may fluctuate between `true` and `false`, or carry a number result, or is it for signalling? We can't be sure just by looking at the channel declaration alone. With an empty struct, there is zero ambiguity.

Often, we'll use a signalling channel so goroutines may inform *main* when they have completed their work, similar to how they would call `wg.Done()` if we implemented a *waitgroup*. That's probably one of the most common applications of signalling channels, but we can do more.

By way of demonstration, in this next example, we'll solve the race condition problem we identified in the previous section. But we won't use a mutex this time, instead, we'll leverage the signalling (and blocking) nature of channels, to implement a guard around the shared `myCounter` variable.

**Example 134 - Signalling channel for mutual exclusion**

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var myCounter int
	var wg sync.WaitGroup

	var lock = make(chan struct{}, 1)

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func() {
			lock <- struct{}{}
			myCounter += 1
			<-lock

			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Printf("Counter total is: %d\n", myCounter)
}

// Go Playground: https://go.dev/play/p/nKGECMF80Z_K
```

[Note: This example appears to be cut off in the source, but shows the pattern of using a buffered channel with capacity 1 as a mutex-like semaphore.]

---

## Analysis for Go 1.20-1.25 Updates Needed

### Current Coverage (Go 1.19)

The chapter currently covers:

1. **Goroutines**: Basic creation, scheduling, GOMAXPROCS
2. **Context**: Background, TODO, WithCancel, WithTimeout patterns
3. **WaitGroups**: sync.WaitGroup for coordination
4. **Mutexes**: sync.Mutex for exclusive access
5. **Channels**: Basic signalling and messaging patterns

### Missing Go 1.20-1.25 Features

1. **WaitGroup.Go() method** (Go 1.20+): New method for simpler goroutine management
2. **testing/synctest package** (Go 1.23+): Testing utilities for concurrent code
3. **Enhanced context features**: New context patterns and improvements
4. **Improved channel operations**: Performance and semantic improvements
5. **sync package enhancements**: Additional synchronization primitives
6. **Structured concurrency patterns**: Modern approaches to goroutine lifecycle management

### Recommendations for Update

1. **Add WaitGroup.Go() section**: Show the new simpler pattern for common use cases
2. **Add testing section**: Cover testing/synctest for concurrent code testing
3. **Update context examples**: Include newer context patterns and best practices
4. **Add modern patterns**: Structured concurrency, worker pools, graceful shutdown
5. **Update performance guidance**: Latest scheduler improvements and recommendations
6. **Add error handling**: Better patterns for error propagation in concurrent code

### Code Examples to Update

- All playground links need verification for Go 1.25 compatibility
- Add examples showing WaitGroup.Go() usage
- Include testing/synctest examples
- Show modern context cancellation patterns
- Demonstrate improved channel patterns

### Style and Approach

The chapter maintains the author's clear, example-driven approach with:
- Simple, focused examples
- Practical gotchas and warnings
- Progressive complexity
- Emphasis on idiomatic Go patterns

This foundation is solid and the updates should preserve this teaching style while adding modern Go concurrency features.