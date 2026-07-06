# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 77 — `malloc()`

---

# Most beginners think:

> "`malloc()` creates memory."

It doesn't.

`malloc()` **does not create RAM**.

Your computer already has RAM.

Instead,

`malloc()` asks the **Operating System** (through the C runtime) for a portion of memory from the **heap**.

Think of it like renting a room in a hotel.

The hotel already exists.

You don't build a new room.

You simply ask:

> "Can I use one of your available rooms?"

If a room is available,

you receive its number.

If not,

your request is rejected.

`malloc()` works in exactly the same way.

---

# Learning Objectives

After this chapter you will understand:

- What `malloc()` is
- Why it exists
- Heap allocation
- What happens internally
- Memory alignment
- Why memory is uninitialized
- Error handling
- Real-world examples

---

# Why Do We Need `malloc()`?

Suppose:

```c
int age = 21;
```

Memory:

```
Stack

↓

21
```

Works fine.

Now suppose you need:

```
10 Users
```

Tomorrow:

```
100 Users
```

Next week:

```
10,000 Users
```

Can you know the exact amount at compile time?

No.

The amount of memory depends on runtime information.

This is why dynamic memory allocation exists.

---

# Stack Allocation

Example:

```c
int numbers[100];
```

Memory:

```
Stack

↓

100 Integers
```

Problem:

The size is fixed.

---

# Dynamic Allocation

Instead:

```c
int n;

scanf("%d", &n);

int *numbers =
    malloc(n * sizeof(int));
```

Now:

```
User Input

↓

Heap Allocation

↓

Exactly n Integers
```

Memory is allocated only when needed.

---

# What is `malloc()`?

Prototype:

```c
void *malloc(size_t size);
```

Meaning:

```
Allocate

↓

size Bytes

↓

Heap

↓

Return Address
```

---

# Why `void *`?

`malloc()` doesn't know what you're storing.

It only knows:

```
Bytes
```

Maybe you want:

```
int

char

float

struct Student

Tree Node
```

It simply returns the address of the allocated memory.

Example:

```c
int *p =
malloc(sizeof(int));
```

The returned `void *` is automatically converted to `int *` in C.

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

Output:

```
25
```

---

# Memory Visualization

Before:

```
Heap

↓

Free

Free

Free

Free
```

After:

```c
malloc(sizeof(int))
```

```
Heap

↓

Allocated

↓

25

Free

Free

Free
```

---

# Why `sizeof()`?

Many beginners write:

```c
malloc(4);
```

Works on many systems,

but it's poor practice.

Instead:

```c
malloc(sizeof(int));
```

Why?

Because `sizeof(int)` depends on the system.

On most modern systems:

```
4 Bytes
```

but the language does not require that.

`sizeof()` makes the program portable.

---

# Allocating an Array

```c
int n = 100;

int *arr =
malloc(n * sizeof(int));
```

Memory:

```
Heap

↓

100 Integers
```

Exactly what you requested.

---

# What Happens Internally?

Suppose:

```c
malloc(64);
```

Flow:

```
Program

↓

malloc()

↓

C Runtime

↓

Heap Manager

↓

Operating System

↓

Heap Memory
```

If successful:

```
Pointer

↓

Returned
```

Otherwise:

```
NULL
```

---

# Why Can `malloc()` Fail?

Suppose:

Need:

```
10 TB
```

Computer has:

```
16 GB RAM
```

The request cannot be satisfied.

`malloc()` returns:

```
NULL
```

Always check the result.

---

# Always Check for NULL

Correct:

```c
int *p =
malloc(sizeof(int));

if(p == NULL)
{
    printf("Allocation Failed\n");
    return 1;
}
```

Never assume allocation succeeds.

---

# Uninitialized Memory

One of the most important facts.

Suppose:

```c
int *p =
malloc(sizeof(int));
```

What value is inside?

Many beginners think:

```
0
```

Wrong.

The contents are **indeterminate**.

Example:

```
Heap

↓

8492

↓

-100

↓

98231

↓

???
```

The bytes contain whatever happened to be there previously.

---

# Why Doesn't `malloc()` Clear Memory?

Initializing every byte to zero would take extra time.

`malloc()` is designed to be fast.

If you need zero-initialized memory,

use:

```c
calloc()
```

We'll learn that in the next chapter.

---

# Memory Alignment

Suppose:

Need:

```
1 Byte
```

