# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 43 — `malloc()`

---

## Most beginners think:

> "`malloc()` creates a variable."

It doesn't.

`malloc()` **does not create variables**.

Instead, it asks the operating system (through the C runtime memory allocator) for a block of memory from the **heap**.

It simply returns the **address** of that memory.

Without `malloc()`, programs couldn't create data structures whose size is only known while the program is running.

---

# Learning Objectives

After this chapter you will understand:

- What `malloc()` is
- Why `malloc()` exists
- How `malloc()` works
- Memory allocation on the heap
- Return value of `malloc()`
- Why `malloc()` can fail
- Checking for `NULL`
- Common mistakes
- Real-world uses

---

# Why Do We Need `malloc()`?

Imagine a program that stores students.

```c
int studentCount;

scanf("%d", &studentCount);
```

The user enters:

```text
500
```

Tomorrow another user enters:

```text
500000
```

You don't know the required memory size before the program starts.

Stack memory isn't suitable for this.

You need memory that can be allocated dynamically.

That's why `malloc()` exists.

---

# What is `malloc()`?

Prototype:

```c
#include <stdlib.h>

void *malloc(size_t size);
```

It means:

> "Allocate **size** bytes on the heap and return its address."

---

# How `malloc()` Works

```text
Program

      │

malloc(100)

      │

      ▼

Memory Allocator

      │

      ▼

Operating System

      │

      ▼

Heap Memory
```

If successful:

```text
Pointer

↓

Allocated Block
```

If unsuccessful:

```text
NULL
```

---

# Basic Example

```c
#include <stdlib.h>

int *p = malloc(sizeof(int));
```

Execution:

```text
malloc()

↓

Heap

↓

4 bytes allocated

↓

Pointer returned
```

Memory:

```text
Stack

┌──────────────┐
│ p = 0x5000   │
└──────────────┘
        │
        ▼

Heap

┌──────────────┐
│ Uninitialized│
└──────────────┘
```

---

# Allocating Arrays

```c
int *numbers = malloc(5 * sizeof(int));
```

Memory:

```text
Heap

┌─────────────────────────────┐
│ int │ int │ int │ int │ int │
└─────────────────────────────┘
```

The memory is contiguous.

---

# Important

The allocated memory is **not initialized**.

Example:

```c
int *p = malloc(sizeof(int));

printf("%d\n", *p);
```

The value is unpredictable.

It contains whatever data happened to already be in that memory.

---

# Always Check the Return Value

```c
int *p = malloc(sizeof(int));

if (p == NULL)
{
    printf("Allocation failed\n");
    return 1;
}
```

If memory cannot be allocated,

`malloc()` returns:

```text
NULL
```

Never assume allocation always succeeds.

---

# Writing Data

```c
*p = 100;

printf("%d\n", *p);
```

Memory:

```text
Heap

100
```

Now the memory contains valid data.

---

# Freeing Memory

Every successful allocation should eventually be released.

```c
free(p);

p = NULL;
```

Otherwise memory leaks occur.

We'll study `free()` in detail later.

---

# Memory Diagram

```text
Stack

Pointer

↓

Heap

Allocated Block

↓

Use Memory

↓

free()

↓

Memory Returned
```

---

# Why Doesn't `malloc()` Return an `int *`?

Prototype:

```c
void *malloc(size_t size);
```

It returns:

```text
void *
```

Because it doesn't know what type you want.

Examples:

```c
int *

char *

double *

struct Student *
```

The same function works for every type.

---

# Real-World Uses

`malloc()` is used in:

- Linked Lists
- Trees
- Hash Tables
- Graphs
- Dynamic Arrays
- Network Buffers
- Database Engines
- Browsers
- Redis
- PostgreSQL

---

# Common Mistakes

## Forgetting `sizeof()`

Wrong:

```c
int *p = malloc(10);
```

Correct:

```c
int *p = malloc(10 * sizeof(int));
```

---

## Ignoring NULL

Wrong:

```c
int *p = malloc(100);

*p = 5;
```

Always verify allocation first.

---

## Forgetting `free()`

```c
malloc()

↓

No free()

↓

Memory Leak
```

---

# Hands-on Labs

### Lab 1

Allocate one integer.

Store:

```text
100
```

Print it.

---

### Lab 2

Allocate an array of 20 integers.

Fill it using a loop.

---

### Lab 3

Print the address of every allocated element.

---

### Lab 4

Free the memory.

Set the pointer to:

```c
NULL
```

---

### Lab 5

Run:

```bash
valgrind ./program
```

Verify there are no memory leaks.

---

# Interview Questions

### What does `malloc()` do?

Allocates memory on the heap and returns its address.

---

### What does `malloc()` return on failure?

```text
NULL
```

---

### Is memory returned by `malloc()` initialized?

No.

It contains indeterminate data.

---

### Why does `malloc()` return `void *`?

Because it is a generic allocator that works for every data type.

---

### Who releases memory allocated by `malloc()`?

The programmer must call:

```c
free()
```

---

# Summary

```text
Program
     │
     ▼
malloc(size)
     │
     ▼
Heap Allocation
     │
     ▼
Pointer Returned
     │
     ▼
Use Memory
     │
     ▼
free()
```

Key Takeaways:

- `malloc()` allocates memory from the heap.
- It returns a pointer to the allocated block.
- Memory is **not initialized**.
- Always check for `NULL`.
- Every successful `malloc()` should eventually have a matching `free()`.
- `malloc()` is the foundation of dynamic memory management in C.

---

# Next Chapter

## Chapter 44 — `calloc()`

You'll learn:

- How `calloc()` differs from `malloc()`
- Why it initializes memory to zero
- Performance considerations
- When to use `calloc()` instead of `malloc()`