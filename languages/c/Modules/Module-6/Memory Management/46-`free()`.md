# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 46 — `free()`

---

## Most beginners think:

> "`free()` deletes the pointer."

It doesn't.

`free()` **does not delete the pointer**.

It **returns the allocated heap memory back to the memory allocator**.

The pointer variable still exists.

It still contains the same address.

That's why calling `free()` incorrectly can lead to:

- Dangling pointers
- Double free bugs
- Use-after-free vulnerabilities
- Program crashes
- Security issues

Understanding `free()` is one of the most important topics in C.

---

# Learning Objectives

After this chapter you will understand:

- What `free()` is
- Why `free()` exists
- How `free()` works internally
- What happens after `free()`
- Dangling pointers
- Double free
- Use-after-free
- Best practices
- Common mistakes

---

# Why Do We Need `free()`?

Suppose a program keeps allocating memory.

```text
malloc()

↓

malloc()

↓

malloc()

↓

malloc()
```

If nothing is released,

memory usage keeps increasing.

Eventually:

```text
Heap

↓

Full

↓

No Memory Left
```

Programs must return memory they no longer need.

That's why `free()` exists.

---

# What is `free()`?

Prototype:

```c
#include <stdlib.h>

void free(void *ptr);
```

Meaning:

```text
Allocated Memory

↓

Return To Allocator
```

Unlike `malloc()`,

`free()` returns nothing.

---

# Basic Example

```c
int *p = malloc(sizeof(int));

*p = 100;

free(p);
```

Execution:

```text
malloc()

↓

Heap Block

↓

Use Memory

↓

free()

↓

Heap Block Released
```

---

# What Happens After `free()`?

Before:

```text
Stack

p

↓

Heap

100
```

After:

```text
Stack

p

↓

Old Address ❌

Heap

Released
```

Notice:

The pointer still stores the old address.

The memory is **no longer yours**.

---

# `free()` Does NOT Set the Pointer to NULL

Wrong assumption:

```text
free(p);

↓

p == NULL
```

False.

Actual:

```text
free(p);

↓

p still contains old address
```

You must do this yourself.

```c
free(p);

p = NULL;
```

---

# Why Set It to NULL?

Suppose later:

```c
if(p != NULL)
{
    /* use pointer */
}
```

If you forgot:

```c
p = NULL;
```

the program may accidentally use freed memory.

---

# Use-After-Free

Wrong:

```c
int *p = malloc(sizeof(int));

free(p);

*p = 10;
```

Possible results:

- Crash
- Random value
- Silent memory corruption
- Security vulnerability

This is called a **use-after-free** bug.

---

# Double Free

Wrong:

```c
free(p);

free(p);
```

The same block is released twice.

Modern memory allocators usually detect this and terminate the program.

Typical message:

```text
double free detected
```

---

# Safe Pattern

```c
free(p);

p = NULL;
```

Now:

```c
free(p);
```

is safe because:

```c
free(NULL);
```

does nothing.

---

# `free(NULL)`

This is perfectly valid.

```c
int *p = NULL;

free(p);
```

Nothing happens.

No crash.

This behavior is defined by the C standard.

---

# Memory Diagram

Before:

```text
Pointer

↓

Heap

□□□□□□□□
```

After:

```text
Pointer

↓

Old Address ❌

Heap

Free Block
```

After resetting:

```text
Pointer

↓

NULL ✔
```

---

# How Does `free()` Work Internally?

Conceptually:

```text
free()

↓

Memory Allocator

↓

Mark Block Free

↓

Future malloc() Can Reuse It
```

The allocator usually **marks the block as available**.

It doesn't necessarily erase the bytes immediately.

---

# Real-World Uses

Every major C application relies on `free()`:

- Linux kernel (kernel allocators)
- Redis
- PostgreSQL
- SQLite
- Nginx
- Browsers
- Compilers
- Embedded firmware

---

# Common Mistakes

## Forgetting `free()`

```text
malloc()

↓

No free()

↓

Memory Leak
```

---

## Using Memory After Free

```c
free(ptr);

ptr[0] = 5;
```

Undefined behavior.

---

## Double Free

```c
free(ptr);

free(ptr);
```

Undefined behavior.

---

## Freeing Stack Memory

Wrong:

```c
int value = 10;

free(&value);
```

