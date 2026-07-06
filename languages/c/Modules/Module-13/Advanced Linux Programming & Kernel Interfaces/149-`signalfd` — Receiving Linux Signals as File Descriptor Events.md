# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 149 — `signalfd` — Receiving Linux Signals as File Descriptor Events

---

# Most developers learn signals like this:

```c
signal(SIGINT, handler);
```

or

```c
sigaction(...);
```

The handler executes **asynchronously**.

This works.

But large Linux servers often avoid traditional signal handlers whenever possible.

Instead they use:

```
signalfd()
```

Why?

Because Linux prefers one unified event model:

```
Sockets

↓

Timers

↓

File Changes

↓

Signals

↓

Everything

↓

File Descriptors
```

This chapter explains how Linux converts asynchronous signals into ordinary readable events.

---

# Learning Objectives

After this chapter you will understand:

- What `signalfd()` is
- Traditional signal handling
- Signal masks
- Signal queues
- Receiving signals through file descriptors
- Integration with `epoll`
- Why event-driven servers prefer `signalfd`

---

# The Problem

Suppose your server receives:

```
SIGINT
```

Traditional Linux:

```
Signal

↓

Interrupt Current Code

↓

Run Handler
```

Problem:

The handler may interrupt your code almost anywhere.

Example:

```
malloc()

↓

Interrupted

↓

Handler

↓

Return
```

Asynchronous execution is powerful,

but it increases complexity.

---

# Linux Solution

Instead of:

```
Signal

↓

Handler
```

Linux can do:

```
Signal

↓

Queue

↓

Readable FD
```

Now signals behave like:

```
Socket Data
```

---

# What is `signalfd()`?

Definition:

> **`signalfd()` creates a file descriptor that receives Linux signals as readable events instead of invoking traditional signal handlers.**

---

# Visualization

```
Signal

↓

Kernel Queue

↓

signalfd

↓

read()

↓

Application
```

---

# Think About Email

Traditional handler:

```
Phone Rings

↓

Answer Immediately
```

`signalfd`:

```
Email Arrives

↓

Read Inbox

↓

When Ready
```

The application decides when to process signals.

---

# Traditional Signal Handling

```
SIGINT

↓

CPU Interrupts Program

↓

Handler Executes

↓

Resume
```

---

# `signalfd` Handling

```
SIGINT

↓

Kernel Queue

↓

FD Readable

↓

epoll()

↓

Application Reads Signal
```

Everything becomes event-driven.

---

# Creating a `signalfd`

Program calls:

```c
signalfd(...)
```

Kernel returns:

```
File Descriptor
```

Initially:

```
No Pending Signals

↓

FD Not Readable
```

---

# Signal Mask

Important concept.

Before using `signalfd`,

applications typically block the signals they intend to receive through the file descriptor.

Otherwise,

the default signal action or handler may run instead.

---

# Visualization

```
SIGINT

↓

Blocked

↓

Kernel Queue

↓

signalfd
```

---

# Signal Arrives

Suppose user presses:

```
Ctrl + C
```

Normally:

```
SIGINT

↓

Handler
```

With `signalfd`:

```
SIGINT

↓

Queue

↓

FD Ready
```

---

# Reading Signals

Application calls:

```c
read(fd,...)
```

Kernel returns structured information describing the pending signal.

Typical information includes:

- Signal number
- Sender PID
- Sender UID

and other metadata.

---

# Multiple Signals

Suppose:

```
SIGUSR1

↓

SIGTERM

↓

SIGINT
```

Kernel queues them.

Application reads them in order according to Linux signal semantics.

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

signalfd

↓

epoll_wait()
```

One event loop handles everything.

---

# Event Loop Example

```
Incoming Client

↓

Socket Event

----------------------

Timeout

↓

timerfd

----------------------

Signal

↓

signalfd

----------------------

Filesystem

↓

inotify
```

No separate signal handler required.

---

# Graceful Shutdown

Suppose server receives:

```
SIGTERM
```

Event loop wakes.

Application performs:

```
Close Clients

↓

Save Data

↓

Exit
```

Very clean.

---

# Signal Queue

Kernel stores pending signals until the application reads them.

This avoids unexpected asynchronous execution inside arbitrary functions.

---

# Relationship with Traditional Signals

Traditional:

```
Signal

↓

Handler

↓

Interrupt
```

Modern Event Loop:

```
Signal

↓

Queue

↓

FD

