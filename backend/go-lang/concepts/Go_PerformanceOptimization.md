# Go Performance Optimization – Complete Interview Revision Guide

> Performance Optimization is a senior-level Go topic.
>
> Most developers can write code that works.
>
> Senior engineers write code that:
>
> - Scales
> - Uses less memory
> - Uses less CPU
> - Handles more traffic
> - Runs faster
>
> Common interview topics:
>
> - pprof
> - CPU Profiling
> - Memory Profiling
> - Goroutine Leaks
> - Reducing Allocations

---

# 1. What is Performance Optimization?

Performance Optimization means improving:

```text
Speed
Memory Usage
CPU Usage
Latency
Throughput
```

without changing functionality.

---

# Example

Bad:

```text
Response Time = 2 Seconds
```

---

Optimized:

```text
Response Time = 100 ms
```

---

# Why Important?

Benefits:

```text
Lower Costs
Better User Experience
Higher Throughput
Better Scalability
```

---

# Performance Bottlenecks

Common causes:

```text
Slow CPU
High Memory Usage
Too Many Allocations
Database Queries
Goroutine Leaks
Lock Contention
```

---

# Optimization Process

Always:

```text
Measure
Identify
Optimize
Measure Again
```

---

# Rule

Never optimize blindly.

---

# Quote

```text
Measure first,
optimize second.
```

---

# 2. pprof

Most Important Performance Tool.

---

# What is pprof?

Built-in Go profiling tool.

Used to analyze:

```text
CPU Usage
Memory Usage
Goroutines
Allocations
Blocking Operations
```

---

# Package

```go
import _ "net/http/pprof"
```

---

# Enable pprof

```go
go func() {

    http.ListenAndServe(
        "localhost:6060",
        nil,
    )

}()
```

---

Now available at:

```text
http://localhost:6060/debug/pprof/
```

---

# Why pprof?

Answers:

```text
What uses CPU?
What allocates memory?
What causes slowdowns?
```

---

# pprof Profiles

---

## CPU Profile

CPU usage.

---

## Heap Profile

Memory usage.

---

## Goroutine Profile

Active goroutines.

---

## Block Profile

Blocking operations.

---

## Mutex Profile

Lock contention.

---

# Most Common Command

```bash
go tool pprof
```

---

# Example

```bash
go tool pprof cpu.prof
```

---

# Interactive Mode

```bash
top
```

Shows hottest functions.

---

# Example Output

```text
50% main.processData
20% json.Marshal
10% strings.Split
```

---

# Interpretation

Most CPU time spent in:

```go
processData()
```

---

# Optimization target found.

---

# 3. CPU Profiling

Most Asked Performance Topic.

---

# What is CPU Profiling?

Measures where CPU time is spent.

---

# Goal

Find:

```text
Expensive Functions
Hot Spots
Slow Algorithms
```

---

# Example

```go
func process() {

    for i := 0; i < 1000000; i++ {
        heavyWork()
    }
}
```

---

CPU Profile may show:

```text
90% CPU
```

inside:

```go
heavyWork()
```

---

# Create CPU Profile

```go
f, _ := os.Create(
    "cpu.prof",
)

pprof.StartCPUProfile(f)

defer pprof.StopCPUProfile()
```

---

Run program.

---

Analyze:

```bash
go tool pprof cpu.prof
```

---

# Useful Commands

---

## Top Functions

```bash
top
```

---

## Graph

```bash
web
```

---

## List Function

```bash
list functionName
```

---

# What To Look For?

Functions consuming:

```text
High CPU
```

---

# Common CPU Problems

```text
Nested Loops
Expensive JSON
String Operations
Sorting Large Data
Regex
```

---

# Example Optimization

Bad:

```go
for {
    expensiveWork()
}
```

---

Improve algorithm.

Reduce work.

---

# CPU Profiling Summary

```text
Find Where CPU Time Goes
```

---

# 4. Memory Profiling

Most Important Backend Topic.

---

# What is Memory Profiling?

Analyzing memory usage.

---

# Goal

Find:

```text
Large Allocations
Memory Waste
Memory Leaks
GC Pressure
```

---

# Create Heap Profile

```go
f, _ := os.Create(
    "heap.prof",
)

pprof.WriteHeapProfile(f)
```

---

Analyze:

```bash
go tool pprof heap.prof
```

---

# Useful Commands

---

## Top

```bash
top
```

---

## Memory Consumers

Shows:

```text
Largest Allocations
```

---

# Example Output

```text
40% bytes.Buffer
30% JSON Marshal
20% Cache
```

---

# Interpretation

Most memory consumed by:

```go
bytes.Buffer
```

---

Optimization target found.

---

# Why Memory Matters?

More memory →

```text
More Garbage Collection
More Cost
Slower Performance
```

---

# Common Memory Problems

---

## Huge Slices

```go
make([]int, 100000000)
```

---

## Large Maps

```go
map[string]interface{}
```

---

## Unreleased References

Objects stay alive.

---

## Too Many Allocations

GC pressure increases.

---

# Memory Profiling Summary

```text
Find Where Memory Is Being Allocated
```

---

# 5. Goroutine Leaks

Extremely Important Interview Topic.

---

# What is a Goroutine Leak?

A goroutine that never exits.

---

# Example

```go
go func() {

    <-ch

}()
```

---

If:

```go
ch
```

never receives data,

goroutine waits forever.

---

# Problem

Thousands of leaked goroutines:

```text
Memory Usage Increases
CPU Usage Increases
Application Slows
```

---

# Visual

```text
Request
   ↓
Goroutine Created
   ↓
Never Stops
   ↓
Leak
```

---

# Common Causes

---

## Unread Channels

```go
<-ch
```

never receives.

---