The allocator may reserve more space internally to satisfy alignment requirements.

For example:

```
Requested

↓

1 Byte

Allocated Block

↓

Aligned Boundary
```

Alignment improves CPU performance and is required for correct access on many architectures.

---

# Heap Fragmentation

Suppose:

```
Allocate

100 Bytes

↓

Free

↓

Allocate

50 Bytes

↓

Allocate

200 Bytes
```

After many allocations and frees,

the heap can become fragmented.

```
Allocated

Free

Allocated

Free

Allocated
```

Modern allocators work to manage fragmentation efficiently.

---

# Common Uses of `malloc()`

- Dynamic Arrays
- Linked Lists
- Trees
- Graphs
- Hash Tables
- Buffers
- Network Packets
- Database Pages

---

# Real-World Example — Web Server

Suppose:

```
Client Request
```

Arrives.

Need memory for:

- HTTP Headers
- Body
- Cookies
- Session Data

The server allocates memory dynamically because request sizes vary.

---

# Real-World Example — Image Viewer

Open:

```
photo.jpg
```

Need memory equal to:

```
Image Size
```

The viewer allocates a buffer using dynamic memory before decoding the image.

---

# Real-World Example — PostgreSQL

When a query executes,

the database allocates memory for:

- Query Plan
- Buffers
- Temporary Results

The amount depends on the query,

so dynamic allocation is essential.

---

# Common Mistakes

---

## Forgetting `free()`

Wrong:

```c
int *p =
malloc(sizeof(int));

/* Never freed */
```

Result:

```
Memory Leak
```

---

## Dereferencing NULL

Wrong:

```c
int *p =
malloc(...);

*p = 10;
```

If `malloc()` returned `NULL`,

this causes undefined behavior.

Always check first.

---

## Using Memory Beyond Allocation

Wrong:

```c
int *p =
malloc(sizeof(int));

p[1] = 20;
```

Only one `int` was allocated.

Writing beyond it causes undefined behavior.

---

## Assuming Memory Is Zero

Wrong:

```c
int *p =
malloc(sizeof(int));

printf("%d\n", *p);
```

The value is indeterminate until you initialize it.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int n = 5;

    int *arr =
        malloc(n * sizeof(int));

    if(arr == NULL)
        return 1;

    for(int i = 0; i < n; i++)
    {
        arr[i] = (i + 1) * 10;
    }

    for(int i = 0; i < n; i++)
    {
        printf("%d ", arr[i]);
    }

    printf("\n");

    free(arr);

    return 0;
}
```

Output:

```
10 20 30 40 50
```

---

# Hands-on Labs

## Lab 1

Allocate memory for:

```
1 Integer
```

Store a value.

Print it.

Free the memory.

---

## Lab 2

Ask the user for:

```
N
```

Allocate an array of:

```
N Integers
```

---

## Lab 3

Allocate memory for a structure.

Store values.

Print them.

---

## Lab 4

Attempt to read an allocated integer before initializing it.

Observe that the value is unpredictable.

---

## Lab 5

Allocate several arrays of different sizes.

Print their addresses.

Observe that all are allocated on the heap.

---

# Interview Questions

### What does `malloc()` do?

It allocates a block of memory on the heap and returns a pointer to it.

---

### Where does `malloc()` allocate memory?

On the heap.

---

### What does `malloc()` return on failure?

```c
NULL
```

---

### Does `malloc()` initialize memory?

No.

The allocated memory contains indeterminate values.

---

### Why use `sizeof()` with `malloc()`?

To allocate the correct number of bytes in a portable way.

---

# Summary

```
Program

↓

malloc()

↓

C Runtime

↓

Heap Manager

↓

Operating System

↓

Heap Memory

↓

Pointer Returned
```

## Key Takeaways

- `malloc()` allocates memory dynamically from the heap.
- It returns a pointer to the allocated block or `NULL` on failure.
- The memory is **not initialized**.
- Always use `sizeof()` when calculating allocation sizes.
- Always check for `NULL`.
- Always release allocated memory with `free()`.
- `malloc()` is the foundation of dynamic data structures and systems programming.

---

# Next Chapter

## Chapter 78 — `calloc()`

You'll learn:

- How `calloc()` differs from `malloc()`
- Zero-initialized memory
- Array allocation
- Performance trade-offs
- When to use `calloc()` vs `malloc()`
- How operating systems optimize zero-filled pages