# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 148 — `timerfd` — Turning Timers into File Descriptors

---

# Most developers create timers like this:

```c
sleep(5);
```

or

```c
setitimer();
```

or

```c
alarm();
```

These work.

But modern Linux servers rarely use them.

Instead they use:

```
timerfd()
```

Why?

Because Linux has a simple philosophy:

> **Everything should become a file descriptor whenever possible.**

Just like:

- sockets
- pipes
- eventfd
- inotify

Timers also become:

```
File Descriptors
```

This allows one event loop to manage:

- Network sockets
- Timers
- File changes
- Notifications

using a single API.

---

# Learning Objectives

After this chapter you will understand:

- What `timerfd()` is
- One-shot timers
- Periodic timers
- Timer expiration counters
- High-resolution timers
- Integration with `epoll`
- Why event loops use `timerfd`

---

# The Problem

Suppose you're writing:

```
Redis

↓

Nginx

↓

Node.js

↓

Database
```

You need to wait for:

- Client sockets
- Timeout events
- Background jobs
- Scheduled tasks

Without `timerfd` you might need:

```
epoll()

+

Signals

+

sleep()

+

Threads
```

Complex.

---

# Linux Solution

Linux says:

```
Timer

↓

File Descriptor
```

Now:

```
Socket

↓

eventfd

↓

timerfd

↓

inotify

↓

epoll()
```

Everything fits into one event loop.

---

# What is `timerfd()`?

Definition:

> **`timerfd()` creates a file descriptor that becomes readable when a timer expires.**

Notice:

```
Readable

↓

Like A File
```

---

# Visualization

```
Application

↓

timerfd()

↓

Kernel Timer

↓

Expiration

↓

FD Becomes Readable
```

---

# Think About an Alarm Clock

Traditional timer:

```
Alarm Rings

↓

Signal
```

`timerfd`:

```
Alarm Rings

↓

FD Ready

↓

epoll()
```

Much easier to integrate.

---

# Creating a Timer

Program calls:

```c
timerfd_create(...)
```

Kernel returns:

```
FD
```

Initially:

```
Not Ready
```

---

# Setting the Timer

Program configures:

```
5 Seconds
```

Kernel starts counting.

---

# Visualization

```
5

↓

4

↓

3

↓

2

↓

1

↓

Expire
```

---

# Timer Expires

Kernel marks:

```
FD

↓

Readable
```

Your event loop wakes immediately.

---

# Reading the Timer

When you call:

```c
read(fd,...)
```

You receive:

```
Expiration Count
```

Usually:

```
1
```

---

# Why a Counter?

Suppose:

```
1 Second Timer
```

Application sleeps for:

```
10 Seconds
```

Instead of losing events,

kernel returns:

```
10 Expirations
```

Very useful.

---

# Visualization

```
Tick

↓

Tick

↓

Tick

↓

Application Reads

↓

3
```

---

# One-Shot Timer

Example:

```
Expire Once

↓

Done
```

Timeline:

```
Start

↓

5 Seconds

↓

Wake

↓

Stop
```

---

# Periodic Timer

Example:

```
Every Second
```

Timeline:

```
Tick

↓

Tick

↓

Tick

↓

Tick
```

Never stops until disabled.

---

# High-Resolution Timers

Linux supports:

```
Nanosecond Precision
```

Internally the actual precision depends on:

- Kernel configuration
- Hardware timer capabilities
- System load

But `timerfd` supports very fine-grained timing.

---

# Absolute vs Relative Time

Relative:

```
Now

+

5 Seconds
```

Absolute:

```
Expire At

↓

12:00:00
```

Both are supported through timer configuration.

---

# Clock Sources

Linux offers multiple clocks.

Common choices:

```
CLOCK_MONOTONIC

↓

Time Since Boot

----------------------

CLOCK_REALTIME

↓

Wall Clock Time
```

Choosing the right clock matters.

---

# CLOCK_REALTIME

Uses:

```
Current Date

↓

Current Time
```

Affected by:

- NTP synchronization
- Manual clock changes

---

# CLOCK_MONOTONIC

Measures elapsed time since boot.

Not affected by wall-clock adjustments.

Preferred for most timeout logic.

---

# Relationship with `epoll`

This is the biggest advantage.

```
Socket

↓

timerfd

↓

eventfd

↓

inotify

↓

epoll_wait()
```

One thread waits for everything.

---

# Event Loop Example

```
Client Socket

↓

Incoming Data

--------------------

Timer

↓

Timeout

--------------------

File Watch

↓

Changed

--------------------

eventfd

↓

Notification
```

