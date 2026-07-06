# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 101 — `restrict` Keyword (The Hidden Performance Feature)

---

# Most beginners have never even heard of:

```
restrict
```

Even many experienced C programmers never use it.

But if you work on:

- Linux
- Databases
- Scientific Computing
- Image Processing
- AI Libraries
- High Performance Computing (HPC)

you'll eventually encounter it.

The purpose of `restrict` is simple:

> **Give the compiler more information so it can generate faster code.**

---

# Learning Objectives

After this chapter you will understand:

- Why `restrict` was introduced
- Pointer aliasing
- Alias analysis
- Compiler optimizations
- Vectorization
- Real-world performance
- When to use `restrict`
- When NOT to use it

---

# First Question

Consider this function:

```c
void add(
    int *a,
    int *b)
{
    *a = *a + *b;
}
```

Question:

Can:

```
a

and

b
```

point to the same memory?

Example:

```c
add(&x, &x);
```

Yes.

Therefore,

the compiler must assume:

```
Possible Aliasing
```

---

# What is Aliasing?

Aliasing means:

```
Multiple Pointers

↓

Same Memory
```

Example:

```c
int x = 10;

int *a = &x;
int *b = &x;
```

Memory:

```
a

↓

x

↑

b
```

Both pointers refer to the same object.

---

# Why Does This Matter?

Suppose:

```c
*a = 5;

printf("%d",
*b);
```

Since:

```
a

↓

x

↑

b
```

The compiler knows:

```
*b

↓

5
```

Because both pointers alias.

---

# Compiler Problem

Suppose:

```c
void compute(
    int *a,
    int *b)
{
    *a = *b + 1;

    *b = 100;

    printf("%d", *a);
}
```

Question:

Can the compiler reorder operations?

Maybe not.

Because:

```
a

↓

Same Memory?

↓

Unknown
```

The compiler must be conservative.

---

# Enter `restrict`

Example:

```c
void compute(
    int *restrict a,
    int *restrict b)
```

Meaning:

> **During this function, `a` and `b` are the only means used to access their respective objects.**

The compiler may now assume:

```
a

≠

b
```

(No aliasing between the objects referenced through these restricted pointers.)

---

# Visualization

Without `restrict`

```
a

↓

?

↑

b

Same Memory?

↓

Possible
```

Compiler:

```
Play Safe
```

---

With `restrict`

```
a

↓

Object A

----------------

b

↓

Object B
```

Compiler:

```
Independent

↓

Optimize
```

---

# Why Is This Faster?

Suppose:

```c
for(int i=0;i<n;i++)
{
    a[i] =
    b[i] +
    c[i];
}
```

Compiler worries:

```
Could

a

and

b

Overlap?
```

Without knowing,

it avoids some optimizations.

---

# With `restrict`

```c
void add(
int *restrict a,
int *restrict b,
int *restrict c,
int n)
```

Compiler knows:

```
No Overlap
```

Now it can:

- Reorder instructions
- Use SIMD
- Vectorize loops
- Reduce memory loads

---

# Vectorization

Suppose:

```
Add

1000 Integers
```

Without `restrict`

Compiler:

```
One At A Time
```

With `restrict`

Compiler may generate instructions that process:

```
4

↓

8

↓

16

Elements At Once
```

using SIMD instructions when appropriate.

---

# Memory Visualization

Without `restrict`

```
a

↓

?

↓

Same As b?

↓

Compiler Unsure
```

With `restrict`

```
a

↓

Array A

----------------

b

↓

Array B

----------------

c

↓

Array C
```

Clearly independent.

---

# Simple Example

Without:

```c
void copy(
char *dest,
char *src)
```

Compiler:

```
Maybe Overlap
```

With:

```c
void copy(
char *restrict dest,
const char *restrict src)
```

Compiler:

```
Definitely Separate
```

More aggressive optimization becomes possible.

---

# Important Rule

Suppose:

```c
int x;

foo(&x, &x);
```

where:

```c
void foo(
int *restrict a,
int *restrict b)
```

This violates the promise made by `restrict`.

Result:

```
Undefined Behavior
```

The compiler is allowed to assume the promise is true.