↓

Application
```

---

# Relationship with `sigaction`

`sigaction()` installs signal handlers.

`signalfd()` replaces many handler use cases by exposing signals as readable events.

Both remain valid Linux APIs.

---

# Relationship with `eventfd`

```
eventfd

↓

Thread Notification

------------------------

signalfd

↓

Signal Notification
```

Both use file descriptors.

---

# Relationship with `timerfd`

```
timerfd

↓

Time Events

-----------------------

signalfd

↓

Signal Events
```

Same programming model.

---

# Relationship with `inotify`

```
inotify

↓

Filesystem Events

-----------------------

signalfd

↓

Signal Events
```

Everything becomes event-driven.

---

# Real-World Example — Server Shutdown

Administrator runs:

```bash
kill -TERM server
```

Server receives:

```
SIGTERM

↓

signalfd

↓

Graceful Shutdown
```

---

# Real-World Example — Supervisors

Process supervisors monitor signals and react without relying on asynchronous handlers.

---

# Real-World Example — Event-Driven Applications

Applications already using `epoll` can incorporate signals without creating a separate control flow.

---

# Real-World Example — Containers

Container entry-point processes commonly handle signals such as `SIGTERM` for graceful shutdown.

A file-descriptor-based model fits naturally into event-driven architectures.

---

# Common Mistakes

---

## Forgetting to Block Signals

Signals intended for `signalfd` should generally be blocked so they are delivered through the file descriptor.

---

## Mixing Handlers and `signalfd` Incorrectly

Using both for the same signal without understanding signal masks can produce unexpected behavior.

---

## Forgetting to Read the FD

Unread pending signals keep the descriptor readable.

---

## Assuming Every Signal Can Be Queued the Same Way

Linux distinguishes between traditional signals and real-time signals.

Their queuing semantics differ.

---

# Hands-on Labs

## Lab 1

Read:

```bash
man 2 signalfd
```

Understand its API.

---

## Lab 2

Create a `signalfd` for:

```
SIGINT
```

Press:

```
Ctrl + C
```

Observe the event.

---

## Lab 3

Integrate:

- Socket
- `timerfd`
- `signalfd`

into one `epoll` loop.

---

## Lab 4

Experiment with:

```
SIGUSR1

SIGTERM

SIGINT
```

Observe the signal information returned by `read()`.

---

## Lab 5

Compare:

- Traditional signal handlers
- `signalfd`

Understand the differences in programming style.

---

# Interview Questions

### What is `signalfd()`?

A Linux system call that delivers signals through a readable file descriptor.

---

### Why do event-driven applications use `signalfd`?

It integrates signal handling with `epoll` and avoids asynchronous signal handlers.

---

### Why should signals be blocked before using `signalfd`?

So the kernel delivers them through the file descriptor instead of invoking their default handling path.

---

### Can `signalfd` be monitored with `epoll`?

Yes.

It is designed specifically for that purpose.

---

### What information is returned when reading from a `signalfd`?

Structured information about pending signals, including the signal number and related metadata.

---

# Summary

Architecture:

```
Signal

↓

Kernel Queue

↓

signalfd

↓

read()

↓

Application
```

Traditional Model:

```
Signal

↓

Handler

↓

Interrupt
```

Modern Event Loop:

```
Socket

↓

timerfd

↓

eventfd

↓

inotify

↓

signalfd

↓

epoll_wait()
```

Comparison:

| Mechanism | Purpose |
|-----------|----------|
| `eventfd` | Thread/process notifications |
| `timerfd` | Timer events |
| `inotify` | Filesystem events |
| `signalfd` | Signal events |

## Key Takeaways

- `signalfd` converts Linux signals into ordinary file descriptor events.
- It fits naturally into event-driven architectures.
- Signals should generally be blocked before using `signalfd`.
- It avoids many complexities associated with asynchronous signal handlers.
- `epoll` can monitor `signalfd` alongside sockets and other event sources.
- `signalfd` completes Linux's philosophy of representing many kernel events as file descriptors.

---

# Next Chapter

## Chapter 150 — Netlink Sockets — How User Space Communicates with the Linux Kernel

You'll learn:

- What Netlink is
- Why normal sockets are not enough
- Kernel ↔ user-space messaging
- Routing table updates
- Network configuration
- `iproute2`
- `udev`
- `systemd`
- How Linux tools communicate directly with kernel subsystems without using ordinary files or traditional system calls