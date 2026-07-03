Most developers think:

> "A process does one thing."

But that's not true.

Open Google Chrome.

Even if you're viewing just one webpage, Chrome is doing many things at the same time:

- Rendering the page
- Downloading images
- Playing audio
- Running JavaScript
- Receiving network packets
- Updating the UI

How can one process do so many things simultaneously?

The answer is **Threads**.

Threads are one of the most important concepts in modern software development.

Java, Go, Node.js, PostgreSQL, Redis, Nginx, Docker, and even the Linux kernel rely heavily on threads.

After this chapter, you'll understand how threads work, why they exist, and why they are much lighter than processes.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 9 — Threads: How One Process Performs Multiple Tasks

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a Thread
> - Why Threads exist
> - Process vs Thread
> - Single-threaded Programs
> - Multi-threaded Programs
> - User Threads
> - Kernel Threads
> - Thread Life Cycle
> - Thread Scheduling
> - Shared Memory
> - Race Conditions
> - Mutex
> - Semaphore
> - Deadlock
> - Thread Pools

---

# 📖 Why Do Threads Exist?

Suppose Chrome only had one thread.

```
Download Image

↓

Run JavaScript

↓

Render Screen

↓

Play Audio

↓

Handle Mouse Click
```

Everything would happen one after another.

The browser would feel slow.

Instead,

Chrome creates multiple threads.

```
Chrome Process

│

├── UI Thread

├── Render Thread

├── JavaScript Thread

├── Network Thread

└── Audio Thread
```

Now many tasks happen simultaneously.

---

# What is a Thread?

A **Thread** is the smallest unit of execution inside a process.

Think of a process as a company.

```
Company

↓

Employees
```

The company is the process.

The employees are the threads.

Each employee performs different work,

but everyone works for the same company.

---

# Process vs Thread

```
Process

↓

Container of Resources
```

```
Thread

↓

Unit of Execution
```

A process owns resources.

Threads use those resources.

---

# Process Structure

```
+---------------------------+

Process

|

|-- Code

|-- Heap

|-- Files

|-- Sockets

|

|-- Thread 1

|-- Thread 2

|-- Thread 3

+---------------------------+
```

The process owns memory.

Threads execute code.

---

# Single-Threaded Process

```
Process

↓

Thread

↓

CPU
```

Only one task executes at a time.

Example:

```
Calculator
```

Simple applications often use one thread.

---

# Multi-Threaded Process

```
Process

│

├── Thread 1

├── Thread 2

├── Thread 3

└── Thread 4
```

Many tasks execute simultaneously.

Example:

```
Chrome

VS Code

Java Application

PostgreSQL
```

---

# What Does a Thread Have?

Every thread has its own:

- Program Counter
- Registers
- Stack

Threads **share**:

- Heap
- Code
- Files
- Sockets
- Process Memory

---

# Shared Memory

Suppose:

```java
int counter = 0;
```

Two threads execute:

```java
counter++;
```

Both access the same variable.

```
Thread A

↓

Heap

↑

Thread B
```

Shared memory is powerful,

but also dangerous.

---

# Process vs Thread Memory

### Processes

```
Process A

Heap A

Stack A
```

```
Process B

Heap B

Stack B
```

Completely isolated.

---

### Threads

```
Process

↓

Shared Heap

↓

Thread A Stack

Thread B Stack

Thread C Stack
```

Only the stacks are separate.

Everything else is shared.

---

# Why Threads Are Faster

Creating a process requires:

- New memory
- New PCB
- New page tables
- New resources

Creating a thread requires only:

- New stack
- Registers
- Thread information

Therefore,

threads are much lighter.

---

# Thread Life Cycle

```
New

↓

Ready

↓

Running

↓

Waiting

↓

Running

↓

Terminated
```

Very similar to processes.

---

# User Threads

User threads are managed by a runtime library.

Examples:

- Older Java runtimes
- Green Threads

Advantages:

✔ Fast creation

✔ Fast switching

Disadvantages:

❌ Kernel cannot schedule them directly.

---

# Kernel Threads

Kernel threads are managed by the Operating System.

Linux schedules kernel threads directly.

Modern Java,

Go,

C++,

Rust,

PostgreSQL,

and many other applications use kernel threads.

---

# Thread Scheduling

The Operating System scheduler decides:

```
Thread A

↓

CPU
```

Then:

```
Thread B

↓

CPU
```

Threads compete for CPU time,

just like processes.

---

# Context Switching Between Threads

Suppose:

```
Thread A

↓

Thread B
```

The Operating System saves:

- Registers
- Program Counter
- Stack Pointer

Then loads another thread.

Since threads share memory,

thread switching is usually cheaper than process switching.

---

# Real Example

Chrome:

```
Chrome Process

↓

Network Thread

↓

Receives Data
```

At the same time:

```
Render Thread

↓

Draws Screen
```

At the same time:

```
JavaScript Thread

↓

Executes JavaScript
```

