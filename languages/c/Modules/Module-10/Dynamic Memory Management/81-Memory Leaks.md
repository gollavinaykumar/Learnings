# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 81 — Memory Leaks

---

# Most beginners think:

> "A memory leak means RAM becomes full."

That's only the **final symptom**.

A memory leak begins much earlier.

A memory leak happens when:

```
Memory

↓

Allocated

↓

Never Released

↓

Pointer Lost

↓

Impossible To Free
```

The operating system cannot magically determine that your program no longer needs that memory.

As long as the process is running,

that memory remains unavailable for reuse.

For short-lived programs,

this may not be noticeable.

For long-running software like:

- Web Servers
- Databases
- Browsers
- Operating Systems

memory leaks can eventually cause serious problems.

---

# Learning Objectives

After this chapter you will understand:

- What a memory leak is
- How leaks occur
- Why leaks are dangerous
- Detecting leaks
- Preventing leaks
- Tools like Valgrind and AddressSanitizer
- Real-world examples

---

# What Is a Memory Leak?

Suppose:

```c
int *p =
malloc(sizeof(int));
```

Memory:

```
Heap

↓

Allocated
```

Now:

```c
p = NULL;
```

Oops.

What happened?

```
Heap

↓

Allocated

↓

No Pointer Exists
```

The allocated memory still exists,

but you've lost the only pointer to it.

You can never call:

```c
free()
```

on that block.

This is a **memory leak**.

---

# Visualizing a Leak

Before:

```
Pointer

↓

Heap Block
```

After:

```
Pointer

↓

NULL

----------------

Heap Block

↓

Still Allocated
```

The block is now unreachable.

---

# Why Is It Called a Leak?

Imagine a water tank.

Small leak:

```
💧
```

Not a problem.

After:

```
Hours

↓

Days

↓

Weeks
```

The tank becomes empty.

Memory leaks behave similarly.

Each leak may be tiny,

but over time,

they accumulate.

---

# Example 1 — Lost Pointer

```c
#include <stdlib.h>

int main()
{
    int *p =
        malloc(sizeof(int));

    p = NULL;

    return 0;
}
```

Leak:

```
malloc()

↓

Allocated

↓

Pointer Lost
```

---

# Example 2 — Returning Early

```c
char *buffer =
malloc(1024);

if(error)
{
    return;
}

free(buffer);
```

If:

```
error

↓

true
```

then:

```
free()

↓

Skipped
```

Memory leak.

---

# Example 3 — Inside a Loop

```c
while(1)
{
    malloc(1024);
}
```

Every iteration:

```
Allocate

↓

Never Free
```

Memory usage grows continuously.

---

# Memory Leak Timeline

Program starts.

```
10 MB
```

After:

```
1 Hour

↓

100 MB
```

After:

```
5 Hours

↓

1 GB
```

Eventually:

```
Allocation Failure

↓

Crash

↓

OOM Killer (Linux)

↓

Process Terminated
```

---

# Why Short Programs Often Hide Leaks

Suppose:

```c
int main()
{
    malloc(100);

    return 0;
}
```

Program exits immediately.

The operating system reclaims the process's memory.

So you might never notice the leak.

However,

the leak still exists from the program's perspective,

and memory-checking tools will report it.

---

# Why Servers Cannot Leak

Suppose:

```
Web Server

↓

Running

↓

365 Days
```

Each request leaks:

```
256 Bytes
```

After:

```
1 Million Requests
```

Leak:

```
256 MB
```

After:

```
100 Million Requests
```

Leak:

```
25.6 GB
```

Eventually,

the server becomes unstable or runs out of memory.

---

# Common Leak Patterns

## Pattern 1 — Lost Pointer

```c
p = malloc(...);

p = malloc(...);
```

The first allocation is leaked.

---

## Pattern 2 — Missing `free()`

```c
malloc(...);

return;
```

No:

```c
free();
```

---

## Pattern 3 — Error Handling

```c
malloc(...);

if(error)
{
    return;
}

free(...);
```

Early returns often bypass cleanup.

---

## Pattern 4 — Linked Lists

Allocate:

```
Node

↓

Node

↓

Node
```

Free only:

```
First Node
```

Remaining nodes leak.

---

# Detecting Leaks — Valgrind

Linux:

```bash
valgrind --leak-check=full ./program
```

Example output:

```
HEAP SUMMARY:

Definitely Lost:

1024 Bytes
```

Valgrind reports:

- Memory leaks
- Invalid reads
- Invalid writes
- Use-after-free
- Double free

---

# Detecting Leaks — AddressSanitizer

Compile:

```bash
gcc -fsanitize=address \
    main.c
```

