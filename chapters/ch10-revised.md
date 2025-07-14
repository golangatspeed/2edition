# Chapter 10 - Concurrency

One of the most attractive features of Go is its support for *concurrency* which provides the ability to run multiple tasks at once.

In this chapter, we're going to explore Go's concurrency features, including *goroutines*, *waitgroups*, and *channels*. We'll learn how to use them, why they work well together, and how *context* also contributes to writing safer concurrent programs.

We'll also look at another more traditional style of concurrent programming, one where we share memory but use *mutexes* to lock memory for use by a single process at a time.

By the end of this chapter, we'll have the ability to identify opportunities for using concurrency in our programs. We'll know how to use the appropriate tools and, understand the risks associated with introducing asynchronous code blocks into our codebase.

> *Concurrency* is not the same as *parallelism*. With concurrency, Go's included *scheduler* manages tasks which are in a *runnable* state, starting and stopping them based on several factors. The result is that *usually*, tasks complete much faster than if they were run sequentially, **but**, they are not all running at the same time.

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

**Example 104 - Setting GOMAXPROCS during runtime**
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

> The return value of `runtime.GOMAXPROCS()` is the previously set value, not the value just set. We ran the function twice to demonstrate that.

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

**Example 105 - Where is the output?**
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

**Example 106 - Using context.WithCancel()**
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

    go func() {
        for {
            select {
            case <-time.After(1 * time.Second):
                fmt.Println("Working...")
            case <-ctx.Done():
                fmt.Println("Cancelled, cleaning up...")
                return
            }
        }
    }()

    time.Sleep(3 * time.Second)
    cancel()
    time.Sleep(1 * time.Second)
}

// Go Playground: https://go.dev/play/p/contextCancel
```

### 10.2.5 Context with timeout

Context with timeout automatically cancels after a specified duration.

**Example 107 - Using context.WithTimeout()**
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    go func() {
        for {
            select {
            case <-time.After(1 * time.Second):
                fmt.Println("Working with timeout...")
            case <-ctx.Done():
                fmt.Printf("Timeout reached: %v\n", ctx.Err())
                return
            }
        }
    }()

    time.Sleep(6 * time.Second)
    fmt.Println("Exiting")
}

// Go Playground: https://go.dev/play/p/contextTimeout
```

> Note that `context.WithTimeout()` also returns a `cancel` function and we're using *defer* to call it. This ensures that context is cancelled regardless of the time elapsed should the program exit for another reason such as a *panic*. This avoids what the compiler terms, "context leak".

### 10.2.6 Enhanced context patterns

Go 1.20+ introduced several improvements to context handling and new patterns for better resource management.

**Example 108 - Modern context patterns**
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func processWithContext(ctx context.Context, id int) error {
    select {
    case <-time.After(2 * time.Second):
        fmt.Printf("Task %d completed\n", id)
        return nil
    case <-ctx.Done():
        fmt.Printf("Task %d cancelled: %v\n", id, ctx.Err())
        return ctx.Err()
    }
}

func main() {
    // Context with both timeout and cancellation
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    // Start multiple tasks
    for i := 1; i <= 3; i++ {
        go func(id int) {
            if err := processWithContext(ctx, id); err != nil {
                fmt.Printf("Task %d failed: %v\n", id, err)
            }
        }(i)
    }

    time.Sleep(4 * time.Second)
}

// Go Playground: https://go.dev/play/p/modernContext
```

## 10.3 Blocking execution with waitgroups

Waitgroups provide a simple way to wait for multiple goroutines to complete, before continuing program execution. The alternative would be a naive pause for some arbitrary amount of time as we saw in the previous examples, **which is not recommended**, or implementing more complex communication over channels, purely to signal completion.

Waitgroups are most useful where there is no requirement to communicate results or data between goroutines, something which would mandate the use of channels.

Goroutines managed by a waitgroup are capable of updating simple shared state, for example, a counter, or an array of known size.

**Example 109 - WaitGroup updating shared state safely**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    t := time.Now()
    var results [5]int

    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            results[i-1] = i * 10
        }(i)
    }

    wg.Wait()
    fmt.Println(results)
    fmt.Println("Completed in", time.Since(t))
}

// Go Playground: https://go.dev/play/p/waitGroupBasic
```

