# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 38 --- NULL Pointer

------------------------------------------------------------------------

## Most beginners think:

> "NULL is just the number 0."

Not exactly.

`NULL` represents **"this pointer intentionally points to nothing."**

It is **not** valid memory.

It is **not** the same as a wild pointer.

It is **not** the same as a dangling pointer.

Using `NULL` is one of the simplest and most effective defensive
programming techniques in C.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What `NULL` is
-   Why `NULL` exists
-   `NULL` vs Wild Pointer
-   `NULL` vs Dangling Pointer
-   Why operating systems protect address 0
-   Checking for `NULL`
-   Real-world uses
-   Best practices

------------------------------------------------------------------------

# What is NULL?

A NULL pointer is a pointer that intentionally points to **no valid
object**.

``` c
int *p = NULL;
```

Conceptually:

``` text
p
 │
 ▼
NULL

(No valid memory)
```

------------------------------------------------------------------------

# Why Does NULL Exist?

Sometimes a pointer has no object to reference yet.

Examples:

-   Memory hasn't been allocated.
-   A search found nothing.
-   Memory has already been freed.
-   Initialization hasn't finished.

Instead of leaving a random address:

``` text
????????
```

we explicitly write:

``` text
NULL
```

------------------------------------------------------------------------

# Wild Pointer vs NULL Pointer

Wrong:

``` c
int *p;
```

``` text
Unknown Address ❌
```

Correct:

``` c
int *p = NULL;
```

``` text
Known Invalid State ✔
```

A `NULL` pointer is predictable.

A wild pointer is not.

------------------------------------------------------------------------

# Dangling Pointer vs NULL Pointer

Dangling:

``` c
free(p);
```

``` text
p

↓

Old Address ❌
```

Safer:

``` c
free(p);
p = NULL;
```

``` text
p

↓

NULL ✔
```

------------------------------------------------------------------------

# Checking Before Use

``` c
if (p != NULL)
{
    *p = 10;
}
```

This prevents dereferencing a known invalid pointer.

------------------------------------------------------------------------

# Why Address 0 Is Special

Modern operating systems intentionally keep virtual address **0**
unmapped.

Conceptually:

``` text
Virtual Memory

0x00000000

↓

No Access
```

If your program dereferences `NULL`:

``` c
*p = 10;
```

the operating system usually terminates the program with a segmentation
fault.

This helps detect bugs early instead of silently corrupting memory.

------------------------------------------------------------------------

# Common APIs

Many C functions return `NULL` to indicate failure.

``` c
FILE *fp = fopen("data.txt","r");

if (fp == NULL)
{
    /* handle error */
}
```

Another example:

``` c
char *p = malloc(100);

if (p == NULL)
{
    /* allocation failed */
}
```

Always check when failure is possible.

------------------------------------------------------------------------

# Memory Diagram

``` text
Valid Pointer

p
 │
 ▼
Object

---------------------

NULL Pointer

p
 │
 ▼
NULL

---------------------

Dangling Pointer

p
 │
 ▼
Released Memory ❌

---------------------

Wild Pointer

p
 │
 ▼
Unknown Address ❌
```

------------------------------------------------------------------------

# Best Practices

-   Initialize pointers to `NULL`.
-   Set pointers to `NULL` after `free()`.
-   Check returned pointers before use.
-   Never dereference `NULL`.
-   Treat compiler warnings seriously.

------------------------------------------------------------------------

# Common Mistakes

## Assuming NULL Is Valid Memory

Wrong:

``` c
int *p = NULL;

*p = 5;
```

Undefined behavior (typically a crash).

------------------------------------------------------------------------

## Forgetting Allocation Checks

Wrong:

``` c
char *buf = malloc(100);

strcpy(buf,"Hello");
```

If `malloc()` failed, `buf` is `NULL`.

Always verify first.

------------------------------------------------------------------------

# Real-World Uses

NULL checks appear throughout:

-   Linux kernel
-   PostgreSQL
-   Redis
-   Nginx
-   SQLite
-   Device drivers
-   Embedded firmware

Defensive pointer handling improves reliability.

------------------------------------------------------------------------

# Hands-on Labs

1.  Initialize pointers to `NULL`.
2.  Check a pointer before dereferencing it.
3.  Allocate memory with `malloc()` and verify the result.
4.  Call `free()` and immediately set the pointer to `NULL`.
5.  Observe a NULL dereference under `gdb` or AddressSanitizer.

------------------------------------------------------------------------

# Interview Questions

### What is a NULL pointer?

A pointer that intentionally points to no valid object.

### Is NULL the same as a wild pointer?

No. A wild pointer contains an unknown address. NULL is a known invalid
value.

### Why set a pointer to NULL after `free()`?

To avoid accidentally using a dangling pointer.

### Why do operating systems leave address 0 unmapped?

To detect NULL pointer dereferences and prevent silent memory
corruption.

------------------------------------------------------------------------

# Summary

``` text
Pointer
   │
   ├── Valid Object ✔
   ├── NULL ✔
   ├── Wild Pointer ❌
   └── Dangling Pointer ❌
```

Key Takeaways:

-   `NULL` represents "no valid object".
-   Initialize pointers with `NULL` when appropriate.
-   Check pointers before dereferencing if failure is possible.
-   Set pointers to `NULL` after `free()`.
-   NULL handling is a fundamental defensive programming practice.

------------------------------------------------------------------------

# 🎉 Module 5 Complete

You have learned:

-   Memory Addresses
-   Pointer Basics
-   Pointer Arithmetic
-   Arrays vs Pointers
-   Pointer to Pointer
-   Void Pointer
-   Function Pointers (Advanced)
-   Double Pointer
-   Const Pointer vs Pointer to Const
-   Dangling Pointer
-   Wild Pointer
-   NULL Pointer

Pointers are the foundation of:

-   Memory Management
-   Dynamic Allocation
-   POSIX Programming
-   Linux Kernel
-   Networking
-   Databases
-   Embedded Systems

------------------------------------------------------------------------

# Next Module

## Module 6 --- Memory Management

You'll learn:

-   Stack Memory
-   Heap Memory
-   Global Memory
-   Static Memory
-   malloc()
-   calloc()
-   realloc()
-   free()
-   Memory Leaks
-   Memory Fragmentation
-   Memory Alignment
-   Memory Corruption
