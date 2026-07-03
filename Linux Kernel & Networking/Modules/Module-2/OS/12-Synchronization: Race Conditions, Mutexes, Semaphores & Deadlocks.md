Most developers think:

> "If two threads execute the same code, they'll produce the same result."

Unfortunately, that's not always true.

Consider this simple code:

```java
counter++;
```

Looks harmless.

Now imagine **100 threads** execute it at the same time.

Expected:

```
100
```

Actual:

```
83

91

97

100

...

(Random)
```

Why?

Because multiple threads are modifying the same memory simultaneously.

This problem is called a **Race Condition**.

Synchronization is one of the hardest topics in Computer Science.

Operating Systems, Databases, Java, Go, PostgreSQL, Docker, and even the Linux Kernel all rely on synchronization mechanisms.

After this chapter, you'll understand how concurrent programs safely share resources.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 12 — Synchronization: Race Conditions, Mutexes, Semaphores & Deadlocks

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Synchronization is needed
> - Shared Resources
> - Critical Section
> - Race Condition
> - Atomic Operations
> - Mutex
> - Spinlock
> - Read-Write Lock
> - Semaphore
> - Condition Variable
> - Monitor
> - Deadlock
> - Livelock
> - Starvation
> - Synchronization inside the Linux Kernel

---

# 📖 Why Do We Need Synchronization?

Imagine a bank.

Two employees update the same account balance.

Initial balance:

```
₹1000
```

Employee A deposits:

```
₹500
```

Employee B withdraws:

```
₹300
```

Correct balance:

```
₹1200
```

If both update simultaneously,

the balance might become:

```
₹700

or

₹1500
```

Both are incorrect.

The same problem occurs inside software.

---

# Shared Resource

A Shared Resource is anything accessed by multiple threads or processes.

Examples:

- Variables
- Files
- Database Records
- Memory
- Network Connections
- Printer
- Log Files

Whenever multiple threads access the same resource,

synchronization may be required.

---

# Example

```java
int counter = 0;
```

Thread A:

```java
counter++;
```

Thread B:

```java
counter++;
```

Both access:

```
Shared Memory
```

---

# What Actually Happens?

Most people think:

```java
counter++;
```

is one operation.

It isn't.

Internally it becomes:

```
Read counter

↓

Add 1

↓

Write counter
```

Three operations.

---

# Race Condition

Suppose:

```
counter = 5
```

Thread A:

```
Read 5
```

Thread B:

```
Read 5
```

Thread A:

```
Write 6
```

Thread B:

```
Write 6
```

Expected:

```
7
```

Actual:

```
6
```

This is called a:

```
Race Condition
```

The result depends on timing.

---

# Critical Section

A Critical Section is code that accesses shared resources.

Example:

```java
counter++;
```

Only one thread should execute this code at a time.

---

# Synchronization Goal

```
Thread A

↓

Critical Section

↓

Exit
```

Only after Thread A exits:

```
Thread B

↓

Critical Section
```

This guarantees correctness.

---

# Atomic Operation

An Atomic Operation is indivisible.

```
Start

↓

Complete
```

No other thread can observe it halfway through.

Example:

```
Atomic Increment
```

Modern CPUs provide atomic instructions.

---

# Mutex

A Mutex is the simplest synchronization mechanism.

Think of it as a room key.

Only one person can hold the key.

```
Thread A

↓

Lock

↓

Critical Section

↓

Unlock
```

Meanwhile:

```
Thread B

↓

Wait
```

---

# Mutex Example

```
Shared Counter

↓

Mutex

↓

Increment
```

Only one thread increments the counter at a time.

---

# Lock Flow

```
Acquire Lock

↓

Critical Section

↓

Release Lock
```

If the lock is busy,

other threads wait.

---

# Spinlock

A Spinlock is similar to a Mutex.

Difference:

Instead of sleeping,

the waiting thread repeatedly checks:

```
Is Lock Free?

No

↓

Check Again

↓

Check Again

↓

Check Again
```

This is called:

```
Spinning
```

---

# When Are Spinlocks Useful?

Suppose the lock will be released very quickly.

Sleeping would cost more than waiting.

Linux Kernel frequently uses Spinlocks for short operations.

---

# Mutex vs Spinlock

| Mutex | Spinlock |
|--------|-----------|
| Waiting Thread Sleeps | Waiting Thread Keeps Checking |
| Better for Long Waits | Better for Very Short Waits |
| Used in User Space | Common in Kernel Space |

---

# Read-Write Lock

Sometimes many threads only read data.

Example:

```
Database Cache
```

Many readers.

Few writers.

A Read-Write Lock allows:

```
Reader A

Reader B

Reader C
```

Simultaneously.

But:

```
Writer
```

Must execute alone.

---

# Read-Write Lock Rules

Allowed:

```
Many Readers
```

Allowed:

```
One Writer
```

Not Allowed:

```
Readers + Writer Together
```

---

# Semaphore

A Semaphore allows multiple threads to enter.

Suppose:

```
Semaphore = 3
```

Then:

```
Thread A

↓

Allowed

Thread B

↓

Allowed

Thread C

↓

Allowed

Thread D

↓

Wait
```

Semaphores control access to limited resources.

---

# Real Example

Suppose a printer room has:

```
3 Printers
```

Ten users want to print.

Semaphore:

```
3
```

Only three users print simultaneously.

Others wait.

---

# Binary Semaphore

A Binary Semaphore has only two values.

