# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 89 — Building a Simple Memory Allocator (`Mini malloc`)

---

# Most beginners think:

> "`malloc()` is a magic function built into C."

It isn't.

`malloc()` is **just another function**.

Someone wrote it.

In fact,

every operating system,

every C library,

and every language runtime contains its own memory allocator.

Examples:

- glibc (`malloc`)
- musl (`malloc`)
- jemalloc
- tcmalloc
- mimalloc

All of them solve the same problem:

> **"Given a request for memory, which free block should I return?"**

Today,

we'll build a simplified version to understand the core ideas.

---

# Learning Objectives

After this chapter you will understand:

- How `malloc()` works internally
- Heap manager
- Block metadata
- Free list
- Allocation algorithms
- Block splitting
- Block merging
- Fragmentation

---

# The Big Picture

When your program calls:

```c
malloc(100);
```

The flow is roughly:

```
Program

↓

malloc()

↓

Memory Allocator

↓

Find Free Block

↓

Return Pointer
```

No magic.

Just an algorithm.

---

# Imagine a Hotel

Suppose a hotel has:

```
Room 1

Room 2

Room 3

Room 4
```

Guest arrives.

Needs:

```
One Room
```

Reception checks:

```
Which Room Is Free?
```

Assigns it.

Memory allocation works the same way.

---

# Heap Before Allocation

Imagine the heap:

```
+----------------------+
| Free                 |
+----------------------+
| Free                 |
+----------------------+
| Free                 |
+----------------------+
| Free                 |
+----------------------+
```

Nothing allocated yet.

---

# First Allocation

Program:

```c
malloc(32);
```

Allocator:

```
Find Free Block

↓

Reserve It
```

Heap:

```
+----------------------+
| Used (32 Bytes)      |
+----------------------+
| Free                 |
+----------------------+
| Free                 |
+----------------------+
| Free                 |
+----------------------+
```

---

# Block Metadata

Every allocation contains hidden information.

Conceptually:

```
+----------------------+
| Metadata             |
+----------------------+
| User Memory          |
+----------------------+
```

Metadata typically stores:

- Block Size
- Allocation Status
- Links to Other Blocks

Example:

```
Size = 32

Allocated = Yes
```

The user never sees this.

---

# User View

Program:

```c
char *p =
malloc(32);
```

Program sees:

```
Pointer

↓

User Memory
```

The metadata exists just before it.

```
Metadata

↓

Returned Pointer
```

---

# Free List

How does the allocator know which blocks are available?

It keeps a:

```
Free List
```

Conceptually:

```
Free Block

↓

Free Block

↓

Free Block
```

Usually implemented as a linked list.

---

# Allocation Algorithm

Request:

```
64 Bytes
```

Allocator:

```
Start

↓

Check Block 1

↓

Too Small

↓

Check Block 2

↓

Large Enough

↓

Allocate
```

---

# First-Fit Algorithm

One simple strategy:

```
Take The First Block

That Fits
```

Visualization:

```
Block 1

↓

Too Small

Block 2

↓

Fits

↓

Use It
```

Fast and easy.

---

# Best-Fit Algorithm

Another strategy:

```
Find The Smallest Block

That Fits
```

May reduce wasted space,

but requires searching more blocks.

---

# Worst-Fit Algorithm

Another possibility:

```
Use Largest Free Block
```

Rarely used in general-purpose allocators,

but useful in some specialized systems.

---

# Splitting Blocks

Suppose:

Need:

```
32 Bytes
```

Free block:

```
100 Bytes
```

Instead of wasting:

```
68 Bytes
```

Allocator splits it.

Before:

```
100 Bytes Free
```

After:

```
32 Used

↓

68 Free
```

Much more efficient.

---

# Visualizing Split

Before:

```
+----------------------+
| Free (100 Bytes)     |
+----------------------+
```

After:

```
+----------------------+
| Used (32)            |
+----------------------+
| Free (68)            |
+----------------------+
```

---

# Freeing Memory

Program:

```c
free(ptr);
```

Allocator:

```
Used

↓

Free
```

Heap:

```
Used

↓

Free

↓

Used
```

---

# Fragmentation

Suppose:

```
Used

Free

Used

Free

Used
```

Many small free blocks exist.

Need:

```
100 Bytes
```

Problem:

No single block is large enough,

even though the total free memory might be sufficient.

This is called:

```
Fragmentation
```

---

# External Fragmentation

Visualization:

```
20 Free

↓

30 Free

↓

10 Free

↓

40 Free
```

Need:

```
60 Bytes
```

Total free:

```
100 Bytes
```

Largest block:

```
40 Bytes
```

Allocation fails.

---

# Merging Blocks (Coalescing)

Suppose:

```
Free

↓

Free
```

Adjacent blocks.

Allocator merges them.

Before:

```
20 Free

↓

30 Free
```

After:

```
50 Free
```

This process is called:

```
Coalescing
```

It reduces fragmentation.

---

# Simplified Block Structure

```c
typedef struct Block
{
    size_t size;

    int free;

    struct Block *next;

} Block;
```