> Beyond a demonstration, this would be a poor application for concurrency. The code itself is very simple, and our inference *should* be that is unlikely to be faster than it's synchronous counterpart. In my testing it was between two and three times slower to schedule goroutines, than to populate the array directly.

Running through the code. The `wg` variable is of type *sync.WaitGroup*. We're making use of closures so not providing the `wg` argument as a parameter to the anonymous function. When a *sync.Waitgroup* typed variable is passed as an argument to a named function it should be passed by reference and not by value.

We add `1` to the waitgroup every time we invoke a new goroutine with `wg.Add(1)` and when each function has completed its work we call the `wg.Done()` receiver.

`wg.Wait()` blocks execution until all goroutines report they have completed via `wg.Done()`.

### 10.3.1 Enhanced WaitGroup with Go method

Go 1.20 introduced the `WaitGroup.Go()` method, which provides a more convenient way to start goroutines with automatic waitgroup management.

**Example 110 - Using WaitGroup.Go() method**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func processTask(id int) {
    time.Sleep(time.Duration(id) * 100 * time.Millisecond)
    fmt.Printf("Task %d completed\n", id)
}

func main() {
    var wg sync.WaitGroup

    // Traditional approach
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            processTask(id)
        }(i)
    }

    // Modern approach with WaitGroup.Go()
    for i := 4; i <= 6; i++ {
        wg.Go(func() {
            processTask(i)
        })
    }

    wg.Wait()
    fmt.Println("All tasks completed")
}

// Go Playground: https://go.dev/play/p/waitGroupGo
```

The `WaitGroup.Go()` method automatically calls `Add(1)` before starting the goroutine, eliminating the possibility of forgetting to call `Add()` or mismatching the count.

**Example 111 - Error handling with WaitGroup.Go()**
```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    var wg sync.WaitGroup
    var errorCount int64

    tasks := []string{"task1", "task2", "task3", "task4", "task5"}

    for _, task := range tasks {
        wg.Go(func() {
            if err := processTaskWithError(task); err != nil {
                atomic.AddInt64(&errorCount, 1)
                fmt.Printf("Task %s failed: %v\n", task, err)
            }
        })
    }

    wg.Wait()
    fmt.Printf("Completed with %d errors\n", errorCount)
}

func processTaskWithError(task string) error {
    // Simulate some tasks failing
    if task == "task3" {
        return fmt.Errorf("simulated error")
    }
    fmt.Printf("Task %s succeeded\n", task)
    return nil
}

// Go Playground: https://go.dev/play/p/waitGroupGoError
```

To summarise, waitgroups are simple to reason about. The waitgroup understands how many completion notifications it should expect, and each goroutine informs the waitgroup when it has completed its work. When all goroutines are accounted for, the waitgroup unblocks and program execution continues.

## 10.4 Sharing variables using mutexes

In the previous example, we used goroutines to populate the elements of an array which had a predefined size and capacity. It's safe to do this, because, as the code is written, each goroutine mutates just one element in the array. We are sharing the array, but we're not sharing the elements which comprise the array.

But few values can be safely shared in this way. Usually, unguarded concurrent access is unsafe, potentially giving rise to a race condition. Certainly any concurrent operation which reads a shared value and then changes that value is unsafe.

Consider the following example, which creates multiple goroutines, each incrementing the `myCounter` variable by a value of `1`. There is a waitgroup in place, so the program blocks until all goroutines have completed.

**Example 112 - Race condition demonstration**
```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var myCounter int
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            myCounter++ // Race condition!
        }()
    }

    wg.Wait()
    fmt.Printf("Counter: %d (expected: 1000)\n", myCounter)
}

// Go Playground: https://go.dev/play/p/raceCondition
```

We print the `myCounter` variable once the work is done and expect the printed output to be `1000`. However, you'll likely see a different number each time you run this program due to the race condition.

**Example 113 - Safe concurrent access with mutex**
```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var myCounter int
    var mu sync.Mutex
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            myCounter++
            mu.Unlock()
        }()
    }

    wg.Wait()
    fmt.Printf("Counter: %d (expected: 1000)\n", myCounter)
}

