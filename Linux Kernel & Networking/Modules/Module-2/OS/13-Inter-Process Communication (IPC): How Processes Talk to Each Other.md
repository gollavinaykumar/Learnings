Most developers think:

> "Processes run independently."

That's true.

But then another question arises:

How does one process communicate with another?

Examples:

- Chrome talks to the GPU process.
- Docker CLI talks to Docker Engine.
- PostgreSQL clients talk to the PostgreSQL server.
- Nginx workers communicate with the master process.
- Kubernetes components constantly exchange information.

Processes are isolated for security.

So how do they exchange data?

The answer is:

**Inter-Process Communication (IPC).**

IPC is one of the core services provided by every Operating System.

After this chapter, you'll understand all major IPC mechanisms used in Linux and where they are used in real systems.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 13 — Inter-Process Communication (IPC): How Processes Talk to Each Other

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why IPC is needed
> - Process Isolation
> - Pipes
> - Named Pipes (FIFO)
> - Message Queues
> - Shared Memory
> - Semaphores in IPC
> - Signals
> - UNIX Domain Sockets
> - Memory-Mapped Files (`mmap`)
> - Socket-based IPC
> - Choosing the right IPC mechanism
> - Real-world IPC examples

---

# 📖 Why Do We Need IPC?

Suppose Chrome has multiple processes.

```
Chrome Browser

↓

Renderer Process

↓

GPU Process

↓

Network Process
```

These processes are isolated.

How do they exchange data?

They cannot directly read each other's memory.

They need:

```
IPC
```

---

# Process Isolation

Remember from the previous chapter:

Every process has its own:

- Memory
- Stack
- Heap
- Registers

```
Process A

↓

Own Memory
```

```
Process B

↓

Own Memory
```

This isolation improves security.

But it creates a communication problem.

---

# What is IPC?

IPC stands for:

```
Inter-Process Communication
```

It allows processes to:

✔ Exchange data

✔ Synchronize work

✔ Notify each other

✔ Share resources

---

# IPC Overview

```
Process A

↓

IPC Mechanism

↓

Process B
```

Linux provides many IPC mechanisms.

Each solves different problems.

---

# Types of IPC

```
IPC

├── Pipes

├── Named Pipes

├── Message Queues

├── Shared Memory

├── Semaphores

├── Signals

├── UNIX Domain Sockets

├── Memory Mapping

└── Network Sockets
```

Let's study each one.

---

# Pipes

A Pipe is the simplest IPC mechanism.

Think of a water pipe.

```
Process A

↓

Pipe

↓

Process B
```

One process writes.

Another process reads.

---

# Pipe Example

Linux command:

```bash
ls | grep txt
```

Flow:

```
ls

↓

Pipe

↓

grep
```

`ls` writes.

`grep` reads.

No temporary file is created.

---

# Pipe Characteristics

✔ Simple

✔ Fast

✔ Parent ↔ Child Communication

❌ One-way by default

❌ Usually related processes only

---

# Named Pipe (FIFO)

Suppose two unrelated processes need communication.

Linux provides:

```
FIFO

First In

First Out
```

Unlike normal pipes,

Named Pipes exist as files.

```
Process A

↓

FIFO File

↓

Process B
```

---

# Message Queue

Instead of streaming bytes,

processes exchange messages.

```
Sender

↓

Queue

↓

Receiver
```

Messages remain in the queue until read.

---

# Queue Example

```
Message 1

↓

Message 2

↓

Message 3
```

The receiver processes them one by one.

---

# Shared Memory

Shared Memory is the fastest IPC mechanism.

Instead of copying data,

multiple processes access the same memory.

```
Process A

↓

Shared Memory

↑

Process B
```

No copying occurs.

---

# Why Is Shared Memory Fast?

Suppose:

```
100 MB Data
```

Pipe:

```
Copy

↓

Copy Again
```

Shared Memory:

```
No Copy

↓

Same Memory
```

Much faster.

---

# Problem with Shared Memory

Since both processes access the same memory,

they can overwrite each other's data.

Therefore,

Shared Memory is almost always combined with:

```
Synchronization

↓

Mutex

↓

Semaphore
```

---

# Semaphores in IPC

Semaphores coordinate access to Shared Memory.

Example:

```
Process A

↓

Semaphore

↓

Shared Memory

↑

Process B
```

Only the allowed number of processes may access the resource.

---

# Signals

Sometimes you don't need to send data.

You only want to notify another process.

Example:

```
Stop

Continue

Terminate

Reload Configuration
```

Linux uses:

```
Signals
```

---

# Common Signals

```
SIGINT

Ctrl + C
```

```
SIGTERM

Graceful Shutdown
```

```
SIGKILL

Immediate Termination
```

```
SIGHUP

Reload Configuration
```

Signals are lightweight notifications.

---

# UNIX Domain Sockets

Sockets aren't only for networking.

Linux provides:

```
UNIX Domain Socket
```

for communication on the same machine.

```
Process A

↓

Socket

↓

Process B
```

Unlike Internet sockets,

no network card is involved.

---

# Why UNIX Domain Sockets Are Fast

Data never leaves the operating system.

```
Application

↓

Kernel

↓

Application
```

No TCP/IP routing.

No Ethernet.

No Internet.

This makes them very efficient.

