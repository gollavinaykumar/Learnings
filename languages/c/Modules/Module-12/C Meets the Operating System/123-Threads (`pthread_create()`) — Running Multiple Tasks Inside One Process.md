# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 123 — Threads (`pthread_create()`) — Running Multiple Tasks Inside One Process

---

# Most beginners think:

> "A program can only do one thing at a time."

Example:

```
Download File

↓

Wait

↓

Read Keyboard

↓

Wait

↓

Update UI

↓

Wait
```

Everything happens one after another.

Modern software doesn't work like this.

Think about:

- Chrome
- VS Code
- Docker
- PostgreSQL
- Redis
- Games

They perform **many tasks simultaneously**.

How?

The answer is:

```
Threads
```

---

# Learning Objectives

After this chapter you will understand:

- What a thread is
- Process vs thread
- Why threads exist
- `pthread_create()`
- `pthread_join()`
- Thread lifecycle
- Shared memory
- Thread scheduling

---

# What is a Process?

A process is:

```
Running Program
```

It has:

```
Code

↓

Heap

↓

Stack

↓

Registers

↓

File Descriptors
```

Every process owns its own virtual memory.

---

# What is a Thread?

A thread is:

> **A single path of execution inside a process.**

One process can contain:

```
One Thread

↓

Two Threads

↓

Hundreds

↓

Thousands
```

---

# Visualization

Single Thread

```
Process

↓

Thread
```

Multiple Threads

```
Process

↓

Thread A

↓

Thread B

↓

Thread C
```

One process.

Multiple execution paths.

---

# Think About a Restaurant

Restaurant:

```
Kitchen

↓

One Cook
```

Orders finish one by one.

Now:

```
Kitchen

↓

Five Cooks
```

Orders are prepared simultaneously.

Threads are like those cooks.

---

# Process vs Thread

Process:

```
Own Memory

↓

Own Resources
```

Thread:

```
Shares Process Resources
```

Threads belong to a process.

---

# Shared Resources

All threads share:

```
Code

↓

Heap

↓

Global Variables

↓

Open Files

↓

Sockets
```

---

# Private Resources

Each thread has its own:

```
Stack

↓

Registers

↓

Program Counter
```

This allows each thread to execute independently.

---

# Visualization

```
Process

↓

Code

↓

Heap

↓

Globals

↓

Files

-------------------------

Thread A

↓

Own Stack

-------------------------

Thread B

↓

Own Stack

-------------------------

Thread C

↓

Own Stack
```

---

# Why Threads?

Suppose:

```
Download File

↓

5 Seconds
```

Without threads:

```
Application Freezes
```

With threads:

```
Thread 1

↓

Download

---------------------

Thread 2

↓

UI

↓

Responsive
```

---

# POSIX Threads

Linux provides:

```
pthread
```

Include:

```c
#include <pthread.h>
```

---

# Creating a Thread

Prototype:

```c
pthread_create()
```

Example:

```c
pthread_create(
&tid,
NULL,
worker,
NULL);
```

Now:

```
Main Thread

↓

Worker Thread
```

Both execute simultaneously.

---

# Thread Function

Example:

```c
void *worker(
void *arg)
{
    printf("Hello\n");

    return NULL;
}
```

Every thread begins executing its start routine.

---

# First Example

```c
#include <pthread.h>
#include <stdio.h>

void *worker(void *arg)
{
    printf("Worker\n");
    return NULL;
}

int main()
{
    pthread_t tid;

    pthread_create(
        &tid,
        NULL,
        worker,
        NULL);

    pthread_join(
        tid,
        NULL);

    return 0;
}
```

Output:

```
Worker
```

---

# Why `pthread_join()`?

Suppose:

Main thread exits immediately.

Worker:

```
Still Running
```

Process terminates,

and the worker is destroyed.

Instead:

```
Main

↓

pthread_join()

↓

Wait

↓

Worker Finishes
```

---

# Visualization

```
Main Thread

↓

Create Worker

↓

Wait

↓

Worker Ends

↓

Continue
```

---

# Thread Scheduling

Suppose:

```
Thread A

↓

Running

↓

Scheduler

↓

Thread B

↓

Running
```

The operating system decides which thread executes next.

---

# Shared Memory

Example:

```c
int counter = 0;
```

Thread A:

```
counter++
```

Thread B:

```
counter++
```

Same variable.

Same memory.

This is why mutexes are needed.

---

