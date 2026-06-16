# Go Concurrency – Complete Interview Revision Guide

> Concurrency is one of Go's biggest strengths and one of the most important topics for backend interviews.
>
> Almost every Go backend application uses:
>
> - Goroutines
> - Channels
> - WaitGroups
> - Mutexes
> - RWMutexes
> - Atomic Operations
> - Select Statements
>
> If you understand this chapter well, you're already ahead of many Go developers.

---

# 1. What is Concurrency?

Concurrency means multiple tasks can make progress at the same time.

Example:

Instead of:

```text
Task A
↓
Task B
↓
Task C
```

Go can do:

```text
Task A
Task B
Task C
```

concurrently.

---

# Concurrency vs Parallelism

Very Common Interview Question.

---

## Concurrency

Multiple tasks are managed together.

```text
A
B
A
C
B
```

Tasks take turns.

---

## Parallelism

Multiple tasks actually run simultaneously on different CPU cores.

```text
CPU 1 → Task A
CPU 2 → Task B
CPU 3 → Task C
```

---

# Simple Interview Answer

```text
Concurrency = dealing with many tasks

Parallelism = executing many tasks simultaneously
```

---

# 2. Goroutines

Most Important Go Feature.

---

# What is a Goroutine?

A goroutine is a lightweight thread managed by Go.

---

# Normal Function

```go
sendEmail()
```

Runs synchronously.

---

# Goroutine

```go
go sendEmail()
```

Runs concurrently.

---

# Example

```go
package main

import (
    "fmt"
    "time"
)

func sendEmail() {
    fmt.Println("Sending Email...")
}

func main() {

    go sendEmail()

    time.Sleep(time.Second)
}
```

Output:

```
Sending Email...
```

---

# What Does `go` Keyword Do?

```go
go sendEmail()
```

Tells Go:

```text
Run this function in a separate goroutine.
```

---

# Multiple Goroutines

```go
go sendEmail()
go generateReport()
go processPayment()
```

All execute concurrently.

---

# Why Goroutines Are Powerful

Traditional OS threads:

```text
Expensive
Heavy
Memory intensive
```

---

Goroutines:

```text
Very lightweight
Cheap to create
Thousands can run
```

---

# Example

```go
for i := 0; i < 100000; i++ {
    go worker()
}
```

Possible in Go.

---

# Goroutine Problem

Main function may exit before goroutines finish.

---

Example:

```go
go sendEmail()

fmt.Println("Done")
```

Output:

```text
Done
```

Email may never execute.

---

# Solution

Use:

```go
WaitGroup
```

---

# 3. WaitGroup

Used to wait for goroutines to finish.

---

# Why Needed?

Without WaitGroup:

```text
Main exits
↓
Program stops
↓
Goroutines die
```

---

# WaitGroup Declaration

```go
var wg sync.WaitGroup
```

---

# Important Methods

```go
wg.Add()
wg.Done()
wg.Wait()
```

---

# Example

```go
package main

import (
    "fmt"
    "sync"
)

func worker(wg *sync.WaitGroup) {

    defer wg.Done()

    fmt.Println("Working")
}

func main() {

    var wg sync.WaitGroup

    wg.Add(1)

    go worker(&wg)

    wg.Wait()

    fmt.Println("Finished")
}
```

Output:

```text
Working
Finished
```

---

# How It Works

```go
wg.Add(1)
```

Add one task.

---

```go
wg.Done()
```

Task completed.

---

```go
wg.Wait()
```

Wait until counter becomes zero.

---

# Visual

```text
Add(3)

Counter = 3

Done()
Counter = 2

Done()
Counter = 1

Done()
Counter = 0

Wait() Unblocks
```

---

# 4. Race Condition

Before learning Mutex, understand race conditions.

---

# Example

```go
counter := 0

go increment()
go increment()
```

Both goroutines update same variable.

---

Possible output:

```text
98
99
100
```

Unpredictable.

---

# Why?

Multiple goroutines access same memory simultaneously.

---

# This Is Called

```text
Race Condition
```

---

# 5. Mutex

Most Asked Interview Topic.

---

# What is Mutex?

Mutex = Mutual Exclusion.

Allows:

```text
Only one goroutine
at a time
```

to access critical section.

---

# Mutex Declaration

```go
var mutex sync.Mutex
```

---

# Lock

```go
mutex.Lock()
```

---

# Unlock

```go
mutex.Unlock()
```

