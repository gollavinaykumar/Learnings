Most developers know that programs use memory.

They've heard terms like:

- Stack Overflow
- Heap Memory
- RAM
- Virtual Memory
- Memory Leak

But very few understand **where variables actually live** or **how the operating system gives memory to programs**.

Every application—Chrome, VS Code, Docker, Java, Node.js, PostgreSQL—relies on memory management.

If you understand memory, you'll later understand:

- Linux Kernel Memory Manager
- JVM Memory
- Node.js V8 Memory
- Docker Memory Limits (cgroups)
- Kubernetes Memory Requests & Limits
- Database Buffer Pools
- Memory Leaks
- Out Of Memory (OOM)

After this chapter, you'll understand how computer memory works from hardware to applications.

---

# 👑 Linux Kernel & Networking Mastery

# Module 1 — Computer Fundamentals

# Chapter 3 — Memory: RAM, ROM, Stack, Heap & Virtual Memory

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What memory is
> - Why memory is needed
> - RAM
> - ROM
> - Stack Memory
> - Heap Memory
> - Program Memory Layout
> - Static Memory
> - Dynamic Memory
> - Virtual Memory
> - Paging
> - Swap Memory
> - Memory Addresses
> - Memory Allocation
> - Memory Leaks
> - Out Of Memory (OOM)
> - How programs use memory

---

# 📖 What is Memory?

Memory is where the computer stores information **while it is working**.

Imagine you're solving a math problem.

You don't keep every number in your head forever.

You write them on paper.

That paper is like computer memory.

Without memory:

```
CPU

↓

Cannot Remember Anything
```

---

# Why Does the CPU Need Memory?

The CPU is extremely fast.

But it cannot remember everything.

Example:

```
int a = 10;

int b = 20;

int c = a + b;
```

Where are these values stored?

```
CPU

↓

Memory
```

The CPU continuously reads from and writes to memory.

---

# Types of Memory

```
Computer Memory

│

├── ROM

│

└── RAM
```

---

# ROM (Read Only Memory)

ROM stores instructions needed to start the computer.

Example:

```
Power On

↓

BIOS / UEFI

↓

Hardware Check

↓

Load Operating System
```

ROM keeps its contents even after power is turned off.

It is permanent memory.

---

# RAM (Random Access Memory)

RAM stores programs that are currently running.

Example:

```
Open Chrome

↓

Chrome Loads into RAM

↓

CPU Executes Chrome
```

Close Chrome:

```
RAM

↓

Memory Released
```

Turn off the computer:

Everything inside RAM disappears.

RAM is temporary memory.

---

# RAM vs ROM

| RAM | ROM |
|------|------|
| Temporary | Permanent |
| Read & Write | Mostly Read Only |
| Fast | Slower |
| Stores Running Programs | Stores Boot Instructions |
| Data Lost After Shutdown | Data Remains |

---

# Program Memory Layout

When a program starts, the operating system divides its memory into sections.

```
+----------------------+

|       Stack          |

+----------------------+

|                      |

|       Heap           |

|                      |

+----------------------+

| Initialized Data     |

+----------------------+

| Uninitialized Data   |

+----------------------+

| Program Code         |

+----------------------+
```

Each section has a different purpose.

---

# Program Code (Text Segment)

This contains the machine instructions.

Example:

```java
System.out.println("Hello");
```

After compilation:

```
Machine Instructions

↓

Stored Here
```

This section usually cannot be modified.

---

# Initialized Data Segment

Stores global variables that already have values.

Example:

```c
int count = 10;
```

---

# Uninitialized Data Segment (BSS)

Stores global variables without initial values.

Example:

```c
int count;
```

The operating system initializes them to zero.

---

# Stack Memory

The stack stores:

- Function calls
- Local variables
- Parameters
- Return addresses

Example:

```java
void test() {
    int x = 5;
}
```

Variable:

```
x
```

is stored on the stack.

---

# Stack Growth

Every function call creates a new stack frame.

```
main()

↓

login()

↓

validate()

↓

encrypt()
```

Memory:

```
+-------------+

encrypt()

+-------------+

validate()

+-------------+

login()

+-------------+

main()

+-------------+
```

When a function returns, its stack frame disappears.

---

# Heap Memory

The heap stores data created during runtime.

Example:

```java
new User();
```

Example:

```js
const obj = {};
```

These objects live in the heap.

Unlike the stack, heap memory is managed dynamically.

---

# Stack vs Heap

| Stack | Heap |
|--------|------|
| Automatic | Dynamic |
| Fast | Slower |
| Small | Large |
| Stores Local Variables | Stores Objects |
| Automatically Freed | Freed by GC or Programmer |

---

# Static Memory

Variables that exist throughout the program.

Example:

```java
static int counter = 0;
```

Allocated once.

Destroyed when the program exits.

---

# Dynamic Memory

Allocated while the program is running.

Example:

```java
new Employee();
```

The operating system provides memory as needed.

---

# Memory Addresses

Every byte of memory has an address.

Example:

```
0x1000

0x1001

0x1002

0x1003
```

Think of memory as a huge apartment building.

Each apartment has its own number.

---

# Pointers (Concept)

A pointer stores a memory address.

