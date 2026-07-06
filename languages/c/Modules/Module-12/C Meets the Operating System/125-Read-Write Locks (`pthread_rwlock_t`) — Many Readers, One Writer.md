# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 125 — Read-Write Locks (`pthread_rwlock_t`) — Many Readers, One Writer

---

# Most beginners think:

> "A mutex is always the best synchronization primitive."

A mutex works well,

but imagine this situation:

```
100 Threads

↓

Read Data
```

Only:

```
1 Thread

↓

Updates Data
```

Should all 100 readers wait for one another?

No.

Reading doesn't change the data.

Linux provides a better solution:

```
Read-Write Lock
```

---

# Learning Objectives

After this chapter you will understand:

- Why mutexes can reduce performance
- Read-write locks
- Readers
- Writers
- `pthread_rwlock_t`
- Reader lock
- Writer lock
- Starvation
- Real-world usage

---

# The Problem

Suppose:

```
Cache

↓

100 Readers

↓

1 Writer
```

With a mutex:

```
Reader A

↓

Lock

↓

Read

↓

Unlock

↓

Reader B

↓

Lock

↓

Read
```

Only one reader at a time.

Very inefficient.

---

# Observation

Multiple readers can safely access:

```
Read Only Data
```

at the same time.

Conflict only happens when:

```
Writer

↓

Modifies Data
```

---

# What is a Read-Write Lock?

Definition:

> **A synchronization object that allows multiple concurrent readers or one exclusive writer.**

Rules:

```
Readers

↓

Many Allowed

---------------------

Writer

↓

Only One

↓

Exclusive Access
```

---

# Visualization

Readers:

```
Reader A

↓

Shared Lock

--------------------

Reader B

↓

Shared Lock

--------------------

Reader C

↓

Shared Lock
```

All run simultaneously.

---

# Writer

```
Writer

↓

Exclusive Lock

↓

Readers Wait

↓

Other Writers Wait
```

Only one writer may proceed.

---

# POSIX Read-Write Locks

Include:

```c
#include <pthread.h>
```

Type:

```c
pthread_rwlock_t
```

---

# Initialization

```c
pthread_rwlock_t rwlock;

pthread_rwlock_init(
&rwlock,
NULL);
```

Now the lock is ready.

---

# Reader Lock

Acquire:

```c
pthread_rwlock_rdlock(
&rwlock);
```

Meaning:

```
Read Access
```

Multiple readers may hold the lock simultaneously.

---

# Visualization

```
Reader A

↓

Read Lock

--------------------

Reader B

↓

Read Lock

--------------------

Reader C

↓

Read Lock
```

No blocking between readers.

---

# Writer Lock

Acquire:

```c
pthread_rwlock_wrlock(
&rwlock);
```

Meaning:

```
Exclusive Access
```

All readers and writers must wait.

---

# Unlock

Both readers and writers release the lock using:

```c
pthread_rwlock_unlock(
&rwlock);
```

---

# Complete Flow

Reader:

```
Read Lock

↓

Read

↓

Unlock
```

Writer:

```
Write Lock

↓

Modify

↓

Unlock
```

---

# Example

Reader:

```c
pthread_rwlock_rdlock(
&rwlock);

printf("%d\n", value);

pthread_rwlock_unlock(
&rwlock);
```

Writer:

```c
pthread_rwlock_wrlock(
&rwlock);

value++;

pthread_rwlock_unlock(
&rwlock);
```

---

# Reader Priority

Suppose:

```
Reader

↓

Reader

↓

Reader

↓

Writer Waiting
```

If new readers continue arriving,

the writer may wait for a long time.

This is called:

```
Writer Starvation
```

---

# Writer Priority

Some implementations favor writers.

Example:

```
Reader

↓

Writer Arrives

↓

New Readers Wait

↓

Writer Runs
```

This reduces writer starvation,

though implementation details vary.

---

# Reader Starvation

The opposite is also possible.

If writers are constantly favored,

readers may wait excessively.

Different operating systems and libraries choose different policies.

---

# Comparison

Mutex:

```
One Thread

↓

Critical Section
```

Read-Write Lock:

```
Many Readers

↓

One Writer
```

---

# Performance

