# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 113 — Process Memory Layout (How Linux Organizes Every Process)

---

# Most beginners think:

> "RAM is just one big block of memory."

It isn't.

Every Linux process gets its own **virtual address space**, and the operating system divides it into different regions.

When you write:

```c
int x = 10;

char name[20];

int *p = malloc(100);
```

These variables **do not** live in the same place.

They are stored in different memory regions.

Understanding these regions is one of the biggest steps toward becoming a Linux systems programmer.

---

# Learning Objectives

After this chapter you will understand:

- Virtual memory
- Process address space
- Text segment
- Data segment
- BSS
- Heap
- Stack
- Command-line arguments
- Environment variables
- Memory layout after `fork()`

---

# Every Process Gets Its Own Memory

Suppose two programs are running.

```
Chrome

↓

Virtual Memory

---------------------

VS Code

↓

Virtual Memory
```

Each process believes it owns its own memory.

In reality,

the operating system maps virtual addresses to physical RAM.

---

# High-Level Memory Layout

A typical Linux process looks like this:

```
High Memory
+------------------------+
| Stack                  |
+------------------------+
|                        |
|        Free Space      |
|                        |
+------------------------+
| Heap                   |
+------------------------+
| BSS                    |
+------------------------+
| Initialized Data       |
+------------------------+
| Text (Code)            |
+------------------------+
Low Memory
```

> **Note:** The exact addresses and arrangement vary between operating systems, CPU architectures, and features such as ASLR.

---

# Why Multiple Segments?

Different kinds of data have different lifetimes.

Example:

```
Machine Code

↓

Never Changes

-------------------

Global Variables

↓

Whole Program

-------------------

malloc()

↓

Dynamic

-------------------

Function Variables

↓

Temporary
```

Each belongs in a different region.

---

# Text Segment (Code Segment)

Contains:

```
Machine Instructions
```

Example:

```c
int add(int a, int b)
{
    return a + b;
}
```

The compiled instructions are stored here.

---

# Properties

The text segment is typically:

- Readable
- Executable
- Not writable

This helps protect program code from accidental modification.

---

# Initialized Data Segment

Contains:

Global and static variables **with explicit initial values**.

Example:

```c
int count = 10;

static int age = 25;
```

Stored in:

```
Initialized Data
```

---

# BSS Segment

BSS stands for:

```
Block Started by Symbol
```

Contains:

Global and static variables **without explicit initialization**, or initialized to zero.

Example:

```c
int total;

static int value;
```

Conceptually:

```
Zero Initialized
```

before `main()` begins.

---

# Why Separate BSS?

Suppose:

```c
int huge[1000000];
```

If all zeros were stored in the executable,

the executable would become enormous.

Instead,

the executable simply records:

```
Allocate

↓

Zero Memory

↓

At Program Startup
```

Much more efficient.

---

# Heap

The heap stores:

```
Dynamic Memory
```

Allocated with:

```c
malloc()

calloc()

realloc()
```

Example:

```c
int *p =
malloc(100);
```

Memory comes from:

```
Heap
```

---

# Heap Growth

Conceptually:

```
Text

↓

Data

↓

BSS

↓

Heap

↑

Growing Up
```

As more memory is allocated,

the heap expands.

---

# Stack

Every function call creates:

```
Stack Frame
```

Example:

```c
void fun()
{
    int x = 10;
}
```

Variable:

```
x

↓

Stack
```

---

# Stack Growth

Conceptually:

```
High Memory

↓

Stack

↓

Growing Down
```

The stack and heap grow toward one another.

---

# Stack Frame

Calling:

```c
main()

↓

fun()

↓

another()
```

Creates:

```
another()

↓

fun()

↓

main()
```

Each function has its own stack frame containing:

- Local variables
- Return address
- Saved registers (implementation-dependent)
- Function parameters (depending on the calling convention)

---

# Stack vs Heap

Stack:

```
Automatic

↓

Fast

↓

Temporary
```

Heap:

```
Manual

↓

malloc()

↓

free()
```

You control its lifetime.

---

# Command-Line Arguments

Program:

```bash
./app hello world
```

Memory contains:

```
argv[0]

↓

"./app"

argv[1]

↓

"hello"

argv[2]

↓

"world"
```

These strings exist in the process's address space.

---

# Environment Variables

Example:

```bash
PATH

HOME

USER
```

These are also stored in the process's virtual memory.

Program:

```c
int main(
int argc,
char *argv[],
char *envp[])
```

can access them.

We'll explore this in a dedicated chapter.

---

# Free Space

Between:

```
Heap

↓

Free Space

↓

Stack
```

The operating system leaves unused virtual address space,

