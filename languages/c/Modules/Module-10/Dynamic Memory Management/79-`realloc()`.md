# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 79 — `realloc()`

---

# Most beginners think:

> "If I need more memory, I must allocate a new block and copy everything manually."

Years ago,

that was often true.

Today,

C provides a function that can resize an existing allocation.

That function is:

```c
realloc()
```

`realloc()` is one of the most powerful memory management functions in C.

Dynamic arrays,

database buffers,

web servers,

language runtimes,

and containers like C++'s `std::vector`

all rely on the same idea.

---

# Learning Objectives

After this chapter you will understand:

- What `realloc()` is
- Growing memory
- Shrinking memory
- Internal working
- When memory moves
- Safe usage
- Common mistakes
- Real-world examples

---

# Why Do We Need `realloc()`?

Suppose:

```c
int *arr =
malloc(5 * sizeof(int));
```

Memory:

```
Heap

↓

10

20

30

40

50
```

Later,

the user wants:

```
100 Integers
```

What now?

The original allocation is too small.

Instead of manually allocating a larger block and copying the data,

we can ask:

```c
realloc()
```

---

# What is `realloc()`?

Prototype:

```c
void *realloc(
    void *ptr,
    size_t new_size
);
```

Meaning:

```
Resize

↓

Existing Heap Block

↓

Return Pointer
```

---

# Basic Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *arr =
        malloc(5 * sizeof(int));

    if(arr == NULL)
        return 1;

    arr =
        realloc(arr,
                10 * sizeof(int));

    if(arr == NULL)
        return 1;

    free(arr);

    return 0;
}
```

Memory now has room for:

```
10 Integers
```

---

# What Happens Internally?

Suppose:

Current:

```
5 Integers
```

Need:

```
10 Integers
```

The allocator checks:

```
Can This Block Grow Here?
```

---

# Case 1 — Enough Space Exists

Suppose:

```
Allocated

↓

Free

↓

Free

↓

Free
```

The allocator simply extends the existing block.

```
Before

↓

AAAAA_____

After

↓

AAAAAAAAAA
```

Pointer:

```
Same
```

No copying needed.

---

# Case 2 — No Space Available

Suppose:

```
AAAAA

BBBBB

CCCCC
```

Need:

```
10 Integers
```

Impossible.

Allocator:

```
Find Larger Block

↓

Copy Data

↓

Free Old Block

↓

Return New Pointer
```

Memory:

```
Old

↓

AAAAA

↓

Copied

↓

New Location

↓

AAAAAAAAAA
```

The pointer changes.

---

# Why the Pointer May Change

Many beginners write:

```c
realloc(arr,
        new_size);
```

Wrong.

Suppose memory moved.

```
Old Pointer

↓

Invalid
```

You lost the new address.

Always store the return value.

---

# Safe Pattern

Wrong:

```c
arr =
realloc(arr,
        new_size);
```

If `realloc()` fails,

it returns:

```
NULL
```

and the original pointer is lost,

creating a memory leak.

---

Correct:

```c
int *temp =
    realloc(arr,
            new_size);

if(temp != NULL)
{
    arr = temp;
}
else
{
    /* Allocation failed.
       arr still points to
       the original block. */
}
```

This is the professional approach.

---

# Growing an Array

Example:

```c
int capacity = 5;

int *arr =
malloc(capacity *
sizeof(int));
```

Need more space.

```c
capacity *= 2;

int *temp =
realloc(arr,
capacity *
sizeof(int));

if(temp != NULL)
{
    arr = temp;
}
```

Now:

```
5

↓

10

↓

20

↓

40

↓

80
```

Exactly how many dynamic array implementations grow.

---

# Shrinking Memory

Suppose:

```
1000 Integers
```

Need only:

```
100
```

Call:

```c
realloc(arr,
100 *
sizeof(int));
```

Allocator may reduce the block size.

Unused memory becomes available for future allocations.

---

# What Happens to Existing Data?

Suppose:

```
10

20

30

40

50
```

Grow the block.

Result:

```
10

20

30

40

50

?

?

?

?
```

Existing bytes are preserved (up to the smaller of the old and new sizes).

The newly added portion is **not initialized**.

---

# `realloc()` Does NOT Initialize New Memory

Many beginners expect:

```
New Area

↓

