Most developers write code like this:

```java
FileReader reader = new FileReader("data.txt");
```

or

```javascript
fetch("https://api.example.com")
```

or

```go
conn.Read(buffer)
```

The code looks simple.

But behind the scenes, something incredible happens.

The Operating System communicates with:

- SSD
- Keyboard
- Mouse
- Monitor
- Network Card
- USB Devices

Questions arise:

- How does Linux read a file?
- How does a keyboard send key presses?
- How does a network packet reach your application?
- Why are SSDs slower than RAM?
- Why is asynchronous I/O faster?

The answer is **Input/Output (I/O)**.

I/O is one of the most performance-critical parts of every Operating System.

After this chapter, you'll understand how Linux communicates with hardware devices and why technologies like `epoll` and `io_uring` exist.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 15 — Input/Output (I/O): How Linux Talks to Hardware

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is I/O
> - Why I/O is slower than CPU
> - Device Drivers
> - Character Devices
> - Block Devices
> - Device Files
> - System Calls for I/O
> - Blocking I/O
> - Non-Blocking I/O
> - Synchronous I/O
> - Asynchronous I/O
> - Interrupts
> - Polling
> - DMA (Direct Memory Access)
> - `select()`
> - `poll()`
> - `epoll()`
> - `io_uring()`

---

# 📖 What is Input/Output (I/O)?

Input/Output means communication between software and hardware.

Examples:

Input:

```
Keyboard

↓

Mouse

↓

Microphone

↓

Network Packet
```

Output:

```
Monitor

↓

Speaker

↓

Printer

↓

SSD
```

Every application constantly performs I/O.

---

# Why Do We Need I/O?

Suppose Chrome wants to display a webpage.

It needs to:

```
Read HTML

↓

Read Images

↓

Receive Network Packets

↓

Draw Screen
```

All of these are I/O operations.

---

# CPU vs I/O Speed

The CPU is extremely fast.

```
CPU

↓

Billions of Operations / Second
```

A disk is much slower.

```
SSD

↓

Microseconds
```

Network communication is slower still.

```
Internet

↓

Milliseconds
```

Therefore,

I/O often becomes the bottleneck.

---

# Typical Application Flow

```
Application

↓

System Call

↓

Kernel

↓

Device Driver

↓

Hardware
```

The application never communicates directly with hardware.

---

# Device Drivers

Every hardware device behaves differently.

Examples:

- SSD
- Keyboard
- Printer
- Network Card
- GPU

Linux uses:

```
Device Drivers
```

A driver understands one specific device.

---

# Device Driver Architecture

```
Application

↓

Kernel

↓

Driver

↓

Hardware
```

Without drivers,

Linux wouldn't know how to communicate with hardware.

---

# Device Files

Linux represents devices as files.

Examples:

```
/dev/sda

↓

Disk
```

```
/dev/tty

↓

Terminal
```

```
/dev/null

↓

Discard Data
```

```
/dev/random

↓

Random Numbers
```

Everything looks like a file.

---

# Why "Everything is a File"

In Linux,

the same system calls work for:

- Files
- Pipes
- Sockets
- Devices

Example:

```
read()

write()

close()
```

Applications don't need different APIs for every device.

---

# Character Devices

Character devices transfer data one character or byte at a time.

Examples:

- Keyboard
- Mouse
- Terminal
- Serial Port

Flow:

```
Keyboard

↓

Driver

↓

Application
```

---

# Block Devices

Block devices transfer data in fixed-size blocks.

Examples:

- SSD
- HDD
- USB Storage

Flow:

```
Application

↓

Driver

↓

Disk Blocks
```

---

# Character vs Block Devices

| Character Device | Block Device |
|------------------|--------------|
| Byte Stream | Fixed Blocks |
| Sequential Access | Random Access |
| Keyboard | SSD |
| Mouse | HDD |
| Terminal | USB Storage |

---

# File Read Flow

Suppose:

```bash
cat file.txt
```

Internally:

```
Application

↓

read()

↓

Kernel

↓

File System

↓

Driver

↓

SSD

↓

RAM

↓

Application
```

