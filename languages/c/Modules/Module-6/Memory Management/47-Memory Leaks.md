# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 47 — Memory Leaks

---

## Most beginners think:

> "When my program finishes, the operating system frees all memory, so memory leaks don't matter."

That's only partially true.

Yes, **when a process exits**, the operating system reclaims its memory.

But what if your program **never exits**?

Examples:

- Linux Server
- PostgreSQL Database
- Redis
- Nginx
- Docker Daemon
- Kubernetes
- Chrome Browser

These programs run for:

- Days
- Weeks
- Months
- Even Years

A small memory leak every minute eventually becomes **gigabytes of wasted memory**.

This is why memory leaks are one of the most serious problems in systems programming.

---

# Learning Objectives

After this chapter you will understand:

- What a memory leak is
- Why memory leaks happen
- Different types of memory leaks
- Memory leak examples
- Detecting leaks
- Preventing leaks
- Real-world consequences

---

# What is a Memory Leak?

A **memory leak** happens when:

```text
Memory Allocated

↓

Program Loses Access

↓

Memory Never Freed
```

The allocated memory still exists,

but your program can no longer use it or release it.

---

# Simple Example

```c
int *p = malloc(sizeof(int));

*p = 100;
```

Memory:

```text
Stack

p

↓

Heap

100
```

Now:

```c
p = NULL;
```

Memory becomes:

```text
Stack

p

↓

NULL

Heap

100
```

What happened?

The heap memory still exists.

But its address has been lost.

Nobody can free it anymore.

This is a **memory leak**.

---

# Another Example

```c
void process()
{
    int *buffer = malloc(1024);

    /* forgot free() */
}
```

Every function call allocates:

```text
1024 Bytes
```

After:

```text
1000 Calls
```

Memory leaked:

```text
1 MB
```

After:

```text
1,000,000 Calls
```

Memory leaked:

```text
~1 GB
```

---

# Memory Leak Visualization

Normal:

```text
malloc()

↓

Use Memory

↓

free()

↓

Heap Released
```

Leak:

```text
malloc()

↓

Use Memory

↓

Pointer Lost

↓

Heap Never Released ❌
```

---

# Types of Memory Leaks

---

## 1. Forgetting `free()`

```c
char *buf = malloc(100);
```

Never calling:

```c
free(buf);
```

---

## 2. Overwriting the Pointer

Wrong:

```c
char *buf = malloc(100);

buf = malloc(200);
```

The first allocation is now unreachable.

---

## 3. Returning Without Cleanup

```c
char *buf = malloc(100);

if(error)
{
    return;
}
```

Forgot:

```c
free(buf);
```

---

## 4. Linked Data Structures

Suppose:

```text
Linked List

↓

Node

↓

Node

↓

Node
```

Only freeing:

```text
Head
```

Leaks every remaining node.

---

# Why Are Memory Leaks Dangerous?

Imagine:

Server

↓

100 Users

↓

1000 Users

↓

100000 Users

↓

Memory Leak Every Request

↓

RAM Full

↓

Crash

---

Real companies spend weeks debugging memory leaks because they often appear only after long periods of uptime.

---

# Detecting Memory Leaks

## Valgrind

Linux:

```bash
valgrind --leak-check=full ./program
```

Example output:

```text
LEAK SUMMARY

Definitely Lost:
```

Valgrind tells you:

- Where memory was allocated
- Where it leaked

---

## AddressSanitizer (LeakSanitizer)

Compile:

```bash
gcc -fsanitize=address program.c
```

or

```bash
gcc -fsanitize=address -g program.c
```

On many systems, LeakSanitizer automatically reports leaked memory when the program exits.

---

# Real Example

Wrong:

```c
int *arr =
malloc(100);

return;
```

Leak:

```text
100 Bytes Lost
```

Correct:

```c
free(arr);

return;
```

---

# Preventing Memory Leaks

Always remember this rule:

```text
Every

malloc()

↓

Must Have

↓

free()
```

The same applies to:

```text
calloc()

↓

free()

----------------

realloc()

↓

free()
```

---

# Ownership Rule

When writing large applications,

always ask:

> **Who owns this memory?**

Example:

```text
Function A

↓

Allocates Memory

↓

Function B

↓

Uses Memory

↓

Function C

↓

Frees Memory
```

Clearly defining ownership prevents leaks.

---

# Smart Cleanup Pattern

```c
char *buf = malloc(100);

if(buf == NULL)
{
    return;
}

/* use buffer */

free(buf);
buf = NULL;
```

Simple.

Safe.

Professional.

---

# Real-World Uses

Memory leak prevention is critical for:

- Linux Kernel
- Redis
- PostgreSQL
- Chrome
- Firefox
- Docker
- Kubernetes
- Nginx
- Embedded Devices

---

# Common Mistakes

---

## Forgetting `free()`

Most common cause.

---

## Losing the Pointer

```c
ptr = malloc(...);

ptr = malloc(...);
```

First allocation leaked.

---

## Returning Early

Always clean up before returning.

---

## Error Handling

Every error path must release allocated resources.

Professional code often uses a cleanup section before returning.

---

# Hands-on Labs

## Lab 1

Write a program with an intentional memory leak.

Detect it using:

```bash
valgrind
```

---

## Lab 2

Fix the leak.

Verify:

```text
0 Bytes Lost
```

---

## Lab 3

Allocate memory inside a loop.

Forget `free()`.

Observe increasing memory usage.

---

## Lab 4

Write a linked list.

Free every node correctly.

---

## Lab 5

Compile using:

```bash
gcc -fsanitize=address -g program.c
```

Observe leak reports.

---

# Interview Questions

### What is a memory leak?

Allocated heap memory that is no longer reachable and therefore can never be freed.

---

### Why are memory leaks dangerous?

They waste memory and eventually cause long-running programs to slow down or crash.

---

### How can memory leaks be detected?

Using tools like:

- Valgrind
- AddressSanitizer / LeakSanitizer

---

### Does the operating system free leaked memory?

Yes.

When the process exits.

But long-running programs may never exit.

---

### What is the golden rule of memory management?

Every successful:

```text
malloc()

calloc()

realloc()
```

should eventually have a matching:

```text
free()
```

---

# Summary

```text
malloc()

↓

Pointer

↓

Use Memory

↓

free()

✔ Correct

-------------------

malloc()

↓

Pointer Lost

↓

Memory Leak ❌
```

Key Takeaways:

- A memory leak happens when allocated memory is never released.
- Leaks are especially dangerous in long-running applications.
- Every allocation should have a matching `free()`.
- Clearly define memory ownership.
- Use Valgrind and AddressSanitizer to detect leaks.
- Memory leaks are one of the most common production issues in systems programming.

---

# Next Chapter

## Chapter 48 — Memory Fragmentation

You'll learn:

- What memory fragmentation is
- Internal vs External Fragmentation
- Why fragmentation reduces performance
- How memory allocators manage fragmented memory
- Why databases and operating systems care deeply about fragmentation
```