# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 48 — Memory Fragmentation

---

## Most beginners think:

> "If my computer has 8 GB of free RAM, I can always allocate a 4 GB block."

Unfortunately, that's not always true.

Imagine your bookshelf.

You have plenty of free space,

but it's scattered into many small gaps.

Although the total free space is large,

there isn't one **continuous space** large enough for a new book.

Memory works exactly the same way.

This problem is called **Memory Fragmentation**.

It is one of the biggest challenges faced by:

- Operating Systems
- Memory Allocators
- Databases
- Browsers
- Game Engines
- Embedded Systems

---

# Learning Objectives

After this chapter you will understand:

- What memory fragmentation is
- Why fragmentation happens
- Internal Fragmentation
- External Fragmentation
- How memory allocators work
- Why fragmentation hurts performance
- How modern allocators reduce fragmentation
- Real-world examples

---

# Imagine a Parking Lot

Suppose a parking lot has:

```text
□□□□□□□□□□□□□□□□□□□□□□□□
```

Every block is free.

Now cars start arriving.

```text
■■□□□□■■■□□□■■□□□□■□□
```

Some cars leave.

```text
■■□□□□□□□■■□□□■■□□□□□
```

You now have:

- Lots of free parking spaces
- But scattered everywhere

A bus arrives.

Needs:

```text
□□□□□□□□
```

Eight consecutive spaces.

Even though many spaces are free,

the bus can't park.

This is fragmentation.

---

# What is Memory Fragmentation?

Memory fragmentation happens when:

```text
Allocated Memory

↓

Freed

↓

Allocated Again

↓

Many Small Free Blocks
```

Instead of one large free block,

memory becomes fragmented.

---

# Memory Layout

Initially:

```text
□□□□□□□□□□□□□□□□□□□□□□□□
```

Allocate:

```text
AAAA BBBB CCCC DDDD
```

Memory:

```text
AAAABBBBCCCCDDDD
```

Free:

```text
BBBB
```

Memory:

```text
AAAA____CCCCDDDD
```

Allocate:

```text
EE
```

Memory:

```text
AAAAEE__CCCCDDDD
```

Now the remaining free space is split into many small pieces.

---

# Why Is Fragmentation a Problem?

Suppose you need:

```text
████████
```

Eight continuous blocks.

Memory:

```text
██__██__██__██__
```

Total free memory:

```text
8 Blocks
```

Continuous memory:

```text
2 Blocks
```

Allocation fails.

---

# Two Types of Fragmentation

---

# 1. Internal Fragmentation

Memory allocator gives you **more memory than requested**.

Example:

You request:

```text
30 Bytes
```

Allocator uses fixed blocks:

```text
32 Bytes
```

Result:

```text
30 Used

2 Wasted
```

Those extra 2 bytes are **internal fragmentation**.

---

# Visualization

Requested:

```text
□□□□□□□□□□□□□□
```

Allocated:

```text
□□□□□□□□□□□□□□□□
```

Unused:

```text
■■
```

Waste exists **inside** the allocated block.

---

# 2. External Fragmentation

Memory outside allocations becomes scattered.

Example:

```text
AA__BB___CC__DD
```

Free space exists,

but it's separated.

Large allocations may fail.

---

# Internal vs External

| Internal Fragmentation | External Fragmentation |
|-------------------------|------------------------|
| Waste inside allocated blocks | Waste between allocated blocks |
| Caused by fixed-size allocations | Caused by scattered free space |
| Memory is allocated but partially unused | Enough total memory exists, but not contiguously |

---

# How Does Fragmentation Happen?

Example:

```text
malloc()

↓

malloc()

↓

free()

↓

malloc()

↓

free()

↓

malloc()
```

Thousands of allocations and deallocations slowly scatter free memory.

Long-running applications experience this naturally.

---

# Why Is It Bad?

Fragmentation causes:

- More memory usage
- Slower allocations
- Allocation failures
- Poor cache performance
- Lower throughput

---

# Real-World Example

Imagine Redis running for six months.

Millions of keys are:

```text
Created

↓

Deleted

↓

Created Again

↓

Deleted Again
```

The heap becomes fragmented.

Redis may use significantly more memory than the actual data requires.

---

# How Do Memory Allocators Reduce Fragmentation?

Modern allocators use techniques such as:

- Free lists
- Memory pools
- Segregated size classes
- Slab allocation
- Buddy allocator
- Compaction (when supported)

Their goal is to keep free memory organized.

---

# Buddy Allocator (Concept)

Memory:

```text
1024 KB
```

Split:

```text
512

512
```

Split again:

```text
256

256

512
```

When neighboring free blocks become available,

they are merged back together.

This reduces fragmentation.

Linux uses buddy allocation for physical memory management.

---

# Memory Pools

Instead of many small allocations:

```text
malloc()

malloc()

malloc()
```

Programs allocate one large block.

Then divide it internally.

Benefits:

- Faster allocation
- Less fragmentation
- Better cache locality

Many game engines and databases use memory pools.

---

# Slab Allocation

Linux kernel often allocates:

```text
task_struct

inode

socket

buffer
```

Objects of the same size.

Instead of calling the general allocator repeatedly,

it uses slab caches.

Advantages:

- Faster allocation
- Reduced fragmentation
- Object reuse

---

# Real-World Uses

Fragmentation matters in:

- Linux Kernel
- Redis
- PostgreSQL
- Browsers
- Databases
- Embedded Systems
- High-performance servers

---

# Common Mistakes

## Many Tiny Allocations

Wrong:

```text
malloc(8)

malloc(12)

malloc(5)

malloc(3)
```

Thousands of tiny allocations increase fragmentation.

---

## Frequent Allocation and Free

Repeated allocation/free cycles increase fragmentation over time.

Whenever possible,

reuse memory.

---

## Ignoring Memory Pools

For performance-critical applications,

custom allocators are often better than repeated `malloc()` calls.

---

# Hands-on Labs

## Lab 1

Allocate and free many different-sized blocks.

Observe heap behavior using:

```bash
valgrind
```

---

## Lab 2

Build a simple memory pool.

Compare allocation speed.

---

## Lab 3

Allocate one million small objects.

Measure performance.

---

## Lab 4

Study the Linux Buddy Allocator.

Search:

```text
Linux Buddy Allocator
```

---

## Lab 5

Read about slab allocation in the Linux kernel.

---

# Interview Questions

### What is memory fragmentation?

Memory becoming divided into many small pieces, reducing allocation efficiency.

---

### What is internal fragmentation?

Unused space inside an allocated block.

---

### What is external fragmentation?

Free memory exists, but it's scattered into many small blocks.

---

### Why is fragmentation bad?

It wastes memory, slows allocation, and may prevent large allocations.

---

### How do modern allocators reduce fragmentation?

Using techniques like:

- Buddy allocation
- Slab allocation
- Memory pools
- Free lists
- Size classes

---

# Summary

```text
Allocate

↓

Free

↓

Allocate Again

↓

Scattered Free Blocks

↓

Fragmentation

↓

Slower Allocation
```

Key Takeaways:

- Fragmentation is a natural consequence of dynamic allocation.
- Internal fragmentation wastes space inside allocated blocks.
- External fragmentation scatters free memory.
- Long-running applications are most affected.
- Modern allocators use sophisticated algorithms to reduce fragmentation.
- Understanding fragmentation is essential for operating systems, databases, and high-performance software.

---

# Next Chapter

## Chapter 49 — Memory Alignment

You'll learn:

- Why CPUs prefer aligned memory
- Alignment requirements for different data types
- Structure padding
- Cache efficiency
- Misaligned memory access
- How alignment affects performance and system design