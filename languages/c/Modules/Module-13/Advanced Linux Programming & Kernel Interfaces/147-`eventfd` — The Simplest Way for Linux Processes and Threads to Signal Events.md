# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 147 — `eventfd` — The Simplest Way for Linux Processes and Threads to Signal Events

---

# Most developers know about:

```
Mutex

↓

Condition Variable

↓

Semaphore

↓

Pipe
```

But very few know about:

```
eventfd()
```

Even though it is used internally by:

- Docker
- QEMU
- KVM
- High-performance web servers
- Event loops
- Databases

If you have ever wondered:

> "How do two Linux threads notify each other without using a pipe?"

The answer is often:

```
eventfd()
```

---

# Learning Objectives

After this chapter you will understand:

- What `eventfd()` is
- Counter-based notifications
- Thread signaling
- Process signaling
- Event-driven synchronization
- Integration with `epoll`
- Why `eventfd` is faster than pipes for notifications

---

# The Problem

Suppose Thread A finishes work.

It needs to tell Thread B:

```
Job Completed
```

How?

One option:

```
Pipe
```

Thread A writes:

```
"done"
```

Thread B reads:

```
"done"
```

Works.

But...

We don't actually care about the message.

We only care that:

```
Something Happened
```

Using a pipe for this is unnecessary overhead.

---

# Linux Solution

Linux created:

```
eventfd()
```

Instead of sending bytes,

it maintains:

```
One 64-bit Counter
```

---

# What is `eventfd()`?

Definition:

> **`eventfd()` creates a kernel-managed 64-bit counter that processes and threads can use to notify one another.**

Think of it as:

```
Doorbell

↓

Counter
```

---

# Visualization

```
Thread A

↓

Increment Counter

↓

Kernel

↓

Thread B

↓

Read Counter
```

---

# Why a Counter?

Imagine:

```
Task Finished

↓

+1
```

Another task:

```
Finished

↓

+1
```

Counter becomes:

```
2
```

No messages required.

---

# Creating an `eventfd`

Program calls:

```c
eventfd(...)
```

Kernel returns:

```
File Descriptor
```

Again,

everything in Linux becomes:

```
FD
```

---

# Internal Structure

Kernel stores:

```
64-bit Counter
```

Initially:

```
0
```

---

# Writing

Suppose:

```
Counter

↓

0
```

Thread A writes:

```
1
```

Counter becomes:

```
1
```

---

# Another Write

Counter:

```
1
```

Write:

```
5
```

Counter becomes:

```
6
```

Simple addition.

---

# Reading

Thread B reads.

Receives:

```
6
```

Counter resets:

```
0
```

(Default behavior.)

---

# Visualization

```
Counter

↓

6

↓

Read

↓

Application Gets 6

↓

Counter = 0
```

---

# Blocking Behavior

Suppose:

```
Counter

↓

0
```

Thread B reads.

Nothing available.

It sleeps.

Later:

Thread A writes:

```
1
```

Kernel wakes Thread B.

---

# Event Notification

```
Thread A

↓

eventfd Write

↓

Kernel

↓

Wake Thread B
```

Exactly what we wanted.

---

# Why Not a Pipe?

Pipe:

```
Allocate Buffer

↓

Copy Bytes

↓

Read Bytes
```

---

# `eventfd`

```
Increment Counter

↓

Wake Waiter
```

Much simpler.

Often more efficient for pure notifications.

---

# Relationship with `epoll`

`eventfd` integrates naturally with:

```
epoll
```

Event loop:

```
Socket

↓

Timer

↓

eventfd

↓

inotify

↓

epoll_wait()
```

One thread can wait for all event sources together.

---

# Multi-thread Example

Worker Threads:

```
Worker 1

↓

eventfd

↑

Worker 2

↑

Worker 3
```

Main thread:

```
epoll_wait()

↓

Wake

↓

Process Jobs
```

---

# Thread Pool Example

Workers finish tasks.

Instead of:

```
Mutex

↓

Condition Variable
```

Workers can notify the dispatcher using:

```
eventfd
```

---

# Process Communication

`eventfd` is not limited to threads.

Processes can share an `eventfd` descriptor (for example, through inheritance or descriptor passing) and use it for notifications.

---

# Relationship with `fork()`

After:

```
fork()
```

Both parent and child may hold references to the same `eventfd`.