Multiple threads improve responsiveness.

---

# Problem — Race Condition

Suppose:

```java
counter++;
```

Two threads execute simultaneously.

Thread A:

```
Read Counter = 5
```

Thread B:

```
Read Counter = 5
```

Both increment.

Both write:

```
6
```

Expected:

```
7
```

Actual:

```
6
```

This is called a **Race Condition**.

---

# Why Race Conditions Happen

Because threads share memory.

```
Thread A

↓

Shared Variable

↑

Thread B
```

Without coordination,

unexpected results occur.

---

# Mutex

A Mutex allows only one thread to access a resource at a time.

```
Thread A

↓

Lock

↓

Critical Section

↓

Unlock
```

Meanwhile:

```
Thread B

↓

Wait
```

Only one thread enters the critical section.

---

# Critical Section

A Critical Section is code that accesses shared resources.

Example:

```java
counter++;
```

This must be protected.

---

# Semaphore

A Semaphore allows a limited number of threads to enter.

Example:

```
Semaphore = 3
```

Three threads may enter simultaneously.

The fourth waits.

Semaphores are useful when multiple resources are available.

---

# Mutex vs Semaphore

| Mutex | Semaphore |
|--------|-----------|
| One Thread | Multiple Threads |
| Binary Lock | Counter Based |
| Protect Shared Data | Control Resource Access |

---

# Deadlock

Imagine:

Thread A:

```
Lock A

↓

Wait for B
```

Thread B:

```
Lock B

↓

Wait for A
```

Neither proceeds.

```
Thread A

↓

Waiting

↑

Thread B
```

This is a **Deadlock**.

---

# Thread Pool

Creating threads repeatedly is expensive.

Instead,

applications create a pool.

```
Thread Pool

├── Thread 1

├── Thread 2

├── Thread 3

└── Thread 4
```

Tasks reuse existing threads.

This improves performance.

---

# Real Example

Java Server:

```
Incoming Request

↓

Thread Pool

↓

Worker Thread

↓

Database

↓

Response
```

Instead of creating thousands of new threads,

existing threads are reused.

---

# Why Threads Matter

Modern software uses threads everywhere.

Examples:

- Java ExecutorService
- Go Goroutines (mapped to OS threads)
- PostgreSQL Workers
- Docker Engine
- Chrome Browser
- VS Code
- Linux Kernel

Without threads,

modern applications would be much slower.

---

# Hands-on Lab

## Lab 1 — View Threads

Linux:

```bash
ps -eLf
```

Observe multiple threads belonging to one process.

---

## Lab 2 — Monitor Threads

```bash
top -H
```

Observe thread-level CPU usage.

---

## Lab 3 — View Thread Count

```bash
ps -o pid,nlwp,cmd
```

`NLWP` means:

```
Number of Lightweight Processes (Threads)
```

---

## Lab 4 — Java Thread Dump

```bash
jstack <PID>
```

Observe hundreds of Java threads in a running application.

---

# Interview Questions

## What is a Thread?

A Thread is the smallest unit of execution inside a process.

---

## What is the difference between a Process and a Thread?

A Process owns resources such as memory and files.

Threads execute within a process and share those resources.

---

## Why are Threads lighter than Processes?

Threads share the same address space, code, heap, and resources, so creating and switching between them requires much less work.

---

## What is a Race Condition?

A Race Condition occurs when multiple threads access shared data simultaneously without proper synchronization.

---

## What is a Mutex?

A Mutex is a synchronization mechanism that allows only one thread to enter a critical section at a time.

---

## What is a Deadlock?

A Deadlock occurs when two or more threads wait indefinitely for each other to release resources.

---

## Why are Thread Pools used?

Thread Pools reduce the overhead of repeatedly creating and destroying threads by reusing existing worker threads.

---

# Summary

Threads allow one process to perform multiple tasks concurrently.

```
Process

↓

Shared Resources

↓

Thread 1

Thread 2

Thread 3

↓

CPU Scheduler

↓

Execution
```

Key concepts:

- A Thread is the smallest unit of execution.
- Processes own resources; threads share them.
- Threads have separate stacks but share the heap and code.
- Threads are faster to create than processes.
- Shared memory enables efficient communication but introduces race conditions.
- Mutexes and Semaphores synchronize thread access.
- Deadlocks occur when threads wait forever for each other.
- Thread Pools improve performance by reusing worker threads.

Understanding threads is essential before learning CPU scheduling, synchronization, Linux kernel scheduling, high-performance servers, Docker, and Kubernetes.

---

# Next Chapter

## Chapter 10 — CPU Scheduling: How Linux Decides Which Process Runs Next

We'll explore:

- Why scheduling is needed
- Scheduler goals
- Ready Queue
- Dispatcher
- Context Switching
- FCFS
- SJF
- Round Robin
- Priority Scheduling
- Completely Fair Scheduler (CFS)
- Real-Time Scheduling
- CPU Starvation
- CPU Affinity