Read-heavy workloads:

```
Mutex

↓

Slow

--------------------

Read-Write Lock

↓

Fast
```

Because readers execute concurrently.

---

# When Not to Use Read-Write Locks

Suppose:

```
50 Readers

↓

50 Writers
```

There is little advantage.

Frequent writes reduce the benefit.

Sometimes a mutex is simpler and just as effective.

---

# Destroy

When finished:

```c
pthread_rwlock_destroy(
&rwlock);
```

Release associated resources.

---

# Real-World Example — Database

Many clients execute:

```sql
SELECT
```

Few clients execute:

```sql
UPDATE
```

Read-write locks allow many concurrent readers while protecting updates.

---

# Real-World Example — Cache

Application cache:

```
Thousands Read

↓

Few Write
```

Excellent fit for read-write locking.

---

# Real-World Example — Routing Table

Network server:

```
Read Route

↓

Many Times

↓

Occasional Update
```

Concurrent reads improve throughput.

---

# Real-World Example — Configuration

Server:

```
Read Config

↓

Thousands of Requests

↓

Admin Updates Once
```

Readers proceed together until an update occurs.

---

# Common Mistakes

---

## Using Write Lock for Every Operation

If you're only reading,

use:

```c
pthread_rwlock_rdlock()
```

---

## Forgetting to Unlock

Just like mutexes,

every successful lock must eventually be unlocked.

---

## Assuming Every Implementation Has the Same Fairness Policy

Reader/writer preference depends on the implementation.

Don't rely on a specific starvation policy unless documented.

---

## Using Read-Write Locks for Write-Heavy Workloads

Frequent writers reduce concurrency benefits.

A mutex may be a better choice.

---

# Complete Example

Reader:

```c
pthread_rwlock_rdlock(
&rwlock);

/* Read Shared Data */

pthread_rwlock_unlock(
&rwlock);
```

Writer:

```c
pthread_rwlock_wrlock(
&rwlock);

/* Modify Shared Data */

pthread_rwlock_unlock(
&rwlock);
```

---

# Hands-on Labs

## Lab 1

Create multiple reader threads.

Observe that they execute concurrently.

---

## Lab 2

Add one writer thread.

Observe that readers pause during writes.

---

## Lab 3

Compare execution time using:

- Mutex
- Read-write lock

on a read-heavy workload.

---

## Lab 4

Create many readers and a single writer.

Observe possible starvation depending on your platform.

---

## Lab 5

Build a thread-safe in-memory cache protected by a read-write lock.

---

# Interview Questions

### What is a read-write lock?

A synchronization object that allows multiple readers or one exclusive writer.

---

### When should read-write locks be used?

When reads greatly outnumber writes.

---

### Can multiple writers hold the lock simultaneously?

No.

Only one writer may hold the lock at a time.

---

### Why are read-write locks faster for read-heavy workloads?

Because multiple readers can access shared data concurrently.

---

### What is writer starvation?

A situation where a writer waits indefinitely because readers continuously acquire the lock.

---

# Summary

Mutex:

```
One Thread

↓

Critical Section
```

Read-Write Lock:

```
Readers

↓

Reader A

↓

Reader B

↓

Reader C

↓

Concurrent

----------------------

Writer

↓

Exclusive
```

Comparison:

| Mutex | Read-Write Lock |
|--------|-----------------|
| One owner | Many readers or one writer |
| Simple | Optimized for read-heavy workloads |
| Good general choice | Best when reads dominate |

## Key Takeaways

- Read-write locks improve concurrency when shared data is read frequently and written infrequently.
- Multiple readers may hold the lock simultaneously.
- Writers require exclusive access.
- Reader and writer starvation are important design considerations.
- Read-write locks are widely used in databases, caches, routing tables, and configuration systems.
- Choosing the correct synchronization primitive depends on the workload, not just correctness.

---

# Next Chapter

## Chapter 126 — Thread Pools — How High-Performance Servers Handle Millions of Requests

You'll learn:

- Why creating a thread per request is expensive
- What a thread pool is
- Worker threads
- Task queues
- Producer-consumer design
- Fixed vs dynamic thread pools
- How Java, Go, Nginx, and web servers efficiently process massive workloads