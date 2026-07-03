Most developers write code like this:

```c
open("file.txt");
```

```java
Files.readString(path);
```

```javascript
fetch("https://google.com");
```

```go
os.Open("file.txt")
```

The code looks simple.

But here's the important question:

**How does your program actually talk to the Linux kernel?**

Remember:

Applications run in **User Space**.

Hardware is controlled by the **Kernel**.

Applications **cannot** directly:

- Read the SSD
- Send packets
- Allocate physical memory
- Create processes
- Mount file systems

So how do they request these operations?

The answer is:

**System Calls (syscalls).**

System Calls are the **gateway between User Space and Kernel Space**.

Every modern application—from Chrome to Docker to PostgreSQL—uses thousands of system calls every second.

After this chapter, you'll understand exactly how your code enters the Linux kernel.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 22 — System Calls: How User Space Talks to the Linux Kernel

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a System Call?
> - Why System Calls exist
> - User Space vs Kernel Space
> - CPU Privilege Levels
> - Ring 0 vs Ring 3
> - Trap Instructions
> - `syscall` Instruction
> - System Call Table
> - Kernel Entry
> - Kernel Exit
> - Common Linux System Calls
> - `strace`
> - Complete System Call Flow

---

# 📖 Why Do We Need System Calls?

Suppose your Java application wants to read a file.

```java
Files.readString(path);
```

Can Java directly access the SSD?

No.

Why?

Because Java runs in:

```
User Space
```

Only the kernel can communicate with hardware.

Therefore Java asks:

```
Java

↓

System Call

↓

Kernel

↓

SSD
```

---

# User Space vs Kernel Space

Remember:

```
Applications

↓

User Space
```

```
Linux Kernel

↓

Kernel Space
```

Applications are isolated.

The kernel has complete hardware access.

---

# Why Can't Applications Access Hardware?

Imagine Chrome directly controlling RAM.

It could:

- Read passwords from another process
- Delete kernel memory
- Crash the Operating System

Instead:

```
Application

↓

Request Permission

↓

Kernel
```

The kernel decides whether the request is allowed.

---

# What is a System Call?

A System Call is a controlled request from User Space to Kernel Space.

```
Application

↓

System Call

↓

Kernel

↓

Hardware
```

It is the only safe way for applications to use operating system services.

---

# Examples of System Calls

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

Create a process:

```
fork()
```

Create a socket:

```
socket()
```

Allocate memory:

```
mmap()
```

Exit a process:

```
exit()
```

Thousands of applications use these every second.

---

# Example — Reading a File

Suppose:

```bash
cat notes.txt
```

Internally:

```
cat

↓

read()

↓

Kernel

↓

Filesystem

↓

SSD

↓

Kernel

↓

cat
```

---

# Example — Network Connection

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

TCP/IP Stack

↓

Network Card
```

---

# CPU Privilege Levels

Modern CPUs support different privilege levels.

```
Ring 0

↓

Kernel
```

```
Ring 3

↓

Applications
```

Ring 0 has unrestricted access.

Ring 3 has limited access.

---

# Ring 0

Kernel Mode

Capabilities:

✔ Access Any Memory

✔ Execute Privileged Instructions

✔ Configure Hardware

✔ Manage CPU

---

# Ring 3

User Mode

Restrictions:

❌ Cannot Access Kernel Memory

❌ Cannot Execute Privileged Instructions

❌ Cannot Access Hardware Directly

---

# How Does the CPU Switch Modes?

Suppose:

```
Application

↓

read()
```

The CPU must switch from:

```
Ring 3

↓

Ring 0
```

This switch happens using a special CPU instruction.

---

# Trap Instruction

Historically,

Linux used:

```
int 0x80
```

to enter the kernel.

Modern 64-bit Linux usually uses:

```
syscall
```

The CPU immediately switches into Kernel Mode.

---

# System Call Flow

```
Application

↓

syscall Instruction

↓

CPU Switches to Ring 0

↓

Kernel Executes Request

↓

Return to Ring 3
```

This happens extremely quickly.

---

# The System Call Table

Every System Call has a number.

Example:

```
0

↓

read
```

```
1

↓

write
```

```
2

↓

open
```

The kernel stores these mappings inside the:

```
System Call Table
```

---

# Example

Suppose:

```
System Call Number

=

0
```

Kernel checks:

```
Table

↓

read()

↓

Execute
```

---

# Arguments to System Calls

System Calls often require parameters.

Example:

```c
read(fd, buffer, size);
```

Parameters include:

```
File Descriptor

Buffer

Size
```

The CPU places these values into registers before executing `syscall`.

---

# Kernel Entry

When the CPU executes:

```
syscall
```

The kernel performs:

```
Save User Registers

↓

Switch Stack

