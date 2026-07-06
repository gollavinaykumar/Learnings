# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 80 — `free()`

---

# Most beginners think:

> "`free()` deletes memory."

Not exactly.

`free()` **does not erase RAM**.

It simply tells the memory allocator:

> **"I'm finished using this block. You may reuse it later."**

Think of it like returning a rented hotel room.

The room still exists.

You don't destroy the building.

You simply tell the hotel:

> "I'm checking out."

The hotel can now rent the room to someone else.

`free()` works exactly the same way.

---

# Learning Objectives

After this chapter you will understand:

- What `free()` does
- Why it exists
- Heap allocator internals
- Memory reuse
- Memory leaks
- Dangling pointers
- Double free
- Use-after-free
- Best practices

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

Allocated

↓

25
```

When you're done,

that memory should be returned to the allocator.

That's the job of:

```c
free(p);
```

---

# What is `free()`?

Prototype:

```c
void free(void *ptr);
```

Meaning:

```
Heap Block

↓

Return To Allocator

↓

Available Again
```

Notice:

It returns:

```
Nothing
```

---

# Basic Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *p =
        malloc(sizeof(int));

    if(p == NULL)
        return 1;

    *p = 25;

    printf("%d\n", *p);

    free(p);

    return 0;
}
```

---

# What Happens Internally?

Suppose:

```
Heap

↓

AAAA

BBBB

CCCC

DDDD
```

You call:

```c
free(BBBB);
```

Allocator:

```
AAAA

FREE

CCCC

DDDD
```

The memory isn't erased.

It is simply marked as available.

---

# Memory Reuse

Later:

```c
malloc(...)
```

may return:

```
FREE
```

again.

Visualization:

```
Allocate

↓

Use

↓

free()

↓

Reuse Later
```

This is how heap memory is recycled.

---

# Important Point

After:

```c
free(p);
```

the pointer variable still exists.

```
p

↓

Old Address
```

The pointer was **not** changed.

Only the memory ownership changed.

---

# Dangling Pointer

Example:

```c
int *p =
malloc(sizeof(int));

free(p);
```

Now:

```
p

↓

Old Address
```

But that memory no longer belongs to your program.

This is called a:

```
Dangling Pointer
```

---

# Why Dangling Pointers Are Dangerous

Wrong:

```c
free(p);

*p = 100;
```

This is called:

```
Use-After-Free
```

The program is accessing memory that has already been released.

This causes **undefined behavior**.

The program might:

- Crash
- Corrupt memory
- Appear to work
- Introduce security vulnerabilities

---

# Best Practice

Immediately after:

```c
free(p);
```

write:

```c
p = NULL;
```

Now:

```
p

↓

NULL
```

Accidentally dereferencing `NULL` is also an error, but it's much easier to detect than silently accessing freed memory.

---

# Double Free

Wrong:

```c
free(p);

free(p);
```

The same block is released twice.

This is called:

```
Double Free
```

It causes undefined behavior and can corrupt the allocator's internal data structures.

---

# Safe Pattern

```c
free(p);

p = NULL;
```

Later:

```c
free(p);
```

Calling `free(NULL)` is defined by the C standard to do nothing.

So this pattern helps avoid accidental double frees.

---

# Memory Leak

Suppose:

```c
int *p =
malloc(sizeof(int));

/* Program exits function */

return;
```

No:

```c
free(p);
```

Memory remains allocated.

This is called a:

```
Memory Leak
```

Repeated leaks eventually waste memory.

---

# Visualizing a Memory Leak

Program:

```
malloc()

↓

100 Bytes
```

Never freed.

```
Heap

↓

Allocated Forever
```

Allocate again.

```
100 Bytes

↓

Still Allocated
```

After many leaks:

```
Heap Full
```

New allocations fail.

---

# Ownership

A useful concept in professional software:

```
Who Owns This Memory?
```

If your code allocates memory,

it is usually responsible for releasing it,

unless ownership is explicitly transferred.

Clear ownership rules help prevent leaks and double frees.

---

# Freeing Arrays

```c
int *arr =
malloc(100 *
sizeof(int));

free(arr);
```

Only **one** call is needed.

The allocator remembers the size internally.

---

# Freeing Structures

