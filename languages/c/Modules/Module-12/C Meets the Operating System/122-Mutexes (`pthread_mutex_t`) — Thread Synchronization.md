# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 122 — Mutexes (`pthread_mutex_t`) — Thread Synchronization

---

# Most beginners think:

> "Mutexes and semaphores are the same."

They solve similar problems,

but they are **not the same thing**.

A semaphore controls:

```
Resource Count
```

A mutex protects:

```
Critical Section
```

Mutexes are specifically designed for **mutual exclusion** between threads.

If you're writing multithreaded applications,

mutexes are one of the most important synchronization tools you'll ever use.

---

# Learning Objectives

After this chapter you will understand:

- What a mutex is
- Why mutexes exist
- Mutex vs semaphore
- `pthread_mutex_t`
- Locking
- Unlocking
- Deadlocks
- Recursive mutexes
- Best practices

---

# The Problem

Suppose two threads execute:

```c
balance += 100;
```

At the same time.

Thread A:

```
Read

↓

1000
```

Scheduler switches.

Thread B:

```
Read

↓

1000

↓

Write

1100
```

Scheduler switches.

Thread A:

```
Write

1100
```

Expected:

```
1200
```

Actual:

```
1100
```

One update disappeared.

---

# Race Condition

This is another example of a:

```
Race Condition
```

The result depends on execution timing.

---

# Critical Section

The shared code:

```c
balance += 100;
```

is the:

```
Critical Section
```

Only one thread should execute it at a time.

---

# What is a Mutex?

Mutex means:

```
Mutual Exclusion
```

Definition:

> **A mutex is a synchronization object that allows only one thread to own a protected section of code at a time.**

---

# Think About a Bathroom

Imagine:

```
One Bathroom

↓

Ten People
```

Door lock:

```
Unlocked

↓

One Person Enters

↓

Locked

↓

Others Wait

↓

Unlock

↓

Next Person
```

A mutex works exactly like that lock.

---

# Visualization

```
Thread A

↓

Lock

↓

Critical Section

↓

Unlock

↓

Thread B
```

---

# POSIX Threads

Include:

```c
#include <pthread.h>
```

Main mutex type:

```c
pthread_mutex_t
```

---

# Declaring a Mutex

Example:

```c
pthread_mutex_t lock;
```

Initially,

it must be initialized.

---

# Initialization

```c
pthread_mutex_init(
&lock,
NULL);
```

Now the mutex is ready.

---

# Locking

Example:

```c
pthread_mutex_lock(
&lock);
```

Meaning:

```
Acquire Lock
```

If another thread already owns it,

the caller blocks.

---

# Visualization

Mutex:

```
Unlocked
```

Thread A:

```
Lock

↓

Success
```

Thread B:

```
Lock

↓

Wait
```

---

# Unlocking

After the critical section:

```c
pthread_mutex_unlock(
&lock);
```

Meaning:

```
Release Lock
```

Now another waiting thread may continue.

---

# Complete Flow

```
Lock

↓

Critical Section

↓

Unlock
```

---

# Example

```c
pthread_mutex_lock(&lock);

counter++;

pthread_mutex_unlock(&lock);
```

Now:

```
Safe
```

No race condition on `counter`.

---

# Mutex Ownership

Important rule:

```
Thread A

↓

Locks
```

Only:

```
Thread A
```

should unlock that mutex.

Unlike semaphores,

mutexes have ownership semantics.

---

# Binary Semaphore vs Mutex

Binary Semaphore:

```
Count

↓

0 / 1
```

Mutex:

```
Owner

↓

Thread
```

A mutex knows which thread currently owns it.

---

# Deadlock

Suppose:

Thread A:

```
Lock A

↓

Waiting

↓

Lock B
```

Thread B:

```
Lock B

↓

Waiting

↓

Lock A
```

Result:

```
Nobody Moves
```

This is called:

```
Deadlock
```

---

# Visualization

```
Thread A

↓

Lock A

↓

Waiting Lock B

-----------------------

Thread B

↓

Lock B

↓

Waiting Lock A
```

Infinite waiting.

---

# Avoiding Deadlocks

Good practice:

Always acquire locks in the same order.

Example:

```
Lock A

↓

Lock B
```

Never sometimes:

```
Lock B

↓

Lock A
```

