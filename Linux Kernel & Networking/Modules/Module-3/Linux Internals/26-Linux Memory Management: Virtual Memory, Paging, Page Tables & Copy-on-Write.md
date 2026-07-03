Most developers think memory is simple.

```java
int x = 10;
```

```javascript
let user = {};
```

```go
arr := make([]int, 100)
```

Memory gets allocated.

Done.

But inside Linux, memory management is one of the most advanced parts of the kernel.

Questions you may have:

- Why does every process think it owns all memory?
- What is Virtual Memory?
- What are Page Tables?
- What is a Page Fault?
- What is `mmap()`?
- Why is `fork()` so fast?
- What is Copy-on-Write?
- Why does Linux swap memory to disk?
- What happens when RAM becomes full?
- How does the OOM Killer decide what to kill?

The answer is:

**Linux Virtual Memory Management.**

Modern operating systems do **not** allow applications to access physical RAM directly.

Instead, Linux creates a virtual world for every process.

After this chapter, you'll understand one of the most important concepts in Computer Science.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 26 — Linux Memory Management: Virtual Memory, Paging, Page Tables & Copy-on-Write

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Virtual Memory exists
> - Physical Memory
> - Virtual Memory
> - Address Space
> - Memory Layout
> - Pages
> - Page Tables
> - MMU (Memory Management Unit)
> - TLB (Translation Lookaside Buffer)
> - Demand Paging
> - Page Faults
> - `mmap()`
> - Copy-on-Write (COW)
> - Swapping
> - OOM Killer

---

# 📖 Why Do We Need Virtual Memory?

Imagine your computer has:

```
16 GB RAM
```

Now imagine Chrome asks for:

```
Memory Address

0x1000
```

PostgreSQL also asks for:

```
0x1000
```

Docker also asks for:

```
0x1000
```

How can they all use the same address?

The answer is:

```
Virtual Memory
```

---

# Physical Memory

Physical Memory is the actual RAM installed in your computer.

```
RAM

↓

16 GB
```

The CPU ultimately reads and writes here.

---

# Problem Without Virtual Memory

Suppose two programs use:

```
Address

0x1000
```

Without isolation:

```
Chrome

↓

0x1000
```

```
PostgreSQL

↓

0x1000
```

They would overwrite each other's data.

System crashes.

---

# Virtual Memory

Linux gives every process its own virtual address space.

```
Chrome

↓

0x1000
```

```
Docker

↓

0x1000
```

```
PostgreSQL

↓

0x1000
```

All are valid,

because they are **virtual addresses**.

---

# Address Translation

Virtual addresses are translated into physical addresses.

```
Application

↓

Virtual Address

↓

MMU

↓

Physical Address

↓

RAM
```

Applications never see physical RAM directly.

---

# Every Process Gets Its Own Address Space

```
Process A

↓

Virtual Memory
```

```
Process B

↓

Virtual Memory
```

```
Process C

↓

Virtual Memory
```

Each process believes it owns memory.

In reality,

Linux maps them into shared physical RAM safely.

---

# Process Memory Layout

Typical process layout:

```
High Addresses

↓

Stack

↓

Memory Mappings

↓

Heap

↓

Data

↓

Code

↓

Low Addresses
```

Each region serves a different purpose.

---

# Code Segment

Contains:

```
Machine Instructions
```

Normally:

```
Read

+

Execute
```

Not writable.

---

# Data Segment

Contains:

```
Global Variables

Static Variables
```

---

# Heap

Used for dynamic allocation.

Examples:

```c
malloc()
```

```cpp
new
```

Java Objects

Go Objects

Node.js Objects

---

# Stack

Contains:

```
Function Calls

↓

Local Variables

↓

Return Addresses
```

Every thread has its own stack.

---

# Memory Pages

Linux does not manage memory byte-by-byte.

Instead,

it divides memory into:

```
Pages
```

Typical page size:

```
4 KB
```

---

# Example

Suppose:

```
16 KB Memory
```

Linux divides it into:

```
Page 1

↓

Page 2

↓

Page 3

↓

Page 4
```

---

# Why Pages?

Pages make memory management:

✔ Simpler

✔ Faster

✔ Easier to swap

✔ Easier to protect

---

# Page Tables

How does Linux know where a virtual page exists?

Using:

```
Page Table
```

Example:

```
Virtual Page

↓

Physical Page
```

Mappings are stored here.

---

# Example Page Table

| Virtual Page | Physical Page |
|--------------|---------------|
| 0 | 15 |
| 1 | 42 |
| 2 | 8 |
| 3 | 101 |

The application never knows these mappings.

---

# Memory Management Unit (MMU)

The MMU is hardware inside the CPU.

Its job:

```
Virtual Address

↓

Translate

↓

Physical Address
```

Every memory access passes through the MMU.

---

# TLB (Translation Lookaside Buffer)

Looking up Page Tables every time would be slow.

The CPU keeps a cache.

```
TLB

↓

Recent Address Translations
```

If found:

```
Fast Access
```

Otherwise:

```
Read Page Table
```

---

# TLB Flow

```
Virtual Address

↓

TLB

↓

Hit?

↓

Yes

↓

Physical Address
```

or

```
Miss

↓

Page Table

↓

Update TLB
```

---

# Demand Paging

Linux does **not** load all memory immediately.