# Thread Stack

Each thread gets:

```
Own Stack
```

Example:

```c
void worker()
{
    int x = 10;
}
```

Thread A:

```
Own x
```

Thread B:

```
Own x
```

Local variables are **not shared**.

---

# Global Variables

Example:

```c
int total = 0;
```

All threads access:

```
Same total
```

This creates the possibility of race conditions.

---

# Process vs Thread Creation

Process:

```
fork()

↓

Duplicate Address Space
```

Thread:

```
pthread_create()

↓

Same Address Space
```

Creating threads is generally less expensive than creating processes.

---

# Thread Lifecycle

```
Create

↓

Running

↓

Blocked

↓

Running

↓

Finished

↓

Joined
```

---

# Detached Threads

Normally:

```
pthread_join()
```

collects thread completion.

Detached thread:

```
Ends

↓

Automatically Cleaned
```

No join required.

---

# Real-World Example — Chrome

Browser:

```
UI Thread

↓

Renderer Thread

↓

Network Thread

↓

GPU Thread
```

Different tasks execute concurrently.

---

# Real-World Example — Web Server

Server:

```
Thread Pool

↓

Worker

↓

Worker

↓

Worker
```

Each thread handles incoming requests.

---

# Real-World Example — Database

Database:

```
Client Thread

↓

Worker Thread

↓

Checkpoint Thread

↓

Logging Thread
```

Each performs specialized work.

---

# Real-World Example — Game Engine

Separate threads may handle:

```
Rendering

↓

Physics

↓

Audio

↓

Networking
```

Improving responsiveness and performance.

---

# Common Mistakes

---

## Forgetting `pthread_join()`

The main thread may exit before workers finish.

---

## Sharing Local Variables Incorrectly

Each thread has its own stack.

Only globals and heap allocations are shared.

---

## Ignoring Race Conditions

Shared variables require synchronization.

---

## Creating Too Many Threads

Thousands of unnecessary threads increase:

- Memory usage
- Scheduling overhead
- Context switching

Thread pools are often a better solution.

---

# Complete Example

```c
#include <pthread.h>
#include <stdio.h>

void *worker(void *arg)
{
    printf("Hello from worker\n");
    return NULL;
}

int main()
{
    pthread_t tid;

    pthread_create(
        &tid,
        NULL,
        worker,
        NULL);

    pthread_join(
        tid,
        NULL);

    printf("Main finished\n");

    return 0;
}
```

Possible output:

```
Hello from worker

Main finished
```

---

# Hands-on Labs

## Lab 1

Create a single worker thread.

Wait for it using:

```c
pthread_join()
```

---

## Lab 2

Create five worker threads.

Observe execution order.

---

## Lab 3

Share a global counter.

Observe race conditions.

Protect it using a mutex.

---

## Lab 4

Print the addresses of local variables in different threads.

Notice each thread has its own stack.

---

## Lab 5

Create a detached thread.

Observe automatic cleanup after it exits.

---

# Interview Questions

### What is a thread?

A lightweight execution unit that runs within a process and shares its resources.

---

### What resources are shared between threads?

- Code
- Heap
- Global variables
- Open file descriptors
- Sockets

---

### What resources are private to each thread?

- Stack
- Registers
- Program counter

---

### Why is `pthread_join()` used?

To wait for a thread to finish and clean up its resources.

---

### Why are threads generally faster than processes?

Because they share the same address space and require less overhead to create and switch between.

---

# Summary

Process:

```
Own Memory

↓

Own Resources
```

Threads:

```
Process

↓

Thread A

↓

Thread B

↓

Thread C
```

Shared:

```
Code

↓

Heap

↓

Globals

↓

Files
```

Private:

```
Stack

↓

Registers

↓

Program Counter
```

## Key Takeaways

- A process can contain multiple threads.
- Threads share the process's memory and resources.
- Each thread has its own stack and CPU state.
- `pthread_create()` starts a new thread.
- `pthread_join()` waits for thread completion.
- Thread synchronization is essential when accessing shared data.

---

# Next Chapter

## Chapter 124 — Condition Variables (`pthread_cond_t`) — Making Threads Wait Efficiently

You'll learn:

- Why mutexes are not enough
- What condition variables are
- `pthread_cond_wait()`
- `pthread_cond_signal()`
- `pthread_cond_broadcast()`
- Producer-consumer synchronization
- Avoiding busy waiting
- Building efficient multithreaded applications