---

# Think of `restrict` as a Contract

You tell the compiler:

```
These Pointers

↓

Will Not Alias
```

Compiler replies:

```
Great!

↓

I'll Optimize
```

If you lie,

the behavior is undefined.

---

# Real-World Example — Image Processing

Function:

```
Input Image

↓

Output Image
```

Pointers:

```
Input

↓

restrict

Output

↓

restrict
```

Compiler optimizes pixel processing heavily.

---

# Real-World Example — Matrix Multiplication

Scientific libraries like BLAS often rely on aliasing information.

Matrix multiplication:

```
A

↓

restrict

B

↓

restrict

C

↓

restrict
```

Allows aggressive optimization.

---

# Real-World Example — Video Encoding

Frame buffers:

```
Source

↓

restrict

Destination

↓

restrict
```

Enables faster processing of millions of pixels.

---

# Real-World Example — Databases

Large in-memory scans:

```
Input Buffer

↓

restrict

Output Buffer

↓

restrict
```

Reduce unnecessary memory dependency checks.

---

# Common Mistakes

---

## Thinking `restrict` Makes Code Faster Automatically

It only enables optimizations **when the compiler can safely use them**.

Performance improvements depend on:

- Compiler
- Optimization level
- Code structure
- Target CPU

---

## Using `restrict` When Aliasing Exists

Wrong:

```c
foo(&x, &x);
```

Undefined behavior.

---

## Confusing `restrict` with `const`

`const`

```
Cannot Modify
```

`restrict`

```
No Aliasing Promise
```

They solve different problems.

---

## Using `restrict` Everywhere

Only use it when you can guarantee the pointers refer to independent objects.

---

# Complete Example

```c
#include <stdio.h>

void add(
int *restrict a,
int *restrict b,
int *restrict c)
{
    *a = *b + *c;
}

int main()
{
    int x = 0;
    int y = 10;
    int z = 20;

    add(&x, &y, &z);

    printf("%d\n", x);

    return 0;
}
```

Output:

```
30
```

---

# Hands-on Labs

## Lab 1

Write a function that copies one array into another.

Compile with optimization both with and without `restrict`.

Inspect the generated assembly.

---

## Lab 2

Implement vector addition using:

```c
restrict
```

Compare performance on a large array.

---

## Lab 3

Pass overlapping pointers to a `restrict` function.

Understand why this violates the contract.

---

## Lab 4

Read compiler optimization reports (`-O2`, `-O3`) and observe differences.

---

## Lab 5

Study how high-performance numerical libraries document pointer aliasing assumptions.

---

# Interview Questions

### What is `restrict`?

A C99 keyword that promises restricted pointers are the only means used to access their referenced objects during their lifetime in a given scope.

---

### Why does `restrict` improve performance?

It allows the compiler to assume there is no pointer aliasing, enabling more aggressive optimizations.

---

### Does `restrict` prevent modification?

No.

It only provides aliasing information.

---

### What happens if the `restrict` promise is violated?

The program has undefined behavior.

---

### Where is `restrict` commonly used?

- Scientific computing
- Image processing
- Numerical libraries
- High-performance computing
- Signal processing

---

# Summary

```
Normal Pointer

↓

May Alias

↓

Compiler Conservative

----------------------------

restrict Pointer

↓

No Aliasing Promise

↓

Compiler Optimizes
```

Key Idea:

```
restrict

↓

Performance

NOT

↓

Safety
```

## Key Takeaways

- `restrict` was introduced in C99 to improve optimization opportunities.
- It tells the compiler that restricted pointers access independent objects.
- Eliminating aliasing assumptions enables better instruction scheduling and vectorization.
- Violating the `restrict` contract results in undefined behavior.
- `restrict` is widely used in high-performance numerical and systems software.
- Use `restrict` only when you can guarantee that the pointers do not alias.

---

# Next Chapter

## Chapter 102 — Inline Functions (`inline`)

You'll learn:

- Why function calls have overhead
- What `inline` really means
- Compiler inlining decisions
- Macros vs inline functions
- Performance trade-offs
- When inlining helps
- When inlining hurts performance