allowing both heap and stack to grow.

---

# Virtual Memory

Suppose:

Program sees:

```
0x7ffe...

↓

Stack
```

This is a **virtual address**.

The kernel translates it into:

```
Physical RAM
```

using the memory management hardware.

---

# Memory After `fork()`

Before:

```
Parent

↓

Text

↓

Data

↓

Heap

↓

Stack
```

After:

```
Parent

↓

Own Virtual Memory

---------------------

Child

↓

Own Virtual Memory
```

Initially,

many pages are shared using:

```
Copy-On-Write
```

---

# Memory After `exec()`

Suppose:

```
Program A

↓

exec()

↓

Program B
```

Old memory layout disappears.

New program receives:

```
New Text

↓

New Data

↓

New Heap

↓

New Stack
```

Everything is rebuilt.

---

# Real-World Example — Web Browser

Browser:

```
Code

↓

Text

Cache

↓

Heap

Functions

↓

Stack
```

Each tab has its own process and memory layout.

---

# Real-World Example — Database

Database:

```
Indexes

↓

Heap

Parser

↓

Stack

Code

↓

Text
```

Millions of allocations occur on the heap.

---

# Real-World Example — Linux Kernel

Each user process has:

```
Own Virtual Memory
```

The kernel maintains separate memory mappings for every process.

---

# Real-World Example — Docker

Every container process has:

```
Own Virtual Address Space
```

The kernel isolates process memory just as it does for ordinary Linux processes.

---

# Common Mistakes

---

## Thinking Heap and Stack Are the Same

They serve different purposes and have different lifetimes.

---

## Forgetting `free()`

Heap memory remains allocated until:

```c
free()
```

or the process exits.

---

## Assuming Global Variables Are on the Stack

Global and static variables live in the data or BSS segments,

not on the stack.

---

## Thinking Virtual Addresses Are Physical RAM

Applications see virtual addresses.

The kernel and CPU translate them to physical memory.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int global = 10;

int main()
{
    int local = 20;

    int *heap =
        malloc(sizeof(int));

    *heap = 30;

    printf("%p\n", (void *)&global);
    printf("%p\n", (void *)&local);
    printf("%p\n", (void *)heap);

    free(heap);

    return 0;
}
```

Notice that the addresses typically belong to different memory regions.

---

# Hands-on Labs

## Lab 1

Print the addresses of:

- Global variable
- Static variable
- Local variable
- Heap allocation

Compare their locations.

---

## Lab 2

Allocate increasing amounts of heap memory.

Observe memory growth using Linux tools.

---

## Lab 3

Write a recursive function.

Observe how the stack grows with each call.

---

## Lab 4

Use:

```bash
cat /proc/self/maps
```

(on Linux)

Inspect the memory layout of a running process.

---

## Lab 5

Run the program multiple times.

Notice that addresses may change because of:

```
ASLR

(Address Space Layout Randomization)
```

---

# Interview Questions

### What is the text segment?

The region containing the program's executable machine code.

---

### What is stored in the data segment?

Initialized global and static variables.

---

### What is the BSS segment?

Uninitialized (or zero-initialized) global and static variables.

---

### What is the difference between the heap and the stack?

The heap stores dynamically allocated memory.

The stack stores function call frames and automatic local variables.

---

### What happens to memory after `exec()`?

The old process image is replaced with a completely new memory layout for the new program.

---

# Summary

Typical Process Memory Layout:

```
High Memory
+------------------------+
| Stack                  |
+------------------------+
| Free Space             |
+------------------------+
| Heap                   |
+------------------------+
| BSS                    |
+------------------------+
| Initialized Data       |
+------------------------+
| Text (Code)            |
+------------------------+
Low Memory
```

Memory Types:

| Region | Stores |
|---------|--------|
| Text | Machine code |
| Data | Initialized globals/statics |
| BSS | Zero/uninitialized globals/statics |
| Heap | `malloc()` allocations |
| Stack | Function frames and local variables |

## Key Takeaways

- Every process has its own virtual address space.
- Different types of data are stored in different memory regions.
- The stack is automatic and temporary; the heap is dynamic and manually managed.
- The BSS stores zero-initialized globals without increasing executable size.
- `fork()` initially shares memory pages using Copy-on-Write.
- `exec()` completely replaces the process memory layout.

---

# Next Chapter

## Chapter 114 — Virtual Memory (How Every Process Thinks It Owns the Entire RAM)

You'll learn:

- Virtual vs physical memory
- Page tables
- MMU (Memory Management Unit)
- Address translation
- Page faults
- Demand paging
- Swap memory
- Why two processes can both use the same virtual address without conflict