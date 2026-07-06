# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 82 — Dangling Pointers & Use-After-Free

---

# Most beginners think:

> "After calling `free()`, the pointer disappears."

It doesn't.

This is one of the most dangerous misconceptions in C.

Consider:

```c
int *p =
malloc(sizeof(int));

free(p);
```

Many people imagine:

```
p

↓

Gone
```

Reality:

```
p

↓

Still Exists

↓

Still Contains Address

↓

Memory No Longer Belongs To You
```

The pointer is still pointing somewhere,

but that memory has already been returned to the allocator.

This creates a **dangling pointer**.

Dangling pointers are responsible for countless crashes,

security vulnerabilities,

and even remote code execution exploits.

---

# Learning Objectives

After this chapter you will understand:

- What a dangling pointer is
- Use-after-free
- Why these bugs happen
- Why they're dangerous
- Real-world security issues
- Prevention techniques
- Best practices

---

# What is a Dangling Pointer?

A dangling pointer is:

> A pointer that still holds an address,
> but the memory at that address is no longer valid to access.

Example:

```c
int *p =
malloc(sizeof(int));

free(p);
```

Memory:

Before:

```
p

↓

Heap

↓

25
```

After:

```
p

↓

Old Address

↓

Freed Memory
```

The pointer still exists.

Only the ownership changed.

---

# Why Is This Dangerous?

Suppose:

```c
free(p);

printf("%d",
       *p);
```

Question:

What should be printed?

Answer:

Nobody knows.

The C standard calls this:

```
Undefined Behavior
```

Possible outcomes:

- Old value
- Random value
- Crash
- Silent corruption
- Appears to work

All are possible.

---

# Use-After-Free

Accessing memory after:

```c
free()
```

is called:

```
Use-After-Free
```

Example:

```c
free(p);

*p = 100;
```

You are writing into memory that no longer belongs to your program.

---

# Visualizing Use-After-Free

Before:

```
Heap

↓

Allocated

↓

25
```

After:

```
free()

↓

Available For Reuse
```

Then:

```c
*p = 100;
```

You may overwrite memory that has already been given to another allocation.

---

# Memory Reuse Example

Suppose:

```c
int *a =
malloc(sizeof(int));

free(a);

int *b =
malloc(sizeof(int));
```

Allocator may reuse the same block.

Visualization:

```
Heap

↓

a

↓

Freed

↓

Reused

↓

b
```

Now:

```c
*a = 50;
```

may accidentally modify:

```
b
```

This can corrupt unrelated program data.

---

# Example Program

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *p =
        malloc(sizeof(int));

    if(p == NULL)
        return 1;

    *p = 10;

    free(p);

    printf("%d\n", *p);

    return 0;
}
```

This program has undefined behavior.

It might print:

```
10
```

today,

and crash tomorrow.

Never rely on it.

---

# Why Does It Sometimes "Work"?

Many beginners say:

> "I tried it, and it printed the correct value."

Example:

```
free()

↓

Memory Not Reused Yet
```

The bytes may still happen to contain the old data.

That does **not** make the program correct.

The allocator is free to reuse or modify that memory at any time.

---

# Security Vulnerability

Suppose:

```
Authentication Object

↓

free()

↓

Attacker Controls Reused Memory
```

Later:

Program accesses:

```
Dangling Pointer
```

The attacker may influence program behavior.

Historically,

use-after-free bugs have led to serious vulnerabilities in browsers,

operating systems,

and other large software systems.

---

# Double Free

Example:

```c
free(p);

free(p);
```

First:

```
Memory Returned
```

Second:

```
Allocator Receives Same Block Again
```

This is undefined behavior.

It can corrupt allocator metadata and lead to crashes or security issues.

---

# Stack Dangling Pointer

Example:

```c
int *test()
{
    int x = 10;

    return &x;
}
```

After:

```
Function Returns
```

Stack frame disappears.

Returned pointer becomes dangling.

---

# Visualization

```
Function

↓

Stack Frame

↓

x = 10
```

Return.

```
Stack Frame Removed
```

Pointer:

```
Still Points Here

↓

Invalid
```

---

# Multiple Pointers

Example:

```c
int *a =
malloc(sizeof(int));

int *b = a;
```

Now:

```
a

↓

Heap

↑

b
```

Call:

```c
free(a);
```

Result:

```
a

↓

Dangling

----------------

b

↓