Example:

```
Address

↓

0x7ffe1234
```

Languages like C use pointers directly.

Java and JavaScript hide them from developers.

---

# Virtual Memory

Suppose your laptop has:

```
16 GB RAM
```

But all running programs together need:

```
20 GB
```

How?

The operating system creates **Virtual Memory**.

```
Program

↓

Virtual Address

↓

Operating System

↓

Physical RAM
```

Programs believe they have their own large memory space.

---

# Physical Memory vs Virtual Memory

```
Application

↓

Virtual Memory

↓

Operating System

↓

Physical RAM
```

Applications never directly access RAM.

The operating system translates addresses.

---

# Paging

Memory is divided into fixed-size blocks.

```
Virtual Memory

↓

Pages

↓

Physical Memory

↓

Frames
```

Example:

```
Page 1

↓

Frame 5

Page 2

↓

Frame 10
```

This mapping is managed by the operating system.

---

# Page Table

The operating system keeps a table.

```
Virtual Page

↓

Physical Frame
```

Whenever the CPU accesses memory, this mapping is used.

---

# Page Fault

Suppose a program requests data.

But the page is not in RAM.

```
CPU

↓

Memory Request

↓

Page Missing

↓

Page Fault

↓

Load From Disk

↓

Continue Execution
```

This process is slower than accessing RAM directly.

---

# Swap Memory

If RAM becomes full:

```
RAM

↓

Move Less Used Pages

↓

Disk (Swap Area)
```

Now RAM has space for active programs.

Swap is much slower than RAM.

---

# Memory Allocation

Suppose:

```java
User user = new User();
```

Steps:

```
Program Requests Memory

↓

Operating System

↓

Heap Allocation

↓

Address Returned

↓

Program Uses Object
```

---

# Memory Leak

Suppose a program continuously allocates memory.

But never releases it.

```
Allocate

↓

Allocate

↓

Allocate

↓

Allocate

↓

RAM Full
```

This is called a Memory Leak.

---

# Out Of Memory (OOM)

When no memory remains:

```
Program Requests Memory

↓

Operating System

↓

No Free Memory

↓

OOM Error
```

Linux may invoke the **OOM Killer** to terminate a process and recover memory.

We'll study this in the Linux Kernel module.

---

# Real Example — Opening Chrome

```
Click Chrome

↓

Read Program From SSD

↓

Load into RAM

↓

Create Stack

↓

Create Heap

↓

Allocate Objects

↓

CPU Executes Instructions
```

Everything a running application does depends on memory.

---

# Why This Matters

Understanding memory is essential for:

- Java Garbage Collection
- Node.js V8 Engine
- Docker Memory Limits
- Kubernetes Resource Management
- PostgreSQL Buffer Cache
- Redis
- Linux Kernel Memory Manager

All of these build on the concepts in this chapter.

---

# Hands-on Lab

## Lab 1 — View RAM Usage

### Linux

```bash
free -h
```

### macOS

```bash
vm_stat
```

---

## Lab 2 — View Process Memory

### Linux

```bash
ps aux
```

### macOS

```bash
ps aux
```

Observe how each running process consumes memory.

---

## Lab 3 — Monitor Memory Continuously

### Linux

```bash
top
```

### macOS

```bash
top
```

---

## Lab 4 — View Virtual Memory Statistics

### Linux

```bash
cat /proc/meminfo
```

### macOS

```bash
vm_stat
```

---

# Interview Questions

## Why is RAM needed?

RAM stores programs and data while they are running so the CPU can access them quickly.

---

## What is the difference between Stack and Heap?

The Stack stores function calls and local variables.

The Heap stores dynamically allocated objects.

---

## What is Virtual Memory?

Virtual Memory allows each process to have its own logical address space while the operating system maps it to physical RAM.

---

## What is a Page Fault?

A Page Fault occurs when the requested memory page is not currently in RAM and must be loaded from disk.

---

## What is a Memory Leak?

A Memory Leak happens when allocated memory is never released, eventually exhausting available memory.

---

## What is the Linux OOM Killer?

The OOM Killer is a Linux kernel mechanism that terminates processes when the system runs out of memory.

---

# Summary

Memory allows the CPU to execute programs efficiently.

```
SSD

↓

RAM

↓

Program Memory

↓

Stack

↓

Heap

↓

CPU
```

Key concepts:

- RAM stores running programs
- ROM stores boot instructions
- Stack stores function calls and local variables
- Heap stores dynamically allocated objects
- Every memory location has an address
- Virtual Memory gives each process its own address space
- Paging maps virtual pages to physical frames
- Swap extends available memory using disk
- Memory leaks waste memory
- OOM occurs when no memory is available

Understanding memory is the foundation for mastering operating systems, Linux internals, JVM, Docker, Kubernetes, databases, and high-performance software.

---

# Next Chapter

## Chapter 4 — CPU Architecture: Cores, Threads, Pipelines & Context Switching

We'll explore:

- CPU cores
- Hardware threads
- Hyper-Threading / SMT
- Instruction pipelines
- Branch prediction
- CPU scheduling basics
- Context switching
- Multi-core execution
- CPU affinity
- How operating systems utilize modern processors