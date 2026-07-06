# 👑 C Mastery Roadmap

# Module 6 --- Memory Management

# Chapter 40 --- Heap Memory

------------------------------------------------------------------------

## Most beginners think:

> "All variables are stored in memory the same way."

They are not.

Some variables are created automatically on the **stack**.

Others must be created manually on the **heap**.

The heap exists because sometimes your program needs memory whose:

-   Size isn't known at compile time
-   Lifetime is longer than a function call
-   Amount can grow or shrink while the program is running

Without the heap, dynamic data structures like linked lists, trees, hash
tables, databases, browsers, and operating systems would be impossible.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What heap memory is
-   Why the heap exists
-   Stack vs heap
-   Dynamic allocation
-   Heap lifetime
-   How the operating system provides heap memory
-   Why heap allocation is slower
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Heap Memory?

Suppose the user enters how many students there are.

``` c
int count;
scanf("%d", &count);
```

You don't know `count` until the program is running.

You cannot safely do this with a fixed-size local array in many
situations.

Instead, request memory dynamically from the heap.

------------------------------------------------------------------------

# What is Heap Memory?

The heap is a region of memory used for **dynamic memory allocation**.

Unlike the stack:

-   The programmer requests memory.
-   The programmer must release memory.

``` text
Program

     │

malloc()

     │

     ▼

Heap

┌──────────────────┐
│ Allocated Block  │
└──────────────────┘
```

------------------------------------------------------------------------

# Stack vs Heap

  Stack                   Heap
  ----------------------- -------------------------------------
  Automatic               Manual
  Very fast               Slower
  Small                   Much larger
  Function lifetime       Programmer-controlled lifetime
  Managed automatically   Managed using `malloc()` / `free()`

------------------------------------------------------------------------

# Example

``` c
#include <stdlib.h>

int *numbers = malloc(5 * sizeof(int));
```

Execution:

``` text
Program

↓

malloc()

↓

Heap Block

↓

numbers points to block
```

The pointer is stored on the stack.

The allocated memory is stored on the heap.

------------------------------------------------------------------------

# Memory Layout

``` text
Stack

┌──────────────┐
│ numbers      │
│ 0x5000       │
└──────────────┘
        │
        ▼

Heap

┌────────────────────┐
│ int int int int int│
└────────────────────┘
```

The stack and heap are different memory regions.

------------------------------------------------------------------------

# Heap Lifetime

Stack variable:

``` c
void demo()
{
    int x = 10;
}
```

`x` disappears automatically.

Heap memory:

``` c
int *p = malloc(sizeof(int));
```

The allocated memory remains until:

``` c
free(p);
```

or the process exits.

------------------------------------------------------------------------

# Why Isn't Heap Memory Freed Automatically?

Imagine this function:

``` c
char *createBuffer()
{
    char *buf = malloc(1024);
    return buf;
}
```

The caller still needs the memory after the function returns.

If the heap were freed automatically, returning allocated memory would
be impossible.

------------------------------------------------------------------------

# Where Does Heap Memory Come From?

Your process asks the operating system for memory.

Conceptually:

``` text
Application

↓

malloc()

↓

Memory Allocator

↓

Operating System

↓

RAM
```

The C library allocator manages heap blocks and requests additional
pages from the operating system when necessary.

------------------------------------------------------------------------

# Why Is Heap Allocation Slower?

Allocating on the stack:

``` text
Move Stack Pointer
```

Allocating on the heap:

``` text
Search Free Block

↓

Reserve Block

↓

Update Metadata

↓

Return Address
```

The allocator performs much more work.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting to Free Memory

``` c
int *p = malloc(sizeof(int));
```

If `free(p)` is never called, memory leaks occur.

------------------------------------------------------------------------

## Losing the Pointer

``` c
p = malloc(100);

p = malloc(200);
```

The address of the first allocation is lost.

That memory can no longer be freed.

------------------------------------------------------------------------

## Using Memory After Free

``` c
free(p);

*p = 10;
```

This is a use-after-free bug.

------------------------------------------------------------------------

# Real-World Uses

Heap memory is used for:

-   Linked lists
-   Trees
-   Hash tables
-   Dynamic arrays
-   Web servers
-   Databases
-   Browsers
-   Linux kernel subsystems (through kernel allocators)

------------------------------------------------------------------------

# Hands-on Labs

1.  Allocate an array using `malloc()`.
2.  Store values in the allocated memory.
3.  Print the addresses of each element.
4.  Free the memory.
5.  Detect leaks using:

``` bash
valgrind ./program
```

------------------------------------------------------------------------

# Interview Questions

### What is heap memory?

A memory region used for dynamically allocated objects.

### Who manages heap memory?

The programmer requests and releases it using allocation functions such
as `malloc()` and `free()`.

### Why is heap allocation slower than stack allocation?

Because the allocator must manage free blocks and bookkeeping
information.

### When is heap memory released?

When `free()` is called or when the process terminates.

### Can heap memory outlive a function?

Yes. That's one of its primary purposes.

------------------------------------------------------------------------

# Summary

``` text
Program
    │
    ▼
malloc()
    │
    ▼
Heap Allocation
    │
    ▼
Use Memory
    │
    ▼
free()
    │
    ▼
Memory Returned
```

Key Takeaways:

-   The heap stores dynamically allocated memory.
-   Heap objects can outlive function calls.
-   `malloc()` allocates heap memory.
-   `free()` releases heap memory.
-   Heap allocation is slower but much more flexible than stack
    allocation.

------------------------------------------------------------------------

# Next Chapter

## Chapter 41 --- Global Memory

You'll learn where global variables are stored, how they differ from
stack and heap memory, and how the operating system initializes global
data when a program starts.