Only memory allocated by:

- `malloc()`
- `calloc()`
- `realloc()`

may be released using `free()`.

---

# Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *p = malloc(sizeof(int));

    if (p == NULL)
        return 1;

    *p = 42;

    printf("%d\n", *p);

    free(p);

    p = NULL;

    return 0;
}
```

Output:

```text
42
```

---

# Hands-on Labs

## Lab 1

Allocate memory.

Store values.

Release it using `free()`.

---

## Lab 2

Print the pointer before and after:

```c
p = NULL;
```

---

## Lab 3

Intentionally perform a double free.

Observe the runtime error.

---

## Lab 4

Compile using AddressSanitizer.

```bash
gcc -fsanitize=address program.c
```

Observe the diagnostics.

---

## Lab 5

Run:

```bash
valgrind ./program
```

Verify there are no leaks or invalid frees.

---

# Interview Questions

### What does `free()` do?

It returns dynamically allocated heap memory to the memory allocator.

---

### Does `free()` delete the pointer?

No.

The pointer variable still exists.

Only the memory block is released.

---

### What is a dangling pointer?

A pointer that still contains the address of memory that has already been freed.

---

### Is `free(NULL)` safe?

Yes.

The C standard guarantees that it does nothing.

---

### Can you call `free()` on stack memory?

No.

Only heap memory allocated by allocation functions may be passed to `free()`.

---

# Summary

```text
malloc()

↓

Heap Allocation

↓

Use Memory

↓

free()

↓

Memory Returned

↓

Pointer Still Exists

↓

Set Pointer = NULL ✔
```

Key Takeaways:

- `free()` releases heap memory.
- It does **not** delete the pointer.
- After `free()`, the pointer becomes dangling until reset.
- Set pointers to `NULL` after freeing them.
- Never use memory after `free()`.
- Never free the same block twice.
- Never call `free()` on stack or global variables.

---

# Next Chapter

## Chapter 47 — Memory Leaks

You'll learn:

- What a memory leak is
- Why memory leaks happen
- How leaks affect long-running applications
- Detecting leaks with Valgrind and AddressSanitizer
- Best practices to prevent leaks
- How companies debug memory leaks in production# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 46 — `free()`

---

## Most beginners think:

> "`free()` deletes the pointer."

It doesn't.

`free()` **does not delete the pointer**.

It **returns the allocated heap memory back to the memory allocator**.

The pointer variable still exists.

It still contains the same address.

That's why calling `free()` incorrectly can lead to:

- Dangling pointers
- Double free bugs
- Use-after-free vulnerabilities
- Program crashes
- Security issues

Understanding `free()` is one of the most important topics in C.

---

# Learning Objectives

After this chapter you will understand:

- What `free()` is
- Why `free()` exists
- How `free()` works internally
- What happens after `free()`
- Dangling pointers
- Double free
- Use-after-free
- Best practices
- Common mistakes

---

# Why Do We Need `free()`?

Suppose a program keeps allocating memory.

```text
malloc()

↓

malloc()

↓

malloc()

↓

malloc()
```

If nothing is released,

memory usage keeps increasing.

Eventually:

```text
Heap

↓

Full

↓

No Memory Left
```

Programs must return memory they no longer need.

That's why `free()` exists.

---

# What is `free()`?

Prototype:

```c
#include <stdlib.h>

void free(void *ptr);
```

Meaning:

```text
Allocated Memory

↓

Return To Allocator
```

Unlike `malloc()`,

`free()` returns nothing.

---

# Basic Example

```c
int *p = malloc(sizeof(int));

*p = 100;

free(p);
```

Execution:

```text
malloc()

↓

Heap Block

↓

Use Memory

↓

free()

↓

Heap Block Released
```

---

# What Happens After `free()`?

Before:

```text
Stack

p

↓

Heap

100
```

After:

```text
Stack

p

↓

Old Address ❌

Heap

Released
```

Notice:

The pointer still stores the old address.

The memory is **no longer yours**.

---

# `free()` Does NOT Set the Pointer to NULL

Wrong assumption:

```text
free(p);

↓

p == NULL
```

False.

Actual:

```text
free(p);

↓

p still contains old address
```

You must do this yourself.

```c
free(p);

