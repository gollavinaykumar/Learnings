# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 133 — System Calls — How User Space Talks to the Linux Kernel

---

# Most developers know system calls because of:

```c
read();

write();

open();

fork();
```

But very few understand:

- What is a system call?
- Why can't C programs directly access hardware?
- What happens after calling:

```c
read(fd, buf, 100);
```

- How does the CPU switch from user mode to kernel mode?

This chapter explains the most fundamental interface between your application and the Linux kernel.

---

# Learning Objectives

After this chapter you will understand:

- What a system call is
- User Mode
- Kernel Mode
- CPU privilege levels
- System Call Interface
- Trap instruction
- Context switch
- Return to user mode
- Why system calls exist

---

# The Problem

Suppose your program wants to:

- Read a file
- Allocate memory
- Create a process
- Send a packet
- Access the disk

Can it directly execute:

```
Disk Controller

↓

Read Sector
```

No.

That would be extremely dangerous.

---

# Why?

Imagine:

```
User Program

↓

Erase Entire Disk
```

Or:

```
Overwrite Kernel Memory
```

Or:

```
Read Other Process Passwords
```

The operating system prevents this.

---

# CPU Privilege Levels

Modern CPUs have protection levels.

Simplified:

```
Kernel Mode

↓

Full Permission

-----------------------

User Mode

↓

Restricted Permission
```

Applications always start in:

```
User Mode
```

---

# Visualization

```
+----------------------+
| User Space           |
| Your Program         |
+----------------------+

System Call

↓

+----------------------+
| Kernel Space         |
| Linux Kernel         |
+----------------------+
```

---

# User Space

Applications execute here.

Examples:

```
Chrome

↓

VS Code

↓

Redis

↓

Python

↓

Node.js
```

Restrictions:

❌ Cannot directly

- Access hardware
- Change page tables
- Modify scheduler
- Access kernel memory

---

# Kernel Space

Kernel executes here.

Privileges:

✅ Access:

- RAM
- CPU
- Disk
- Network
- Interrupts
- Device Drivers

Everything important.

---

# What is a System Call?

Definition:

> **A system call is the controlled entry point from user space into kernel space.**

Think of it as a secure doorway.

---

# Think About a Bank

Customer:

```
Needs Money
```

Cannot enter:

```
Vault
```

Instead:

```
Customer

↓

Counter

↓

Bank Employee

↓

Vault

↓

Money
```

System calls work exactly like the bank counter.

---

# Example

Program:

```c
write(1, "Hello", 5);
```

Flow:

```
Program

↓

System Call

↓

Kernel

↓

Terminal Driver

↓

Screen
```

---

# Without System Calls

Applications would need:

```
Know Disk Hardware

↓

Know SSD Protocol

↓

Know GPU Registers

↓

Know Keyboard Controller
```

Impossible.

Linux hides all of this.

---

# System Call Flow

```
User Program

↓

System Call

↓

Kernel

↓

Hardware

↓

Kernel

↓

Return
```

---

# Example — `read()`

Program:

```c
read(fd, buf, 100);
```

Kernel:

```
Locate File

↓

Filesystem

↓

Disk Driver

↓

SSD

↓

Copy Data

↓

Return
```

---

# Example — `fork()`

Program:

```c
fork();
```

Kernel:

```
Create Process

↓

Copy Resources

↓

Scheduler

↓

Return
```

---

# Example — `mmap()`

Program:

```c
mmap();
```

Kernel:

```
Virtual Memory Manager

↓

Page Tables

↓

Return Pointer
```

---

# Example — `socket()`

Program:

```c
socket();
```

Kernel:

```
Networking Stack

↓

Allocate Socket

↓

Return FD
```

---

# CPU Mode Switch

This is the most important concept.

Initially:

```
User Mode
```

Program executes:

```c
read();
```

CPU performs:

```
Privilege Switch

↓

Kernel Mode
```

Kernel executes.

After completion:

```
Return

↓

User Mode
```

---

# Visualization

```
User Mode

↓

System Call

↓

Kernel Mode

↓

Return

↓

User Mode
```

This happens millions of times every second.

---

# Trap Instruction

Your C code does **not** directly jump into the kernel.

Instead,

the C library eventually executes a special CPU instruction.

Historically:

```
int 0x80
```

Modern x86-64:

```
syscall
```

ARM uses different instructions,

but the concept is identical.

---

# Context Switch

During a system call,

CPU saves:

```
Registers

↓

Program Counter

↓

Flags
```

Kernel performs the requested work,

then restores execution.

---

# System Call Numbers

Every system call has an ID.

Example (x86-64):

```
read

↓

0

-------------------

write

↓

1

-------------------

open

↓

2
```