Run:

```bash
./a.out
```

If a leak exists,

AddressSanitizer reports it.

It also detects:

- Buffer overflows
- Use-after-free
- Heap corruption

Very useful during development.

---

# Prevention Strategy

Rule:

```
Every

malloc()

↓

Must Have

↓

free()
```

Similarly:

```
calloc()

↓

free()
```

```
realloc()

↓

Eventually

↓

free()
```

---

# Ownership Principle

Suppose:

```c
char *name =
malloc(100);
```

Ask:

```
Who Owns This Memory?
```

Answer:

```
Current Function?
```

or

```
Caller?
```

Every allocation should have a clearly defined owner responsible for freeing it.

---

# Cleanup Pattern

Instead of:

```c
if(error1)
    return;

if(error2)
    return;

free(ptr);
```

Use:

```c
if(error1)
    goto cleanup;

if(error2)
    goto cleanup;

cleanup:

free(ptr);
```

In C,

a cleanup label is a common technique for ensuring resources are released exactly once.

---

# Real-World Example — Browser

Open:

```
100 Tabs
```

Each tab allocates:

- Images
- JavaScript Objects
- DOM Trees

Close:

```
100 Tabs
```

If memory isn't released,

the browser keeps consuming more RAM over time.

---

# Real-World Example — Database

Each SQL query allocates:

- Buffers
- Query Plans
- Temporary Results

After the query finishes,

all temporary memory must be released.

Otherwise,

the database gradually consumes more memory.

---

# Real-World Example — Linux Kernel

Kernel memory leaks are especially serious.

Unlike user-space processes,

parts of the kernel run for the entire system uptime.

A kernel memory leak can eventually affect the whole operating system.

---

# Common Mistakes

---

## Forgetting `free()`

Wrong:

```c
malloc(...);

/* Finished */
```

Always release the allocation.

---

## Losing the Pointer

Wrong:

```c
ptr = malloc(...);

ptr = NULL;
```

Memory leaked.

---

## Overwriting the Pointer

Wrong:

```c
ptr = malloc(...);

ptr = anotherPointer;
```

Original allocation becomes unreachable.

---

## Returning Early

Wrong:

```c
malloc(...);

if(error)
    return;
```

Cleanup skipped.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *arr =
        malloc(100 * sizeof(int));

    if(arr == NULL)
        return 1;

    for(int i = 0; i < 100; i++)
    {
        arr[i] = i;
    }

    free(arr);

    arr = NULL;

    return 0;
}
```

No leak.

---

# Hands-on Labs

## Lab 1

Write a program that intentionally leaks memory.

Run it using:

```bash
valgrind --leak-check=full
```

Observe the report.

---

## Lab 2

Compile the same program with:

```bash
-fsanitize=address
```

Observe the diagnostics.

---

## Lab 3

Create a linked list.

Free every node.

Verify that no memory leaks remain.

---

## Lab 4

Write a function with multiple error paths.

Implement a single cleanup section.

---

## Lab 5

Allocate memory inside a loop.

Free it before the next iteration.

Monitor memory usage while the program runs.

---

# Interview Questions

### What is a memory leak?

Allocated heap memory that is never released and becomes unreachable.

---

### Why are memory leaks dangerous?

They waste memory and can eventually cause allocation failures or degrade long-running applications.

---

### Does the operating system free memory when a process exits?

Yes.

The operating system reclaims the process's resources when it terminates.

However,

this does not excuse leaks in long-running programs or libraries.

---

### Which tools detect memory leaks?

- Valgrind
- AddressSanitizer

---

### What is the golden rule of dynamic memory?

Every successful allocation should eventually have a matching `free()`.

---

# Summary

```
malloc()

↓

Heap Block

↓

Pointer

↓

Pointer Lost

↓

Cannot Call free()

↓

Memory Leak
```

Golden Rule:

```
malloc()

↓

free()

----------------

calloc()

↓

free()

----------------

realloc()

↓

free()
```

## Key Takeaways

- A memory leak occurs when allocated memory is never released.
- Losing the last pointer to an allocation makes it impossible to free.
- Leaks are especially dangerous in long-running software.
- Use tools like Valgrind and AddressSanitizer to detect leaks.
- Design clear ownership rules for dynamically allocated memory.
- Every successful allocation should eventually be matched by a corresponding `free()`.

---

# Next Chapter

## Chapter 82 — Dangling Pointers & Use-After-Free

You'll learn:

- What dangling pointers are
- Why they occur
- Use-after-free bugs
- Why these bugs are security vulnerabilities
- Real-world exploitation techniques
- How professional developers prevent them