## Unclosed Channels

Workers wait forever.

---

## Missing Context Cancellation

```go
ctx.Done()
```

not handled.

---

# Bad Example

```go
func worker() {

    for {

        job := <-jobs

        process(job)
    }
}
```

---

If:

```go
jobs
```

never closes,

worker lives forever.

---

# Better

```go
for job := range jobs {

}
```

---

# Use Context

```go
select {

case <-ctx.Done():
    return
}
```

---

# Detect Goroutine Leaks

pprof:

```bash
go tool pprof
```

---

Check:

```text
Goroutine Profile
```

---

# Runtime Count

```go
runtime.NumGoroutine()
```

---

Example

```go
fmt.Println(
    runtime.NumGoroutine(),
)
```

---

Unexpected growth:

```text
Leak Possible
```

---

# Goroutine Leak Summary

```text
Goroutine Exists Forever
Without Useful Work
```

---

# 6. Reducing Allocations

Most Important Optimization Technique.

---

# Why Reduce Allocations?

Every allocation:

```text
Consumes Memory
Creates GC Work
Costs CPU
```

---

Goal:

```text
Allocate Less
Reuse More
```

---

# Example

Bad:

```go
for i := 0; i < 1000; i++ {

    s := []int{}
}
```

---

Creates:

```text
1000 Allocations
```

---

# Optimization #1

Slice Preallocation

---

Bad:

```go
var nums []int
```

---

Good:

```go
nums :=
    make(
        []int,
        0,
        1000,
    )
```

---

Why?

Avoids repeated reallocations.

---

# Optimization #2

strings.Builder

---

Bad:

```go
s := ""

for i := 0; i < 1000; i++ {

    s += "Go"
}
```

---

Creates many allocations.

---

Good:

```go
var b strings.Builder

for i := 0; i < 1000; i++ {

    b.WriteString("Go")
}
```

---

Much faster.

---

# Optimization #3

Reuse Objects

---

Bad:

```go
for i := 0; i < 100000; i++ {

    obj := User{}
}
```

---

Creates huge allocations.

---

Good:

Reuse objects.

---

# Optimization #4

sync.Pool

Advanced Interview Topic.

---

# What is sync.Pool?

Reusable object pool.

---

Example

```go
var pool = sync.Pool{

    New: func() interface{} {

        return &bytes.Buffer{}
    },
}
```

---

Get Object

```go
buf :=
    pool.Get().
    (*bytes.Buffer)
```

---

Return Object

```go
pool.Put(buf)
```

---

Benefits:

```text
Less Allocation
Less GC
Better Performance
```

---

# Optimization #5

Avoid Interface Conversions

---

Bad:

```go
var x interface{} = user
```

May allocate.

---

Prefer concrete types when possible.

---

# Optimization #6

Escape Analysis

Important Interview Topic.

---

Check:

```bash
go build -gcflags="-m"
```

---

Output:

```text
escapes to heap
```

---

Goal:

```text
Keep More Objects On Stack
```

---

# Optimization Process

Senior Engineer Workflow.

---

Step 1

```text
Measure
```

---

Step 2

```text
Profile
```

---

Step 3

```text
Identify Bottleneck
```

---

Step 4

```text
Optimize
```

---

Step 5

```text
Measure Again
```

---

# Common Performance Problems

| Problem | Solution |
|----------|----------|
| High CPU | CPU Profiling |
| High Memory | Heap Profiling |
| Too Many Goroutines | Leak Detection |
| High GC | Reduce Allocations |
| Slow Strings | strings.Builder |
| Repeated Slice Growth | Preallocate Capacity |

---

# Most Asked Interview Questions

---

## What is pprof?

Go's built-in profiling tool.

---

## What Can pprof Analyze?

```text
CPU
Memory
Goroutines
Blocking
Mutexes
```

---

## How Do You Analyze Profiles?

```bash
go tool pprof
```

---

## What is CPU Profiling?

Finding where CPU time is spent.

---

## What is Memory Profiling?

Finding where memory allocations occur.

---

## What is a Goroutine Leak?

A goroutine that never exits.

---

## How Do You Detect Goroutine Leaks?

```go
runtime.NumGoroutine()
```

or:

```text
pprof goroutine profile
```

---

## Why Reduce Allocations?

Less GC and better performance.

---

## Common Allocation Optimizations?

- Slice Preallocation
- strings.Builder
- sync.Pool
- Object Reuse

---

# Quick Revision Cheat Sheet

## Enable pprof

```go
import _ "net/http/pprof"
```

---

## Start pprof Server

```go
http.ListenAndServe(
    ":6060",
    nil,
)
```

---

## Analyze Profile

```bash
go tool pprof
```

---

## CPU Profile

```go
pprof.StartCPUProfile()
```

---

## Heap Profile

```go
pprof.WriteHeapProfile()
```

---

## Goroutine Count

```go
runtime.NumGoroutine()
```

---

## Preallocate Slice

```go
make([]int, 0, 1000)
```

---

## String Builder

```go
strings.Builder
```

---

## Object Pool

```go
sync.Pool
```

---

## Escape Analysis

```bash
go build -gcflags="-m"
```

---

# Golden Interview Answer

### How Do You Optimize Performance In Go?

The first step is always profiling, not guessing.

Go provides `pprof` to analyze:

- CPU usage
- Memory allocations
- Goroutines
- Blocking operations

Common optimizations include:

- Reducing allocations
- Preallocating slices
- Using `strings.Builder`
- Reusing objects with `sync.Pool`
- Fixing goroutine leaks
- Improving algorithms

A senior Go engineer typically follows:

```text
Measure
→ Profile
→ Optimize
→ Measure Again
```

to make data-driven performance improvements.

---

# End of Performance Optimization Revision