The exact numbering depends on the architecture.

---

# C Library

You write:

```c
printf();
```

Actually:

```
printf()

↓

write()

↓

syscall
```

Many standard library functions eventually invoke system calls.

---

# Standard Flow

```
Application

↓

glibc

↓

syscall

↓

Linux Kernel
```

---

# Why Not Call Kernel Functions Directly?

Kernel functions:

```
Not Accessible

↓

User Space
```

Only system calls form the supported interface.

---

# Real-World Example — Chrome

```
Read File

↓

read()

↓

Kernel

↓

SSD
```

Thousands of system calls occur every second.

---

# Real-World Example — Redis

```
Client

↓

epoll_wait()

↓

Kernel

↓

Ready Events
```

Redis spends much of its time interacting with the kernel through system calls.

---

# Real-World Example — Docker

Docker constantly invokes:

```
clone()

↓

mount()

↓

setns()

↓

ioctl()
```

to manage containers.

---

# Real-World Example — JVM

Java:

```java
FileInputStream
```

Eventually becomes:

```
read()

↓

Kernel
```

Every language ultimately reaches Linux system calls.

---

# Common System Calls

| System Call | Purpose |
|-------------|----------|
| `read()` | Read data |
| `write()` | Write data |
| `open()` | Open file |
| `close()` | Close file |
| `fork()` | Create process |
| `execve()` | Execute program |
| `mmap()` | Map memory |
| `socket()` | Create socket |
| `accept()` | Accept client |
| `epoll_wait()` | Wait for I/O |

---

# Why System Calls Are Expensive

Compared to normal functions:

Normal function:

```
Call

↓

Return
```

System call:

```
Privilege Switch

↓

Kernel Work

↓

Privilege Switch Back
```

More overhead.

---

# Common Mistakes

---

## Thinking `printf()` Is a System Call

It is a C library function.

Eventually,

it usually uses:

```
write()
```

---

## Thinking Kernel Runs in User Mode

Kernel always executes in privileged mode.

---

## Thinking Applications Access Hardware Directly

They must go through system calls.

---

## Assuming Every Function Is a System Call

Most C functions never enter the kernel.

Only specific operations require it.

---

# Hands-on Labs

## Lab 1

Run:

```bash
strace ls
```

Observe all system calls made by `ls`.

---

## Lab 2

Write a program using:

```c
open()

read()

write()

close()
```

Trace it using:

```bash
strace
```

---

## Lab 3

Run:

```bash
man 2 read
```

Notice:

Section 2 = System Calls.

---

## Lab 4

Compare:

```bash
man 3 printf
```

and

```bash
man 2 write
```

Observe the difference between library functions and system calls.

---

## Lab 5

Research the x86-64 `syscall` instruction.

Understand how it transfers execution into the Linux kernel.

---

# Interview Questions

### What is a system call?

A controlled interface that allows user-space programs to request services from the Linux kernel.

---

### Why are system calls necessary?

Because user-space programs cannot directly access privileged hardware or kernel resources.

---

### What happens during a system call?

The CPU switches from user mode to kernel mode, executes kernel code, and then returns to user mode.

---

### Is `printf()` a system call?

No.

It is a C library function that typically uses `write()` internally.

---

### Why are system calls slower than normal function calls?

Because they require privilege transitions and kernel execution.

---

# Summary

Architecture:

```
User Program

↓

glibc

↓

System Call

↓

Linux Kernel

↓

Hardware
```

CPU Mode:

```
User Mode

↓

Kernel Mode

↓

User Mode
```

Examples:

| Operation | System Call |
|-----------|-------------|
| Read File | `read()` |
| Write File | `write()` |
| Create Process | `fork()` |
| Execute Program | `execve()` |
| Create Socket | `socket()` |
| Memory Mapping | `mmap()` |

## Key Takeaways

- System calls are the gateway between user space and the Linux kernel.
- User-space programs cannot directly access hardware.
- The CPU switches privilege levels during every system call.
- The C standard library wraps many system calls with higher-level APIs.
- Understanding system calls is essential for mastering Linux internals.
- Every major programming language ultimately relies on Linux system calls.

---

# Next Chapter

## Chapter 134 — Inside `read()` — Following One System Call Through the Entire Linux Kernel

You'll trace a single call:

```c
read(fd, buffer, 1024);
```

through every major Linux subsystem:

- glibc wrapper
- `syscall` instruction
- CPU privilege switch
- System call table
- Virtual File System (VFS)
- Filesystem driver
- Page cache
- Block layer
- Device driver
- SSD
- Return path

By the end, you'll understand exactly what happens when you read one byte from a file.