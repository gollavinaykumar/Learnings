# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 158 — io_uring — The Future of High-Performance Asynchronous I/O in Linux

---

# Imagine you're building:

- Nginx
- PostgreSQL
- Redis
- Cloud Storage
- Object Storage
- High-speed File Server

Thousands of clients connect simultaneously.

Every client performs:

```
read()

↓

write()

↓

read()

↓

write()
```

Traditional Linux handles this well.

But every operation still requires:

```
User Space

↓

System Call

↓

Kernel

↓

Return
```

Millions of system calls per second become expensive.

Linux engineers asked:

> **Can we perform I/O with fewer system calls, fewer context switches, and higher throughput?**

The answer became:

```
io_uring
```

Today it is one of the most important Linux performance technologies.

---

# Learning Objectives

After this chapter you will understand:

- Why `io_uring` exists
- Asynchronous I/O
- Shared Ring Buffers
- Submission Queue
- Completion Queue
- SQEs
- CQEs
- Kernel workers
- Zero-copy concepts
- Modern server architectures

---

# The Problem

Suppose your application does:

```c
read(fd, buf, 4096);
```

Flow:

```
Application

↓

System Call

↓

Kernel

↓

Disk

↓

Kernel

↓

Return
```

Every request crosses:

```
User

↓

Kernel
```

Multiple times.

---

# Another Problem

Imagine:

```
100,000

↓

Concurrent Reads
```

Traditional blocking I/O struggles.

Even non-blocking I/O still requires many system calls and event notifications.

---

# Linux Solution

Instead of:

```
System Call

↓

Every Operation
```

Linux shares memory between:

```
Kernel

↓

Application
```

using:

```
Ring Buffers
```

---

# What is `io_uring`?

Definition:

> **`io_uring` is a Linux asynchronous I/O interface that uses shared ring buffers between user space and the kernel to reduce system-call overhead and improve I/O performance.**

Think of it as:

```
Shared Work Queue

↓

Between

↓

Application

↓

Kernel
```

---

# Why the Name?

```
I/O

+

Ring Buffer

↓

io_uring
```

Everything revolves around circular queues.

---

# Traditional I/O

```
Application

↓

read()

↓

Kernel

↓

Return
```

Every request:

```
System Call
```

---

# `io_uring`

```
Application

↓

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

# The Big Idea

Instead of repeatedly saying:

```
Read This

↓

Wait

↓

Read That

↓

Wait
```

Application says:

```
Here Are

↓

1000 Jobs
```

Kernel processes them.

Application later collects completed results.

---

# Shared Memory

Kernel and application share:

```
Submission Ring

↓

Completion Ring
```

No repeated allocation.

Minimal copying.

---

# Visualization

```
Application

↓

Shared Rings

↑

Kernel
```

---

# Submission Queue (SQ)

Application places work requests here.

Example:

```
Read File

↓

Write Socket

↓

Accept Client

↓

Open File
```

Kernel later consumes them.

---

# Visualization

```
Application

↓

SQ

↓

Kernel
```

---

# Submission Queue Entry (SQE)

Each request becomes:

```
SQE
```

Example:

```
Operation

↓

READ

↓

File Descriptor

↓

Buffer

↓

Length
```

One SQE describes one operation.

---

# Completion Queue (CQ)

Kernel finishes work.

Places result into:

```
Completion Queue
```

Application reads completed entries.

---

# Completion Queue Entry (CQE)

Contains:

```
Result

↓

Status

↓

User Data
```

Application knows:

```
Which Request

↓

Completed
```

---

# Visualization

```
Kernel

↓

CQ

↓

Application
```

---

# Workflow

```
Create SQE

↓

Submit

↓

Kernel Executes

↓

CQE Produced

↓

Application Reads Result
```

---

# Example

Application submits:

```
Read File A

Read File B

Read File C
```

Kernel processes them asynchronously.

Application later reads:

```
Completed A

Completed B

Completed C
```

---

# Blocking vs Asynchronous

Blocking:

```
Read

↓

Wait

↓

Next Read
```

Asynchronous:

```
Read A

Read B

Read C

↓

Continue Working

↓

Collect Results
```

Much higher concurrency.

---

# Batching

Traditional:

```
100 Reads

↓

100 System Calls
```

`io_uring`:

```
100 Reads

↓

Batch Submission

↓

Much Fewer System Calls
```

Lower overhead.

---

# Kernel Workers

Some operations cannot complete immediately.

Kernel may schedule helper workers internally.

Application continues running.

Completion arrives later.

---

# Relationship with Ring Buffers

Remember AF_XDP?

```
RX Ring

↓

TX Ring
```

`io_uring` also uses ring buffers,

but for general I/O operations instead of networking.

---

# Relationship with `epoll`

Traditionally:

```
Socket

↓

epoll()

↓