0
```

Wrong.

The newly added bytes contain indeterminate values.

Initialize them yourself if needed.

---

# What If `new_size == 0`?

Many implementations behave like:

```c
free(ptr);
```

However,

the C standard has changed over time, and behavior may be implementation-defined.

For portable code,

prefer:

```c
free(ptr);

ptr = NULL;
```

instead of relying on:

```c
realloc(ptr, 0);
```

---

# What If `ptr == NULL`?

Example:

```c
int *arr =
realloc(NULL,
10 *
sizeof(int));
```

This behaves the same as:

```c
malloc(
10 *
sizeof(int));
```

A useful feature.

---

# Memory Visualization

Before:

```
Heap

↓

10

20

30

40

50
```

After:

```
10

20

30

40

50

?

?

?

?
```

Old data preserved.

New memory uninitialized.

---

# Real-World Example — Dynamic Array

Suppose:

```
Array Capacity

↓

10
```

User inserts:

```
11th Element
```

Runtime:

```
Allocate Larger Array

↓

Copy

↓

Free Old

↓

Continue
```

This strategy is used in many libraries and languages.

---

# Real-World Example — Web Server

A request body arrives.

Initially:

```
4 KB Buffer
```

Client sends:

```
20 KB
```

Server grows the buffer dynamically.

---

# Real-World Example — Database

A query returns more rows than expected.

Database:

```
Buffer

↓

realloc()

↓

Continue Processing
```

---

# Common Mistakes

---

## Losing the Original Pointer

Wrong:

```c
arr =
realloc(arr,
new_size);
```

Always use a temporary pointer.

---

## Forgetting That Memory May Move

After:

```c
realloc()
```

The old address may no longer be valid.

Always use the returned pointer.

---

## Assuming New Memory Is Zero

Wrong.

The additional memory is uninitialized.

---

## Using the Old Pointer After Movement

Wrong:

```c
int *old = arr;

arr =
realloc(arr,
new_size);

/* Using old here is unsafe if
   the allocation moved. */
```

Only use the updated pointer.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *arr =
        malloc(3 * sizeof(int));

    if(arr == NULL)
        return 1;

    arr[0] = 10;
    arr[1] = 20;
    arr[2] = 30;

    int *temp =
        realloc(arr,
                6 * sizeof(int));

    if(temp == NULL)
    {
        free(arr);
        return 1;
    }

    arr = temp;

    arr[3] = 40;
    arr[4] = 50;
    arr[5] = 60;

    for(int i = 0; i < 6; i++)
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
10 20 30 40 50 60
```

---

# Hands-on Labs

## Lab 1

Allocate:

```
5 Integers
```

Resize to:

```
10 Integers
```

Verify the first five values remain unchanged.

---

## Lab 2

Resize an array from:

```
100 Integers

↓

20 Integers
```

Observe that only the first 20 elements remain valid.

---

## Lab 3

Use the safe temporary-pointer pattern for every `realloc()` call.

---

## Lab 4

Print addresses before and after `realloc()`.

Observe that sometimes the address changes and sometimes it doesn't.

---

## Lab 5

Build a dynamic array that doubles its capacity whenever it becomes full.

---

# Interview Questions

### What does `realloc()` do?

It resizes an existing dynamically allocated memory block.

---

### Can `realloc()` move memory?

Yes.

If the current block cannot be expanded,

it may allocate a new block, copy the data, free the old block, and return a new pointer.

---

### Does `realloc()` preserve existing data?

Yes,

up to the smaller of the old and new allocation sizes.

---

### Is the newly added memory initialized?

No.

The additional bytes contain indeterminate values.

---

### What is the safest way to use `realloc()`?

Store the result in a temporary pointer first.

Only replace the original pointer if the allocation succeeds.

---

# Summary

```
Existing Heap Block

↓

realloc()

↓

Can Grow?

↓

Yes

↓

Extend

----------------

No

↓

Allocate New Block

↓

Copy Data

↓

Free Old Block

↓

Return New Pointer
```

## Key Takeaways

- `realloc()` changes the size of a heap allocation.
- The returned pointer may be different from the original.
- Existing data is preserved.
- Newly allocated bytes are not initialized.
- Use a temporary pointer to avoid memory leaks if reallocation fails.
- `realloc()` is the foundation of dynamic arrays and many high-performance software systems.

---

# Next Chapter

## Chapter 80 — `free()`

You'll learn:

- Why freeing memory is essential
- What happens inside `free()`
- Heap allocator internals
- Memory leaks
- Double free
- Use-after-free
- Why garbage collection doesn't exist in C