↓

Kernel Stack

↓

Locate System Call

↓

Execute
```

Now the kernel safely handles the request.

---

# Kernel Exit

After completing the operation:

```
Kernel

↓

Return Value

↓

Restore Registers

↓

Switch to User Mode

↓

Continue Program
```

The application resumes exactly where it stopped.

---

# File Descriptor

Most System Calls use File Descriptors.

Example:

```
0

↓

stdin
```

```
1

↓

stdout
```

```
2

↓

stderr
```

When opening a file:

```c
fd = open("file.txt");
```

The kernel returns another file descriptor.

---

# Real Example

Suppose:

```bash
echo Hello
```

Internally:

```
echo

↓

write()

↓

stdout

↓

Kernel

↓

Terminal Driver

↓

Screen
```

---

# System Calls Are Everywhere

Examples:

Java

```
Files.readAllBytes()

↓

read()
```

Go

```
net.Listen()

↓

socket()
```

Python

```
open()

↓

open()
```

Node.js

```
fs.readFile()

↓

read()
```

Every language eventually reaches Linux System Calls.

---

# `strace`

Want to see System Calls?

Linux provides:

```bash
strace
```

Example:

```bash
strace ls
```

Output:

```
open()

↓

read()

↓

close()

↓

write()
```

You can watch every System Call the program performs.

---

# Why `strace` Is Useful

Developers use it to:

✔ Debug applications

✔ Find missing files

✔ Analyze performance

✔ Understand Linux behavior

---

# Real Example — Docker

Suppose:

```bash
docker run nginx
```

Docker performs many System Calls:

```
clone()

↓

mount()

↓

sethostname()

↓

unshare()

↓

execve()
```

Everything eventually enters the Linux kernel through System Calls.

---

# Why System Calls Matter

Every Linux application depends on them.

Examples:

- Docker
- Kubernetes
- PostgreSQL
- Redis
- Nginx
- Chrome
- Java
- Go
- Python
- Node.js

Without System Calls,

applications cannot interact with the Operating System.

---

# Hands-on Lab

## Lab 1 — Trace `ls`

```bash
strace ls
```

Observe dozens of system calls.

---

## Lab 2 — Trace File Reads

```bash
strace cat README.md
```

Observe:

```
open()

read()

close()
```

---

## Lab 3 — Trace Network Activity

```bash
strace curl https://example.com
```

Observe:

```
socket()

connect()

sendto()

recvfrom()
```

---

## Lab 4 — View System Call Manual

```bash
man 2 open
```

Section 2 contains Linux System Calls.

---

## Lab 5 — Observe File Descriptors

```bash
ls -l /proc/$$/fd
```

---

# Interview Questions

## What is a System Call?

A System Call is the controlled interface through which applications request services from the Linux kernel.

---

## Why do applications need System Calls?

Because applications run in User Space and cannot directly access hardware or privileged kernel functionality.

---

## What is the difference between Ring 0 and Ring 3?

Ring 0 is Kernel Mode with full privileges.

Ring 3 is User Mode with restricted privileges.

---

## What is the purpose of the `syscall` instruction?

It switches the CPU from User Mode to Kernel Mode so the kernel can execute a requested service.

---

## What is the System Call Table?

It is the kernel's lookup table that maps system call numbers to their corresponding kernel functions.

---

## What does `strace` do?

`strace` traces and displays the system calls made by a running program.

---

## Why do all programming languages ultimately depend on System Calls?

Because all interactions with files, networks, processes, memory, and devices must be performed by the Linux kernel.

---

# Summary

System Calls provide the secure bridge between User Space and the Linux kernel.

```
Application

↓

System Call

↓

syscall Instruction

↓

CPU Switches to Ring 0

↓

Linux Kernel

↓

Hardware

↓

Return to User Space
```

Key concepts:

- Applications cannot access hardware directly.
- System Calls are the official interface to the Linux kernel.
- User Space runs in Ring 3; the kernel runs in Ring 0.
- The `syscall` instruction transfers control to the kernel.
- The System Call Table maps syscall numbers to kernel functions.
- `strace` reveals the system calls made by applications.
- Every high-level language ultimately relies on Linux System Calls.

Understanding System Calls is essential before learning namespaces, cgroups, networking internals, Docker, Kubernetes, and Linux kernel programming.

---

# Next Chapter

## Chapter 23 — Linux Kernel Modules: Extending the Kernel Without Rebooting

We'll explore:

- What are Kernel Modules?
- Why Kernel Modules exist
- Loadable Kernel Modules (LKM)
- Module Lifecycle
- `insmod`
- `rmmod`
- `modprobe`
- Module Dependencies
- Device Drivers as Modules
- How Docker depends on Kernel Modules
- Writing Your First Kernel Module