---

# Example

```go
var (
    counter int
    mutex sync.Mutex
)

func increment() {

    mutex.Lock()

    counter++

    mutex.Unlock()
}
```

Now safe.

---

# Why?

Visual:

Without Mutex:

```text
G1 → counter++
G2 → counter++
```

Collision.

---

With Mutex:

```text
G1 → Lock
G1 → Update
G1 → Unlock

G2 → Lock
G2 → Update
G2 → Unlock
```

Safe.

---

# Best Practice

Use defer.

```go
mutex.Lock()
defer mutex.Unlock()
```

---

# 6. RWMutex

Read-Write Mutex.

---

# Problem With Mutex

Even readers wait.

Example:

```text
Read
Read
Read
```

All blocked one by one.

---

# Solution

```go
sync.RWMutex
```

---

# Rules

```text
Multiple Readers Allowed

Single Writer Allowed
```

---

# Declaration

```go
var rw sync.RWMutex
```

---

# Read Lock

```go
rw.RLock()
```

---

# Read Unlock

```go
rw.RUnlock()
```

---

# Write Lock

```go
rw.Lock()
```

---

# Write Unlock

```go
rw.Unlock()
```

---

# Example

```go
rw.RLock()

fmt.Println(data)

rw.RUnlock()
```

Multiple readers can execute together.

---

# Visual

```text
Reader 1 ✔
Reader 2 ✔
Reader 3 ✔

Writer ✖ waits
```

---

# Interview Question

When should RWMutex be used?

Answer:

```text
Read-heavy workloads
```

Example:

```text
Cache
Configuration
In-memory lookup tables
```

---

# 7. Atomic Operations

Very Important Backend Topic.

---

# Problem

Mutex has locking overhead.

For simple counters:

```go
counter++
```

Mutex may be overkill.

---

# Solution

Use Atomic Operations.

---

# Package

```go
sync/atomic
```

---

# Example

```go
atomic.AddInt64(&counter, 1)
```

---

# Why?

Atomic means:

```text
Operation happens
completely or not at all.
```

No race condition.

---

# Example

```go
var counter int64

atomic.AddInt64(&counter, 1)
```

---

# Read Value

```go
atomic.LoadInt64(&counter)
```

---

# Benefits

```text
Very Fast
Thread Safe
No Locks
```

---

# Use Cases

```text
Counters
Metrics
Request Counts
Statistics
```

---

# Mutex vs Atomic

| Feature | Mutex | Atomic |
|----------|---------|---------|
| Complex Logic | Yes | No |
| Counter | Yes | Yes |
| Performance | Lower | Higher |
| Lock Needed | Yes | No |

---

# 8. Channels

Most Important Go Concurrency Feature.

---

# What is a Channel?

Channels allow goroutines to communicate safely.

---

# Create Channel

```go
ch := make(chan int)
```

---

# Send Data

```go
ch <- 10
```

---

# Receive Data

```go
value := <-ch
```

---

# Example

```go
func main() {

    ch := make(chan int)

    go func() {
        ch <- 100
    }()

    value := <-ch

    fmt.Println(value)
}
```

Output:

```
100
```

---

# Visual

```text
Sender
   ↓
Channel
   ↓
Receiver
```

---

# Why Channels?

Instead of:

```text
Shared Memory
```

Go prefers:

```text
Share Memory By Communicating
```

---

# Unbuffered Channels

Default channel.

---

## Example

```go
ch := make(chan int)
```

---

# Behavior

Sender waits until receiver is ready.

Receiver waits until sender is ready.

---

# Visual

```text
Sender
   ↔
Receiver
```

Both synchronize.

---

# Example

```go
ch := make(chan int)

go func() {
    ch <- 10
}()

fmt.Println(<-ch)
```

Output:

```
10
```

---

# Buffered Channels

Can hold values before receiver consumes them.

---

## Example

```go
ch := make(chan int, 3)
```

Capacity:

```text
3 values
```

---

# Send

```go
ch <- 1
ch <- 2
ch <- 3
```

All succeed immediately.

---

# Visual

```text
[1][2][3]
```

Stored inside channel.

---

# Buffered vs Unbuffered

| Feature | Unbuffered | Buffered |
|----------|------------|-----------|
| Capacity | 0 | >0 |
| Synchronization | Immediate | Delayed |
| Sender Waits | Yes | Only when full |

---

# Close Channel

Used to indicate:

```text
No more values will be sent.
```

---

## Example

```go
close(ch)
```

---

# Example

```go
ch := make(chan int)

close(ch)
```

---

# Reading Closed Channel

```go
value, ok := <-ch
```

Output:

```go
0 false
```

---

# Why Use Close?

Useful for:

```text
Workers
Pipelines
Streams
```

---

# Range Over Channel

```go
for value := range ch {
    fmt.Println(value)
}
```

Stops automatically when:

```go
close(ch)
```

is called.

---

# Directional Channels

Advanced Interview Topic.

---

# Send-Only Channel

```go
func producer(ch chan<- int)
```

Meaning:

```text
Can only send
```

---

# Receive-Only Channel

```go
func consumer(ch <-chan int)
```

Meaning:

```text
Can only receive
```

---

# Example

```go
func producer(ch chan<- int) {
    ch <- 10
}
```

---

```go
func consumer(ch <-chan int) {
    fmt.Println(<-ch)
}
```

---

# Why Use Directional Channels?

Provides compile-time safety.

---

# Select Statement

Most Asked Channel Interview Question.

---

# Problem

Need to wait on multiple channels.

---

# Solution

Use:

```go
select
```

---

# Example

```go
select {

case msg := <-ch1:
    fmt.Println(msg)

case msg := <-ch2:
    fmt.Println(msg)
}
```

---

# How It Works

Whichever channel is ready first gets executed.

---

# Example

```go
select {

case value := <-emailChannel:
    fmt.Println(value)

case value := <-smsChannel:
    fmt.Println(value)
}
```

---

# Default Case

```go
select {

case msg := <-ch:
    fmt.Println(msg)

default:
    fmt.Println("No Data")
}
```

Output:

```
No Data
```

if channel not ready.

---

# Timeout Pattern

Very Common Backend Example.

---

```go
select {

case result := <-ch:
    fmt.Println(result)

case <-time.After(2 * time.Second):
    fmt.Println("Timeout")
}
```

---

# Why Useful?

Prevent goroutines from waiting forever.

---

# Most Asked Interview Questions

---

## What is a Goroutine?

A lightweight thread managed by Go runtime.

---

## Why Use WaitGroup?

To wait for goroutines to finish.

---

## What is a Race Condition?

Multiple goroutines accessing shared data causing unpredictable results.

---

## What is a Mutex?

Allows only one goroutine at a time to access critical section.

---

## What is RWMutex?

Allows:

```text
Multiple Readers
Single Writer
```

---

## What are Atomic Operations?

Thread-safe operations without locks.

Example:

```go
atomic.AddInt64()
```

---

## What is a Channel?

A communication mechanism between goroutines.

---

## Difference Between Buffered and Unbuffered Channels?

Unbuffered:

```text
Sender waits for receiver.
```

Buffered:

```text
Can store values before receiver reads.
```

---

## Why Close a Channel?

To signal no more data will be sent.

---

## What is a Directional Channel?

Send-only:

```go
chan<- int
```

Receive-only:

```go
<-chan int
```

---

## What is Select?

Allows waiting on multiple channels simultaneously.

---

# Quick Revision Cheat Sheet

## Goroutine

```go
go sendEmail()
```

---

## WaitGroup

```go
wg.Add(1)
wg.Done()
wg.Wait()
```

---

## Mutex

```go
mutex.Lock()
mutex.Unlock()
```

---

## RWMutex

```go
rw.RLock()
rw.RUnlock()

rw.Lock()
rw.Unlock()
```

---

## Atomic

```go
atomic.AddInt64(&counter, 1)
```

---

## Create Channel

```go
ch := make(chan int)
```

---

## Send

```go
ch <- 10
```

---

## Receive

```go
value := <-ch
```

---

## Buffered Channel

```go
ch := make(chan int, 5)
```

---

## Close Channel

```go
close(ch)
```

---

## Send Only

```go
chan<- int
```

---

## Receive Only

```go
<-chan int
```

---

## Select

```go
select {
case <-ch1:
case <-ch2:
}
```

---

# Golden Interview Answer

### Why Is Concurrency So Powerful In Go?

Go provides first-class concurrency support through:

- Goroutines
- Channels
- WaitGroups
- Mutexes
- Atomic Operations
- Select Statements

This allows developers to build highly scalable backend systems that efficiently handle thousands of concurrent requests while keeping code simple and maintainable.

---

# End of Go Concurrency Revision