```
0

or

1
```

It behaves similarly to a Mutex,

but ownership rules differ.

---

# Condition Variable

Sometimes a thread waits until something happens.

Example:

Producer:

```
Add Item
```

Consumer:

```
Wait Until Item Exists
```

Condition Variables allow threads to sleep until a condition becomes true.

---

# Producer-Consumer Example

Producer:

```
Produce Data

↓

Notify
```

Consumer:

```
Wait

↓

Wake Up

↓

Consume Data
```

This avoids unnecessary CPU usage.

---

# Monitor

A Monitor combines:

- Shared Data
- Mutex
- Condition Variables

Languages like Java provide monitors automatically using:

```java
synchronized
```

---

# Deadlock

Imagine:

Thread A:

```
Lock A

↓

Needs Lock B
```

Thread B:

```
Lock B

↓

Needs Lock A
```

Neither proceeds.

```
Thread A

↓

Waiting

↑

Thread B
```

This is a:

```
Deadlock
```

---

# Four Conditions for Deadlock

Deadlock occurs when all four conditions exist:

1.

```
Mutual Exclusion
```

2.

```
Hold and Wait
```

3.

```
No Preemption
```

4.

```
Circular Wait
```

Break any one,

and deadlock cannot occur.

---

# Deadlock Prevention

Common techniques:

✔ Lock resources in the same order

✔ Avoid nested locks

✔ Use timeouts

✔ Keep critical sections short

---

# Livelock

Suppose two people meet in a hallway.

Person A:

```
Moves Left
```

Person B:

```
Moves Right
```

Then both move again.

Neither passes.

Both remain active,

but no progress occurs.

This is:

```
Livelock
```

---

# Starvation

Suppose:

```
High Priority Threads
```

always obtain the lock.

One thread:

```
Waits Forever
```

It never gets access.

This is:

```
Starvation
```

---

# Synchronization in Linux Kernel

The Linux Kernel uses many synchronization mechanisms.

Examples:

- Spinlocks
- Mutexes
- Read-Write Locks
- RCU (Read-Copy Update)
- Atomic Variables
- Seqlocks
- Completions

Different situations require different tools.

---

# Real Example

Suppose PostgreSQL updates a row.

```
Client A

↓

Lock Row

↓

Update
```

Meanwhile:

```
Client B

↓

Wait
```

Without synchronization,

database corruption could occur.

---

# Why Synchronization Matters

Synchronization is everywhere.

Examples:

- Java Multithreading
- Go Goroutines
- PostgreSQL
- Redis
- Linux Kernel
- Docker Engine
- Kubernetes Controllers

Correct synchronization ensures:

✔ Correctness

✔ Data Integrity

✔ Thread Safety

---

# Hands-on Lab

## Lab 1 — Observe Race Condition (Java)

```java
int counter = 0;
```

Start multiple threads incrementing the counter without synchronization.

Observe inconsistent results.

---

## Lab 2 — Fix Using synchronized

Protect the increment using:

```java
synchronized
```

Observe consistent results.

---

## Lab 3 — View Kernel Locks

Linux:

```bash
cat /proc/locks
```

Observe active file locks.

---

## Lab 4 — Observe Thread Synchronization

Run:

```bash
top -H
```

Observe multiple threads executing within the same process.

---

# Interview Questions

## What is a Race Condition?

A Race Condition occurs when multiple threads access shared data concurrently and the final result depends on execution timing.

---

## What is a Critical Section?

A Critical Section is a portion of code that accesses shared resources and must be executed by only one thread at a time.

---

## What is a Mutex?

A Mutex is a synchronization primitive that allows only one thread to access a critical section simultaneously.

---

## What is a Spinlock?

A Spinlock is a lock where waiting threads repeatedly check whether the lock has become available instead of sleeping.

---

## When is a Read-Write Lock useful?

When many threads read shared data but only a few modify it.

---

## What is a Semaphore?

A Semaphore controls access to a limited number of shared resources using a counter.

---

## What is a Deadlock?

A Deadlock occurs when threads wait indefinitely for resources held by each other.

---

## What is Starvation?

Starvation occurs when a thread never receives the resources it needs because other threads continuously obtain them first.

---

# Summary

Synchronization allows multiple threads to safely share resources.

```
Multiple Threads

↓

Shared Resource

↓

Synchronization Primitive

↓

Correct Execution
```

Key concepts:

- Shared resources require synchronization.
- Race Conditions occur due to unsynchronized access.
- Critical Sections protect shared data.
- Atomic Operations execute as indivisible units.
- Mutexes allow one thread at a time.
- Spinlocks are efficient for very short waits.
- Read-Write Locks allow multiple readers or one writer.
- Semaphores control access to limited resources.
- Condition Variables allow efficient waiting.
- Deadlocks, Livelocks, and Starvation are common concurrency problems.
- The Linux Kernel uses specialized synchronization mechanisms to safely manage shared data.

Understanding synchronization is essential before learning file systems, Linux kernel internals, databases, Docker, Kubernetes, and high-performance distributed systems.

---

# Next Chapter

## Chapter 13 — Inter-Process Communication (IPC): How Processes Talk to Each Other

We'll explore:

- Why IPC is needed
- Pipes
- Named Pipes (FIFO)
- Message Queues
- Shared Memory
- Semaphores in IPC
- Signals
- UNIX Domain Sockets
- Memory-Mapped Files
- Real-world IPC in Linux
- IPC used by Docker and Kubernetes