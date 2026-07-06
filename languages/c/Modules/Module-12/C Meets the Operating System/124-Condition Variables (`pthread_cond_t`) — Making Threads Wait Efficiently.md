# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 124 — Condition Variables (`pthread_cond_t`) — Making Threads Wait Efficiently

---

# Most beginners think:

> "If a thread needs to wait, it should keep checking in a loop."

Example:

```c
while(data_ready == 0)
{
    // keep checking
}
```

Looks simple.

But this is actually a terrible solution.

Why?

Because the thread continuously consumes CPU while doing absolutely nothing useful.

This is called:

```
Busy Waiting

(or)

Busy Spinning
```

Linux provides a much better mechanism:

```
Condition Variables
```

---

# Learning Objectives

After this chapter you will understand:

- Why mutexes are not enough
- Busy waiting
- Condition variables
- `pthread_cond_t`
- `pthread_cond_wait()`
- `pthread_cond_signal()`
- `pthread_cond_broadcast()`
- Producer-consumer synchronization

---

# The Problem

Suppose:

Thread A produces data.

```
Thread A

↓

Generate Data

↓

Buffer
```

Thread B:

```
Needs Data
```

Without condition variables:

```c
while(data_ready == 0)
{
}
```

CPU usage:

```
100%
```

Just waiting.

---

# Think About a Restaurant

Customer:

```
Waiting For Food
```

Bad approach:

```
Every Second

↓

"Is My Food Ready?"

↓

Again

↓

Again

↓

Again
```

Good approach:

```
Sit

↓

Wait

↓

Waiter Calls You

↓

Eat
```

Condition variables work like the waiter.

---

# Busy Waiting

Visualization:

```
Check

↓

No

↓

Check

↓

No

↓

Check

↓

No

↓

Forever
```

CPU keeps working unnecessarily.

---

# Efficient Waiting

Instead:

```
Sleep

↓

Producer Signals

↓

Wake Up

↓

Continue
```

No wasted CPU cycles.

---

# What is a Condition Variable?

Definition:

> **A synchronization object that allows threads to sleep until a specific condition becomes true.**

It is always used **together with a mutex**.

---

# Why With a Mutex?

Condition variables protect:

```
State

↓

Shared Variable
```

The mutex ensures the shared state is examined and modified safely.

---

# POSIX Condition Variables

Include:

```c
#include <pthread.h>
```

Type:

```c
pthread_cond_t
```

---

# Initialization

```c
pthread_cond_t cond;

pthread_cond_init(
&cond,
NULL);
```

Now the condition variable is ready.

---

# Waiting

Example:

```c
pthread_cond_wait(
&cond,
&lock);
```

Something very important happens here.

---

# What `pthread_cond_wait()` Does

It performs these operations atomically:

```
Unlock Mutex

↓

Sleep

↓

Signal Arrives

↓

Wake Up

↓

Lock Mutex Again

↓

Return
```

This avoids race conditions between sleeping and signaling.

---

# Visualization

```
Thread

↓

Lock

↓

Wait

↓

Mutex Released

↓

Sleeping

↓

Signal

↓

Wake

↓

Mutex Reacquired
```

---

# Signaling

Producer:

```c
pthread_cond_signal(
&cond);
```

Meaning:

```
Wake

↓

One Waiting Thread
```

---

# Broadcasting

Instead:

```c
pthread_cond_broadcast(
&cond);
```

Meaning:

```
Wake

↓

All Waiting Threads
```

---

# Complete Flow

Consumer:

```
Lock

↓

Condition False?

↓

Wait

↓

Wake

↓

Continue
```

Producer:

```
Lock

↓

Update Data

↓

Signal

↓

Unlock
```

---

# Example

Shared variable:

```c
int ready = 0;
```

Consumer:

```c
pthread_mutex_lock(&lock);

while(!ready)
{
    pthread_cond_wait(
        &cond,
        &lock);
}

pthread_mutex_unlock(&lock);
```

Producer:

```c
pthread_mutex_lock(&lock);

ready = 1;

pthread_cond_signal(&cond);

pthread_mutex_unlock(&lock);
```

---

# Why Use `while` Instead of `if`?

Many beginners write:

```c
if(!ready)
{
    pthread_cond_wait(...);
}
```

Wrong.

Correct:

```c
while(!ready)
{
    pthread_cond_wait(...);
}
```

Because:

- Multiple threads may wake
- Conditions may change before the thread runs again
- Spurious wakeups are possible on POSIX systems

Always recheck the condition.