p = NULL;
```

---

# Why Set It to NULL?

Suppose later:

```c
if(p != NULL)
{
    /* use pointer */
}
```

If you forgot:

```c
p = NULL;
```

the program may accidentally use freed memory.

---

# Use-After-Free

Wrong:

```c
int *p = malloc(sizeof(int));

free(p);

*p = 10;
```

Possible results:

- Crash
- Random value
- Silent memory corruption
- Security vulnerability

This is called a **use-after-free** bug.

---

# Double Free

Wrong:

```c
free(p);

free(p);
```

The same block is released twice.

Modern memory allocators usually detect this and terminate the program.

Typical message:

```text
double free detected
```

---

# Safe Pattern

```c
free(p);

p = NULL;
```

Now:

```c
free(p);
```

is safe because:

```c
free(NULL);
```

does nothing.

---

# `free(NULL)`

This is perfectly valid.

```c
int *p = NULL;

free(p);
```

Nothing happens.

No crash.

This behavior is defined by the C standard.

---

# Memory Diagram

Before:

```text
Pointer

↓

Heap

□□□□□□□□
```

After:

```text
Pointer

↓

Old Address ❌

Heap

Free Block
```

After resetting:

```text
Pointer

↓

NULL ✔
```

---

# How Does `free()` Work Internally?

Conceptually:

```text
free()

↓

Memory Allocator

↓

Mark Block Free

↓

Future malloc() Can Reuse It
```

The allocator usually **marks the block as available**.

It doesn't necessarily erase the bytes immediately.

---

# Real-World Uses

Every major C application relies on `free()`:

- Linux kernel (kernel allocators)
- Redis
- PostgreSQL
- SQLite
- Nginx
- Browsers
- Compilers
- Embedded firmware

---

# Common Mistakes

## Forgetting `free()`

```text
malloc()

↓

No free()

↓

Memory Leak
```

---

## Using Memory After Free

```c
free(ptr);

ptr[0] = 5;
```

Undefined behavior.

---

## Double Free

```c
free(ptr);

free(ptr);
```

Undefined behavior.

---

## Freeing Stack Memory

Wrong:

```c
int value = 10;

free(&value);
```

Only memory allocated by:

- `malloc()`
- `calloc()`
- `realloc()`

may be released using `free()`.

---

# Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *p = malloc(sizeof(int));

    if (p == NULL)
        return 1;

    *p = 42;

    printf("%d\n", *p);

    free(p);

    p = NULL;

    return 0;
}
```

Output:

```text
42
```

---

# Hands-on Labs

## Lab 1

Allocate memory.

Store values.

Release it using `free()`.

---

## Lab 2

Print the pointer before and after:

```c
p = NULL;
```

---

## Lab 3

Intentionally perform a double free.

Observe the runtime error.

---

## Lab 4

Compile using AddressSanitizer.

```bash
gcc -fsanitize=address program.c
```

Observe the diagnostics.

---

## Lab 5

Run:

```bash
valgrind ./program
```

Verify there are no leaks or invalid frees.

---

# Interview Questions

### What does `free()` do?

It returns dynamically allocated heap memory to the memory allocator.

---

### Does `free()` delete the pointer?

No.

The pointer variable still exists.

Only the memory block is released.

---

### What is a dangling pointer?

A pointer that still contains the address of memory that has already been freed.

---

### Is `free(NULL)` safe?

Yes.

The C standard guarantees that it does nothing.

---

### Can you call `free()` on stack memory?

No.

Only heap memory allocated by allocation functions may be passed to `free()`.

---

# Summary

```text
malloc()

↓

Heap Allocation

↓

Use Memory

↓

free()

↓

Memory Returned

↓

Pointer Still Exists

↓

Set Pointer = NULL ✔
```

Key Takeaways:

- `free()` releases heap memory.
- It does **not** delete the pointer.
- After `free()`, the pointer becomes dangling until reset.
- Set pointers to `NULL` after freeing them.
- Never use memory after `free()`.
- Never free the same block twice.
- Never call `free()` on stack or global variables.

---

# Next Chapter

## Chapter 47 — Memory Leaks

You'll learn:

- What a memory leak is
- Why memory leaks happen
- How leaks affect long-running applications
- Detecting leaks with Valgrind and AddressSanitizer
- Best practices to prevent leaks
- How companies debug memory leaks in production