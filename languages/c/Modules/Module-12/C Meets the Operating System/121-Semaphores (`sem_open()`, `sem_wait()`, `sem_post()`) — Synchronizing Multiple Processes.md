# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 121 — Semaphores (`sem_open()`, `sem_wait()`, `sem_post()`) — Synchronizing Multiple Processes

---

# Most beginners think:

> "If two processes share memory, everything will work automatically."

No.

Sharing memory is easy.

Sharing it **correctly** is difficult.

Suppose:

```
Process A

↓

balance = 100

↓

+50

-------------------------

Process B

↓

balance = 100

↓

-20
```

Expected:

```
130
```

Possible Result:

```
80

100

150

130
```

Why?

Because both processes modify the same memory simultaneously.

This problem is called a:

```
Race Condition
```

The solution is:

```
Synchronization
```

One of the oldest synchronization mechanisms is the:

```
Semaphore
```

---

# Learning Objectives

After this chapter you will understand:

- Race conditions
- Critical sections
- Mutual exclusion
- Binary semaphores
- Counting semaphores
- `sem_open()`
- `sem_wait()`
- `sem_post()`
- Producer-consumer synchronization

---

# The Problem

Suppose two processes execute:

```c
counter++;
```

Looks harmless.

Internally it is:

```
Read Counter

↓

Add 1

↓

Write Counter
```

Three separate operations.

---

# Example

Initial value:

```
counter = 0
```

Process A:

```
Read

↓

0
```

Before writing,

scheduler switches.

Process B:

```
Read

↓

0

↓

Write 1
```

Switch again.

Process A:

```
Write 1
```

Expected:

```
2
```

Actual:

```
1
```

One update was lost.

---

# Race Condition

A race condition occurs when:

> Multiple threads or processes access shared data concurrently and the final result depends on the timing of execution.

---

# Visualization

```
Process A

↓

Shared Memory

↑

Process B

↓

Writes

↓

Conflict
```

---

# Critical Section

The code that accesses shared data is called the:

```
Critical Section
```

Example:

```c
counter++;
```

Only one execution context should enter this section at a time.

---

# Mutual Exclusion

Goal:

```
One Process

↓

Critical Section

↓

Exit

↓

Next Process
```

Never:

```
Two Processes

↓

Same Critical Section

↓

Same Time
```

---

# What is a Semaphore?

A semaphore is:

> **A synchronization object used to coordinate access to shared resources.**

Think of it as a traffic signal.

---

# Traffic Light Analogy

One-lane bridge.

```
Car A

↓

Green

↓

Cross

↓

Red

↓

Car B Waits
```

Only one car uses the bridge.

Semaphores provide the same control for shared resources.

---

# Binary Semaphore

Values:

```
0

↓

Locked

-------------------

1

↓

Unlocked
```

Similar to a simple lock.

---

# Counting Semaphore

Values:

```
0

1

2

3

...

N
```

Represents multiple available resources.

Example:

```
5 Database Connections
```

At most five clients may use them simultaneously.

---

# POSIX Semaphores

Include:

```c
#include <semaphore.h>
```

Common functions:

```
sem_open()

sem_wait()

sem_post()

sem_close()

sem_unlink()
```

---

# Creating a Semaphore

Example:

```c
sem_t *sem =
sem_open(
"/lock",
O_CREAT,
0666,
1);
```

Initial value:

```
1
```

Meaning:

```
Unlocked
```

---

# `sem_wait()`

Conceptually:

```
Semaphore

↓

Available?

↓

Yes

↓

Enter

--------------------

No

↓

Sleep
```

If the value is zero,

the caller blocks.

---

# Visualization

Semaphore:

```
1
```

Process A:

```
sem_wait()

↓

0

↓

Enter
```

Process B:

```
sem_wait()

↓

Wait
```

---

# `sem_post()`

After leaving the critical section:

```c
sem_post(sem);
```

Conceptually:

```
0

↓

1

↓

Wake Waiting Process
```

---

# Complete Flow

```
sem_wait()

↓

Critical Section

↓

sem_post()
```

Only one process enters at a time (for a binary semaphore).

---

# Example

```c
sem_wait(sem);

counter++;

sem_post(sem);
```

Now:

```
No Race Condition
```

---

# Why Blocking Is Good

Instead of:

```
Loop Forever

↓

Checking Lock
```

