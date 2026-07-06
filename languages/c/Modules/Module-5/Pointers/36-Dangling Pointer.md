# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 36 --- Dangling Pointer

------------------------------------------------------------------------

## Most beginners think:

> "If I have a pointer, I can safely use it forever."

That's not true.

A pointer is only valid while the memory it points to is valid.

If the memory is destroyed but the pointer still holds its old address,
the pointer becomes a **dangling pointer**.

Dangling pointers are one of the most common causes of:

-   Crashes
-   Memory corruption
-   Security vulnerabilities
-   Undefined behavior

Understanding dangling pointers is essential for writing safe C
programs.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a dangling pointer is
-   How dangling pointers are created
-   Returning addresses of local variables
-   Using memory after `free()`
-   Memory diagrams
-   How to prevent dangling pointers
-   Real-world consequences

------------------------------------------------------------------------

# What is a Dangling Pointer?

A dangling pointer is a pointer that still stores an address **after the
object at that address no longer exists**.

``` text
Pointer

↓

0x1000

❌ Memory is no longer valid
```

The address remains.

The object does not.

------------------------------------------------------------------------

# Scenario 1 --- Returning a Local Variable

Wrong:

``` c
int *getValue(void)
{
    int x = 100;
    return &x;
}
```

Why?

`x` lives on the stack.

When the function returns:

``` text
Stack Frame Destroyed

↓

x no longer exists
```

But the returned pointer still contains the old address.

It now dangles.

------------------------------------------------------------------------

# Memory Diagram

During function execution:

``` text
Stack

┌──────────────┐
│ x = 100      │
└──────────────┘
        ▲
        │
     Pointer
```

After the function returns:

``` text
Stack Frame Removed

Pointer
   │
   ▼
Old Address ❌
```

The pointer references invalid memory.

------------------------------------------------------------------------

# Scenario 2 --- Using Memory After `free()`

``` c
int *p = malloc(sizeof(int));

*p = 42;

free(p);
```

At this point:

``` text
Memory Released
```

But:

``` text
p

↓

Old Address
```

The pointer still stores the address.

Using it is undefined behavior.

------------------------------------------------------------------------

# Use-After-Free

Wrong:

``` c
free(p);

printf("%d\n", *p);
```

Possible results:

-   Crash
-   Garbage value
-   Appears to work
-   Security vulnerability

Never rely on the behavior.

------------------------------------------------------------------------

# Why Is This Dangerous?

After `free()`:

The memory may be reused.

``` text
Old Program

↓

free()

↓

Another Allocation

↓

Same Address
```

Your dangling pointer may accidentally modify unrelated data.

------------------------------------------------------------------------

# Safe Practice

Immediately reset the pointer.

``` c
free(p);

p = NULL;
```

Now:

``` text
p

↓

NULL
```

Accidentally dereferencing `NULL` is easier to detect than silently
corrupting memory.

------------------------------------------------------------------------

# Another Example

``` c
char *buffer = malloc(100);

free(buffer);

buffer = NULL;
```

Now you can safely test:

``` c
if (buffer == NULL)
{
    /* no valid memory */
}
```

------------------------------------------------------------------------

# Real-World Bugs

Dangling pointers have caused:

-   Browser security exploits
-   Kernel vulnerabilities
-   Database crashes
-   Remote code execution bugs

Many security advisories involve **use-after-free** errors.

------------------------------------------------------------------------

# Common Mistakes

## Returning Stack Addresses

``` c
return &localVariable;
```

Never do this.

------------------------------------------------------------------------

## Forgetting to Reset Pointers

``` c
free(ptr);

/* ptr still contains old address */
```

Set it to `NULL`.

------------------------------------------------------------------------

## Sharing Freed Memory

Two pointers:

``` text
p ──► Memory
q ──► Memory
```

Calling:

``` c
free(p);
```

makes **both** pointers invalid if they referred to the same allocation.

------------------------------------------------------------------------

# Real-World Uses

Understanding dangling pointers is critical for:

-   Linux kernel
-   Device drivers
-   Memory allocators
-   Game engines
-   Browsers
-   Databases
-   Embedded systems

------------------------------------------------------------------------

# Hands-on Labs

1.  Return the address of a local variable and observe the compiler
    warning.
2.  Allocate memory, free it, then inspect the pointer value.
3.  Set the pointer to `NULL` after `free()`.
4.  Use `valgrind` to detect a use-after-free bug.
5.  Compile with AddressSanitizer:

``` bash
gcc -fsanitize=address program.c
```

Observe the runtime error.

------------------------------------------------------------------------

# Interview Questions

### What is a dangling pointer?

A pointer that references memory that is no longer valid.

### How are dangling pointers created?

-   Returning addresses of local variables
-   Using memory after `free()`
-   Destroying an object while pointers still reference it

### Why are dangling pointers dangerous?

They cause undefined behavior, crashes, memory corruption, and security
vulnerabilities.

### How can you reduce the risk?

Free memory once and set the pointer to `NULL`.

------------------------------------------------------------------------

# Summary

``` text
Valid Memory
     │
     ▼
Pointer
     │
     ▼
free()
     │
     ▼
Memory Released
     │
     ▼
Dangling Pointer ❌
```

Key Takeaways:

-   A dangling pointer references invalid memory.
-   Never return pointers to local variables.
-   Never access memory after `free()`.
-   Set pointers to `NULL` after freeing them.
-   Dangling pointers are a major source of software bugs and security
    issues.

------------------------------------------------------------------------

# Next Chapter

## Chapter 37 --- Wild Pointer

You'll learn what wild pointers are, how they differ from dangling
pointers, why uninitialized pointers are dangerous, and how to avoid
them.
