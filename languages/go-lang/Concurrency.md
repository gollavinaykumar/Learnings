# Go (Golang) Concurrency — Complete Study & Interview Revision Notes

> **Goal:** Learn Go concurrency from the basics to advanced production
> patterns, with simple explanations, diagrams, runnable examples,
> common bugs, debugging techniques, and interview questions.
>
> **Audience:** Developers who know basic Go syntax and want to
> understand how to build safe concurrent services.

------------------------------------------------------------------------

## Table of Contents

1.  [Concurrency vs. Parallelism](#1-concurrency-vs-parallelism)
2.  [Why Go Uses Goroutines](#2-why-go-uses-goroutines)
3.  [Your First Goroutine](#3-your-first-goroutine)
4.  [The Main Function and Program
    Lifetime](#4-the-main-function-and-program-lifetime)
5.  [Goroutines vs. OS Threads](#5-goroutines-vs-os-threads)
6.  [Channels: Sending Data Between
    Goroutines](#6-channels-sending-data-between-goroutines)
7.  [Buffered and Unbuffered
    Channels](#7-buffered-and-unbuffered-channels)
8.  [Channel Direction and Closing
    Channels](#8-channel-direction-and-closing-channels)
9.  [The `select` Statement](#9-the-select-statement)
10. [Race Conditions and Shared
    Memory](#10-race-conditions-and-shared-memory)
11. [Mutexes and RWMutex](#11-mutexes-and-rwmutex)
12. [Atomic Operations](#12-atomic-operations)
13. [WaitGroup](#13-waitgroup)
14. [Context, Cancellation, and
    Timeouts](#14-context-cancellation-and-timeouts)
15. [Worker Pools](#15-worker-pools)
16. [Fan-Out, Fan-In, and Pipelines](#16-fan-out-fan-in-and-pipelines)
17. [Rate Limiting and Semaphores](#17-rate-limiting-and-semaphores)
18. [Deadlocks, Goroutine Leaks, and Other Failure
    Modes](#18-deadlocks-goroutine-leaks-and-other-failure-modes)
19. [Error Handling in Concurrent
    Code](#19-error-handling-in-concurrent-code)
20. [Generics and Reusable Concurrency
    Helpers](#20-generics-and-reusable-concurrency-helpers)
21. [Go Scheduler: G, M, and P](#21-go-scheduler-g-m-and-p)
22. [Memory Model and
    Happens-Before](#22-memory-model-and-happens-before)
23. [Performance: When Concurrency
    Helps](#23-performance-when-concurrency-helps)
24. [HTTP Servers and Real Production
    Scenarios](#24-http-servers-and-real-production-scenarios)
25. [Testing Concurrent Code](#25-testing-concurrent-code)
26. [Debugging and Profiling Tools](#26-debugging-and-profiling-tools)
27. [Advanced Patterns and Newer Go
    Features](#27-advanced-patterns-and-newer-go-features)
28. [Common Mistakes Checklist](#28-common-mistakes-checklist)
29. [Java vs. Go Concurrency](#29-java-vs-go-concurrency)
30. [Interview Questions and Short
    Answers](#30-interview-questions-and-short-answers)
31. [Learning Path](#31-learning-path)
32. [One-Minute Revision Cheat
    Sheet](#32-one-minute-revision-cheat-sheet)

------------------------------------------------------------------------

## 1. Concurrency vs. Parallelism

### Concurrency — handling multiple tasks in overlapping time

Concurrency is about **structuring a program so several tasks can make
progress** during the same period.

Example: a logistics service receives a load request, checks a carrier,
fetches a route distance, and writes an audit event. These tasks may
overlap while waiting for I/O.

### Parallelism — doing multiple tasks at exactly the same time

Parallelism means multiple operations physically execute simultaneously,
usually on different CPU cores.

``` text
Concurrency: one worker switches between tasks

Time ──────────────────────────────────────>
Task A: [work][wait........][work]
Task B:       [work][wait][work]
Task C:              [work][wait][work]

Parallelism: tasks execute at the same time on multiple cores

Core 1: [ Task A ][ Task A ][ Task A ]
Core 2: [ Task B ][ Task B ][ Task B ]
Core 3: [ Task C ][ Task C ][ Task C ]
```

**Remember:** Concurrency is a way to design a program; parallelism is
one way work can execute.

### When does concurrency help?

- **I/O-bound work:** HTTP requests, database calls, file reads, message
  queues. A task can wait while other tasks progress.
- **CPU-bound work:** image processing, compression, calculations.
  Parallel execution may help if work is split across cores.
- **Not always helpful:** tiny tasks, highly contended shared state, or
  work that is already limited by a remote API.

------------------------------------------------------------------------

## 2. Why Go Uses Goroutines

Go gives you **goroutines**: lightweight units of concurrent execution
managed by the Go runtime.

A goroutine is started with the `go` keyword.

``` go
go doWork()
```

Compared with creating a native OS thread for every task, goroutines
typically have a much smaller initial stack and the runtime schedules
many goroutines over a smaller set of OS threads. They can grow their
stacks as needed.

Go’s commonly used concurrency building blocks are:

- **Goroutines:** run functions concurrently.
- **Channels:** transfer values and coordinate goroutines.
- **`sync` package:** mutexes, wait groups, once-only initialization,
  and related primitives.
- **`sync/atomic`:** atomic operations on supported values.
- **`context`:** cancellation, deadlines, and request-scoped values.
- **`select`:** wait on multiple channel operations.

A useful principle from Go’s design culture is: **communicate by sharing
data through channels, and share memory by coordinating access**. Both
channels and locks are valid; choose the clearest tool for the job.

------------------------------------------------------------------------

## 3. Your First Goroutine

### Sequential version

``` go
package main

import (
    "fmt"
    "time"
)

func printMessage(message string) {
    for i := 1; i <= 3; i++ {
        fmt.Println(message, i)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    printMessage("A")
    printMessage("B")
}
```

The second function starts only after the first finishes.

### Concurrent version

``` go
package main

import (
    "fmt"
    "sync"
    "time"
)

func printMessage(wg *sync.WaitGroup, message string) {
    defer wg.Done()

    for i := 1; i <= 3; i++ {
        fmt.Println(message, i)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    var wg sync.WaitGroup
    wg.Add(2)

    go printMessage(&wg, "A")
    go printMessage(&wg, "B")

    wg.Wait()
    fmt.Println("Both tasks completed")
}
```

The output order of `A` and `B` is **not guaranteed**. They can
interleave differently on different runs.

### What does `go` actually do?

``` go
go sendNotification()
```

This asks the runtime to start `sendNotification` in a new goroutine. It
does **not** mean “run on another CPU core immediately,” and it does not
make the caller wait.

### A common bug: main exits too soon

``` go
func main() {
    go fmt.Println("Hello from goroutine")
}
```

The program may exit before the goroutine gets to print. A goroutine
does not keep the process alive by itself. Coordinate completion using a
`WaitGroup`, channel, or another suitable mechanism.

------------------------------------------------------------------------

## 4. The Main Function and Program Lifetime

`main.main` is the entry point of a Go executable. When it returns, the
process exits; Go does not wait automatically for other goroutines.

### Correct coordination with a channel

``` go
package main

import "fmt"

func main() {
    done := make(chan struct{})

    go func() {
        fmt.Println("Background work finished")
        close(done)
    }()

    <-done
}
```

A `chan struct{}` is often used for a signal because the empty struct
carries no data.

**Production note:** Do not use arbitrary `time.Sleep` calls as a
substitute for synchronization. Sleep may make a demo appear to work,
but it does not establish a reliable completion or visibility
relationship.

------------------------------------------------------------------------

## 5. Goroutines vs. OS Threads

| Feature               | Goroutine                           | OS thread                                        |
|-----------------------|-------------------------------------|--------------------------------------------------|
| Managed by            | Go runtime                          | Operating system                                 |
| Typical initial stack | Small, grows as needed              | Usually larger fixed/reserved stack              |
| Scheduling            | Go scheduler multiplexes goroutines | OS scheduler schedules threads                   |
| Creation cost         | Usually low                         | Usually higher                                   |
| Best use              | Many concurrent tasks               | Native execution unit used by the runtime and OS |

These are general characteristics, not guarantees about exact memory or
timing. Costs depend on the Go version, operating system, workload, and
runtime configuration.

### Does one goroutine equal one thread?

No. Thousands of goroutines may run on a smaller number of OS threads. A
goroutine can be paused while waiting and another goroutine can use the
available execution resources.

### `GOMAXPROCS`

`GOMAXPROCS` controls how many OS threads may execute Go code
simultaneously in the Go scheduler’s P execution resources. It is not a
limit on the number of goroutines. Modern Go versions can adjust the
default based on available CPU resources and runtime behavior.

``` go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Println("Logical CPUs:", runtime.NumCPU())
    fmt.Println("GOMAXPROCS:", runtime.GOMAXPROCS(0))
    fmt.Println("Goroutines now:", runtime.NumGoroutine())
}
```

Use these values for diagnostics, not as proof that a workload will
scale linearly.

------------------------------------------------------------------------

## 6. Channels: Sending Data Between Goroutines

A **channel** is a typed communication path between goroutines.

``` go
numbers := make(chan int)
```

Send a value using `<-`:

``` go
numbers <- 42
```

Receive a value:

``` go
value := <-numbers
```

### Diagram

``` text
Goroutine A                         Goroutine B
     |                                   |
     | -------- numbers <- 42 ---------> |
     |                                   |
     |                              value := <-numbers
```

### Complete example

``` go
package main

import "fmt"

func calculate(result chan<- int) {
    result <- 6 * 7
}

func main() {
    result := make(chan int)

    go calculate(result)

    answer := <-result
    fmt.Println(answer) // 42
}
```

`chan<- int` means the function may send integers to the channel but
cannot receive from it. This documents the intended direction and helps
catch mistakes at compile time.

### Why use channels?

- Pass results from a worker to a caller.
- Signal that an operation is complete.
- Build pipelines and worker pools.
- Apply backpressure by limiting buffered capacity.
- Coordinate cancellation and shutdown.

### Channels are not automatically the best solution

A mutex is often simpler when several goroutines need safe access to one
shared map or struct. A channel is useful when the program naturally
transfers ownership or sends events/results.

------------------------------------------------------------------------

## 7. Buffered and Unbuffered Channels

### Unbuffered channel

``` go
ch := make(chan int)
```

A send and receive must rendezvous: the sender waits until a receiver is
ready, and a receiver waits until a sender is ready.

``` text
Sender: ch <- 10  ── waits ──┐
                             ├── transfer happens
Receiver: <-ch     ── waits ─┘
```

### Buffered channel

``` go
ch := make(chan int, 3)
```

The channel can hold up to three values without a receiver immediately
taking each one.

``` text
Capacity = 3

+-----+-----+-----+
|  10 |  20 |  30 |  Buffer full
+-----+-----+-----+
```

A send blocks when the buffer is full. A receive blocks when the buffer
is empty.

``` go
package main

import "fmt"

func main() {
    ch := make(chan string, 2)

    ch <- "first"
    ch <- "second"

    fmt.Println(<-ch) // first
    fmt.Println(<-ch) // second
}
```

### How do you choose capacity?

- Start with unbuffered if synchronization is important and you have no
  measured reason to buffer.
- Use a small buffer to absorb short bursts or decouple producers from
  consumers.
- Use a bounded buffer to create backpressure.
- Avoid arbitrarily huge buffers: they can hide slow consumers and
  increase memory usage.

**Important:** Buffering does not fix a missing receiver forever. If
producers can outpace consumers without bound, a finite buffer
eventually fills.

------------------------------------------------------------------------

## 8. Channel Direction and Closing Channels

### Directional channel types

``` go
func produce(out chan<- int) { // send-only
    out <- 100
}

func consume(in <-chan int) { // receive-only
    value := <-in
    println(value)
}
```

Directional types make function contracts clearer.

### Closing a channel

`close(ch)` means **no more values will be sent**. It does not destroy
the channel, and it does not mean “cancel every receiver.”

``` go
package main

import "fmt"

func main() {
    ch := make(chan int)

    go func() {
        defer close(ch)
        for i := 1; i <= 3; i++ {
            ch <- i
        }
    }()

    for value := range ch {
        fmt.Println(value)
    }
}
```

A `for range` over a channel continues receiving until the channel is
closed and all buffered values have been drained.

### Rules to remember

- The goroutine responsible for producing values usually owns the
  decision to close the channel.
- Do not close a channel from the receiving side just because the
  receiver is finished.
- Sending to a closed channel panics.
- Closing a closed channel panics.
- Receiving from a closed, drained channel returns the zero value
  immediately.
- Use the two-value receive to distinguish a real zero value from a
  closed channel:

``` go
value, ok := <-ch
if !ok {
    fmt.Println("Channel is closed and drained")
    return
}
fmt.Println(value)
```

### Avoid double-close bugs

If multiple producers share one channel, one producer must not close it
while others may still send. A coordinator can wait for all producers
and then close the output channel.

------------------------------------------------------------------------

## 9. The `select` Statement

`select` waits for one of several channel operations.

``` go
select {
case value := <-results:
    fmt.Println("Got:", value)
case <-time.After(2 * time.Second):
    fmt.Println("Timed out")
}
```

If several cases are ready, Go chooses one ready case pseudo-randomly;
do not assume a fixed priority order.

### Cancellation example

``` go
func worker(ctx context.Context, jobs <-chan int) {
    for {
        select {
        case <-ctx.Done():
            return
        case job, ok := <-jobs:
            if !ok {
                return
            }
            process(job)
        }
    }
}
```

A `default` case makes `select` non-blocking:

``` go
select {
case ch <- value:
    fmt.Println("Sent")
default:
    fmt.Println("Would block; doing something else")
}
```

Use `default` carefully. A tight loop around a non-blocking `select` can
consume CPU while doing no useful work.

------------------------------------------------------------------------

## 10. Race Conditions and Shared Memory

A **data race** occurs when two goroutines access the same memory
concurrently, at least one access is a write, and the accesses are not
properly synchronized.

### Unsafe counter

``` go
var count int

func increment() {
    count++ // read, add, write; not one indivisible operation
}
```

If many goroutines call `increment`, updates can be lost.

``` text
count = 0

Goroutine A reads 0
Goroutine B reads 0
Goroutine A writes 1
Goroutine B writes 1

Expected after two increments: 2
Actual: 1
```

### Fix 1: mutex

``` go
var (
    mu    sync.Mutex
    count int
)

func increment() {
    mu.Lock()
    count++
    mu.Unlock()
}
```

Prefer `defer mu.Unlock()` in functions where early returns or panics
could otherwise leave the lock held:

``` go
func increment() {
    mu.Lock()
    defer mu.Unlock()
    count++
}
```

### Fix 2: atomic integer

``` go
var count atomic.Int64

func increment() {
    count.Add(1)
}
```

Atomic is useful for a single independent counter. It is not enough to
make a multi-step business operation atomic.

### Detect races

``` bash
go test -race ./...
go run -race .
```

The race detector finds many data races exercised by the test or program
run. It does not prove that no race exists; run representative tests and
workloads.

------------------------------------------------------------------------

## 11. Mutexes and RWMutex

### `sync.Mutex`

A mutex allows only one goroutine at a time to hold the lock.

``` go
type SafeCounter struct {
    mu    sync.Mutex
    value int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}
```

### `sync.RWMutex`

`RWMutex` allows multiple readers or one writer. Readers use
`RLock`/`RUnlock`; writers use `Lock`/`Unlock`.

``` go
type Cache struct {
    mu    sync.RWMutex
    items map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    value, ok := c.items[key]
    return value, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.items[key] = value
}
```

Initialize the map before calling `Set`, or add initialization logic
under the write lock.

### Mutex vs. RWMutex

| Situation                                       | Usually consider                                                       |
|-------------------------------------------------|------------------------------------------------------------------------|
| Simple shared state                             | `sync.Mutex`                                                           |
| Many reads, infrequent writes; measured benefit | `sync.RWMutex`                                                         |
| Tiny critical section or frequent writes        | Start with `sync.Mutex`                                                |
| Need to coordinate a multi-step transaction     | A lock around the complete invariant, or a transaction in the database |

`RWMutex` is not automatically faster. It has more coordination overhead
and should be chosen based on measured workload.

### Locking rules

- Keep critical sections short.
- Do not call unknown callbacks while holding a lock unless their
  behavior is understood.
- Avoid inconsistent lock ordering across code paths.
- Do not copy a mutex after first use.
- Do not hold a lock while doing slow network I/O unless the design
  explicitly requires it.
- Protect the invariant, not just one individual line.

------------------------------------------------------------------------

## 12. Atomic Operations

The `sync/atomic` package provides atomic operations on supported
values. In modern Go, typed atomic types such as `atomic.Int64` are
usually easier to use.

``` go
var activeRequests atomic.Int64

func requestStarted() {
    activeRequests.Add(1)
}

func requestFinished() {
    activeRequests.Add(-1)
}
```

### What does atomic mean?

An operation is indivisible with respect to concurrent access to that
atomic value. It does **not** automatically make a whole group of
operations or a data structure thread-safe.

### Atomic vs. mutex vs. channel

| Tool    | Best fit                                                   |
|---------|------------------------------------------------------------|
| Atomic  | Simple counters, flags, independent numeric state          |
| Mutex   | Protecting shared maps, structs, and multi-step invariants |
| Channel | Passing values, signaling, ownership transfer, pipelines   |
| Context | Cancellation and deadlines across call boundaries          |

### Common mistake

This is not safe just because one field is atomic:

``` go
type Account struct {
    balance atomic.Int64
}
```

A transfer from account A to account B requires a coordinated operation
across both accounts. Independent atomic reads/writes can still violate
business invariants. Use a suitable locking strategy or database
transaction.

------------------------------------------------------------------------

## 13. WaitGroup

`sync.WaitGroup` waits for a group of goroutines to finish. It does not
transfer result values and does not automatically cancel goroutines.

``` go
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)

    go func(id int) {
        defer wg.Done()
        fmt.Println("worker", id)
    }(i)
}

wg.Wait()
```

### Rules

- Call `Add` before starting the goroutine, not inside it.
- Call `Done` exactly once for each counted task; `defer wg.Done()` is
  common.
- Do not copy a `WaitGroup` after first use.
- Do not reuse it for a new independent batch until the previous `Wait`
  has returned and the previous tasks are complete.
- A `WaitGroup` does not make shared variables safe. If goroutines write
  the same map or counter, you still need synchronization.

### Waiting is not result collection

Use a channel or another result mechanism to collect values. Use
`WaitGroup` to coordinate completion.

------------------------------------------------------------------------

## 14. Context, Cancellation, and Timeouts

The `context` package communicates cancellation, deadlines, and
request-scoped values across API boundaries.

Use context for work that should stop when: - an HTTP request is
canceled; - a parent task is canceled; - a deadline is exceeded; - an
application is shutting down.

### Cancellation example

``` go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("worker stopped:", ctx.Err())
            return
        case <-time.After(200 * time.Millisecond):
            fmt.Println("working...")
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go worker(ctx)

    time.Sleep(650 * time.Millisecond)
    cancel()
    time.Sleep(50 * time.Millisecond) // demo only; use a completion signal in production
}
```

For production code, signal completion explicitly instead of relying on
the final sleep.

### Timeout example

``` go
ctx, cancel := context.WithTimeout(parentCtx, 2*time.Second)
defer cancel()

req, err := http.NewRequestWithContext(
    ctx,
    http.MethodGet,
    "https://example.com",
    nil,
)
if err != nil {
    return err
}

resp, err := http.DefaultClient.Do(req)
if err != nil {
    return err
}
defer resp.Body.Close()
```

### Context best practices

- Pass `context.Context` as the first parameter, usually named `ctx`.
- Do not store a request context in a long-lived struct.
- Call the returned `cancel` function to release resources when
  appropriate.
- Check `ctx.Done()` in long-running loops.
- Cancellation is cooperative: the code and APIs being called must
  respond to it.
- Avoid using context values for optional function parameters or
  arbitrary application state. Use them for request-scoped metadata that
  crosses API boundaries.
- Do not pass `nil` context; use `context.Background()` or an incoming
  context.

------------------------------------------------------------------------

## 15. Worker Pools

A worker pool runs a fixed number of workers that consume jobs from a
shared channel. It limits active work and helps apply backpressure.

### Diagram

``` text
                 jobs channel
Producer ──────> [ job ][ job ][ job ][ job ]
                      |      |      |
                      v      v      v
                   Worker1 Worker2 Worker3
                      |      |      |
                      +------v------+
                        results
```

### Complete example

``` go
package main

import (
    "fmt"
    "sync"
)

type Job struct {
    ID    int
    Value int
}

type Result struct {
    JobID  int
    Square int
}

func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()

    for job := range jobs {
        results <- Result{
            JobID:  job.ID,
            Square: job.Value * job.Value,
        }
    }
}

func main() {
    const workerCount = 3
    const jobCount = 8

    jobs := make(chan Job)
    results := make(chan Result, jobCount)

    var wg sync.WaitGroup
    wg.Add(workerCount)

    for i := 1; i <= workerCount; i++ {
        go worker(i, jobs, results, &wg)
    }

    go func() {
        defer close(jobs)
        for i := 1; i <= jobCount; i++ {
            jobs <- Job{ID: i, Value: i}
        }
    }()

    go func() {
        wg.Wait()
        close(results)
    }()

    for result := range results {
        fmt.Printf("job=%d square=%d\n", result.JobID, result.Square)
    }
}
```

### Why use a worker pool?

- Prevent spawning an unbounded number of expensive tasks.
- Limit concurrent database/API calls.
- Control CPU-heavy work.
- Apply backpressure when a queue is full.
- Make shutdown and observability more predictable.

### How many workers?

- CPU-bound work: benchmark around available CPU parallelism as a
  starting point.
- I/O-bound work: more workers may help, but remote-service limits,
  connection pools, memory, and rate limits matter.
- Always measure throughput, latency, errors, queue depth, and resource
  use.

**Production concern:** A worker pool needs an explicit policy for
overload: block producers, reject jobs, drop low-priority jobs, or
persist them. Do not silently accumulate unlimited work.

------------------------------------------------------------------------

## 16. Fan-Out, Fan-In, and Pipelines

### Fan-out

Send jobs to multiple goroutines to process independent tasks
concurrently.

### Fan-in

Combine results from several goroutines into one output stream.

``` text
                +--> Worker A --+
Input --> split +--> Worker B --+--> merge --> Output
                +--> Worker C --+
```

### Example: fan-out/fan-in

``` go
package main

import (
    "fmt"
    "sync"
)

func worker(input <-chan int, output chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    for n := range input {
        output <- n * 2
    }
}

func main() {
    input := make(chan int)
    output := make(chan int)

    var wg sync.WaitGroup
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go worker(input, output, &wg)
    }

    go func() {
        for n := 1; n <= 5; n++ {
            input <- n
        }
        close(input)
    }()

    go func() {
        wg.Wait()
        close(output)
    }()

    for result := range output {
        fmt.Println(result)
    }
}
```

The result order is not guaranteed because workers finish in different
orders. If order matters, include sequence numbers and reorder at the
consumer.

### Pipeline design checklist

1.  Define who owns and closes each channel.
2.  Decide what happens when a stage errors.
3.  Propagate cancellation downstream and upstream.
4.  Ensure blocked sends can be interrupted or that receivers are
    guaranteed to continue.
5.  Bound queues so slow consumers create backpressure instead of
    unbounded memory use.
6.  Decide whether partial results are valid.

------------------------------------------------------------------------

## 17. Rate Limiting and Semaphores

### Rate limit vs. concurrency limit

- **Rate limit:** limits operations per unit of time, such as 20
  requests per second.
- **Concurrency limit:** limits how many operations are in progress at
  once, such as at most 8 simultaneous calls.

These solve different problems.

### Simple concurrency limit using a buffered channel

``` go
limit := make(chan struct{}, 8)

for _, item := range items {
    limit <- struct{}{} // acquire a slot; blocks when all slots are used

    go func(item Item) {
        defer func() { <-limit }() // release slot
        process(item)
    }(item)
}
```

This is a simple semaphore pattern, but production code should also
include cancellation, error collection, and a bounded task-submission
strategy. If `items` is huge, spawning one goroutine per item and making
most wait for the semaphore can still consume too much memory.

### Rate limiter with `golang.org/x/time/rate`

The `x/time/rate` package is commonly used for token-bucket rate
limiting. Add the dependency using your project’s module tooling and
consult its documentation for the version you use.

``` go
// Conceptual example; import golang.org/x/time/rate
limiter := rate.NewLimiter(rate.Limit(10), 5) // 10 events/second, burst 5

for _, item := range items {
    if err := limiter.Wait(ctx); err != nil {
        return err
    }
    process(item)
}
```

This limits the rate at which processing begins. A concurrency semaphore
is still needed if in-flight work must be capped independently.

------------------------------------------------------------------------

## 18. Deadlocks, Goroutine Leaks, and Other Failure Modes

### Deadlock

A deadlock happens when goroutines wait forever for progress that cannot
occur.

``` go
func main() {
    ch := make(chan int)
    ch <- 1 // deadlocks: no receiver can run
}
```

Another example is a lock cycle:

``` text
Goroutine A holds Lock 1, waits for Lock 2
Goroutine B holds Lock 2, waits for Lock 1
Neither can continue.
```

**Prevention:** use consistent lock ordering, review channel
send/receive ownership, and ensure every blocking operation has a valid
path to progress or cancellation.

### Goroutine leak

A goroutine leak occurs when a goroutine remains blocked or running
after its work is no longer useful.

Common causes: - sending to a channel that nobody receives from; -
receiving from a channel that is never sent to or closed; - a worker
blocked on a network call without timeout; - a producer that cannot stop
after its consumer exits; - a retry loop that ignores cancellation.

### Example of a leak-prone send

``` go
func load() <-chan int {
    out := make(chan int)

    go func() {
        out <- 42 // can block forever if caller stops receiving
        close(out)
    }()

    return out
}
```

### Make the send cancellable

``` go
func load(ctx context.Context) <-chan int {
    out := make(chan int, 1)

    go func() {
        defer close(out)
        select {
        case out <- 42:
        case <-ctx.Done():
        }
    }()

    return out
}
```

A small buffer can help in this specific example, but cancellation is
the more general way to make the goroutine stoppable. Buffering alone is
not a universal leak fix.

### Starvation and livelock

- **Starvation:** a goroutine repeatedly fails to get the resources or
  scheduling opportunities it needs.
- **Livelock:** goroutines keep reacting to each other but make no
  useful progress.
- **Contention:** many goroutines compete for the same lock or resource,
  reducing throughput.

### Blocked goroutine checklist

Ask: - What event unblocks this send/receive/lock? - Can the caller
return early? - Does cancellation reach this goroutine? - Who closes
this channel? - Is a network/database call bounded by a timeout? - Can
the queue grow without limit?

------------------------------------------------------------------------

## 19. Error Handling in Concurrent Code

Goroutines do not automatically return their errors to the caller.
Choose an explicit strategy.

### Result struct over a channel

``` go
type Result struct {
    Value string
    Err   error
}

results := make(chan Result, 1)

go func() {
    value, err := doWork()
    results <- Result{Value: value, Err: err}
}()

result := <-results
if result.Err != nil {
    return result.Err
}
use(result.Value)
```

For multiple workers, include task identifiers if results need to be
correlated.

### Fail-fast groups with `errgroup`

The `golang.org/x/sync/errgroup` package is commonly used to coordinate
related goroutines and return the first non-nil error.
`errgroup.WithContext` also provides a context canceled when a task
returns an error.

``` go
// Import golang.org/x/sync/errgroup
g, ctx := errgroup.WithContext(parentCtx)

g.Go(func() error {
    return fetchProfile(ctx)
})

g.Go(func() error {
    return fetchOrders(ctx)
})

if err := g.Wait(); err != nil {
    return err
}
```

The snippet assumes `fetchProfile` and `fetchOrders` are defined. Add an
explicit concurrency limit when launching a large number of tasks; do
not assume `errgroup` alone limits concurrency.

### Panic behavior

A panic in a goroutine is not automatically returned as an error to the
parent goroutine. An unrecovered panic normally terminates the process.
Recover only at deliberate boundaries where you can safely handle the
failure; do not use `recover` to hide programming bugs.

------------------------------------------------------------------------

## 20. Generics and Reusable Concurrency Helpers

Generics can make worker and pipeline helpers reusable across types.
Keep abstractions small; an overly generic concurrency framework can
make ownership and cancellation harder to understand.

### Example: map each input concurrently with a bounded worker count

``` go
package main

import (
    "context"
    "fmt"
    "sync"
)

func MapBounded[T any, R any](
    ctx context.Context,
    inputs []T,
    workers int,
    fn func(context.Context, T) (R, error),
) ([]R, error) {
    if workers < 1 {
        return nil, fmt.Errorf("workers must be at least 1")
    }

    type task struct {
        index int
        value T
    }
    type result struct {
        index int
        value R
        err   error
    }

    tasks := make(chan task)
    results := make(chan result)
    var wg sync.WaitGroup

    worker := func() {
        defer wg.Done()
        for t := range tasks {
            value, err := fn(ctx, t.value)
            select {
            case results <- result{index: t.index, value: value, err: err}:
            case <-ctx.Done():
                return
            }
        }
    }

    wg.Add(workers)
    for i := 0; i < workers; i++ {
        go worker()
    }

    go func() {
        defer close(tasks)
        for i, value := range inputs {
            select {
            case tasks <- task{index: i, value: value}:
            case <-ctx.Done():
                return
            }
        }
    }()

    go func() {
        wg.Wait()
        close(results)
    }()

    output := make([]R, len(inputs))
    var firstErr error
    received := 0

    for r := range results {
        received++
        if r.err != nil && firstErr == nil {
            firstErr = r.err
        }
        output[r.index] = r.value
    }

    if firstErr != nil {
        return nil, firstErr
    }
    if err := ctx.Err(); err != nil {
        return nil, err
    }
    if received != len(inputs) {
        return nil, fmt.Errorf("incomplete results: got %d of %d", received, len(inputs))
    }
    return output, nil
}

func main() {
    output, err := MapBounded(
        context.Background(),
        []int{1, 2, 3, 4},
        2,
        func(ctx context.Context, n int) (int, error) {
            return n * n, nil
        },
    )
    fmt.Println(output, err)
}
```

**Important design note:** This illustrative helper waits for other
tasks after one fails and does not automatically cancel sibling work. In
production, define fail-fast versus collect-all semantics explicitly and
test cancellation paths. `errgroup` may be a simpler choice for many
real applications.

------------------------------------------------------------------------

## 21. Go Scheduler: G, M, and P

Go’s runtime scheduler is often described using three concepts:

- **G — Goroutine:** the task to execute.
- **M — Machine:** an OS thread.
- **P — Processor:** runtime resources required for an M to execute Go
  code.

``` text
             Go scheduler
       +-----------------------+
       | Runnable Gs (goroutines)
       +-----------------------+
                |
                v
        P (execution resources)
                |
                v
             M (OS thread)
                |
                v
              CPU core
```

This is a simplified mental model; actual scheduling, queues, syscalls,
cgo, and runtime internals are more involved.

### Why this design?

- Many goroutines can be scheduled over fewer OS threads.
- Blocking operations can allow other goroutines to make progress.
- The runtime can distribute runnable work across processors.
- Goroutines make asynchronous task decomposition convenient without
  requiring a thread per task.

### Does a blocking syscall block the whole program?

Usually not. The runtime can allow other goroutines to run on other
available threads while a thread is blocked in a syscall. However,
external calls, cgo, CPU saturation, lock contention, and runtime
behavior can still affect throughput. Do not assume every kind of
blocking is free.

### When to learn scheduler internals

Learn the basic G/M/P model for interviews. For production tuning,
measure first with profiles and traces rather than trying to tune based
only on a mental model.

------------------------------------------------------------------------

## 22. Memory Model and Happens-Before

Concurrency correctness is not only about “which line ran first.” It is
also about whether writes in one goroutine are safely visible to
another.

Go’s memory model defines synchronization relationships. Important
examples include: - a channel send is synchronized with the
corresponding receive; - unlocking a mutex synchronizes before a later
successful lock on that mutex; - atomic operations have documented
ordering guarantees; - goroutine creation alone does not mean the parent
waits for the child to finish.

### Unsafe publication idea

If one goroutine writes ordinary shared data while another reads it
without synchronization, the program may have a data race. Do not rely
on timing or on the fact that a value “usually appears.”

### Synchronized example

``` go
var (
    mu    sync.Mutex
    ready bool
    value string
)

func producer() {
    mu.Lock()
    value = "ready"
    ready = true
    mu.Unlock()
}

func consumer() {
    mu.Lock()
    defer mu.Unlock()

    if ready {
        fmt.Println(value)
    }
}
```

Both accesses are protected by the same mutex, so the relationship
between `ready` and `value` is consistent.

### Channel publication

``` go
type Config struct {
    Endpoint string
}

func main() {
    ready := make(chan Config)

    go func() {
        cfg := Config{Endpoint: "https://api.example.com"}
        ready <- cfg
    }()

    cfg := <-ready
    fmt.Println(cfg.Endpoint)
}
```

The send/receive coordinates transfer of the value. If the value
contains pointers to mutable objects that are later changed by multiple
goroutines, those objects still need a clear ownership or
synchronization strategy.

**Rule:** Prefer clear ownership and documented synchronization over
assumptions about CPU caches or execution order.

------------------------------------------------------------------------

## 23. Performance: When Concurrency Helps

### Amdahl’s law, informally

If most of a task is inherently sequential, adding more workers cannot
speed up the entire task without limit. Parallel work also adds
scheduling, coordination, allocation, and communication overhead.

### Common bottlenecks

- Remote API rate limits or latency.
- Database connection pool exhaustion.
- Lock contention.
- Too many goroutines or queued jobs.
- Large buffered channels retaining memory.
- CPU saturation or excessive context switching.
- Garbage collection pressure from allocations.
- Head-of-line blocking caused by slow tasks.

### Measure, don’t guess

Useful measurements: - throughput (jobs/second); - p50, p95, and p99
latency; - error and timeout rates; - active goroutine count; - CPU and
memory; - queue depth and wait time; - lock contention and blocking
profiles.

### Benchmarking

``` go
func BenchmarkWork(b *testing.B) {
    for i := 0; i < b.N; i++ {
        doWork()
    }
}
```

Run benchmarks with:

``` bash
go test -bench=. -benchmem ./...
```

Use realistic inputs. Microbenchmarks can mislead when they omit
network, database, allocation, or contention costs.

------------------------------------------------------------------------

## 24. HTTP Servers and Real Production Scenarios

Go’s `net/http` server handles concurrent requests. In ordinary server
use, you typically do **not** need to start a new goroutine manually for
every request handler.

### Simple HTTP handler

``` go
package main

import (
    "encoding/json"
    "net/http"
    "time"
)

type Health struct {
    Status string `json:"status"`
    Time   string `json:"time"`
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    _ = json.NewEncoder(w).Encode(Health{
        Status: "ok",
        Time:   time.Now().UTC().Format(time.RFC3339),
    })
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/health", healthHandler)

    server := &http.Server{
        Addr:              ":8080",
        Handler:           mux,
        ReadHeaderTimeout: 5 * time.Second,
    }

    if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
        panic(err)
    }
}
```

A production server should also define graceful shutdown, request/body
limits where appropriate, observability, TLS or a trusted TLS
termination layer, and realistic timeouts.

### Scenario A: GoodSeva — find matching carriers

Suppose a shipper creates a load and the service checks many eligible
carriers.

A reasonable flow: 1. Validate the load. 2. Query candidate carriers
from the database. 3. Apply a bounded number of independent external
checks concurrently. 4. Use a timeout/cancellation context. 5. Collect
errors and results. 6. Sort/rank candidates deterministically. 7.
Persist the final result and publish an event if needed.

Do not launch unlimited goroutines for every carrier. Use a bounded
worker pool or `errgroup` with a limit, and respect third-party API
limits.

### Scenario B: GPS location ingestion

For incoming GPS positions: - accept and validate events quickly; - use
bounded queues or a message broker for buffering; - process updates with
explicit partitioning or ordering rules per device/vehicle if order
matters; - avoid unsynchronized shared maps; - batch database writes
only when safe for latency and durability; - monitor queue lag, dropped
messages, and database pool saturation.

A goroutine per event may be fine at low volume but should not be
assumed safe at unlimited scale. Capacity planning and backpressure are
essential.

### Scenario C: Database access

Concurrent goroutines may contend for a finite `database/sql` connection
pool. Increasing worker count beyond available connections can increase
queueing rather than throughput.

Use request contexts and database query timeouts. For business
invariants such as “only one carrier can accept this load,” use a
database transaction or conditional update with an appropriate
uniqueness/locking strategy. A Go mutex protects only one process; it
cannot coordinate multiple application instances.

### Scenario D: Graceful shutdown

Use `http.Server.Shutdown(ctx)` to stop accepting new connections and
wait for active requests up to a deadline. Also cancel background
workers, stop accepting jobs, and wait for them to exit. Shutdown is a
coordinated protocol, not simply a process sleep.

------------------------------------------------------------------------

## 25. Testing Concurrent Code

Concurrent bugs are often intermittent. Test the behavior and invariants
rather than assuming one output order.

### Race detector

``` bash
go test -race ./...
```

Use it during development and CI when practical. It has overhead and
only detects races exercised during that run.

### Avoid timing-based tests

Fragile:

``` go
go doWork()
time.Sleep(100 * time.Millisecond)
if !done {
    t.Fatal("not done")
}
```

Prefer explicit coordination using channels, wait groups, or testable
interfaces.

### Example test with a completion channel

``` go
func TestWorkerCompletes(t *testing.T) {
    done := make(chan struct{})

    go func() {
        doWork()
        close(done)
    }()

    select {
    case <-done:
    case <-time.After(time.Second):
        t.Fatal("worker did not complete in time")
    }
}
```

The timeout prevents the test from hanging forever; the completion
channel is what coordinates the test.

### Stress and repeated runs

``` bash
go test -race -count=50 ./...
```

Repeated runs can expose some intermittent issues. They do not guarantee
detection of all concurrency bugs.

### Test checklist

- Cancellation while a worker is blocked.
- Errors from one or several workers.
- Empty input and one-item input.
- Full queues and slow consumers.
- Shutdown while work is in progress.
- Concurrent access to maps and caches.
- Partial results and ordering requirements.
- Timeouts and retries.

------------------------------------------------------------------------

## 26. Debugging and Profiling Tools

### Runtime information

``` go
fmt.Println(runtime.NumGoroutine())
```

A growing goroutine count can be a warning sign, but a single number is
not enough to diagnose a leak.

### Common commands

``` bash
go test -race ./...
go test -bench=. -benchmem ./...
go test -run TestName -count=50 ./...
go tool pprof
```

### `net/http/pprof`

The `net/http/pprof` package can expose profiles such as CPU, heap,
goroutine, block, and mutex profiles when registered appropriately.
Protect profiling endpoints in production; they can expose sensitive
operational details and consume resources.

### Execution trace

Go’s execution tracer can help understand scheduling, blocking,
syscalls, and latency. A common workflow is:

``` bash
go test -trace=trace.out ./your/package
go tool trace trace.out
```

### What to inspect?

| Symptom                            | What to investigate                                                |
|------------------------------------|--------------------------------------------------------------------|
| CPU is high                        | CPU profile, hot loops, too much work, spinning `select`           |
| Latency rises under load           | queueing, DB pool, remote APIs, lock contention                    |
| Memory grows                       | heap profile, queue capacity, retained references, goroutine leaks |
| Goroutine count keeps growing      | goroutine profile and blocked stack traces                         |
| Throughput falls with more workers | contention, downstream limits, oversubscription, allocation costs  |

------------------------------------------------------------------------

## 27. Advanced Patterns and Newer Go Features

### 27.1 `sync.Once`

Run initialization once, even if many goroutines call it concurrently.

``` go
var (
    once sync.Once
    cfg  *Config
)

func GetConfig() *Config {
    once.Do(func() {
        cfg = loadConfig()
    })
    return cfg
}
```

This example assumes `loadConfig` succeeds or has a deliberate error
strategy. `sync.Once` is not a retry mechanism for failed
initialization; if initialization errors, decide how to store and expose
that error.

### 27.2 `sync.Map`

`sync.Map` is specialized for particular access patterns, such as
write-once/read-many entries or disjoint key sets. For ordinary
application maps, a normal `map` plus a mutex is often easier to reason
about.

### 27.3 `sync.Pool`

`sync.Pool` can reduce allocation pressure for temporary reusable
objects. It is a performance optimization, not a durable cache: values
may disappear at any time, including due to garbage collection. Never
depend on a pooled value being retained.

### 27.4 `singleflight`

The `golang.org/x/sync/singleflight` package suppresses duplicate
in-flight calls for the same key. It is useful when many requests ask
for the same expensive result at once. It does not replace a cache and
does not automatically persist results after a call completes.

### 27.5 Structured concurrency

Structured concurrency means related concurrent tasks have a clear
parent scope: the parent can wait for them, cancel them, and handle
their errors. In Go, this is often built using `context`, `errgroup`,
and explicit lifecycle management. It is a design principle rather than
one built-in keyword.

### 27.6 `WaitGroup.Go` version note

Newer Go releases provide additional concurrency helpers, and APIs vary
by Go version. Check the official documentation for the exact version
you target before using newer methods such as `WaitGroup.Go`. The
classic `Add` + goroutine + `Done` pattern remains broadly recognizable.

### 27.7 Avoid cleverness without a reason

Advanced primitives are not automatically better. Start with a simple
mutex, channel, or wait group; profile; then add specialized patterns
only when they solve a demonstrated problem.

------------------------------------------------------------------------

## 28. Common Mistakes Checklist

### Goroutines

- [ ] Every goroutine has a clear lifetime and exit condition.
- [ ] Errors are returned or recorded through an explicit mechanism.
- [ ] The caller does not assume goroutines finish before `main` or a
  handler returns.
- [ ] Cancellation can reach long-running work.

### Channels

- [ ] The sender/closer ownership is clear.
- [ ] No sender can send after close.
- [ ] Every blocking send/receive has a path to progress or
  cancellation.
- [ ] Buffer sizes are intentional and bounded.
- [ ] `select` does not accidentally spin in a busy loop.

### Shared state

- [ ] Shared maps are protected or ownership is isolated.
- [ ] Counters use a mutex or atomic where appropriate.
- [ ] Locks protect complete invariants, not just isolated lines.
- [ ] Locks are not copied after use.
- [ ] No reliance on timing instead of synchronization.

### Operations

- [ ] Network and database calls have suitable deadlines/timeouts.
- [ ] Worker counts respect downstream capacity.
- [ ] Queue overflow behavior is defined.
- [ ] Shutdown stops producers, drains or rejects work intentionally,
  and waits for workers.
- [ ] Race tests, profiling, and load tests are part of development.

------------------------------------------------------------------------

## 29. Java vs. Go Concurrency

| Concept                     | Java                                                    | Go                                             |
|-----------------------------|---------------------------------------------------------|------------------------------------------------|
| Lightweight concurrent task | Thread / virtual thread                                 | Goroutine                                      |
| Start concurrent work       | `Thread.start()`, executor, virtual thread              | `go function()`                                |
| Transfer values             | Queues, futures, shared objects                         | Channels are a core language/runtime pattern   |
| Mutual exclusion            | `synchronized`, `Lock`                                  | `sync.Mutex`, `sync.RWMutex`                   |
| Atomic values               | `AtomicInteger`, etc.                                   | `sync/atomic` typed values                     |
| Wait for tasks              | `Future`, `CountDownLatch`, executor APIs               | `sync.WaitGroup`, channels, `errgroup`         |
| Cancellation                | Interruption, future cancellation, API-specific support | `context.Context` and cooperative cancellation |
| Structured task grouping    | Structured concurrency APIs, version-dependent          | Often composed from context + `errgroup`       |
| Race detection              | Tooling varies; Java memory model defines guarantees    | Built-in race detector via `go test -race`     |
| CPU parallelism control     | Executor sizing / runtime and JVM settings              | `GOMAXPROCS` and worker design                 |

### Mental model shift from Java to Go

1.  Do not create a goroutine for every unit of work without thinking
    about bounds and cancellation.
2.  Use channels for communication and ownership transfer; use mutexes
    for protecting shared state.
3.  Use `context.Context` consistently for request cancellation and
    deadlines.
4.  Assume output order is nondeterministic unless you explicitly
    preserve it.
5.  Use the race detector as a routine development tool.
6.  Do not assume a process-local lock protects a distributed invariant
    across multiple service instances.

------------------------------------------------------------------------

## 30. Interview Questions and Short Answers

### Fundamentals

**1. What is a goroutine?**  
A lightweight concurrent task scheduled by the Go runtime.

**2. Concurrency vs. parallelism?**  
Concurrency structures multiple tasks so they can make progress;
parallelism executes work simultaneously on multiple execution
resources.

**3. Does `go f()` wait for `f` to finish?**  
No. It starts `f` in another goroutine and the caller continues.

**4. Does Go wait for goroutines when `main` returns?**  
No. The process exits when `main` returns.

**5. Can goroutines run in parallel?**  
Yes, when runtime and machine resources allow it. Concurrency does not
guarantee parallel execution.

### Channels

**6. What is an unbuffered channel?**  
A channel with no storage buffer; send and receive must rendezvous.

**7. What is a buffered channel?**  
A channel with capacity to hold values until received; sends block when
full.

**8. What happens when sending to a closed channel?**  
The send panics.

**9. What happens when receiving from a closed channel?**  
After buffered values are drained, receive returns the element type’s
zero value immediately. The two-value receive reports `ok == false`.

**10. Who should close a channel?**  
Usually the sender or coordinator that knows no more sends will occur.

**11. What does `select` do?**  
It waits for one of multiple channel operations; if multiple cases are
ready, one is chosen without guaranteed priority.

### Synchronization

**12. What is a data race?**  
Concurrent access to the same memory where at least one access writes
and there is no proper synchronization.

**13. Mutex vs. atomic?**  
Use a mutex for shared structures and multi-step invariants; atomics for
simple independent atomic state.

**14. Mutex vs. channel?**  
A mutex protects shared state; a channel communicates values/events or
transfers ownership. Choose based on the problem.

**15. Does `WaitGroup` make writes thread-safe?**  
No. It waits for completion; shared writes still need safe
synchronization.

**16. What is `RWMutex`?**  
A lock that permits multiple readers or one writer; it helps only for
appropriate workloads.

### Lifecycle and errors

**17. What is `context.Context` used for?**  
Cancellation, deadlines, and request-scoped metadata across call
boundaries.

**18. Does canceling a context kill a goroutine?**  
No. The goroutine must observe cancellation and exit, and called APIs
must cooperate.

**19. What is a goroutine leak?**  
A goroutine that remains blocked or running after it is no longer
useful.

**20. What is `errgroup`?**  
A helper for coordinating goroutines and collecting errors;
`WithContext` also propagates cancellation when a task fails.

### Performance and internals

**21. What are G, M, and P?**  
G is a goroutine, M an OS thread, and P runtime execution resources
needed to run Go code.

**22. Does more concurrency always improve performance?**  
No. Contention, queueing, downstream limits, and scheduling overhead can
make performance worse.

**23. How do you detect data races?**  
Run tests or programs with `-race`; the detector only finds races
exercised by that run.

**24. How do you investigate goroutine leaks?**  
Inspect goroutine profiles/stacks, identify blocked operations, and
verify every goroutine has a cancellation and exit path.

**25. How would you limit calls to an external API?**  
Use a bounded worker pool or semaphore for in-flight concurrency and a
rate limiter for requests per time unit; use context timeouts and
monitor failures.

**26. How do you ensure only one carrier accepts a load across multiple
service instances?**  
Use a database transaction or conditional update with a
constraint/locking strategy. A Go mutex only coordinates goroutines in
one process.

**27. Why might a buffered channel help?**  
It can decouple producer and consumer briefly and absorb a bounded
burst. It does not eliminate overload or guarantee higher throughput.

**28. Why is `time.Sleep` not synchronization?**  
It only delays the current goroutine; it does not prove another
goroutine completed or safely publish shared state.

### Practice exercises

1.  Write a goroutine that returns a value through a channel.
2.  Run 100 increments concurrently and fix the race with a mutex.
3.  Implement the same counter with `atomic.Int64`.
4.  Build a worker pool with three workers and 20 jobs.
5.  Add cancellation to a worker pool.
6.  Make a fan-in function that preserves result ordering.
7.  Write a test that detects a worker that fails to stop.
8.  Limit external calls to five concurrent operations and ten per
    second.
9.  Simulate a deadlock, explain why it happens, then fix it.
10. Run `go test -race ./...` and explain each reported race.

------------------------------------------------------------------------

## 31. Learning Path

### Stage 1 — Fundamentals

- `go` keyword and goroutines
- `WaitGroup`
- Channels and channel direction
- Buffered vs. unbuffered channels
- `select`

### Stage 2 — Safe shared state

- Data races
- `Mutex` and `RWMutex`
- Atomics
- Go memory model basics
- `go test -race`

### Stage 3 — Production lifecycle

- Context cancellation and deadlines
- Worker pools and backpressure
- Error handling with `errgroup`
- Graceful shutdown
- Timeouts for HTTP and database operations

### Stage 4 — Advanced engineering

- Pipelines and fan-in/fan-out
- Rate limiters and semaphores
- Profiling and execution traces
- Load testing and capacity planning
- Scheduler and memory-model internals

### Suggested mini-projects

1.  **Concurrent URL checker:** bounded workers, timeouts, result
    collection.
2.  **File processor:** stream jobs through channels and report errors.
3.  **GPS event processor:** bounded queue, device ordering rules,
    graceful shutdown.
4.  **Carrier matching service:** concurrent lookups with API limits and
    request cancellation.
5.  **Metrics service:** atomic counters, mutex-protected maps,
    profiling endpoint protected from public access.

------------------------------------------------------------------------

## 32. One-Minute Revision Cheat Sheet

| Need                                            | Use                                       |
|-------------------------------------------------|-------------------------------------------|
| Start concurrent function                       | `go fn()`                                 |
| Wait for several tasks                          | `sync.WaitGroup`                          |
| Pass values between goroutines                  | Channel                                   |
| Wait on multiple channel operations             | `select`                                  |
| Protect a map or multi-step invariant           | `sync.Mutex`                              |
| Many reads, few writes and measured benefit     | `sync.RWMutex`                            |
| Simple independent counter/flag                 | `sync/atomic`                             |
| Stop work / deadline                            | `context.Context`                         |
| Bound active jobs                               | Worker pool / semaphore                   |
| Bound requests per second                       | Rate limiter                              |
| Coordinate related tasks and errors             | `errgroup`                                |
| Detect data races                               | `go test -race ./...`                     |
| Investigate CPU, memory, blocking               | pprof and execution trace                 |
| Avoid unbounded memory growth                   | Bounded queues + explicit overload policy |
| Stop background goroutines                      | Cancellation + completion coordination    |
| Protect business state across service instances | Database transaction/conditional update   |

### The five rules to remember

1.  **Every goroutine needs a clear lifetime.**
2.  **Every blocking operation needs a valid path to progress or
    cancellation.**
3.  **Shared mutable state needs synchronization.**
4.  **Concurrency must be bounded when work or downstream capacity is
    finite.**
5.  **Measure performance and use the race detector; do not rely on
    timing-based assumptions.**

------------------------------------------------------------------------

## Final mental model

``` text
Goroutines = who does the work
Channels   = how values/signals move between tasks
Mutexes    = how shared memory is protected
WaitGroup  = how completion is coordinated
Context    = how cancellation/deadlines travel
Worker pool = how concurrency is bounded
Race detector + profiles = how correctness and performance are investigated
```

**Interview-ready summary:** Go makes concurrent programming
approachable with lightweight goroutines and channels, but it does not
remove the need to design ownership, synchronization, cancellation,
backpressure, error handling, and shutdown carefully.