Each block stores:

- Size
- Free/Used flag
- Link to next block

A real allocator contains additional metadata and optimizations.

---

# Mini Allocation Logic

Pseudo code:

```
Request Memory

↓

Search Free List

↓

Found?

↓

Yes

↓

Split If Needed

↓

Mark Used

↓

Return Pointer

----------------

No

↓

Request More Memory

↓

Add New Block

↓

Return Pointer
```

---

# Requesting More Memory

If no block fits,

the allocator asks the operating system for additional memory.

On Unix-like systems,

modern allocators commonly use:

- `mmap()`
- and sometimes `brk()`/`sbrk()` (legacy)

We'll study these system calls later.

---

# Real-World Example — glibc

glibc's allocator manages:

- Free bins
- Fast bins
- Thread caches
- Large allocations

Much more sophisticated than our mini allocator,

but based on the same core ideas.

---

# Real-World Example — jemalloc

Designed for:

- Web Servers
- Databases
- Browsers

Optimizes:

- Fragmentation
- Scalability
- Multithreading

---

# Real-World Example — Go Runtime

Go's runtime contains its own allocator.

When you write:

```go
make()
```

or

```go
new()
```

the runtime manages memory using allocator algorithms internally.

---

# Common Mistakes

---

## Thinking `malloc()` Talks Directly to RAM

Usually,

it talks to the memory allocator,

which only occasionally requests more memory from the operating system.

---

## Forgetting Metadata Exists

User memory is only part of the allocated block.

The allocator stores hidden bookkeeping information nearby.

---

## Ignoring Fragmentation

Even with plenty of free memory,

fragmentation can prevent large allocations.

---

## Assuming `free()` Returns Memory Directly to the OS

Often,

`free()` simply returns the block to the allocator for future reuse.

Whether memory is returned to the operating system depends on the allocator and allocation size.

---

# Mini Allocator Example (Conceptual)

```c
typedef struct Block
{
    size_t size;
    int free;
    struct Block *next;
} Block;

/*
    Search the free list.

    If a block fits:

        Mark it used.

    Otherwise:

        Request more memory.

    Return pointer to user data.
*/
```

A production allocator is much more complex,

but the core workflow is the same.

---

# Hands-on Labs

## Lab 1

Implement a simple linked list representing free memory blocks.

---

## Lab 2

Implement a first-fit search algorithm.

---

## Lab 3

Split a large block into:

- Used block
- Remaining free block

---

## Lab 4

Merge adjacent free blocks after freeing them.

---

## Lab 5

Visualize heap state after a sequence of allocations and frees.

---

# Interview Questions

### What does a memory allocator do?

It manages heap memory by allocating and freeing memory blocks.

---

### What is block metadata?

Hidden information stored with each allocation that helps the allocator manage memory.

---

### What is a free list?

A data structure that tracks available memory blocks.

---

### What is fragmentation?

A situation where free memory is divided into many small pieces, making large allocations difficult.

---

### What is coalescing?

Merging adjacent free blocks into a larger block.

---

# Summary

```
Program

↓

malloc()

↓

Memory Allocator

↓

Search Free List

↓

Found?

↓

Split Block

↓

Mark Used

↓

Return Pointer
```

When Freeing:

```
free()

↓

Mark Free

↓

Merge Adjacent Blocks

↓

Reuse Later
```

## Key Takeaways

- `malloc()` is implemented using memory allocation algorithms.
- Allocators manage hidden metadata for every memory block.
- Free lists track available memory.
- Blocks may be split during allocation and merged during deallocation.
- Fragmentation is a major challenge in memory management.
- Modern allocators build on these same core principles with advanced optimizations.

---

# 🎉 Module 10 Complete

Congratulations!

You now understand:

- ✅ Process Memory Layout
- ✅ Stack vs Heap
- ✅ `malloc()`
- ✅ `calloc()`
- ✅ `realloc()`
- ✅ `free()`
- ✅ Memory Leaks
- ✅ Dangling Pointers
- ✅ Double Free
- ✅ Heap Corruption
- ✅ Dynamic Arrays
- ✅ Dynamic Strings
- ✅ Memory Alignment & Padding
- ✅ Memory Debugging (Valgrind & AddressSanitizer)
- ✅ How `malloc()` Works Internally

These topics form the foundation of:

- Operating Systems
- Linux Kernel
- Compilers
- Databases
- Browsers
- Game Engines
- Embedded Systems
- High-Performance Servers

---

# 🚀 Next Module

# Module 11 — Advanced C & System Programming Foundations

You'll learn:

- Function Pointers
- Callback Functions
- Generic Programming with `void *`
- `typedef` Advanced Usage
- Bit Manipulation
- Bit Fields
- Macros (Advanced)
- Conditional Compilation
- Assertions
- `volatile`
- `const` Deep Dive
- `restrict`
- Inline Functions
- Variable Argument Functions (`stdarg.h`)
- Building Generic Libraries

This module bridges the gap between core C programming and professional systems programming, preparing you for Linux kernel development, networking stacks, embedded systems, and high-performance software.