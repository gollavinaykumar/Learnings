# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 120 — Shared Memory (`shm_open()`, `mmap()`) — The Fastest IPC Mechanism

---

# Most beginners think:

> "If two processes want to exchange data, they must use pipes or sockets."

That works.

But there is a problem.

Suppose Process A wants to send **1 GB** of data.

With a pipe:

```
Process A

↓

Kernel Buffer

↓

Process B
```

The data is copied multiple times.

For very large data,

this becomes expensive.

Linux provides a much faster mechanism:

```
Shared Memory
```

---

# Learning Objectives

After this chapter you will understand:

- What shared memory is
- Why it is the fastest IPC mechanism
- `shm_open()`
- `mmap()`
- Shared memory regions
- Synchronization
- Memory mapping
- Real-world usage

---

# The Problem

Suppose:

```
Video Process

↓

1 GB Frame

↓

Another Process
```

Using:

```
Pipe

↓

Copy

↓

Copy Again
```

Very expensive.

Instead:

```
Both Processes

↓

Same Memory
```

No unnecessary copies.

---

# Think About a Whiteboard

Imagine two people.

Instead of:

```
Person A

↓

Paper

↓

Person B
```

They both write on:

```
One Whiteboard
```

No copying.

Shared memory works the same way.

---

# What is Shared Memory?

Shared memory is:

> **A memory region simultaneously mapped into the virtual address spaces of multiple processes.**

Visualization:

```
Process A

↓

Virtual Address

↓

Shared Physical Memory

↑

Virtual Address

↓

Process B
```

Both processes access the same bytes.

---

# Compare with Pipes

Pipe:

```
Process A

↓

Copy

↓

Kernel

↓

Copy

↓

Process B
```

Shared Memory:

```
Process A

↓

Shared Memory

↑

Process B
```

No intermediate data copying during normal access.

---

# Why Is It Fast?

Once mapped:

```
Read

↓

RAM

↓

Done
```

No repeated kernel copying for each read or write operation.

---

# Creating Shared Memory

POSIX provides:

```c
shm_open()
```

Prototype:

```c
int shm_open(
const char *name,
int oflag,
mode_t mode);
```

Creates or opens a shared memory object.

---

# Example

```c
int fd =
shm_open(
"/demo",
O_CREAT | O_RDWR,
0666);
```

Now Linux creates:

```
Shared Memory Object
```

identified by:

```
/demo
```

---

# Setting the Size

New shared memory objects start with size:

```
0 Bytes
```

Use:

```c
ftruncate(
fd,
4096);
```

Now:

```
Shared Memory

↓

4096 Bytes
```

---

# Mapping Memory

Next:

```c
mmap()
```

Prototype:

```c
void *mmap(...);
```

Purpose:

```
Map

↓

Shared Memory

↓

Virtual Address
```

Now the process can access it like ordinary memory.

---

# Visualization

```
Process

↓

Pointer

↓

Shared Memory

↓

RAM
```

From this point onward,

it looks like a normal pointer.

---

# Example

```c
char *ptr =
mmap(...);

ptr[0] = 'A';
```

No:

```
write()

↓

read()
```

Needed.

Just ordinary memory access.

---

# Second Process

Another process:

```c
mmap(...)
```

same object:

```
/demo
```

Result:

```
Process A

↓

Shared RAM

↑

Process B
```

If Process A writes:

```
Hello
```

Process B immediately sees:

```
Hello
```

---

# Synchronization Problem

Suppose:

Process A:

```
Writes
```

Process B:

```
Writes
```

At exactly the same time.

Possible result:

```
Corrupted Data
```

---

# Why?

Both processes access:

```
Same Bytes
```

Without coordination,

their operations may interfere.

---

# Solution

Shared memory is usually combined with:

```
Mutexes

↓

Semaphores

↓

Atomic Operations
```

Synchronization prevents data races.

---

# Shared Memory vs Pipe

Pipe:

```
Communication

↓

Sequential

↓

Temporary
```

Shared Memory:

```
Shared Data

↓

Random Access

↓

Very Fast
```

---

# Shared Memory vs Socket

Socket:

```
Serialize

↓

Copy

↓

Deserialize
```

