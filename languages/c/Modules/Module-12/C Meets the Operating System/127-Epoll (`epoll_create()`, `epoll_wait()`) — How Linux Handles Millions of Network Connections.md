# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 127 — `epoll` — How Linux Handles Millions of Network Connections

---

# Most beginners think:

> "One client = One thread."

This is how many beginners write servers.

```
Client 1

↓

Thread 1

-----------------------

Client 2

↓

Thread 2

-----------------------

Client 3

↓

Thread 3
```

Looks fine...

Until:

```
100,000 Clients
```

Now you have:

```
100,000 Threads
```

Impossible.

Modern Linux servers like:

- Nginx
- Node.js (libuv)
- Redis
- HAProxy

do **NOT** create one thread per connection.

Instead they use one of Linux's greatest inventions:

```
epoll
```

---

# Learning Objectives

After this chapter you will understand:

- Why thread-per-connection fails
- Blocking I/O
- Non-blocking I/O
- I/O Multiplexing
- `select()`
- `poll()`
- `epoll`
- Edge Triggered
- Level Triggered

---

# The Problem

Suppose:

```
100,000 Clients
```

Only:

```
2,000

↓

Sending Data
```

The other:

```
98,000

↓

Idle
```

Should Linux create:

```
100,000 Threads?
```

No.

That wastes:

- Memory
- CPU
- Scheduler time

---

# Why Thread Per Connection Fails

Every thread needs:

```
Stack

↓

Registers

↓

Scheduler Entry
```

Example:

```
100,000 Threads

×

1 MB Stack

=

100 GB Memory
```

Impossible for most systems.

---

# Think About a Teacher

Bad approach:

```
100 Students

↓

100 Teachers
```

Better:

```
1 Teacher

↓

Raise Hand

↓

Teacher Helps
```

Linux works the same way.

---

# Blocking I/O

Example:

```c
read(socket);
```

If no data exists:

```
Thread

↓

Sleep

↓

Waiting
```

CPU isn't busy,

but one thread is occupied.

---

# Visualization

```
Client

↓

No Data

↓

Thread Waiting

↓

Nothing Happens
```

Thousands of waiting threads waste resources.

---

# Non-Blocking I/O

Instead:

```
read()

↓

No Data?

↓

Return Immediately
```

Program continues doing other work.

---

# Problem

Without another mechanism:

```
Loop

↓

read()

↓

No Data

↓

read()

↓

No Data

↓

Forever
```

This becomes:

```
Busy Polling
```

Not efficient.

---

# I/O Multiplexing

Idea:

Instead of asking:

```
Socket 1?

↓

Socket 2?

↓

Socket 3?
```

Ask Linux:

```
Tell Me

↓

When Something Is Ready
```

---

# Visualization

```
Sockets

↓

Kernel

↓

Ready List

↓

Application
```

The kernel monitors all sockets.

---

# `select()`

Oldest solution.

Program gives Linux:

```
Socket List
```

Linux checks:

```
Ready?

↓

Return
```

Problem:

Every call scans every socket.

---

# Complexity

Suppose:

```
100,000 Sockets
```

Kernel scans:

```
1

↓

2

↓

3

↓

...

↓

100,000
```

Even if only:

```
2

↓

Ready
```

Slow.

---

# `poll()`

Improved interface.

Still:

```
Linear Scan

↓

Every Call
```

Better API.

Similar scaling limitations.

---

# Enter `epoll`

Instead of:

```
Check Everything
```

Linux keeps:

```
Ready Socket List
```

Application receives only sockets that are ready.

---

# Visualization

Without epoll

```
100,000 Sockets

↓

Scan All

↓

2 Ready
```

With epoll

```
100,000 Sockets

↓

Kernel Tracks

↓

2 Ready

↓

Return Only Those
```

Huge improvement.

---

# Creating an epoll Instance

Example:

```c
epoll_create1(0);
```

Kernel creates:

```
epoll Object
```

---

# Registering Sockets

Example:

```c
epoll_ctl()
```

Meaning:

```
Watch

↓

This Socket
```

Linux now monitors it.

---

# Waiting

Example:

```c
epoll_wait()
```

Behavior:

```
Sleep

↓

Socket Ready

↓

Wake

↓

Return Ready Events
```

No busy polling.

---

# Workflow

```
epoll_create()

↓

epoll_ctl()

↓

epoll_wait()

↓

Process Events

↓

Repeat
```

---

# Event Loop

Most modern servers:

```
while(1)
{
    epoll_wait();

    Process Ready Events;
}
```

One thread.

Thousands of sockets.

---

# Level Triggered (LT)

Default mode.

If data remains unread:

```
Kernel

↓

Still Ready

↓

Notify Again
```

Easy to use.

---

# Example

Socket contains:

```
100 Bytes
```

Program reads:

```
20 Bytes
```

Remaining:

```
80 Bytes
```

Next:

```
epoll_wait()

↓

Ready Again
```

---