All handled by one event loop.

---

# Timeout Example

Suppose:

```
HTTP Request

↓

30 Seconds
```

If no response:

```
timerfd

↓

Readable

↓

Close Connection
```

No separate timer thread needed.

---

# Heartbeat Example

Every:

```
5 Seconds
```

Timer expires.

Application sends:

```
Heartbeat Packet
```

---

# Scheduler Example

Background cleanup:

```
Every Minute

↓

timerfd

↓

Delete Expired Sessions
```

---

# Relationship with `eventfd`

```
eventfd

↓

Thread Notification

------------------------

timerfd

↓

Time Notification
```

Both integrate identically with `epoll`.

---

# Relationship with Signals

Traditional:

```
Timer

↓

SIGALRM
```

Modern servers often prefer:

```
timerfd

↓

epoll
```

because it integrates cleanly into event-driven architectures.

---

# Relationship with `io_uring`

Modern asynchronous I/O systems can also combine timer mechanisms with asynchronous operations.

`timerfd` remains a useful general-purpose timer interface.

---

# Real-World Example — Nginx

Connection timeouts:

```
timerfd / internal timer mechanisms

↓

Event Loop

↓

Close Idle Connections
```

---

# Real-World Example — Redis

Scheduled maintenance tasks and timeout management fit naturally into an event-driven architecture.

---

# Real-World Example — Database

Periodic:

```
Checkpoint

↓

Cleanup

↓

Statistics
```

Often coordinated using timer facilities.

---

# Real-World Example — Monitoring Agent

Every:

```
10 Seconds
```

Collect:

- CPU
- Memory
- Disk

Send metrics.

---

# Common Mistakes

---

## Thinking `timerfd` Delivers Signals

It delivers readiness through a file descriptor.

---

## Forgetting to Read the FD

If you don't read it,

the timer remains readable,

and `epoll` will continue reporting it.

---

## Using `CLOCK_REALTIME` for Timeouts

Wall-clock adjustments may affect behavior.

For elapsed-time measurements,

`CLOCK_MONOTONIC` is often the better choice.

---

## Creating Many Timer Threads

A single event loop with `timerfd` is often simpler and more scalable.

---

# Hands-on Labs

## Lab 1

Read:

```bash
man 2 timerfd_create
```

Understand the API.

---

## Lab 2

Create a one-shot timer.

Observe when the file descriptor becomes readable.

---

## Lab 3

Create a periodic timer.

Print the expiration count returned by `read()`.

---

## Lab 4

Integrate a `timerfd` into an `epoll` event loop.

---

## Lab 5

Compare:

- `alarm()`
- `sleep()`
- `timerfd()`

Understand the strengths and limitations of each.

---

# Interview Questions

### What is `timerfd()`?

A Linux system call that creates a timer represented by a file descriptor.

---

### Why is `timerfd` useful?

It integrates timer events into event loops using ordinary file descriptor APIs.

---

### What happens when a timer expires?

The file descriptor becomes readable, and `read()` returns the number of expirations.

---

### Why is `CLOCK_MONOTONIC` often preferred?

Because it is not affected by changes to the system's wall clock.

---

### Can `timerfd` be used with `epoll`?

Yes.

It is specifically designed for event-driven programming.

---

# Summary

Architecture:

```
Application

↓

timerfd()

↓

Kernel Timer

↓

Expiration

↓

Readable FD
```

Timer Types:

| Type | Behavior |
|------|----------|
| One-shot | Fires once |
| Periodic | Fires repeatedly |

Event Loop:

```
Socket

↓

timerfd

↓

eventfd

↓

inotify

↓

epoll_wait()
```

Clock Choices:

| Clock | Best For |
|--------|----------|
| `CLOCK_MONOTONIC` | Timeouts and intervals |
| `CLOCK_REALTIME` | Wall-clock scheduling |

## Key Takeaways

- `timerfd` turns timers into ordinary file descriptors.
- It integrates seamlessly with Linux event loops.
- Both one-shot and periodic timers are supported.
- `read()` returns expiration counts rather than arbitrary data.
- `CLOCK_MONOTONIC` is generally preferred for timeout logic.
- `timerfd` is a fundamental building block of modern event-driven Linux applications.

---

# Next Chapter

## Chapter 149 — `signalfd` — Receiving Linux Signals as File Descriptor Events

You'll learn:

- What `signalfd` is
- Why signals are difficult to manage
- Converting signals into readable file descriptors
- Signal masks
- Signal queues
- Integration with `epoll`
- Why modern Linux servers often prefer `signalfd` over traditional signal handlers