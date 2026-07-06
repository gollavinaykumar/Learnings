# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 137 — The Linux Page Cache — Why Reading the Same File Twice Is Much Faster

---

# Most beginners think:

> "Every `read()` goes to the SSD."

Reality:

```
read()

↓

RAM

↓

SSD (Only If Needed)
```

In fact,

most Linux file reads **never touch the disk**.

Instead,

they come from one of Linux's most important performance optimizations:

```
Page Cache
```

Without the page cache:

- Linux would be much slower
- Databases would suffer
- Browsers would lag
- Applications would constantly wait for disk I/O

---

# Learning Objectives

After this chapter you will understand:

- What the page cache is
- Cache hit
- Cache miss
- Dirty pages
- Write-back caching
- `fsync()`
- `sync()`
- Read-ahead
- Memory pressure
- Page eviction

---

# The Problem

Suppose you read:

```text
report.pdf
```

Traditional thinking:

```
Program

↓

SSD

↓

Program

↓

SSD

↓

Program
```

Every read touches the SSD.

Very slow.

---

# SSD vs RAM

Approximate latency:

```
CPU Register

↓

~1 ns

-------------------

RAM

↓

~100 ns

-------------------

SSD

↓

~50–100 µs

-------------------

HDD

↓

~5–10 ms
```

RAM is **orders of magnitude faster** than persistent storage.

---

# The Solution

Linux says:

```
Read Once

↓

Store In RAM

↓

Reuse Later
```

This RAM cache is:

```
Page Cache
```

---

# What is the Page Cache?

Definition:

> **The page cache is a kernel-managed memory cache that stores recently accessed file pages in RAM.**

It is transparent.

Applications don't need to know it exists.

---

# Visualization

```
Application

↓

read()

↓

Page Cache

↓

SSD (Only If Missing)
```

---

# First Read

Suppose:

```bash
cat notes.txt
```

Flow:

```
Application

↓

Page Cache

↓

Miss

↓

SSD

↓

RAM

↓

Application
```

---

# Second Read

Read the same file again.

Flow:

```
Application

↓

Page Cache

↓

Hit

↓

Application
```

No SSD access.

Much faster.

---

# Cache Hit

Definition:

```
Requested Data

↓

Already In RAM
```

Fast path.

---

# Visualization

```
Application

↓

Page Cache

↓

Found

↓

Return
```

---

# Cache Miss

Definition:

```
Requested Data

↓

Not In RAM
```

Kernel loads it from storage.

---

# Visualization

```
Application

↓

Page Cache

↓

Not Found

↓

SSD

↓

RAM

↓

Application
```

---

# What is a Page?

Linux manages memory in:

```
Pages
```

Typically:

```
4096 Bytes

(4 KB)
```

The page cache stores file data page by page,

not byte by byte.

---

# Example

Suppose:

```
File

↓

12 KB
```

Linux stores:

```
Page 1

↓

Page 2

↓

Page 3
```

---

# Read-Ahead

Suppose you read:

```
Page 1
```

Linux predicts:

```
You'll Probably Need

↓

Page 2

↓

Page 3
```

So it loads them early.

---

# Visualization

```
Application

↓

Read Page 1

↓

Kernel Prefetches

↓

Page 2

↓

Page 3
```

This improves sequential reads.

---

# Writing a File

Suppose:

```c
write(fd, data, size);
```

Does Linux immediately write to the SSD?

Usually:

```
No
```

---

# Dirty Pages

Instead:

```
Application

↓

Page Cache

↓

Modified

↓

Dirty Page
```

The page is marked:

```
Dirty
```

Meaning:

```
RAM Has New Data

↓

Disk Does Not
```

---

# Visualization

```
RAM

↓

Dirty

↓

Write Later
```

---

# Why Delay Disk Writes?

Because SSD writes are relatively expensive.

Linux groups writes together.

Benefits:

- Fewer I/O operations
- Better throughput
- Less device overhead

---

# Write-Back

Eventually:

```
Dirty Pages

↓

Background Flush

↓

SSD
```

Linux performs asynchronous write-back.

---

# `fsync()`

Sometimes you **must** ensure data reaches storage.

Example:

```c
fsync(fd);
```

Meaning:

```
Dirty Pages

↓

Write Immediately

↓

Wait Until Complete
```

Useful for databases and critical data.

---

# `sync()`

System-wide version.

```
All Dirty Pages

↓

Flush

↓

Storage
```