# Edge Triggered (ET)

Different behavior.

Kernel notifies only when:

```
State Changes
```

Example:

```
Empty

↓

Data Arrives

↓

Notify Once
```

If you don't read everything,

Linux may not notify again.

---

# Visualization

Level Triggered

```
Data Exists

↓

Ready

↓

Ready

↓

Ready
```

Edge Triggered

```
Data Arrives

↓

Ready Once
```

---

# Which Is Faster?

Edge Triggered:

```
Fewer Notifications
```

Potentially faster,

but harder to program correctly.

---

# Non-Blocking Requirement

Edge-triggered mode is normally used with:

```
Non-Blocking Sockets
```

Otherwise applications can block unexpectedly.

---

# Event Loop Architecture

```
epoll_wait()

↓

Socket Ready

↓

Read

↓

Process

↓

Write

↓

Continue
```

One loop handles many connections.

---

# Real-World Example — Nginx

```
Millions

↓

Connections

↓

Few Workers

↓

epoll
```

Each worker handles many client sockets.

---

# Real-World Example — Redis

Redis:

```
Single Thread

↓

epoll

↓

Thousands Of Clients
```

It relies on efficient event-driven I/O.

---

# Real-World Example — Node.js

Node.js:

```
JavaScript

↓

libuv

↓

epoll (Linux)
```

Your JavaScript doesn't call `epoll()` directly.

`libuv` does it internally.

---

# Real-World Example — HAProxy

```
Incoming TCP

↓

epoll

↓

Forward
```

Very high-performance networking.

---

# Why epoll Changed Linux

Before:

```
Threads

↓

Waiting
```

After:

```
One Event Loop

↓

Ready Events Only
```

Much lower memory usage.

Much higher scalability.

---

# Common Mistakes

---

## Forgetting Non-Blocking Mode

Especially with edge-triggered `epoll`.

---

## Reading Only Part of the Data in ET Mode

Applications should continue reading until no more data is available.

---

## Thinking epoll Creates Threads

It does not.

It is an I/O notification mechanism.

---

## Assuming epoll Exists Everywhere

`epoll` is Linux-specific.

Other operating systems provide different APIs.

---

# Complete Workflow

```
Create Socket

↓

Make Non-Blocking

↓

epoll_create()

↓

epoll_ctl()

↓

epoll_wait()

↓

Read

↓

Write

↓

Repeat
```

---

# Hands-on Labs

## Lab 1

Create an epoll instance.

Register one socket.

Observe readiness notifications.

---

## Lab 2

Compare:

- `select()`
- `poll()`
- `epoll()`

Understand the conceptual differences.

---

## Lab 3

Switch between:

- Level Triggered
- Edge Triggered

Observe behavior.

---

## Lab 4

Build a simple event loop using:

```c
epoll_wait()
```

---

## Lab 5

Read the source code or architecture documentation of:

- Nginx
- Redis
- libuv

Identify where event-driven I/O is used.

---

# Interview Questions

### What problem does `epoll` solve?

Efficiently monitoring large numbers of file descriptors without repeatedly scanning every descriptor.

---

### Why is `epoll` faster than `select()`?

The kernel maintains readiness information and returns only ready file descriptors.

---

### What is the difference between Level Triggered and Edge Triggered modes?

Level Triggered repeatedly reports readable or writable conditions while they remain true.

Edge Triggered reports changes in readiness and typically requires draining all available data.

---

### Why are non-blocking sockets commonly used with `epoll`?

To prevent one socket operation from blocking the entire event loop.

---

### Which popular software uses `epoll`?

- Nginx
- Redis
- HAProxy
- Node.js (through `libuv`)

---

# Summary

Traditional Model:

```
Client

↓

Thread

↓

Waiting
```

Event-Driven Model:

```
Clients

↓

epoll

↓

Ready Events

↓

One Event Loop
```

Comparison:

| API | Scalability | Notes |
|------|------------|-------|
| `select()` | Low | Scans all descriptors each call |
| `poll()` | Medium | Improved interface, still scans |
| `epoll()` | Very High | Returns ready events efficiently |

## Key Takeaways

- Thread-per-connection does not scale to very large numbers of clients.
- `epoll` is Linux's high-performance I/O event notification facility.
- Applications register file descriptors once and wait for readiness events.
- Level Triggered mode is simpler; Edge Triggered mode can reduce notifications but requires more careful programming.
- Modern Linux servers rely heavily on `epoll` for scalability.
- Understanding `epoll` is essential for building high-performance network services.

---

# Next Chapter

## Chapter 128 — `select()` vs `poll()` vs `epoll()` vs `kqueue()` vs `io_uring`

You'll learn:

- Complete comparison of Linux and BSD I/O multiplexing APIs
- Why `epoll` replaced `select()`
- Why `io_uring` is changing Linux again
- `kqueue()` on BSD and macOS
- Performance characteristics
- Which API powers Nginx, Redis, Node.js, and modern high-performance systems