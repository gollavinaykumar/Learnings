# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 44 — `calloc()`

---

## Most beginners think:

> "`calloc()` is just another name for `malloc()`."

It's not.

Both allocate memory from the heap, but there's one very important difference.

`malloc()`

- Allocates memory.

`calloc()`

- Allocates memory.
- **Initializes every byte to zero.**

This small difference makes `calloc()` much safer in many situations.

---

# Learning Objectives

After this chapter you will understand:

- What `calloc()` is
- Why `calloc()` exists
- `calloc()` vs `malloc()`
- Memory initialization
- Performance differences
- Internal working
- Common mistakes
- Real-world uses

---

# Why Do We Need `calloc()`?

Suppose we allocate memory for an array.

```c
int *numbers = malloc(5 * sizeof(int));
```

Memory might contain:

```text
4589
-20
1938
0
88
```

Those values are simply leftover bytes from previous allocations.

Sometimes you don't want random data.

You want everything to start at **zero**.

That's why `calloc()` exists.

---

# What is `calloc()`?

Prototype:

```c
#include <stdlib.h>

void *calloc(size_t count,
             size_t size);
```

Notice the parameters.

Instead of:

```text
Total Bytes
```

You provide:

```text
Number of Objects

×

Size of Each Object
```

---

# Basic Example

```c
int *numbers = calloc(5, sizeof(int));
```

Execution:

```text
Allocate:

5 Integers

↓

Heap

↓

Initialize Every Byte

↓

Return Pointer
```

---

# Memory Layout

Immediately after allocation:

```text
Heap

0

0

0

0

0
```

Everything starts at zero.

---

# `malloc()` vs `calloc()`

```c
malloc(5 * sizeof(int));
```

Result:

```text
???

???

???

???

???
```

Random data.

---

```c
calloc(5, sizeof(int));
```

Result:

```text
0

0

0

0

0
```

---

# Internal Working

Conceptually:

```text
calloc()

↓

Allocate Memory

↓

Fill Memory With Zero

↓

Return Pointer
```

It performs one additional step compared to `malloc()`.

---

# Why Zero Initialization Matters

Suppose:

```c
struct Student
{
    int id;
    int age;
};
```

Using:

```c
calloc(1,sizeof(struct Student));
```

Produces:

```text
id = 0

age = 0
```

Instead of random values.

---

# Checking for Failure

Like `malloc()`,

always verify the result.

```c
int *numbers =
calloc(100,sizeof(int));

if(numbers==NULL)
{
    printf("Allocation failed\n");
}
```

---

# Performance

People often ask:

> "Is `calloc()` slower?"

Usually:

Yes,

because it initializes memory.

However,

modern operating systems and C libraries optimize zero-filled pages heavily.

In many situations,

the performance difference is very small.

Choose the function based on correctness, not assumptions.

---

# Memory Diagram

```text
Program

↓

calloc()

↓

Heap

↓

Zero Filled Memory

↓

Pointer Returned
```

---

# Real-World Uses

`calloc()` is useful for:

- Structures
- Arrays
- Graphs
- Trees
- Hash tables
- Networking buffers
- Database pages
- Embedded systems

Whenever you need memory to start in a known state.

---

# Common Mistakes

## Assuming `malloc()` Returns Zero

Wrong.

```c
malloc()

↓

Random Data
```

Only `calloc()` guarantees zero initialization.

---

## Forgetting `free()`

```text
calloc()

↓

Memory Used

↓

Never Freed

↓

Memory Leak
```

Always call:

```c
free(pointer);
```

---

## Wrong Parameters

Wrong:

```c
calloc(sizeof(int),100);
```

Although the total bytes may match in some cases, the conventional order is:

```c
calloc(100, sizeof(int));
```

This is clearer and less error-prone.

---

# Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *arr = calloc(5, sizeof(int));

    if(arr == NULL)
        return 1;

    for(int i = 0; i < 5; i++)
    {
        printf("%d ", arr[i]);
    }

    free(arr);

    return 0;
}
```

Output:

```text
0 0 0 0 0
```

---

# Hands-on Labs

## Lab 1

Allocate an integer array using `calloc()`.

Print every element.

---

## Lab 2

Compare:

```c
malloc()
```

and

```c
calloc()
```

Observe the initial values.

---

## Lab 3

Allocate a structure using `calloc()`.

Verify all fields are zero.

---

## Lab 4

Use:

```bash
valgrind ./program
```

Ensure no memory leaks remain.

---

# Interview Questions

### What is the difference between `malloc()` and `calloc()`?

`malloc()` allocates memory.

`calloc()` allocates memory and initializes every byte to zero.

---

### Does `calloc()` always return zero values?

Yes.

It initializes the allocated memory to all zero bytes.

---

### Why does `calloc()` accept two parameters?

Because it receives:

- Number of objects
- Size of each object

---

### Is `calloc()` always slower than `malloc()`?

Not necessarily.

Modern operating systems optimize zero-initialized memory efficiently.

---

### Should memory allocated by `calloc()` be freed?

Yes.

Always release it using:

```c
free()
```

---

# Summary

```text
Program
     │
     ▼
calloc(count,size)
     │
     ▼
Heap Allocation
     │
     ▼
Memory Set To Zero
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

- `calloc()` allocates heap memory.
- Every byte is initialized to zero.
- It is safer than `malloc()` when starting with empty data.
- Always check for `NULL`.
- Every successful `calloc()` should eventually have a matching `free()`.

---

# Next Chapter

## Chapter 45 — `realloc()`

You'll learn:

- Why memory sometimes needs to grow or shrink
- How `realloc()` works
- Expanding dynamic arrays
- Shrinking memory blocks
- What happens when memory is moved
- Safe usage patterns