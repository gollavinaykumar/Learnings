# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 128 — `select()` vs `poll()` vs `epoll()` vs `kqueue()` vs `io_uring`

---

# Most beginners think:

> "There is only one way to wait for sockets."

Actually, Linux and Unix have evolved through **five major generations** of I/O event APIs.

```
select()

↓

poll()

↓

epoll()

↓

kqueue()   (BSD/macOS)

↓

io_uring()
```

Understanding **why each one was invented** is more important than memorizing the APIs.

---

# Learning Objectives

After this chapter you will understand:

- Evolution of I/O APIs
- Why `select()` became obsolete
- `poll()`
- `epoll()`
- `kqueue()`
- `io_uring()`
- Performance comparison
- Which software uses which API

---

# Evolution Timeline

```
1980s

↓

select()

↓

1990s

↓

poll()

↓

2002

↓

epoll()

↓

BSD

↓

kqueue()

↓

2019

↓

io_uring()
```

Every generation solved problems in the previous one.

---

# Generation 1 — `select()`

Prototype:

```c
select();
```

Idea:

```
Tell Linux

↓

Watch

↓

These Sockets
```

Linux scans every descriptor.

---

# Visualization

```
Socket 1

↓

Socket 2

↓

Socket 3

↓

...

↓

Socket N
```

Every call.

Even if only:

```
One Socket

↓

Ready
```

---

# Problems with `select()`

### 1. Linear Scan

```
100,000

↓

Scan

↓

Every Time
```

---

### 2. Descriptor Limit

Traditionally:

```
FD_SETSIZE

↓

1024
```

Many systems limit the default `fd_set` size.

---

### 3. Copy Overhead

Every call copies descriptor sets between:

```
User Space

↓

Kernel

↓

User Space
```

---

# Generation 2 — `poll()`

Prototype:

```c
poll();
```

Improved API.

No fixed descriptor limit like `select()`.

Still:

```
Linear Scan
```

---

# Visualization

```
Socket List

↓

Kernel

↓

Scan Entire List
```

Still inefficient for huge numbers of descriptors.

---

# Generation 3 — `epoll()`

Linux developers changed the approach.

Instead of:

```
Scan Everything
```

Linux stores:

```
Ready Events
```

Applications receive only:

```
Ready Descriptors
```

---

# Visualization

Without epoll:

```
100,000 Sockets

↓

Scan All
```

With epoll:

```
100,000 Sockets

↓

Kernel Tracks

↓

Return Ready Ones
```

Much faster.

---

# Complexity

Conceptually:

`select()`:

```
Every Call

↓

Scan N
```

`poll()`:

```
Every Call

↓

Scan N
```

`epoll()`:

```
Wait

↓

Ready Events
```

The kernel maintains readiness information internally.

---

# Generation 4 — `kqueue()`

Operating Systems:

```
FreeBSD

OpenBSD

NetBSD

macOS
```

Linux has:

```
epoll
```

BSD family has:

```
kqueue
```

---

# What is `kqueue()`?

A scalable event notification interface.

Supports:

- Sockets
- Files
- Signals
- Timers
- Process events

Through one unified API.

---

# Visualization

```
Socket

↓

Timer

↓

Signal

↓

File

↓

kqueue

↓

Application
```

One event queue for many event types.

---

# macOS

On macOS,

applications typically use:

```
kqueue
```

instead of:

```
epoll
```

---

# Generation 5 — `io_uring`

Newest Linux interface.

Introduced to reduce:

```
System Calls

↓

Context Switches

↓

Copies
```

---

# Traditional I/O

Application:

```
read()

↓

Kernel

↓

Return

↓

write()

↓

Kernel

↓

Return
```

Many system calls.

---

# `io_uring`

Application places requests into:

```
Submission Queue
```

Kernel places completions into:

```
Completion Queue
```

Very efficient.

---

# Visualization

Application

```
Submission Queue

↓

Kernel

↓

Completion Queue

↓

Application
```

Much less overhead.

---

# Why Is `io_uring` Fast?

Traditional:

```
System Call

↓

Kernel

↓

Return

↓

Repeat
```

`io_uring`:

```
Batch Requests

↓

Kernel

↓

Batch Completions
```

Fewer transitions between user space and kernel space.

---

# Supported Operations

`io_uring` can perform:

- Read
- Write
- Accept
- Connect
- Send
- Receive
- Timeout
- File operations

And more.

---

# Comparison

### `select()`

```
Simple

↓

Old

↓

Small Systems
```

---

### `poll()`

```
Improved

↓

Still Linear
```

---

### `epoll()`

```
Linux

↓

Scalable

↓

Millions Of Connections
```

---

### `kqueue()`

```
BSD

↓

macOS

↓

Scalable
```

---

### `io_uring`

```
Newest

↓

Asynchronous

↓

Extremely Fast
```

---

# Which One Does Node.js Use?