Suppose a program requests:

```
1 GB
```

but uses:

```
10 MB
```

Linux loads only the pages actually used.

This is:

```
Demand Paging
```

---

# Page Fault

Suppose a program accesses:

```
Virtual Page

↓

Not Loaded
```

CPU raises:

```
Page Fault
```

Kernel handles it.

---

# Page Fault Flow

```
CPU

↓

Page Fault

↓

Kernel

↓

Allocate Page

↓

Load Data

↓

Resume Program
```

The application usually never notices.

---

# `mmap()`

Linux allows mapping files directly into memory.

```
Disk File

↓

mmap()

↓

Virtual Memory
```

The application accesses the file as if it were memory.

---

# Why `mmap()`?

Benefits:

✔ Faster File Access

✔ No Manual Buffer Management

✔ Shared Memory

✔ Large Files

Databases use `mmap()` extensively.

---

# Copy-on-Write (COW)

Suppose:

```
fork()
```

Should Linux copy:

```
2 GB RAM
```

Immediately?

Too slow.

Instead:

```
Parent

↓

Shared Pages

↑

Child
```

Both share memory.

---

# Copy-on-Write Flow

Initially:

```
Parent

↓

Shared Page

↑

Child
```

If the child writes:

```
Allocate New Page

↓

Copy Data

↓

Modify
```

Only modified pages are copied.

Huge performance improvement.

---

# Swapping

Suppose:

```
RAM Full
```

Linux moves inactive pages to:

```
Swap Partition

or

Swap File
```

```
RAM

↓

Swap
```

This frees physical memory.

---

# Problem with Swapping

RAM:

```
Nanoseconds
```

SSD:

```
Microseconds
```

Swap is much slower.

Heavy swapping causes:

```
System Slowdown
```

Sometimes called:

```
Thrashing
```

---

# OOM Killer

Suppose:

```
RAM Full

+

Swap Full
```

Linux has no memory left.

The kernel activates:

```
OOM Killer
```

---

# What Does the OOM Killer Do?

It evaluates running processes.

Factors include:

- Memory Usage
- Importance
- OOM Score

Then:

```
Choose Process

↓

Kill Process

↓

Free Memory
```

The goal is to keep the system alive.

---

# Why Memory Management Matters

Every application depends on it.

Examples:

- Java JVM
- Go Runtime
- Node.js
- Docker
- PostgreSQL
- Redis
- Chrome

Virtual Memory allows them to coexist safely.

---

# Hands-on Lab

## Lab 1 — View Memory Usage

```bash
free -h
```

---

## Lab 2 — View Virtual Memory Statistics

```bash
vmstat
```

---

## Lab 3 — View Process Memory Map

```bash
cat /proc/$$/maps
```

---

## Lab 4 — View Detailed Memory Info

```bash
cat /proc/meminfo
```

---

## Lab 5 — View Top Memory Consumers

```bash
top
```

or

```bash
htop
```

---

## Lab 6 — View Swap Usage

```bash
swapon --show
```

---

# Interview Questions

## Why is Virtual Memory used?

Virtual Memory isolates processes, provides a larger address space, and simplifies memory management.

---

## What is a Page?

A Page is a fixed-size block of virtual memory, typically 4 KB on many Linux systems.

---

## What is the purpose of the MMU?

The Memory Management Unit translates virtual addresses into physical addresses.

---

## What is the TLB?

The Translation Lookaside Buffer is a CPU cache that stores recent virtual-to-physical address translations.

---

## What is Demand Paging?

Demand Paging loads memory pages only when they are first accessed.

---

## What is a Page Fault?

A Page Fault occurs when a process accesses a page that is not currently mapped or loaded into physical memory.

---

## Why is Copy-on-Write efficient?

It delays copying memory until one of the processes modifies a shared page.

---

## What does the OOM Killer do?

The OOM Killer terminates one or more processes when the system runs out of available memory to prevent a complete system failure.

---

# Summary

Linux provides each process with its own virtual memory space while efficiently managing physical RAM.

```
Application

↓

Virtual Address

↓

MMU

↓

Page Table

↓

Physical Address

↓

RAM
```

Key concepts:

- Virtual Memory isolates processes.
- Pages divide memory into fixed-size units.
- Page Tables map virtual pages to physical pages.
- The MMU performs address translation.
- The TLB accelerates address translation.
- Demand Paging loads memory only when needed.
- Page Faults trigger kernel-managed page loading.
- `mmap()` maps files directly into memory.
- Copy-on-Write makes `fork()` efficient.
- Swapping extends memory using disk.
- The OOM Killer protects the system when memory is exhausted.

Understanding Linux memory management is essential before studying NUMA, kernel allocators, containers, networking performance, and advanced Linux kernel internals.

---

# Next Chapter

## Chapter 27 — Linux Networking Stack: From Ethernet Frame to User Application

We'll explore:

- Why the Linux Networking Stack exists
- Network Stack Layers
- Network Interface Cards (NICs)
- Ethernet Frames
- ARP
- IP
- Routing
- TCP
- UDP
- Sockets
- `sk_buff` (`skb`)
- Netfilter
- Packet Flow Through the Linux Kernel

> **This chapter begins the networking-focused part of the roadmap and serves as the foundation for Docker networking, Kubernetes networking, firewalls, and high-performance networking.**