```c
struct Student *s =
malloc(sizeof(struct Student));

free(s);
```

Also:

```
One Call
```

---

# What If Pointer Is NULL?

Example:

```c
int *p = NULL;

free(p);
```

Perfectly safe.

Nothing happens.

---

# What If Pointer Was Never Allocated?

Wrong:

```c
int x = 10;

free(&x);
```

`x` lives on the stack,

not the heap.

Only pointers returned by allocation functions like:

```
malloc()

calloc()

realloc()
```

(or `NULL`) may be passed to `free()`.

---

# Real-World Example — Web Server

Request arrives.

Allocate:

```
Headers

Body

Cookies
```

Request finishes.

Release:

```
All Buffers
```

Otherwise,

the server leaks memory with every request.

---

# Real-World Example — Database

Database allocates:

```
Query Buffer

↓

Execute Query

↓

free()
```

The memory becomes available for the next query.

---

# Real-World Example — Browser

Open:

```
100 Tabs
```

Close:

```
90 Tabs
```

The browser frees memory used by those tabs.

Without freeing,

memory usage would continue growing.

---

# Common Mistakes

---

## Forgetting `free()`

Wrong:

```c
char *buffer =
malloc(1024);

/* Never freed */
```

Memory leak.

---

## Double Free

Wrong:

```c
free(ptr);

free(ptr);
```

Undefined behavior.

---

## Use After Free

Wrong:

```c
free(ptr);

printf("%d",
       *ptr);
```

Undefined behavior.

---

## Freeing Stack Memory

Wrong:

```c
int x = 10;

free(&x);
```

Only heap memory should be freed.

---

## Freeing the Same Block Through Multiple Aliases

Example:

```c
int *a =
malloc(sizeof(int));

int *b = a;

free(a);

/* b now also points to freed memory */
```

Both pointers refer to the same block.

After freeing through one pointer,

the other becomes dangling as well.

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
        arr[i] = i + 1;
    }

    free(arr);

    arr = NULL;

    return 0;
}
```

---

# Hands-on Labs

## Lab 1

Allocate:

```
10 Integers
```

Store values.

Free the memory.

Set the pointer to `NULL`.

---

## Lab 2

Write a program that intentionally leaks memory.

Run it under a memory-checking tool such as Valgrind (Linux) or AddressSanitizer.

Observe the reported leak.

---

## Lab 3

Experiment with:

```c
free(NULL);
```

Observe that it safely does nothing.

---

## Lab 4

Create two pointers pointing to the same allocation.

Free one.

Explain why the other becomes a dangling pointer.

---

## Lab 5

Build a program that repeatedly allocates and frees memory.

Observe that the program can reuse heap memory efficiently.

---

# Interview Questions

### What does `free()` do?

It releases a dynamically allocated heap block so the allocator can reuse it.

---

### Does `free()` erase memory?

No.

It marks the memory as available for future allocations.

---

### What is a memory leak?

Memory that was allocated but never released.

---

### What is a dangling pointer?

A pointer that still refers to memory that has already been freed.

---

### Is `free(NULL)` safe?

Yes.

The C standard defines it as a no-op.

---

### Can you call `free()` on stack memory?

No.

Only memory obtained from dynamic allocation functions (or `NULL`) may be passed to `free()`.

---

# Summary

```
malloc()

↓

Heap Block

↓

Program Uses Memory

↓

free()

↓

Allocator Marks Block Free

↓

Future malloc()

↓

Memory Reused
```

Common Problems:

```
Forget free()

↓

Memory Leak

----------------

free()

↓

Use Pointer Again

↓

Use-After-Free

----------------

free()

↓

free()

↓

Double Free
```

## Key Takeaways

- `free()` releases heap memory back to the allocator.
- It does not erase the memory contents.
- Always free dynamically allocated memory when finished.
- Set pointers to `NULL` after freeing them when practical.
- Never use memory after it has been freed.
- Never free the same allocation twice.
- Proper memory management is essential for reliable, secure, and high-performance software.

---

# Next Chapter

## Chapter 81 — Memory Leaks

You'll learn:

- What memory leaks are
- How they happen
- Why long-running servers suffer from them
- Detecting leaks with Valgrind and AddressSanitizer
- Leak prevention strategies
- Real-world production examples