---

# Docker Example

Suppose you execute:

```bash
docker ps
```

Does the Docker CLI start containers?

No.

The CLI communicates with the Docker Engine.

Flow:

```
Docker CLI

↓

/var/run/docker.sock

↓

Docker Engine
```

`docker.sock` is a UNIX Domain Socket.

---

# PostgreSQL Example

Applications connect using sockets.

```
Application

↓

PostgreSQL Socket

↓

Database
```

On the same machine,

this is often faster than TCP.

---

# Memory-Mapped Files (`mmap`)

A file can be mapped into memory.

```
Disk File

↓

mmap()

↓

Virtual Memory
```

Multiple processes can map the same file.

```
Process A

↓

Mapped File

↑

Process B
```

Useful for:

- Databases
- Browsers
- Analytics
- Large Files

---

# Network Sockets

Processes can also communicate across machines.

```
Machine A

↓

TCP Socket

↓

Machine B
```

Examples:

- HTTP
- SSH
- PostgreSQL
- Redis
- Kafka

This is also IPC,

but across systems.

---

# IPC Comparison

| IPC Mechanism | Same Machine | Across Network | Fast | Data Sharing |
|--------------|--------------|----------------|------|--------------|
| Pipe | ✔ | ❌ | High | Stream |
| Named Pipe | ✔ | ❌ | High | Stream |
| Message Queue | ✔ | ❌ | Medium | Messages |
| Shared Memory | ✔ | ❌ | Very High | Memory |
| Signals | ✔ | ❌ | Very High | Notification |
| UNIX Domain Socket | ✔ | ❌ | Very High | Bidirectional |
| TCP Socket | ✔ | ✔ | Medium | Stream |

---

# Which IPC Should You Use?

Need simple parent-child communication?

```
Pipe
```

Need unrelated local processes?

```
Named Pipe

or

UNIX Socket
```

Need structured messages?

```
Message Queue
```

Need maximum performance?

```
Shared Memory
```

Need notifications only?

```
Signals
```

Need communication between computers?

```
TCP Socket
```

---

# Real Examples

Chrome

```
Browser

↓

IPC

↓

Renderer
```

Docker

```
CLI

↓

docker.sock

↓

Docker Engine
```

PostgreSQL

```
Client

↓

Socket

↓

Database
```

Nginx

```
Master

↓

Signals

↓

Workers
```

Redis

```
Client

↓

TCP

↓

Server
```

---

# Why IPC Matters

Modern software constantly uses IPC.

Examples:

- Docker
- Kubernetes
- Chrome
- PostgreSQL
- Redis
- Kafka
- VS Code
- Linux System Services

Without IPC,

processes would be isolated forever.

---

# Hands-on Lab

## Lab 1 — Observe a Pipe

```bash
ls | grep ".md"
```

Observe data flowing from one process to another.

---

## Lab 2 — View Docker Socket

```bash
ls -l /var/run/docker.sock
```

Observe the UNIX Domain Socket used by Docker.

---

## Lab 3 — Send a Signal

Start:

```bash
sleep 300
```

Find its PID:

```bash
ps -ef | grep sleep
```

Terminate gracefully:

```bash
kill <PID>
```

Force terminate:

```bash
kill -9 <PID>
```

---

## Lab 4 — View Open UNIX Sockets

Linux:

```bash
ss -xl
```

Observe active UNIX Domain Sockets.

---

# Interview Questions

## Why do processes need IPC?

Because processes are isolated and cannot directly access each other's memory.

---

## What is the fastest IPC mechanism?

Shared Memory, because data is not copied between processes.

---

## What is the difference between a Pipe and a Named Pipe?

A Pipe usually connects related processes.

A Named Pipe exists as a filesystem object and can connect unrelated processes.

---

## What are Signals used for?

Signals notify processes about events such as termination, interruption, or configuration reload.

---

## Why does Docker use a UNIX Domain Socket?

Because the Docker CLI and Docker Engine usually run on the same machine, making UNIX Domain Sockets faster than TCP.

---

## When should Shared Memory be combined with synchronization?

Whenever multiple processes may read or write shared data simultaneously, synchronization mechanisms like Mutexes or Semaphores are required.

---

# Summary

IPC allows isolated processes to communicate safely and efficiently.

```
Process A

↓

IPC Mechanism

↓

Process B
```

Key concepts:

- Processes are isolated by default.
- IPC enables communication and coordination.
- Pipes provide simple byte streams.
- Named Pipes allow unrelated processes to communicate.
- Message Queues exchange structured messages.
- Shared Memory offers the highest performance.
- Signals provide lightweight notifications.
- UNIX Domain Sockets enable efficient local communication.
- TCP Sockets allow communication across networks.
- Real-world systems like Docker, PostgreSQL, Chrome, and Kubernetes rely heavily on IPC.

Understanding IPC is essential before studying file systems, networking, containers, Linux kernel internals, and distributed systems.

---

# Next Chapter

## Chapter 14 — File Systems: How Linux Stores and Retrieves Data

We'll explore:

- Why File Systems exist
- Files vs Directories
- Inodes
- Superblocks
- Blocks
- Journaling
- ext4
- XFS
- Btrfs
- VFS (Virtual File System)
- Mount Points
- File Permissions
- How Linux locates a file on disk