Shared Memory:

```
Direct RAM Access
```

Much faster for local communication.

---

# Real-World Example — Web Browser

Browser:

```
GPU Process

↓

Shared Memory

↓

Renderer
```

Large image buffers are often exchanged through shared memory.

---

# Real-World Example — Database

Database:

```
Buffer Cache

↓

Shared Memory

↓

Worker Processes
```

Multiple workers efficiently access cached pages.

---

# Real-World Example — Video Streaming

Decoder:

```
Frame

↓

Shared Memory

↓

Renderer
```

Avoids repeatedly copying huge video frames.

---

# Real-World Example — Machine Learning

Inference Process:

```
Tensor

↓

Shared Memory

↓

Serving Process
```

Large tensors can be shared efficiently between cooperating processes.

---

# Memory Lifetime

Shared memory continues to exist until it is explicitly removed.

Cleanup:

```c
shm_unlink("/demo");
```

This removes the shared memory object.

---

# Common Mistakes

---

## Forgetting Synchronization

Shared memory does **not** provide automatic locking.

---

## Forgetting `munmap()`

Mapped memory should be released using:

```c
munmap()
```

when it is no longer needed.

---

## Forgetting `shm_unlink()`

Unused shared memory objects may remain until explicitly removed.

---

## Thinking Shared Memory Copies Data

The whole purpose is:

```
No Extra Copies
```

after the mapping has been established.

---

# Complete Flow

```
shm_open()

↓

ftruncate()

↓

mmap()

↓

Read

↓

Write

↓

munmap()

↓

close()

↓

shm_unlink()
```

---

# Hands-on Labs

## Lab 1

Create a shared memory object using:

```c
shm_open()
```

---

## Lab 2

Map it using:

```c
mmap()
```

Write a string into the shared memory.

---

## Lab 3

Create a second process.

Map the same shared memory object.

Read the string.

---

## Lab 4

Experiment with concurrent writes.

Observe why synchronization is necessary.

---

## Lab 5

Delete the shared memory object using:

```c
shm_unlink()
```

---

# Interview Questions

### What is shared memory?

A memory region mapped into multiple processes so they can directly access the same data.

---

### Why is shared memory faster than pipes?

Because data is accessed directly in RAM instead of being copied through kernel buffers for every transfer.

---

### Which function creates a POSIX shared memory object?

```c
shm_open()
```

---

### What does `mmap()` do?

Maps a file or shared memory object into a process's virtual address space.

---

### Why are mutexes or semaphores needed with shared memory?

To synchronize concurrent access and prevent race conditions.

---

# Summary

Shared Memory Workflow:

```
shm_open()

↓

Shared Object

↓

mmap()

↓

Process A

↑

Process B

↓

Shared RAM
```

Comparison:

| IPC Mechanism | Data Copies | Speed |
|---------------|------------|--------|
| Pipe | Multiple | Medium |
| Socket | Multiple | Medium |
| FIFO | Multiple | Medium |
| Shared Memory | Minimal after mapping | Very High |

## Key Takeaways

- Shared memory is one of the fastest IPC mechanisms available.
- `shm_open()` creates or opens a shared memory object.
- `mmap()` maps the shared memory into a process's address space.
- Multiple processes can access the same physical memory simultaneously.
- Synchronization primitives are essential to avoid race conditions.
- Shared memory is widely used in databases, browsers, multimedia systems, and high-performance applications.

---

# 🎉 IPC Fundamentals Complete!

You now understand the major Unix IPC mechanisms:

- ✅ Signals
- ✅ Anonymous Pipes
- ✅ Named Pipes (FIFOs)
- ✅ Unix Domain Sockets
- ✅ Shared Memory

These are the core communication techniques used throughout Unix and Linux systems.

---

# Next Chapter

## Chapter 121 — Semaphores (`sem_open()`, `sem_wait()`, `sem_post()`) — Synchronizing Multiple Processes

You'll learn:

- What semaphores are
- Binary vs counting semaphores
- Race conditions
- Critical sections
- `sem_open()`
- `sem_wait()`
- `sem_post()`
- Producer-consumer synchronization
- Why synchronization is essential with shared memory