Also Dangling
```

Setting only:

```c
a = NULL;
```

does **not** fix:

```
b
```

Every alias becomes invalid after the memory is freed.

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

Accidentally using:

```c
*p
```

will usually fail immediately instead of silently corrupting memory.

This only protects that specific pointer.

Other aliases must also be handled correctly.

---

# Ownership Principle

Professional software often follows:

```
One Owner

↓

One free()
```

Only the owner is responsible for releasing the allocation.

This greatly reduces dangling pointer bugs.

---

# Real-World Example — Browser

A browser tab allocates:

- DOM Nodes
- JavaScript Objects
- Images

Close the tab.

Memory is freed.

If another part of the browser still uses those objects,

a use-after-free bug may occur.

---

# Real-World Example — Operating System

Kernel Driver:

```
Allocate Buffer

↓

free()

↓

Interrupt Uses Buffer
```

The interrupt handler now accesses invalid memory.

This is a classic systems programming bug.

---

# Real-World Example — Database

A query result is freed.

Another thread mistakenly accesses it.

Possible outcomes:

- Crash
- Corrupted Results
- Data Loss

---

# Detecting Use-After-Free

AddressSanitizer:

Compile:

```bash
gcc -fsanitize=address \
    main.c
```

Run:

```bash
./a.out
```

AddressSanitizer reports:

```
heap-use-after-free
```

with detailed diagnostics.

---

# Common Mistakes

---

## Using Memory After `free()`

Wrong:

```c
free(ptr);

printf("%d",
       *ptr);
```

Undefined behavior.

---

## Returning Stack Addresses

Wrong:

```c
return &local;
```

Local variables disappear when the function returns.

---

## Double Free

Wrong:

```c
free(ptr);

free(ptr);
```

Undefined behavior.

---

## Forgetting About Aliases

Wrong:

```c
int *a =
malloc(sizeof(int));

int *b = a;

free(a);

/* b is now dangling too */
```

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *p =
        malloc(sizeof(int));

    if(p == NULL)
        return 1;

    *p = 100;

    free(p);

    p = NULL;

    return 0;
}
```

Safe.

No dangling pointer remains in `p`.

---

# Hands-on Labs

## Lab 1

Allocate memory.

Free it.

Set the pointer to `NULL`.

Verify the program no longer has a dangling pointer through that variable.

---

## Lab 2

Create two pointers to the same allocation.

Free one.

Explain why the other becomes invalid.

---

## Lab 3

Return the address of a local variable.

Observe the compiler warning (if enabled) and explain why the pointer is invalid.

---

## Lab 4

Compile a program with:

```bash
-fsanitize=address
```

Create a use-after-free bug.

Observe the diagnostic report.

---

## Lab 5

Use Valgrind to detect invalid memory accesses in a program containing a dangling pointer.

---

# Interview Questions

### What is a dangling pointer?

A pointer that refers to memory that is no longer valid to access.

---

### What is use-after-free?

Accessing dynamically allocated memory after it has been released with `free()`.

---

### Why are dangling pointers dangerous?

They can cause crashes, memory corruption, unpredictable behavior, and security vulnerabilities.

---

### Does setting a pointer to `NULL` free memory?

No.

It only changes the pointer variable.

You must call:

```c
free()
```

first.

---

### Does setting one pointer to `NULL` fix all aliases?

No.

Any other pointer referring to the same freed memory also becomes dangling and must be handled appropriately.

---

# Summary

```
malloc()

↓

Pointer

↓

free()

↓

Pointer Still Exists

↓

Dangling Pointer

↓

Use-After-Free

↓

Undefined Behavior
```

Safe Pattern:

```
malloc()

↓

Use Memory

↓

free()

↓

Pointer = NULL
```

## Key Takeaways

- `free()` releases memory but does not modify pointer variables.
- A pointer to freed memory is called a dangling pointer.
- Accessing freed memory results in undefined behavior.
- Setting a pointer to `NULL` after `free()` is a good defensive practice.
- Be careful with multiple pointers (aliases) to the same allocation.
- Use tools like AddressSanitizer and Valgrind to detect dangling pointer bugs.
- Preventing use-after-free bugs is essential for writing secure and reliable systems software.

---

# Next Chapter

## Chapter 83 — Double Free & Heap Corruption

You'll learn:

- What heap corruption is
- How double free damages allocator metadata
- Invalid frees
- Buffer overruns on the heap
- Security implications
- How modern allocators detect corruption
- Best practices for safe heap management