---

# Why Databases Use `fsync()`

Suppose:

```
Transfer Money

↓

Write Balance

↓

Power Failure
```

Without `fsync()`,

the data may still be only in RAM.

Databases often force writes before confirming transactions.

---

# Memory Pressure

RAM is limited.

Suppose:

```
Page Cache

↓

20 GB
```

Application needs memory.

Kernel must free some cached pages.

---

# Page Eviction

Kernel removes:

```
Least Useful Pages
```

Typically pages that have not been accessed recently.

---

# Visualization

```
Cache Full

↓

Evict Old Pages

↓

Load New Pages
```

---

# Clean vs Dirty Pages

Clean Page:

```
Same As Disk
```

Easy to discard.

Dirty Page:

```
Modified

↓

Must Be Written

↓

Before Removal
```

---

# Relationship with `mmap()`

Memory-mapped files also use the page cache.

```
mmap()

↓

Page Cache

↓

Storage
```

Both `read()` and `mmap()` benefit from caching.

---

# Relationship with VFS

Remember:

```
Application

↓

VFS

↓

Page Cache

↓

Filesystem
```

The page cache sits between the VFS/filesystem layer and the storage device.

---

# Real-World Example — Chrome

Images:

```
Read Once

↓

Page Cache

↓

Display Again

↓

No SSD
```

---

# Real-World Example — PostgreSQL

Database pages:

```
Query

↓

Database Buffer

↓

Page Cache

↓

SSD
```

Many databases maintain **their own cache** in addition to the kernel page cache.

---

# Real-World Example — Git

Running:

```bash
git status
```

repeatedly often becomes faster because filesystem metadata and file contents are already cached.

---

# Real-World Example — `cat`

First run:

```bash
cat bigfile.txt
```

Reads from storage.

Second run:

Often served almost entirely from RAM.

---

# Common Mistakes

---

## Thinking `write()` Immediately Writes to Disk

Usually it writes to the page cache first.

---

## Thinking Cached Data Is Permanent

The kernel may evict cached pages when memory is needed.

---

## Ignoring `fsync()`

Critical applications should explicitly synchronize important data when durability matters.

---

## Thinking Page Cache Belongs to One Process

The page cache is shared across the system.

If one process reads a file,

another process may benefit from the cached pages.

---

# Hands-on Labs

## Lab 1

Read a large file twice.

Measure the execution time.

---

## Lab 2

Run:

```bash
free -h
```

Observe cached memory.

---

## Lab 3

Inspect:

```bash
cat /proc/meminfo
```

Find cache-related statistics.

---

## Lab 4

Write a file.

Call:

```c
fsync()
```

Observe the difference in behavior.

---

## Lab 5

Research Linux page replacement algorithms and how the kernel decides which cached pages to reclaim.

---

# Interview Questions

### What is the Linux page cache?

A kernel-managed RAM cache storing recently accessed file pages.

---

### What is a cache hit?

The requested file page is already in RAM and can be returned without accessing storage.

---

### What is a dirty page?

A cached page modified in memory but not yet written back to storage.

---

### Why is `fsync()` important?

It ensures modified data is flushed from memory to persistent storage before returning.

---

### Why are repeated reads usually much faster?

Because the required file pages are often already available in the page cache.

---

# Summary

Read Path:

```
Application

↓

Page Cache

↓

SSD (If Needed)
```

Write Path:

```
Application

↓

Page Cache

↓

Dirty Page

↓

Background Write

↓

SSD
```

Page States:

```
Clean

↓

Discard Anytime

-----------------------

Dirty

↓

Must Write First
```

## Key Takeaways

- The page cache dramatically improves Linux file I/O performance.
- Most repeated file reads are served directly from RAM.
- Writes usually modify cached pages before reaching storage.
- Dirty pages are written back asynchronously unless explicitly synchronized.
- `fsync()` provides durability guarantees for critical data.
- The page cache is a fundamental optimization used throughout the Linux storage stack.

---

# Next Chapter

## Chapter 138 — `/proc` Filesystem — How Linux Exposes Kernel Internals as Files

You'll learn:

- What `/proc` really is
- Why `/proc` contains no real files
- Process directories
- `/proc/self`
- `/proc/cpuinfo`
- `/proc/meminfo`
- `/proc/<pid>/maps`
- `/proc/<pid>/fd`
- How Linux turns kernel data structures into virtual files