Consistency prevents many deadlocks.

---

# Recursive Mutex

Normal mutex:

```
Thread Locks

↓

Locks Again

↓

Deadlock
```

Recursive mutex:

```
Thread Locks

↓

Locks Again

↓

Allowed
```

The same thread can lock it multiple times.

Each lock must eventually be matched with an unlock.

---

# Try Lock

Instead of blocking:

```c
pthread_mutex_trylock()
```

Behavior:

```
Available?

↓

Yes

↓

Lock

-------------------

No

↓

Return Immediately
```

Useful when waiting is undesirable.

---

# Timed Lock

Some systems also provide timed locking,

allowing a thread to wait only for a limited duration before giving up.

---

# Destroying a Mutex

When finished:

```c
pthread_mutex_destroy(
&lock);
```

Releases associated resources.

---

# Real-World Example — Bank

Shared account:

```
Thread A

↓

Mutex

↓

Balance

↑

Thread B
```

Only one update at a time.

---

# Real-World Example — Web Server

Worker threads:

```
Mutex

↓

Request Queue

↓

Next Request
```

Protect shared queues from concurrent modification.

---

# Real-World Example — Browser

Renderer threads:

```
Mutex

↓

Shared Cache
```

Prevent data corruption.

---

# Real-World Example — Database

Worker threads:

```
Mutex

↓

Shared Buffer Pool
```

Coordinate access to cached database pages.

---

# Common Mistakes

---

## Forgetting to Unlock

Thread:

```
Lock

↓

Never Unlock
```

Everyone else waits forever.

---

## Unlocking from the Wrong Thread

Only the owning thread should unlock a mutex.

---

## Locking Too Much Code

Large critical sections reduce concurrency.

Protect only what is necessary.

---

## Destroying a Locked Mutex

Always ensure the mutex is unlocked before destroying it.

---

# Complete Example

```c
pthread_mutex_lock(&lock);

/* Critical Section */

counter++;

pthread_mutex_unlock(&lock);
```

Execution:

```
Acquire

↓

Execute

↓

Release
```

---

# Hands-on Labs

## Lab 1

Create a mutex.

Initialize it.

Destroy it.

---

## Lab 2

Protect a shared counter accessed by multiple threads.

Observe the difference with and without locking.

---

## Lab 3

Create two mutexes.

Intentionally produce a deadlock.

Analyze why it occurs.

---

## Lab 4

Replace:

```c
pthread_mutex_lock()
```

with:

```c
pthread_mutex_trylock()
```

Observe non-blocking behavior.

---

## Lab 5

Create a recursive mutex.

Lock it multiple times from the same thread.

---

# Interview Questions

### What is a mutex?

A synchronization object that provides mutual exclusion for shared resources.

---

### What is the difference between a mutex and a semaphore?

A mutex has ownership and is intended for mutual exclusion.

A semaphore is a counting synchronization primitive.

---

### What is a deadlock?

A situation where two or more threads wait indefinitely for each other to release resources.

---

### Why should critical sections be small?

To reduce contention and improve concurrency.

---

### What is a recursive mutex?

A mutex that allows the owning thread to acquire it multiple times before releasing it.

---

# Summary

Race Condition:

```
Multiple Threads

↓

Shared Data

↓

Conflict
```

Mutex Protection:

```
Lock

↓

Critical Section

↓

Unlock
```

Comparison:

| Mutex | Semaphore |
|--------|-----------|
| Ownership | No ownership |
| Mutual exclusion | Resource counting |
| One owner | Counter-based |
| Thread synchronization | Thread/process synchronization |

## Key Takeaways

- Mutexes provide mutual exclusion for shared data.
- Only one thread can own a mutex at a time.
- Mutexes differ from semaphores because they enforce ownership.
- Deadlocks occur when threads wait indefinitely for one another.
- Keeping critical sections short improves application performance.
- Mutexes are one of the most fundamental synchronization tools in multithreaded programming.

---

# Next Chapter

## Chapter 123 — Threads (`pthread_create()`) — Running Multiple Tasks Inside One Process

You'll learn:

- What a thread is
- Process vs thread
- `pthread_create()`
- `pthread_join()`
- Thread lifecycle
- Shared address space
- Thread scheduling
- Why modern servers use thousands of threads