read()
```

Modern applications may combine:

```
epoll

↓

io_uring
```

or in some cases perform many asynchronous operations directly through `io_uring`, depending on workload and kernel support.

---

# Relationship with `read()`

Traditional:

```c
read()
```

becomes:

```
SQE

↓

Kernel

↓

CQE
```

Same goal.

Different execution model.

---

# Relationship with `write()`

Instead of:

```
write()

↓

Wait
```

Application submits:

```
Write Request

↓

Continue

↓

Completion Later
```

---

# Relationship with Files

Supports operations such as:

- Read
- Write
- Open
- Close
- Sync

depending on kernel capabilities.

---

# Relationship with Networking

Modern kernels support many networking operations through `io_uring`, reducing overhead for high-performance servers.

---

# Relationship with Zero-Copy

`io_uring` includes support for several optimizations,

including zero-copy features for some operations and kernel versions.

Availability depends on the operation and Linux release.

---

# Real-World Example — Database

Database submits:

```
100 Reads

↓

Continue Query Planning

↓

Results Arrive
```

Improved throughput.

---

# Real-World Example — Web Server

Thousands of clients.

Requests submitted asynchronously.

Workers spend less time blocked waiting for I/O.

---

# Real-World Example — Object Storage

Many files:

```
Read

↓

Compress

↓

Write

↓

Upload
```

Pipeline becomes more efficient.

---

# Real-World Example — Logging System

Many writes:

```
SQ

↓

Kernel

↓

CQ
```

Reduced syscall overhead.

---

# Advantages

Compared with traditional I/O:

```
Fewer System Calls

↓

Fewer Context Switches

↓

Batch Processing

↓

Higher Throughput

↓

Lower Latency
```

---

# Common Mistakes

---

## Thinking `io_uring` Eliminates the Kernel

It still relies on the Linux kernel.

It simply changes how requests are submitted and completed.

---

## Assuming Every Operation Is Faster

Performance gains depend on workload, storage, networking, and kernel support.

---

## Confusing `io_uring` with `epoll`

`epoll` waits for readiness.

`io_uring` submits and completes asynchronous operations.

---

## Forgetting Shared Rings

Shared ring buffers are the central design of `io_uring`.

---

# Hands-on Labs

## Lab 1

Read:

```bash
man 7 io_uring
```

Study the architecture.

---

## Lab 2

Install:

```bash
liburing
```

Explore example programs.

---

## Lab 3

Inspect a simple asynchronous file read using `io_uring`.

---

## Lab 4

Compare:

- Blocking I/O
- `epoll`
- `io_uring`

Understand their execution models.

---

## Lab 5

Research which operations your Linux kernel supports through `io_uring`.

---

# Interview Questions

### What is `io_uring`?

A Linux asynchronous I/O interface using shared ring buffers between user space and the kernel.

---

### Why is `io_uring` faster than traditional I/O?

It reduces system-call overhead, context switches, and supports efficient batching of operations.

---

### What is an SQE?

A Submission Queue Entry describing one I/O request.

---

### What is a CQE?

A Completion Queue Entry describing the result of a completed operation.

---

### Does `io_uring` replace the Linux kernel?

No.

It provides a more efficient interface for interacting with the kernel.

---

# Summary

Architecture:

```
Application

↓

Submission Queue

↓

Kernel

↓

Completion Queue

↓

Application
```

Workflow:

```
Create SQE

↓

Submit

↓

Kernel Executes

↓

CQE Produced

↓

Application Reads Result
```

Major Components:

| Component | Purpose |
|-----------|---------|
| Submission Queue (SQ) | Send requests to the kernel |
| SQE | One submitted operation |
| Completion Queue (CQ) | Receive completed operations |
| CQE | One completed result |

Comparison:

| Traditional I/O | `io_uring` |
|-----------------|------------|
| One syscall per operation | Batch submissions possible |
| Frequent context switches | Fewer context switches |
| Blocking or readiness-based | Native asynchronous execution |
| Higher syscall overhead | Lower syscall overhead |

## Key Takeaways

- `io_uring` is the modern Linux interface for high-performance asynchronous I/O.
- It uses shared submission and completion ring buffers.
- SQEs describe requests; CQEs describe completed operations.
- Batching reduces syscall overhead and improves scalability.
- Modern databases, storage systems, and web servers increasingly adopt `io_uring`.
- Understanding `io_uring` is essential for mastering modern Linux systems programming.

---

# Next Chapter

## Chapter 159 — The Linux Networking Stack — Following One Packet from the NIC to Your Application

You'll learn:

- Complete packet journey
- NIC
- DMA
- Interrupts
- NAPI
- XDP
- TC
- Netfilter
- Routing
- Socket buffers (`sk_buff`)
- TCP/IP processing
- Socket delivery
- The complete life cycle of a network packet inside the Linux kernel