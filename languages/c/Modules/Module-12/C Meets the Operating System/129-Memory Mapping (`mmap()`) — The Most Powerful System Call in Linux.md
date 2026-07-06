# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 129 — `mmap()` — The Most Powerful System Call in Linux

---

# Most developers know `mmap()` because of:

```c
mmap(...)
```

But very few understand what it actually does.

Many think:

> "`mmap()` simply allocates memory."

That's **not** what makes it powerful.

`mmap()` can make a file behave like memory.

It can create shared memory.

It powers databases.

It powers browsers.

It powers operating systems.

It is one of the most important Linux system calls ever created.

---

# Learning Objectives

After this chapter you will understand:

- Why `mmap()` exists
- File-backed mappings
- Anonymous mappings
- Shared mappings
- Private mappings
- Copy-on-Write
- Zero-copy concepts
- Real-world uses

---

# The Problem

Suppose you want to read a file.

Traditional approach:

```
Disk

↓

read()

↓

Kernel Buffer

↓

User Buffer

↓

Program
```

Notice:

```
Copy

↓

Copy Again
```

For huge files,

this becomes expensive.

---

# The Idea Behind `mmap()`

Instead of copying,

Linux says:

> "I'll make the file look like memory."

Now your program simply accesses:

```c
buffer[i]
```

without explicitly calling:

```c
read()
```

---

# Visualization

Without `mmap()`

```
Disk

↓

Kernel

↓

Copy

↓

Program Buffer
```

With `mmap()`

```
Disk

↓

Kernel Maps File

↓

Virtual Memory

↓

Program
```

No explicit read loop.

---

# What is `mmap()`?

Prototype:

```c
void *mmap(...);
```

Definition:

> **Maps a file or anonymous memory into a process's virtual address space.**

---

# Think About Google Maps

Imagine:

Instead of copying an entire city,

you simply receive:

```
Map

↓

View Needed Area
```

`mmap()` works similarly.

The kernel maps the file,

and pages are loaded when needed.

---

# File-Backed Mapping

Example:

```
photo.jpg

↓

mmap()

↓

Memory Pointer
```

Now:

```c
ptr[100]
```

reads byte 100 of the file.

---

# Visualization

```
File

↓

Virtual Memory

↓

Pointer
```

The file behaves like an array.

---

# Anonymous Mapping

`mmap()` can also create memory **not backed by a file**.

Example:

```
Anonymous Memory

↓

Virtual Memory

↓

Pointer
```

Useful for:

- Large allocations
- Shared memory
- Custom allocators

---

# Shared Mapping

Option:

```
MAP_SHARED
```

Visualization:

```
Process A

↓

Mapped File

↑

Process B
```

Changes are visible to all processes mapping the same region.

---

# Private Mapping

Option:

```
MAP_PRIVATE
```

Processes initially share pages.

When one process writes:

```
Copy-On-Write

↓

Private Copy
```

Other processes continue seeing the original data.

---

# Copy-on-Write

Before writing:

```
Process A

↓

Page

↑

Process B
```

After one process modifies the page:

```
Process A

↓

New Page

------------------

Process B

↓

Old Page
```

Efficient memory sharing.

---

# Demand Paging

Suppose:

```
10 GB File
```

Does Linux immediately load all 10 GB?

No.

Instead:

```
Map File

↓

Touch Page

↓

Page Fault

↓

Load Only Needed Page
```

Memory is loaded lazily.

---

# Page Faults

Suppose:

```c
ptr[5000000];
```

Page not in RAM.

CPU generates:

```
Page Fault
```

Kernel:

```
Load Page

↓

Resume Program
```

---

# Writing Through `mmap()`

Example:

```c
ptr[0] = 'A';
```

With:

```
MAP_SHARED
```

Changes may eventually be written back to the file.

No explicit:

```c
write()
```

required.

---

# Synchronization

Sometimes applications use:

```c
msync()
```

to request that modified pages be synchronized with the underlying file.

---

# Zero-Copy Concept

Traditional:

```
Disk

↓

Kernel Buffer

↓

User Buffer

↓

Application
```

Multiple copies.

With `mmap()`:

```
Disk

↓

Kernel

↓

Mapped Pages

↓

Application
```

Far fewer copies.