Linux:

```
JavaScript

↓

libuv

↓

epoll
```

macOS:

```
JavaScript

↓

libuv

↓

kqueue
```

Windows:

```
JavaScript

↓

IOCP
```

Node.js automatically chooses the appropriate backend.

---

# Which One Does Nginx Use?

Linux:

```
epoll
```

BSD:

```
kqueue
```

Windows:

```
IOCP
```

Platform-specific optimizations.

---

# Which One Does Redis Use?

Redis:

```
Linux

↓

epoll
```

Other platforms use their native event APIs.

---

# Which One Does Docker Use?

Docker networking ultimately relies on Linux kernel facilities,

including:

- `epoll`
- sockets
- namespaces

depending on the subsystem.

---

# Performance Comparison

| API | Scalability | Platform |
|------|------------|----------|
| `select()` | Low | POSIX |
| `poll()` | Medium | POSIX |
| `epoll()` | Very High | Linux |
| `kqueue()` | Very High | BSD/macOS |
| `io_uring()` | Extremely High | Linux |

---

# When Should You Use Each?

Small educational programs:

```
select()
```

Portable POSIX applications:

```
poll()
```

Linux servers:

```
epoll()
```

macOS/BSD servers:

```
kqueue()
```

Modern Linux high-performance systems:

```
io_uring()
```

---

# Common Mistakes

---

## Thinking `epoll` Exists Everywhere

It is Linux-specific.

---

## Thinking `kqueue` Works on Linux

It belongs to BSD-derived systems, including macOS.

---

## Assuming `io_uring` Replaces Everything

Many production systems still successfully use `epoll`.

The best choice depends on workload, maturity, and compatibility requirements.

---

## Optimizing Too Early

For small applications,

simplicity often matters more than maximum scalability.

---

# Architecture Comparison

Traditional:

```
Application

↓

read()

↓

Kernel

↓

Return
```

`epoll`:

```
Kernel

↓

Ready Events

↓

Application
```

`io_uring`:

```
Submission Queue

↓

Kernel

↓

Completion Queue
```

---

# Hands-on Labs

## Lab 1

Study:

- `select()`
- `poll()`
- `epoll()`

Understand their conceptual differences.

---

## Lab 2

Read the `epoll` backend implementation in:

```
libuv
```

---

## Lab 3

Read the Linux documentation for:

```
io_uring
```

Understand submission and completion queues.

---

## Lab 4

Compare:

- Nginx architecture
- Redis event loop
- Node.js event loop

Identify which event API each uses on Linux.

---

## Lab 5

Draw the complete evolution:

```
select

↓

poll

↓

epoll

↓

io_uring
```

Explain why each generation was introduced.

---

# Interview Questions

### Why is `epoll` faster than `select()`?

Because the kernel tracks readiness internally and returns only ready descriptors instead of repeatedly scanning all descriptors.

---

### What is `kqueue()`?

A scalable BSD/macOS event notification mechanism supporting multiple event types.

---

### What is `io_uring`?

A modern Linux asynchronous I/O interface based on submission and completion queues.

---

### Which API does Node.js use on Linux?

`epoll` through the `libuv` library.

---

### Which API powers most high-performance Linux servers today?

Many production systems still use `epoll`, while some newer applications adopt `io_uring` for additional performance.

---

# Summary

Evolution:

```
select()

↓

poll()

↓

epoll()

↓

kqueue()

↓

io_uring()
```

Platform Mapping:

| Platform | Preferred API |
|----------|---------------|
| Linux | `epoll()` / `io_uring()` |
| macOS | `kqueue()` |
| FreeBSD | `kqueue()` |
| Windows | IOCP |

## Key Takeaways

- Modern event APIs evolved to improve scalability and reduce overhead.
- `select()` and `poll()` repeatedly examine descriptor sets.
- `epoll()` provides scalable event notification on Linux.
- `kqueue()` serves a similar role on BSD systems and macOS.
- `io_uring` introduces a queue-based asynchronous I/O model with reduced system-call overhead.
- Understanding these APIs explains how modern web servers and runtimes achieve high performance.

---

# 🎉 Linux High-Performance I/O Complete!

You now understand:

- ✅ Blocking I/O
- ✅ Non-blocking I/O
- ✅ `select()`
- ✅ `poll()`
- ✅ `epoll()`
- ✅ `kqueue()`
- ✅ `io_uring()`

This knowledge forms the foundation of modern high-performance network programming.

---

# Next Chapter

## Chapter 129 — Memory Mapping (`mmap()`) — The Most Powerful System Call in Linux

You'll learn:

- Why `mmap()` is considered one of Linux's most powerful system calls
- File-backed memory mapping
- Anonymous memory mapping
- Shared vs private mappings
- Copy-on-Write mappings
- Zero-copy I/O concepts
- Why databases, browsers, Redis, JVMs, and operating systems rely heavily on `mmap()`