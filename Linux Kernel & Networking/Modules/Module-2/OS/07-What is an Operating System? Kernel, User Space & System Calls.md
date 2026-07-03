Most people think the Operating System is just:

- Windows
- Linux
- macOS

But that's only what you **see**.

The Operating System is much more than a desktop, icons, or a terminal.

It is the software that sits **between your applications and your hardware**.

Without an Operating System:

- Chrome cannot open.
- Docker cannot create containers.
- Java cannot allocate memory.
- Node.js cannot create sockets.
- PostgreSQL cannot read files.
- Kubernetes cannot run Pods.

Every application depends on the Operating System.

After this chapter, you'll understand why Operating Systems exist, how they work internally, and why the **Linux kernel** is one of the most important software projects ever created.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 7 — What is an Operating System? Kernel, User Space & System Calls

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is an Operating System
> - Why Operating Systems exist
> - Hardware vs Applications
> - Kernel
> - User Space
> - Kernel Space
> - User Mode vs Kernel Mode
> - System Calls
> - Resource Management
> - Process Management
> - Memory Management
> - File Management
> - Device Management
> - Why Linux became dominant
> - Complete OS Architecture

---

# 📖 Life Without an Operating System

Imagine you buy a brand-new computer.

It contains:

- CPU
- RAM
- SSD
- Keyboard
- Mouse
- Monitor

Now imagine there is **no Operating System**.

What happens if Chrome wants to:

- Read a file
- Use RAM
- Send data to the Internet
- Print something

Chrome would have to control every hardware device itself.

```
Chrome

↓

CPU

RAM

SSD

GPU

Keyboard

Mouse

Network Card
```

Every application would need to know how every hardware device works.

That would be impossible.

---

# Why Do We Need an Operating System?

Instead of every application controlling hardware,

we introduce one software layer.

```
Applications

↓

Operating System

↓

Hardware
```

Now applications no longer need to understand hardware.

They simply ask the Operating System.

---

# Real Example

Suppose Chrome wants to save a file.

Without OS:

```
Chrome

↓

Control SSD Directly
```

With OS:

```
Chrome

↓

Operating System

↓

SSD
```

The Operating System performs the hardware communication.

---

# What is an Operating System?

An Operating System is software that:

- Manages hardware
- Runs applications
- Allocates memory
- Schedules CPU
- Controls devices
- Manages files
- Provides security

It acts as the **manager of the computer**.

---

# Complete Architecture

```
+----------------------+

Applications

Chrome

Docker

Java

Node.js

PostgreSQL

+----------------------+

Operating System

+----------------------+

CPU

RAM

SSD

GPU

NIC

+----------------------+
```

Everything passes through the Operating System.

---

# The Kernel

The Kernel is the **core** of the Operating System.

```
Operating System

↓

Kernel
```

The kernel always stays in memory while the computer is running.

It has complete control over the hardware.

---

# Responsibilities of the Kernel

The kernel manages almost everything.

```
Kernel

├── CPU

├── Memory

├── Files

├── Processes

├── Devices

├── Networking

├── Security
```

Without the kernel,

the Operating System cannot function.

---

# User Space vs Kernel Space

The Operating System separates memory into two areas.

```
+----------------------+

User Space

Applications

+----------------------+

Kernel Space

Linux Kernel

+----------------------+
```

Applications cannot directly access Kernel Space.

---

# Why Separate Them?

Suppose Chrome crashes.

If Chrome could directly control memory,

it could crash the entire computer.

Instead:

```
Chrome

↓

User Space

↓

Kernel
```

The kernel protects the system.

---

# User Mode

Applications normally execute in:

```
User Mode
```

Limitations:

❌ Cannot control CPU directly

❌ Cannot access hardware

❌ Cannot modify kernel memory

❌ Cannot stop other processes

---

# Kernel Mode

The kernel executes in:

```
Kernel Mode
```

Privileges:

✔ Full memory access

✔ Device control

✔ CPU scheduling

✔ Network management

✔ File system control

Kernel Mode has unrestricted access to the system.

---

# User Mode vs Kernel Mode

| User Mode | Kernel Mode |
|------------|-------------|
| Limited Privileges | Full Privileges |
| Runs Applications | Runs Kernel |
| Cannot Access Hardware Directly | Controls Hardware |
| Safer | Powerful |

---

# How Can Applications Use Hardware?

Suppose Chrome wants to read a file.

Chrome cannot access the SSD directly.

Instead:

```
Chrome

↓

System Call

↓

Kernel

↓

SSD

↓

Kernel

↓

Chrome
```

This controlled request is called a **System Call**.

---

# What is a System Call?

A System Call is the gateway between User Space and Kernel Space.

```
Application

↓

System Call

↓

Kernel

↓

Hardware
```

Without System Calls,

applications could never use hardware safely.

---

# Common System Calls

Read a file:

```
read()
```

Write a file:

```
write()
```

Open a file:

```
open()
```

Allocate memory:

```
mmap()
```

Create a process:

```
fork()
```

Execute a program:

```
execve()
```

Create a socket:

```
socket()
```

Every application performs thousands of System Calls.

---

# Example — Reading a File

Suppose Java executes:

```java
Files.readString(path);
```

Internally:

```
Java

↓

read()

↓

Kernel

↓

SSD

↓

Kernel

↓

Java
```

Java never touches the SSD directly.

---

# Example — Opening a Network Connection

Suppose Chrome opens:

```
https://google.com
```

Internally:

```
Chrome

↓

socket()

↓

Kernel

↓

Network Card

↓

Internet
```

Networking always passes through the kernel.

---

# Resource Management

Many applications run simultaneously.

Example:

```
Chrome

Docker

VS Code

Spotify

PostgreSQL
```

Who decides:

- CPU usage?
- RAM allocation?
- Disk access?
- Network bandwidth?

The Operating System.

---

# Process Management

Every running application becomes a process.

Example:

```
Chrome

↓

Process
```

```
Docker

↓

Process
```

The kernel creates,

tracks,

and destroys processes.

---

# Memory Management

Every process needs memory.

The kernel decides:

```
How Much RAM?

Which Pages?

Virtual Memory?

Swap?
```

Applications don't decide.

The kernel does.

---

# File Management

Applications request files.

The kernel handles:

```
Permissions

Directories

Reading

Writing

Caching
```

Every file operation goes through the kernel.

---

# Device Management

Suppose you connect:

- Keyboard
- Mouse
- Printer
- USB Drive

The kernel communicates with device drivers.

```
Application

↓

Kernel

↓

Driver

↓

Hardware
```

---

# Networking

Networking is managed by the kernel.

Example:

```
Chrome

↓

socket()

↓

TCP/IP Stack

↓

NIC

↓

Internet
```

We'll study this entire flow later.

---

# Why Linux Became So Popular

Linux became dominant because:

✔ Open Source

✔ Stable

✔ Secure

✔ Highly Performant

✔ Excellent Networking

✔ Supports Thousands of Devices

✔ Used in Servers

✔ Used in Cloud

✔ Used in Docker

✔ Used in Kubernetes

Today,

most servers on the Internet run Linux.

---

# Real Example — Opening Chrome

```
Click Chrome

↓

Kernel Creates Process

↓

Allocate RAM

↓

Read Files

↓

Schedule CPU

↓

Create Network Socket

↓

Chrome Opens
```

Notice how the kernel is involved in every step.

---

# Why This Matters

Everything you'll learn later depends on these concepts.

```
Operating System

↓

Linux Kernel

↓

Processes

↓

Memory

↓

Networking

↓

Containers

↓

Docker

↓

Kubernetes
```

Without understanding the Operating System,

Linux internals will feel confusing.

---

# Hands-on Lab

## Lab 1 — View Running Processes

Linux/macOS:

```bash
ps -ef
```

Observe that every running application is a process.

---

## Lab 2 — View Kernel Version

```bash
uname -r
```

---

## Lab 3 — View System Calls

Linux:

```bash
strace ls
```

Observe how many system calls are made just to execute `ls`.

---

## Lab 4 — View Running Kernel Messages

Linux:

```bash
dmesg
```

---

# Interview Questions

## What is an Operating System?

An Operating System is software that manages hardware resources and provides services for applications.

---

## What is the Kernel?

The Kernel is the core component of the Operating System that controls hardware and manages system resources.

---

## Why can't applications access hardware directly?

Direct hardware access would be unsafe and could crash or compromise the entire system. Applications use System Calls to request services from the kernel.

---

## What is the difference between User Space and Kernel Space?

User Space runs applications with limited privileges, while Kernel Space runs the operating system with full access to hardware.

---

## What is a System Call?

A System Call is the mechanism through which an application requests services from the operating system kernel.

---

# Summary

The Operating System acts as the manager between applications and hardware.

```
Applications

↓

System Calls

↓

Linux Kernel

↓

CPU

RAM

SSD

NIC

GPU
```

Key concepts:

- The Operating System manages all hardware resources.
- The Kernel is the heart of the Operating System.
- Applications run in User Space.
- The Kernel runs in Kernel Space.
- System Calls allow applications to safely use hardware.
- Every file, network, memory, and process operation passes through the kernel.

Understanding these concepts is essential before learning processes, scheduling, memory management, networking, Docker, and the Linux kernel.

---

# Next Chapter

## Chapter 8 — Processes: How Programs Become Running Applications

We'll explore:

- What is a Process
- Process Creation
- Process Life Cycle
- Process Control Block (PCB)
- Process States
- PID
- PPID
- fork()
- exec()
- wait()
- Zombie Processes
- Orphan Processes