This is one reason databases love `mmap()`.

---

# `malloc()` vs `mmap()`

Small allocations:

```
malloc()

↓

Heap
```

Large allocations:

Many implementations internally use:

```
mmap()
```

instead of extending the heap.

---

# Shared Memory

Remember:

```
shm_open()

↓

mmap()
```

Shared memory becomes usable only after mapping it into the process.

---

# Browser Example

Chrome:

```
Image File

↓

mmap()

↓

Renderer
```

Efficient access to large resources.

---

# Database Example

Database:

```
Index File

↓

mmap()

↓

Search
```

No manual buffering required.

The kernel manages page loading.

---

# Redis Example

Redis primarily keeps data in RAM,

but many supporting Linux facilities and persistence mechanisms rely on memory-mapped concepts in the operating system.

---

# JVM Example

The Java Virtual Machine uses memory mapping in several areas,

including loading classes and managing memory-mapped files through its APIs.

---

# Operating System Example

Linux itself uses memory mapping extensively for:

- Executable loading
- Shared libraries
- Shared memory
- Dynamic linking

---

# Common Mistakes

---

## Thinking `mmap()` Copies the File

It maps the file into virtual memory.

Pages are loaded on demand.

---

## Forgetting `munmap()`

Mapped regions should be released:

```c
munmap()
```

when no longer needed.

---

## Assuming `MAP_PRIVATE` Updates the File

It does not.

Writes become private after Copy-on-Write.

---

## Ignoring Page Faults

Page faults are a normal part of memory-mapped file access.

---

# Complete Workflow

```
open()

↓

mmap()

↓

Read

↓

Write

↓

munmap()

↓

close()
```

---

# Hands-on Labs

## Lab 1

Open a text file.

Map it with:

```c
mmap()
```

Print its contents without using:

```c
read()
```

---

## Lab 2

Modify a file using:

```c
MAP_SHARED
```

Observe the updated file contents.

---

## Lab 3

Repeat using:

```c
MAP_PRIVATE
```

Observe that the original file remains unchanged.

---

## Lab 4

Map a large file.

Access only a few bytes.

Observe that Linux loads pages lazily.

---

## Lab 5

Read:

```bash
cat /proc/<pid>/maps
```

Inspect the mapped memory regions of a running process.

---

# Interview Questions

### What does `mmap()` do?

Maps a file or anonymous memory into a process's virtual address space.

---

### What is the difference between `MAP_SHARED` and `MAP_PRIVATE`?

`MAP_SHARED` shares modifications with other mappings and the underlying file.

`MAP_PRIVATE` creates private Copy-on-Write pages for modifications.

---

### Why is `mmap()` faster than repeatedly calling `read()`?

It avoids repeated copying and lets the kernel load pages on demand.

---

### What is demand paging?

Loading memory pages only when they are actually accessed.

---

### Why do databases use `mmap()`?

It allows efficient access to large files while relying on the kernel's virtual memory system for paging and caching.

---

# Summary

Traditional I/O:

```
Disk

↓

read()

↓

Kernel Buffer

↓

Application
```

Memory Mapping:

```
Disk

↓

Virtual Memory

↓

Application
```

Mapping Types:

| Type | Purpose |
|------|---------|
| `MAP_SHARED` | Shared updates |
| `MAP_PRIVATE` | Copy-on-Write |
| Anonymous Mapping | Memory not backed by a file |

## Key Takeaways

- `mmap()` maps files or anonymous memory directly into a process's address space.
- Memory-mapped files simplify file access by treating files as memory.
- `MAP_SHARED` enables shared updates; `MAP_PRIVATE` uses Copy-on-Write.
- Demand paging loads only the pages that are actually accessed.
- `mmap()` is fundamental to databases, browsers, shared libraries, and many operating system features.
- It is one of the most powerful abstractions in modern Unix-like systems.

---

# Next Chapter

## Chapter 130 — Dynamic Linking (`.so`) — How Linux Loads Shared Libraries

You'll learn:

- Static vs dynamic linking
- ELF shared libraries
- `.so` files
- The dynamic linker (`ld-linux`)
- Procedure Linkage Table (PLT)
- Global Offset Table (GOT)
- Lazy binding
- Why almost every Linux program depends on shared libraries