Notifications work across processes.

---

# Semaphore Mode

`eventfd` has an optional semaphore mode.

Normal mode:

```
Counter = 5

↓

Read

↓

Return 5

↓

Counter = 0
```

Semaphore mode:

```
Counter = 5

↓

Read

↓

Return 1

↓

Counter = 4
```

Useful when multiple consumers should acquire events one at a time.

---

# Relationship with Pipes

Pipe transfers:

```
Actual Data
```

`eventfd` transfers:

```
Event Count
```

Different purposes.

---

# Relationship with Condition Variables

Condition variables require:

- Mutex
- Shared memory
- Synchronization logic

`eventfd` provides a file-descriptor-based notification mechanism,

which integrates naturally with Linux I/O multiplexing APIs.

---

# Relationship with `io_uring`

Modern Linux uses:

```
eventfd

↓

Completion Notification
```

in many asynchronous I/O scenarios.

---

# Real-World Example — QEMU

Virtual machines generate events.

`eventfd` efficiently notifies waiting threads.

---

# Real-World Example — KVM

Kernel Virtual Machine uses `eventfd` extensively for communication between virtual devices and userspace.

---

# Real-World Example — High-Performance Servers

Worker threads:

```
Finish Request

↓

eventfd

↓

Main Event Loop
```

Avoiding unnecessary pipe traffic.

---

# Real-World Example — Databases

Database background workers often use lightweight notification mechanisms when coordinating internal tasks.

---

# Common Mistakes

---

## Thinking `eventfd` Transfers Messages

It transfers a counter,

not arbitrary data.

---

## Using `eventfd` for Large Data

It is designed for notifications,

not data transfer.

---

## Confusing It with Pipes

Pipes carry byte streams.

`eventfd` signals events.

---

## Forgetting It Is a File Descriptor

You can use it with:

- `read()`
- `write()`
- `poll()`
- `select()`
- `epoll()`

like many other Linux kernel objects.

---

# Hands-on Labs

## Lab 1

Read:

```bash
man 2 eventfd
```

Understand its API.

---

## Lab 2

Write two threads.

Thread A increments an `eventfd`.

Thread B waits and reads notifications.

---

## Lab 3

Integrate an `eventfd` into an `epoll` loop.

Observe event-driven wakeups.

---

## Lab 4

Experiment with semaphore mode.

Observe the difference in read behavior.

---

## Lab 5

Compare a notification implemented with:

- Pipe
- Condition variable
- `eventfd`

Measure conceptual complexity and API differences.

---

# Interview Questions

### What is `eventfd()`?

A Linux system call that creates a kernel-managed 64-bit event counter for process and thread notification.

---

### Why is `eventfd` useful?

It provides efficient event notification without sending arbitrary message data.

---

### How is `eventfd` different from a pipe?

A pipe transfers byte streams.

An `eventfd` maintains an event counter.

---

### Can `eventfd` be used with `epoll`?

Yes.

It is specifically designed to integrate well with Linux event loops.

---

### What happens after reading an `eventfd` in normal mode?

The current counter value is returned and the counter is reset to zero.

---

# Summary

Architecture:

```
Thread A

↓

eventfd Write

↓

Kernel Counter

↓

Thread B

↓

eventfd Read
```

Counter Flow:

```
Counter = 0

↓

Write 3

↓

Counter = 3

↓

Read

↓

Return 3

↓

Counter = 0
```

Comparison:

| Mechanism | Purpose |
|-----------|---------|
| Pipe | Transfer byte streams |
| Semaphore | Resource counting |
| Condition Variable | Thread synchronization |
| `eventfd` | Event notification via counter |

## Key Takeaways

- `eventfd` is a lightweight Linux event notification mechanism.
- It uses a kernel-managed 64-bit counter instead of byte streams.
- It integrates naturally with `epoll` and event-driven architectures.
- It is useful for both thread and process synchronization.
- Many virtualization, networking, and high-performance systems rely on `eventfd`.
- Understanding `eventfd` helps explain modern Linux asynchronous programming.

---

# Next Chapter

## Chapter 148 — `timerfd` — Turning Timers into File Descriptors

You'll learn:

- What `timerfd` is
- High-resolution timers
- Periodic timers
- One-shot timers
- Timer expiration counters
- Integration with `epoll`
- Why modern event loops use `timerfd` instead of signal-based timers