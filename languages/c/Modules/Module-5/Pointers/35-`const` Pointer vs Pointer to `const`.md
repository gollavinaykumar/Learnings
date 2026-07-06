# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 35 --- `const` Pointer vs Pointer to `const`

------------------------------------------------------------------------

## Most beginners think:

> "Adding `const` to a pointer means the same thing everywhere."

It doesn't.

The position of `const` completely changes its meaning.

These declarations are **different**:

``` c
const int *p;
int *const p;
const int *const p;
```

Understanding them is essential for writing safe APIs and reading Linux
kernel code.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What `const` means
-   Pointer to const
-   Const pointer
-   Const pointer to const
-   Reading declarations
-   Why APIs use `const`
-   Common mistakes

------------------------------------------------------------------------

# Rule to Remember

Read declarations from the variable name outward.

Example:

``` c
const int *p;
```

Start at `p`.

`p` is a pointer to a constant integer.

------------------------------------------------------------------------

# Pointer to `const`

``` c
const int *p;
```

or

``` c
int const *p;
```

Both are identical.

``` text
Pointer
   │
   ▼
Constant Data
```

You **can** change where `p` points.

You **cannot** modify the value through `p`.

Example:

``` c
int a = 10;
int b = 20;

const int *p = &a;

p = &b;      // ✔ Allowed
//*p = 50;   // ❌ Error
```

------------------------------------------------------------------------

# Const Pointer

``` c
int *const p = &a;
```

``` text
Constant Pointer
       │
       ▼
Mutable Data
```

The pointer itself cannot change.

The data can.

``` c
*p = 50;    // ✔ Allowed
//p = &b;   // ❌ Error
```

------------------------------------------------------------------------

# Const Pointer to Const Data

``` c
const int *const p = &a;
```

``` text
Constant Pointer
       │
       ▼
Constant Data
```

Neither the pointer nor the data can be modified through `p`.

``` c
//*p = 5;   // ❌
//p = &b;   // ❌
```

------------------------------------------------------------------------

# Visual Comparison

  Declaration             Change Data   Change Pointer
  ---------------------- ------------- ----------------
  `const int *p`              ❌              ✔
  `int *const p`               ✔              ❌
  `const int *const p`        ❌              ❌

------------------------------------------------------------------------

# Why Use `const`?

Suppose you write:

``` c
void print(const char *text);
```

The function promises not to modify the caller's string.

This improves:

-   Safety
-   Readability
-   Compiler optimizations

------------------------------------------------------------------------

# Real-World Examples

## `strlen()`

Conceptually:

``` c
size_t strlen(const char *s);
```

It reads the string.

It never modifies it.

------------------------------------------------------------------------

## Linux Kernel

Kernel APIs often use `const` for buffers that must not be changed.

------------------------------------------------------------------------

## Library Design

Use `const` whenever a function only reads data.

It communicates intent clearly.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting What Is Constant

``` c
const int *p;
```

The **integer** is constant.

Not the pointer.

------------------------------------------------------------------------

## Casting Away `const`

``` c
const int x = 5;

int *p = (int *)&x;
```

Modifying through `p` causes undefined behavior.

Avoid removing `const` unless absolutely necessary.

------------------------------------------------------------------------

# Hands-on Labs

1.  Create one example of each declaration.
2.  Try modifying the pointer and the data.
3.  Observe compiler errors.
4.  Add `const` to function parameters and see how it prevents
    accidental writes.

------------------------------------------------------------------------

# Interview Questions

### What is the difference between `const int *p` and `int *const p`?

`const int *p`: - Data is read-only. - Pointer can change.

`int *const p`: - Pointer cannot change. - Data can change.

### Why use `const` in function parameters?

To prevent accidental modification and make APIs safer.

### Are `const int *` and `int const *` different?

No. They are exactly the same.

------------------------------------------------------------------------

# Summary

``` text
const int *p

Pointer
   │
   ▼
Read-Only Data

-------------------------

int *const p

Fixed Pointer
      │
      ▼
Writable Data

-------------------------

const int *const p

Fixed Pointer
      │
      ▼
Read-Only Data
```

Key Takeaways:

-   The position of `const` matters.
-   `const int *` protects the data.
-   `int *const` protects the pointer.
-   `const int *const` protects both.
-   `const` is widely used in professional C APIs.

------------------------------------------------------------------------

# Next Chapter

## Chapter 36 --- Dangling Pointer

You'll learn what dangling pointers are, how they are created, why they
cause crashes and security bugs, and how to avoid them.
