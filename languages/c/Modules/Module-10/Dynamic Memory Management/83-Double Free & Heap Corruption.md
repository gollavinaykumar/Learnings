# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 83 — Double Free & Heap Corruption

---

# Most beginners think:

> "Calling `free()` twice is harmless because the memory is already free."

This is one of the most dangerous mistakes in C programming.

Consider:

```c
free(ptr);

free(ptr);
```

Many beginners expect:

```
First free()

↓

Memory Released

Second free()

↓

Nothing Happens
```

Reality:

```
First free()

↓

Allocator Internal Data Updated

Second free()

↓

Allocator Gets Corrupted

↓

Undefined Behavior
```

This is called a:

```
Double Free
```

Double free bugs have caused security vulnerabilities in:

- Browsers
- Operating Systems
- Databases
- Network Servers

Understanding why they occur requires understanding how the heap allocator works.

---

# Learning Objectives

After this chapter you will understand:

- Double free
- Invalid free
- Heap corruption
- Heap metadata
- Buffer overflows on the heap
- Security implications
- Prevention techniques

---

# Review

Suppose:

```c
int *p =
malloc(sizeof(int));
```

Memory:

```
Heap

↓

Allocated Block
```

Call:

```c
free(p);
```

Allocator:

```
Heap

↓

Free Block
```

Everything is correct.

---

# What Happens During `free()`?

Many beginners think:

```
free()

↓

Erase Memory
```

Wrong.

Internally,

the allocator updates bookkeeping information.

Conceptually:

```
Heap

↓

Allocated Block

↓

Allocator Metadata

↓

Free List
```

The allocator now knows that block is available for reuse.

---

# Heap Metadata

Every allocation typically has hidden information.

Conceptually:

```
+------------------+
| Metadata         |
+------------------+
| Your Data        |
+------------------+
```

Metadata may include:

- Block size
- Allocation status
- Links to free blocks
- Other allocator-specific information

Your program normally never sees this metadata,

but the allocator depends on it.

---

# Double Free

Example:

```c
int *p =
malloc(sizeof(int));

free(p);

free(p);
```

First:

```
Allocator

↓

Marks Block Free
```

Second:

```
Allocator

↓

Attempts To Free

Already-Free Block
```

Its internal structures may become inconsistent.

This is undefined behavior.

---

# Visualizing Double Free

Before:

```
Heap

↓

Allocated
```

After first:

```
Heap

↓

Free
```

After second:

```
Allocator State

↓

Corrupted
```

Future allocations may fail or behave unpredictably.

---

# Invalid Free

Example:

```c
int x = 10;

free(&x);
```

Problem:

```
x

↓

Stack
```

Only pointers obtained from:

```
malloc()

calloc()

realloc()
```

(or `NULL`) may be passed to `free()`.

Freeing anything else results in undefined behavior.

---

# Another Invalid Free

Example:

```c
int *p =
malloc(100);

free(p + 1);
```

Problem:

```
Pointer

↓

Middle Of Allocation
```

The allocator expects the original pointer returned by the allocation function.

Only this is valid:

```c
free(p);
```

---

# Heap Buffer Overflow

Suppose:

```c
int *arr =
malloc(5 * sizeof(int));
```

Valid:

```
arr[0]

↓

arr[4]
```

Wrong:

```c
arr[5] = 100;
```

You wrote beyond the allocated block.

Possible consequence:

```
Your Data

↓

Allocator Metadata

↓

Overwritten
```

This can corrupt the heap.

---

# Visualization

Memory:

```
Metadata

↓

Your Array

↓

Next Block Metadata
```

Overflow:

```
Your Array

↓

Writes Here

↓

Next Metadata Damaged
```

The next allocation or free operation may crash because the allocator's bookkeeping has been corrupted.

---

# Why Heap Corruption Is Dangerous

Suppose:

```
Heap

↓

Corrupted Metadata
```

Next:

```c
malloc()
```

Allocator reads invalid metadata.

Possible results:

- Crash
- Incorrect allocation
- Memory corruption
- Security vulnerability

---

# Security Perspective

Historically,

attackers have exploited heap corruption bugs to:

- Modify program behavior
- Crash services
- Gain code execution

Modern allocators include many protections,

but heap corruption is still a serious class of vulnerabilities.

---

# Detecting Heap Corruption

AddressSanitizer:

Compile:

```bash
gcc -fsanitize=address \
    main.c
```

Reports:

- Heap buffer overflow
- Double free
- Invalid free
- Use-after-free

---

Valgrind:

```bash
valgrind ./program
```

Reports:

- Invalid writes
- Invalid frees
- Heap misuse
- Memory leaks

---

# Best Practices

Always follow:

```
malloc()

↓

Use

↓

free()

↓

Pointer = NULL
```

Never:

```
free()

↓

free()
```

Never:

```
free(Stack Memory)
```

Never:

```
Write Beyond Allocation
```

---

# Safe Example

```c
int *p =
malloc(sizeof(int));

if(p == NULL)
    return;

free(p);

p = NULL;
```

Calling:

```c
free(p);
```

again is now safe,

because:

```c
free(NULL);
```

does nothing.

---

# Real-World Example — Web Server

Request:

```
Allocate Buffer
```

Response sent.

```
free(Buffer)
```

A bug accidentally calls:

```
free(Buffer)
```

again.

Result:

```
Heap Corruption

↓

Server Crash
```

---

# Real-World Example — Browser

JavaScript Object:

```
Allocate

↓

Free

↓

Free Again
```

Historically,

similar bugs have contributed to browser security vulnerabilities.

---

# Real-World Example — Database

Database Page:

```
Allocate

↓

Free

↓

Pointer Reused Incorrectly
```

Heap corruption can lead to incorrect query results or crashes.

---

# Common Mistakes

---

## Double Free

Wrong:

```c
free(ptr);

free(ptr);
```

Undefined behavior.

---

## Invalid Free

Wrong:

```c
int x;

free(&x);
```

Stack memory must not be freed.

---

## Freeing Offset Pointers

Wrong:

```c
free(ptr + 1);
```

Only the original pointer returned by the allocator may be freed.

---

## Heap Buffer Overflow

Wrong:

```c
arr[10] = 100;
```

when only five elements were allocated.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *arr =
        malloc(5 * sizeof(int));

    if(arr == NULL)
        return 1;

    for(int i = 0; i < 5; i++)
    {
        arr[i] = i;
    }

    free(arr);

    arr = NULL;

    return 0;
}
```

Safe.

No double free.

No heap corruption.

---

# Hands-on Labs

## Lab 1

Write a program that performs a double free.

Compile with:

```bash
-fsanitize=address
```

Observe the error report.

---

## Lab 2

Write beyond the end of a heap allocation.

Observe AddressSanitizer's heap-buffer-overflow report.

---

## Lab 3

Attempt:

```c
free(ptr + 1);
```

Understand why this is invalid.

---

## Lab 4

Use Valgrind to detect invalid frees in a sample program.

---

## Lab 5

Review a larger program and verify that every allocation has exactly one matching `free()`.

---

# Interview Questions

### What is a double free?

Calling `free()` more than once on the same allocation.

---

### What is heap corruption?

Damage to heap data or allocator metadata caused by invalid memory operations.

---

### Can you call `free()` on stack memory?

No.

Only memory obtained from allocation functions (or `NULL`) may be passed to `free()`.

---

### What is a heap buffer overflow?

Writing beyond the bounds of a dynamically allocated block.

---

### Which tools detect heap corruption?

- AddressSanitizer
- Valgrind

---

# Summary

```
malloc()

↓

Heap Block

↓

free()

↓

Block Returned

↓

free() Again

↓

Double Free

↓

Undefined Behavior
```

Heap Overflow:

```
Allocated Block

↓

Write Past End

↓

Heap Metadata Corrupted

↓

Future malloc()/free()

↓

Undefined Behavior
```

## Key Takeaways

- Every allocation should be freed exactly once.
- Never call `free()` on stack memory or interior pointers.
- Heap buffer overflows can corrupt allocator metadata.
- Double free and heap corruption are serious correctness and security issues.
- Modern tools like AddressSanitizer and Valgrind help detect these bugs early.
- Safe heap management is essential for reliable systems programming.

---

# Next Chapter

## Chapter 84 — Dynamic Arrays (Building Your Own `vector`)

You'll learn:

- How to build a resizable array
- Capacity vs Size
- Automatic growth with `realloc()`
- Growth strategies (2×, 1.5×)
- Amortized complexity
- How `std::vector`, Java `ArrayList`, Go slices, and many runtimes manage dynamic arrays internally