---

# Producer-Consumer Problem

Producer:

```
Create Item

↓

Buffer

↓

Signal
```

Consumer:

```
Wait

↓

Wake

↓

Read Item
```

Efficient.

No busy waiting.

---

# Multiple Consumers

Suppose:

```
Consumer A

↓

Waiting

-------------------

Consumer B

↓

Waiting

-------------------

Consumer C

↓

Waiting
```

Producer:

```
Broadcast
```

All consumers wake and recheck the condition.

---

# Condition Variable Lifetime

Initialize:

```c
pthread_cond_init()
```

Destroy:

```c
pthread_cond_destroy()
```

Release resources when no longer needed.

---

# Real-World Example — Web Server

Worker threads:

```
Request Queue Empty

↓

Wait

↓

New Request

↓

Signal

↓

Worker Processes Request
```

No CPU wasted polling the queue.

---

# Real-World Example — Database

Worker:

```
No Jobs

↓

Sleep

↓

Transaction Arrives

↓

Wake
```

---

# Real-World Example — Thread Pool

Main thread:

```
Task Queue

↓

Signal

↓

Worker Thread
```

Workers sleep until work arrives.

---

# Real-World Example — Media Player

Decoder:

```
Frame Ready

↓

Signal

↓

Renderer Wakes

↓

Display Frame
```

---

# Common Mistakes

---

## Using `if` Instead of `while`

Always recheck the condition after waking.

---

## Forgetting the Mutex

Condition variables must be used with the associated mutex protecting the shared state.

---

## Busy Waiting

Avoid:

```c
while(flag == 0)
{
}
```

Use:

```c
pthread_cond_wait()
```

instead.

---

## Signaling Without Updating State

Always update the shared condition **before** signaling waiting threads.

---

# Complete Example

```c
pthread_mutex_lock(&lock);

while(!ready)
{
    pthread_cond_wait(
        &cond,
        &lock);
}

/* Use shared data */

pthread_mutex_unlock(&lock);
```

Producer:

```c
pthread_mutex_lock(&lock);

ready = 1;

pthread_cond_signal(&cond);

pthread_mutex_unlock(&lock);
```

---

# Hands-on Labs

## Lab 1

Create one producer and one consumer using a condition variable.

---

## Lab 2

Replace busy waiting with:

```c
pthread_cond_wait()
```

Compare CPU usage.

---

## Lab 3

Create three consumer threads.

Use:

```c
pthread_cond_broadcast()
```

Observe all threads waking.

---

## Lab 4

Replace:

```c
while
```

with:

```c
if
```

Experiment and observe why `while` is the correct pattern.

---

## Lab 5

Implement a bounded producer-consumer queue using:

- Mutex
- Condition variable

---

# Interview Questions

### What is a condition variable?

A synchronization object that allows threads to sleep until a specified condition becomes true.

---

### Why are condition variables used?

To avoid busy waiting and efficiently synchronize threads.

---

### Why is a mutex required?

To safely protect and examine the shared condition associated with the condition variable.

---

### What is the difference between `pthread_cond_signal()` and `pthread_cond_broadcast()`?

`pthread_cond_signal()` wakes one waiting thread.

`pthread_cond_broadcast()` wakes all waiting threads.

---

### Why should `pthread_cond_wait()` usually be placed inside a `while` loop?

Because the condition must be rechecked after waking due to possible spurious wakeups or changes made by other threads.

---

# Summary

Busy Waiting:

```
Check

↓

Check

↓

Check

↓

Waste CPU
```

Condition Variable:

```
Sleep

↓

Signal

↓

Wake

↓

Continue
```

Synchronization Pattern:

```
Mutex

↓

Condition Variable

↓

Shared State

↓

Thread Coordination
```

## Key Takeaways

- Condition variables allow threads to wait efficiently without consuming CPU.
- They are always used together with a mutex.
- `pthread_cond_wait()` releases the mutex while sleeping and reacquires it before returning.
- `pthread_cond_signal()` wakes one waiting thread.
- `pthread_cond_broadcast()` wakes all waiting threads.
- Always wait in a `while` loop, not an `if` statement.

---

# Next Chapter

## Chapter 125 — Read-Write Locks (`pthread_rwlock_t`) — Many Readers, One Writer

You'll learn:

- Why mutexes sometimes limit performance
- Read-write locks
- `pthread_rwlock_rdlock()`
- `pthread_rwlock_wrlock()`
- Reader vs writer priority
- Starvation
- Real-world database and cache examples