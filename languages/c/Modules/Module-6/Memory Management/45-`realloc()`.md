# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 45 — `realloc()`

---

## Most beginners think:

> "Once memory is allocated, its size can never change."

That's not true.

Suppose you allocate memory for **10 students**.

Later, you discover there are **100 students**.

Should you:

- Allocate a completely new block?
- Copy everything manually?
- Free the old memory?

You could...

But C already provides a function that does this for you.

That function is **`realloc()`**.

---

# Learning Objectives

After this chapter you will understand:

- What `realloc()` is
- Why `realloc()` exists
- How `realloc()` works
- Expanding memory
- Shrinking memory
- When memory moves
- Safe usage
- Common mistakes
- Real-world uses

---

# Why Do We Need `realloc()`?

Suppose you don't know how much data you'll receive.

Example:

```text
User Uploads File

↓

File Grows

↓

Need More Memory
```

Or:

```text
Network Server

↓

More Requests

↓

Need Larger Buffer
```

Instead of creating a completely new allocation manually,

`realloc()` can resize an existing block.

---

# What is `realloc()`?

Prototype:

```c
#include <stdlib.h>

void *realloc(void *ptr,
              size_t new_size);
```

Meaning:

```text
Existing Memory

↓

Resize

↓

Return New Pointer
```

---

# Basic Example

```c
int *numbers =
malloc(5 * sizeof(int));
```

Memory:

```text
Heap

10

20

30

40

50
```

Now:

```c
numbers =
realloc(numbers,
10 * sizeof(int));
```

The array can now hold:

```text
10 Integers
```

---

# What Happens Internally?

There are **two possibilities**.

---

# Case 1 — Enough Free Space

Current memory:

```text
Heap

Allocated Block

□□□□□

Free Space

□□□□□
```

`realloc()` simply expands the block.

Result:

```text
□□□□□□□□□□
```

Same address.

Very fast.

---

# Case 2 — Not Enough Space

Current heap:

```text
Allocated

□□□□□

Used

■■■■■

Free

□□□
```

No room to grow.

The allocator:

1. Allocates a larger block.
2. Copies the old data.
3. Frees the old block.
4. Returns the new address.

```text
Old Block

↓

Copy

↓

New Block

↓

Old Freed
```

The pointer **may change**.

---

# Why You Must Use the Returned Pointer

Wrong:

```c
realloc(numbers,
20*sizeof(int));
```

If the memory moved,

you've lost the new address.

Correct:

```c
numbers =
realloc(numbers,
20*sizeof(int));
```

Always store the returned pointer.

---

# Safer Pattern

Even better:

```c
int *temp =
realloc(numbers,
20*sizeof(int));

if(temp != NULL)
{
    numbers = temp;
}
```

Why?

If `realloc()` fails,

the original block is **still valid**.

If you overwrite your only pointer immediately, you could lose access to the original memory and create a memory leak.

---

# Shrinking Memory

`realloc()` can also reduce memory.

```c
numbers =
realloc(numbers,
3 * sizeof(int));
```

Unused memory is returned to the allocator.

---

# `realloc(NULL, size)`

Special case:

```c
int *p =
realloc(NULL,
100);
```

This behaves exactly like:

```c
malloc(100);
```

---

# `realloc(ptr, 0)`

Another special case:

```c
realloc(ptr,0);
```

Depending on the C standard and implementation, this may free the block and return `NULL` or behave in an implementation-defined way.

For portable code, call `free(ptr)` directly when you want to release memory.

---

# Memory Diagram

```text
Before

Pointer

↓

Heap

□□□□□

-----------------

After Expansion

Pointer

↓

Heap

□□□□□□□□□□
```

---

# Real-World Uses

`realloc()` is used in:

- Dynamic arrays
- Text editors
- JSON parsers
- Browsers
- Redis
- PostgreSQL
- Log collectors
- Network buffers

Whenever the amount of data changes while the program runs.

---

# Common Mistakes

---

## Ignoring the Return Value

Wrong:

```c
realloc(ptr,100);
```

Always capture the returned pointer.

---

## Overwriting the Pointer Immediately

Risky:

```c
ptr =
realloc(ptr,
newSize);
```

If `realloc()` returns `NULL`, you've lost the original pointer.

Use a temporary variable.

---

## Assuming the Address Never Changes

After `realloc()`:

```text
Old Address

↓

New Address
```

The memory may have moved.

Never assume the pointer value remains the same.

---

# Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *arr = malloc(5 * sizeof(int));

    if (arr == NULL)
        return 1;

    for (int i = 0; i < 5; i++)
        arr[i] = i + 1;

    int *temp = realloc(arr, 10 * sizeof(int));

    if (temp == NULL)
    {
        free(arr);
        return 1;
    }

    arr = temp;

    for (int i = 5; i < 10; i++)
        arr[i] = i + 1;

    for (int i = 0; i < 10; i++)
        printf("%d ", arr[i]);

    free(arr);

    return 0;
}
```

Output:

```text
1 2 3 4 5 6 7 8 9 10
```

---

# Hands-on Labs

## Lab 1

Allocate an array of five integers.

Expand it to ten integers using `realloc()`.

---

## Lab 2

Shrink the array back to three integers.

---

## Lab 3

Print the pointer address before and after `realloc()`.

Observe whether it changed.

---

## Lab 4

Use a temporary pointer while calling `realloc()`.

---

## Lab 5

Run:

```bash
valgrind ./program
```

Verify that there are no leaks.

---

# Interview Questions

### What does `realloc()` do?

It changes the size of an existing heap allocation.

---

### Can `realloc()` move memory?

Yes.

If there isn't enough free space after the current block, it allocates a new block, copies the data, frees the old block, and returns a new pointer.

---

### Why should you use a temporary pointer?

Because if `realloc()` fails, it returns `NULL` while the original allocation remains valid.

Using a temporary pointer prevents losing access to that memory.

---

### What does `realloc(NULL, size)` do?

It behaves like `malloc(size)`.

---

### Can `realloc()` shrink memory?

Yes.

It can both increase and decrease the size of an allocation.

---

# Summary

```text
Existing Block
      │
      ▼
realloc()
      │
      ├── Expand
      ├── Shrink
      └── Move (if needed)
      │
      ▼
New Pointer
```

Key Takeaways:

- `realloc()` resizes an existing heap allocation.
- The memory block may stay in place or move.
- Always use the returned pointer.
- Use a temporary variable to handle failures safely.
- `realloc()` is essential for building dynamic data structures.

---

# Next Chapter

## Chapter 46 — `free()`

You'll learn:

- What `free()` actually does
- How memory is returned to the allocator
- Double free bugs
- Use-after-free
- Best practices for releasing memory
- Why every `malloc()`, `calloc()`, and `realloc()` should have a matching `free()`