---

# Network Read Flow

Suppose Chrome receives a webpage.

```
Internet

↓

Network Card

↓

Driver

↓

Kernel

↓

Socket

↓

Chrome
```

---

# Blocking I/O

Suppose an application reads a file.

```
Application

↓

read()

↓

Wait

↓

Wait

↓

Wait

↓

Data Arrives

↓

Continue
```

During the wait,

the thread does nothing.

---

# Blocking Example

```java
reader.read();
```

The thread cannot continue until data arrives.

---

# Problem with Blocking I/O

Suppose one server thread handles one client.

```
Client 1

↓

Wait

↓

Thread Blocked
```

Thousands of clients:

```
Thousands of Waiting Threads
```

This wastes memory and CPU.

---

# Non-Blocking I/O

Instead of waiting,

the application asks:

```
Data Ready?
```

If not:

```
Continue Other Work
```

Later,

check again.

---

# Non-Blocking Flow

```
read()

↓

No Data

↓

Return Immediately

↓

Do Other Work

↓

Try Again
```

The thread never sleeps unnecessarily.

---

# Synchronous I/O

In synchronous I/O,

the application is responsible for waiting.

```
Request

↓

Wait

↓

Data

↓

Continue
```

---

# Asynchronous I/O

In asynchronous I/O,

the application starts an operation and continues immediately.

```
Request

↓

Continue Working

↓

Kernel Reads Data

↓

Completion Notification

↓

Process Result
```

The application doesn't wait.

---

# Blocking vs Non-Blocking

| Blocking | Non-Blocking |
|-----------|--------------|
| Waits | Returns Immediately |
| Simpler | More Efficient |
| Easier to Write | More Complex |
| Good for Small Programs | Good for High Concurrency |

---

# Synchronous vs Asynchronous

| Synchronous | Asynchronous |
|--------------|--------------|
| Caller Waits | Caller Continues |
| Simpler | Higher Performance |
| Common in Simple Programs | Used in High-Performance Servers |

---

# Interrupts

Suppose you press a key.

The CPU doesn't constantly check the keyboard.

Instead:

```
Keyboard

↓

Interrupt

↓

CPU

↓

Handle Key Press
```

The hardware notifies the CPU.

---

# Polling

Another approach:

```
CPU

↓

Check Keyboard

↓

Nothing

↓

Check Again

↓

Nothing

↓

Check Again
```

This wastes CPU time.

---

# Interrupts vs Polling

| Interrupt | Polling |
|------------|----------|
| Hardware Notifies CPU | CPU Continuously Checks |
| Efficient | Wasteful |
| Used by Most Devices | Used in Special Cases |

---

# DMA (Direct Memory Access)

Suppose the SSD must copy:

```
500 MB
```

Without DMA:

```
SSD

↓

CPU

↓

RAM
```

The CPU copies every byte.

Very expensive.

---

# With DMA

```
SSD

↓

DMA Controller

↓

RAM
```

The CPU simply says:

```
Start Transfer
```

Later:

```
Interrupt

↓

Transfer Complete
```

This frees the CPU to do other work.

---

# Why DMA Matters

DMA is heavily used for:

- SSDs
- Network Cards
- GPUs
- Audio Devices

It dramatically improves performance.

---

# `select()`

Suppose a server manages:

```
100 Sockets
```

Instead of blocking on one,

it asks:

```
Which Socket Has Data?
```

This is the purpose of:

```
select()
```

---

# Problem with `select()`

Suppose:

```
100,000 Connections
```

`select()` must scan all of them every time.

This becomes slow.

---

# `poll()`

`poll()` improves some limitations,

but still scans all file descriptors.

Large servers still suffer.

---

# `epoll()`

Linux introduced:

```
epoll()
```

Instead of asking:

```
Check Everything
```

The kernel notifies the application:

```
These Sockets Are Ready
```

No unnecessary scanning.

---

# Why `epoll()` Is Fast

Instead of:

```
100,000 Checks
```

You only process:

```
Ready Connections
```

This is why:

- Nginx
- Redis
- Node.js
- HAProxy

use `epoll()` on Linux.

---

# `io_uring()`

`io_uring()` is the newest Linux I/O interface.

Instead of repeated system calls,

it uses shared memory between the application and the kernel.

```
Application

↓

Shared Ring Buffer

↓

Kernel
```

Benefits:

✔ Fewer System Calls

✔ Lower Latency

✔ Higher Throughput

✔ Asynchronous Operations

---

# Why `io_uring()` Exists

Traditional I/O:

```
Application

↓

System Call

↓

Kernel

↓

Return

↓

Repeat
```

Many system calls.

With `io_uring()`:

```
Application

↓

Shared Queue

↓

Kernel
```

Much less overhead.

---

# Real Example

Nginx receives:

```
10,000 Clients
```

Internally:

```
epoll()

↓

Ready Connections

↓

Read Request

↓

Send Response
```

Only active connections consume CPU.

---

# Why I/O Matters

Every modern application depends on efficient I/O.

Examples:

- Chrome
- PostgreSQL
- Docker
- Redis
- Nginx
- Kubernetes
- Kafka

Poor I/O design limits scalability.

---

# Hands-on Lab

## Lab 1 — View Block Devices

```bash
lsblk
```

Observe available storage devices.

---

## Lab 2 — View Character Devices

```bash
ls -l /dev
```

Notice entries like:

```
/dev/null

/dev/tty

/dev/random
```

---

## Lab 3 — Monitor Disk I/O

Linux:

```bash
iostat
```

---

## Lab 4 — Observe Open File Descriptors

```bash
lsof
```

Observe that sockets and devices also appear as files.

---

## Lab 5 — Observe Network Connections

```bash
ss -t
```

View active TCP sockets.

---

# Interview Questions

## What is I/O?

I/O (Input/Output) is communication between software and hardware devices.

---

## Why is I/O slower than CPU operations?

Because storage devices, networks, and peripherals operate much more slowly than CPU instruction execution.

---

## What is a Device Driver?

A Device Driver is software inside the kernel that knows how to communicate with a specific hardware device.

---

## What is the difference between a Character Device and a Block Device?

Character Devices transfer data as a byte stream, while Block Devices transfer fixed-size blocks.

---

## What is Blocking I/O?

Blocking I/O causes the calling thread to wait until the operation completes.

---

## What is Non-Blocking I/O?

Non-Blocking I/O returns immediately if data is unavailable, allowing the application to continue other work.

---

## Why is `epoll()` better than `select()` for large servers?

`epoll()` reports only ready file descriptors instead of scanning all monitored descriptors every time.

---

## What is the advantage of `io_uring()`?

`io_uring()` reduces system call overhead and enables highly efficient asynchronous I/O through shared queues.

---

# Summary

Linux provides a unified I/O model for communicating with hardware devices.

```
Application

↓

System Call

↓

Kernel

↓

Device Driver

↓

Hardware
```

Key concepts:

- I/O connects applications to hardware.
- Device Drivers abstract hardware differences.
- Linux treats devices as files.
- Character Devices stream bytes; Block Devices store fixed-size blocks.
- Blocking and Non-Blocking I/O determine waiting behavior.
- Synchronous and Asynchronous I/O determine execution flow.
- Interrupts notify the CPU efficiently.
- DMA transfers data without constant CPU involvement.
- `epoll()` powers high-performance network servers.
- `io_uring()` represents the next generation of Linux I/O.

Understanding I/O is essential before learning networking, Linux kernel internals, Docker, Kubernetes, databases, and high-performance server architecture.

---

# Next Chapter

## Chapter 16 — Security & Permissions: Users, Groups, Permissions, Capabilities & Access Control

We'll explore:

- Why Operating Systems need security
- Users and Groups
- File Permissions
- `rwx` Permissions
- Ownership
- `chmod`
- `chown`
- Special Permissions (SUID, SGID, Sticky Bit)
- Access Control Lists (ACLs)
- Linux Capabilities
- Authentication vs Authorization
- Principle of Least Privilege