// Go Playground: https://go.dev/play/p/mutexSafe
```

By adding a mutex (`sync.Mutex`), we ensure that only one goroutine can access the shared variable at a time, eliminating the race condition.

## 10.5 Channels

Channels provide communication between goroutines and are Go's implementation of the CSP (Communicating Sequential Processes) model. There are two primary use cases for channels: *signalling* and *messaging*.

**Example 114 - Basic channel communication**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Unbuffered channel
    ch := make(chan string)

    go func() {
        time.Sleep(2 * time.Second)
        ch <- "Hello from goroutine"
    }()

    message := <-ch
    fmt.Println(message)
}

// Go Playground: https://go.dev/play/p/basicChannel
```

### 10.5.1 Buffered channels

Buffered channels allow asynchronous communication up to the buffer size.

**Example 115 - Buffered channels**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Buffered channel with capacity 3
    ch := make(chan int, 3)

    // Send values without blocking
    ch <- 1
    ch <- 2
    ch <- 3

    go func() {
        for i := range ch {
            fmt.Printf("Received: %d\n", i)
            time.Sleep(500 * time.Millisecond)
        }
    }()

    time.Sleep(2 * time.Second)
    close(ch)
    time.Sleep(1 * time.Second)
}

// Go Playground: https://go.dev/play/p/bufferedChannel
```

### 10.5.2 Select statement for channel operations

The `select` statement allows non-blocking channel operations and handling multiple channels.

**Example 116 - Select with multiple channels**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "Channel 1"
    }()

    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "Channel 2"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received from", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received from", msg2)
        case <-time.After(3 * time.Second):
            fmt.Println("Timeout")
        }
    }
}

// Go Playground: https://go.dev/play/p/selectChannels
```

## 10.6 Testing concurrent code

Go 1.23 stabilized the `testing/synctest` package, which provides powerful utilities for testing concurrent code reliably.

**Example 117 - Testing concurrent code with synctest**
```go
package main

import (
    "sync"
    "testing"
    "testing/synctest"
    "time"
)

func ConcurrentCounter(iterations int) int {
    var counter int
    var mu sync.Mutex
    var wg sync.WaitGroup

    for i := 0; i < iterations; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            mu.Lock()
            counter++
            mu.Unlock()
        }()
    }

    wg.Wait()
    return counter
}

func TestConcurrentCounter(t *testing.T) {
    synctest.Run(func() {
        result := ConcurrentCounter(100)
        if result != 100 {
            t.Errorf("Expected 100, got %d", result)
        }
    })
}

func TestChannelCommunication(t *testing.T) {
    synctest.Run(func() {
        ch := make(chan int, 2)
        
        go func() {
            ch <- 1
            ch <- 2
            close(ch)
        }()

        var results []int
        for val := range ch {
            results = append(results, val)
        }

        if len(results) != 2 {
            t.Errorf("Expected 2 values, got %d", len(results))
        }
    })
}

// Go Playground: https://go.dev/play/p/testingSynctest
```

The `testing/synctest` package provides:
- **Deterministic scheduling** for reproducible tests
- **Race condition detection** without external tools
- **Controlled timing** for time-sensitive operations
- **Deadlock detection** for concurrent code

## Summary

Go's concurrency model provides powerful tools for building concurrent applications:

- **Goroutines** offer lightweight concurrency
- **Context** provides cancellation and timeout control
- **WaitGroups** coordinate goroutine completion
- **Mutexes** ensure safe shared memory access
- **Channels** enable communication between goroutines
- **testing/synctest** helps test concurrent code reliably

The key to effective concurrent programming in Go is understanding when and how to use each tool. Start simple with goroutines and waitgroups, add context for control, use channels for communication, and employ mutexes when sharing state is necessary.

Modern Go versions have made concurrent programming even more accessible with improvements like `WaitGroup.Go()` and `testing/synctest`, while maintaining the elegant simplicity that makes Go's concurrency model so appealing.