Kernel simply puts the process to sleep.

When the semaphore becomes available,

the kernel wakes it.

Efficient.

---

# Producer-Consumer Problem

Producer:

```
Generate Data

↓

Buffer
```

Consumer:

```
Read Data

↓

Buffer
```

Without synchronization:

```
Producer

↓

Overwrite

↓

Consumer
```

Or:

```
Consumer

↓

Read Empty Buffer
```

Semaphores coordinate both sides.

---

# Visualization

```
Producer

↓

Semaphore

↓

Buffer

↓

Semaphore

↓

Consumer
```

---

# Named Semaphore

Like shared memory,

named semaphores exist in the system until removed.

Cleanup:

```c
sem_unlink("/lock");
```

---

# Binary vs Counting

Binary:

```
Printer

↓

Only One User
```

Counting:

```
Connection Pool

↓

10 Connections
```

Up to ten clients simultaneously.

---

# Real-World Example — Database

Shared cache:

```
Workers

↓

Semaphore

↓

Shared Memory
```

Protects shared structures.

---

# Real-World Example — Web Server

Thread pool:

```
Worker

↓

Semaphore

↓

Request Queue
```

Coordinates access to shared work.

---

# Real-World Example — Video Streaming

Decoder:

```
Frame Ready

↓

Semaphore

↓

Renderer
```

Signals that new data is available.

---

# Real-World Example — Printer Queue

Only one process should send data to the printer at a time.

A semaphore ensures orderly access.

---

# Common Mistakes

---

## Forgetting `sem_post()`

Process enters:

```
sem_wait()
```

Never leaves.

Everyone else waits forever.

This is effectively a deadlock.

---

## Protecting Too Much Code

Keep critical sections as short as possible.

Long critical sections reduce concurrency.

---

## Ignoring Race Conditions

Even:

```c
counter++;
```

is not an atomic operation.

---

## Forgetting Cleanup

Release system resources using:

```c
sem_close()

sem_unlink()
```

when appropriate.

---

# Complete Example

```c
sem_wait(sem);

/* Critical Section */

counter++;

sem_post(sem);
```

Execution:

```
Wait

↓

Enter

↓

Update

↓

Release
```

---

# Hands-on Labs

## Lab 1

Create a named semaphore using:

```c
sem_open()
```

---

## Lab 2

Protect a shared counter with:

```c
sem_wait()

sem_post()
```

---

## Lab 3

Remove the semaphore using:

```c
sem_unlink()
```

---

## Lab 4

Create two processes.

Attempt concurrent writes with and without a semaphore.

Compare the results.

---

## Lab 5

Implement a simple producer-consumer example using shared memory and semaphores.

---

# Interview Questions

### What is a semaphore?

A synchronization object used to coordinate access to shared resources.

---

### What is a race condition?

A situation where the result depends on the timing of concurrent execution.

---

### What is a critical section?

The part of the program that accesses shared resources and must be synchronized.

---

### What is the difference between a binary semaphore and a counting semaphore?

A binary semaphore allows one holder at a time.

A counting semaphore tracks multiple available resources.

---

### Why does `sem_wait()` block?

To prevent concurrent access when the semaphore is unavailable.

---

# Summary

Race Condition:

```
Process A

↓

Shared Data

↑

Process B

↓

Conflict
```

Semaphore Protection:

```
sem_wait()

↓

Critical Section

↓

sem_post()
```

Comparison:

| Type | Value Range | Use Case |
|------|-------------|----------|
| Binary Semaphore | 0 or 1 | Mutual exclusion |
| Counting Semaphore | 0..N | Resource counting |

## Key Takeaways

- Shared memory requires synchronization.
- Race conditions occur when concurrent operations interfere with each other.
- Semaphores coordinate access to shared resources.
- `sem_wait()` acquires access; `sem_post()` releases it.
- Binary semaphores enforce mutual exclusion.
- Counting semaphores manage pools of identical resources.

---

# Next Chapter

## Chapter 122 — Mutexes (`pthread_mutex_t`) — Thread Synchronization

You'll learn:

- Why mutexes exist
- Mutex vs semaphore
- `pthread_mutex_init()`
- `pthread_mutex_lock()`
- `pthread_mutex_unlock()`
- Recursive mutexes
- Deadlocks
- Best practices for multithreaded synchronization