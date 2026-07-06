# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 32 --- Void Pointer (`void *`)

------------------------------------------------------------------------

## Most beginners think:

> "Every pointer must know the type it points to."

Usually that's true.

``` c
int *ip;
char *cp;
double *dp;
```

Each pointer knows the type of data it references.

But sometimes a library doesn't know the data type in advance.

For example:

-   `malloc()` can allocate memory for **any** type.
-   `qsort()` can sort **any** type.
-   Generic containers can store **any** type.

How is that possible?

The answer is the **void pointer**.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a `void *` is
-   Why it exists
-   Generic programming
-   Type casting
-   `malloc()` and `void *`
-   `qsort()` callbacks
-   Limitations of `void *`
-   Common mistakes

------------------------------------------------------------------------

# What is a Void Pointer?

A `void *` is a pointer that can hold the address of **any data type**.

``` c
void *ptr;
```

Think of it as:

``` text
Generic Address Holder
```

It stores an address but **doesn't know what type lives there**.

------------------------------------------------------------------------

# Why Do We Need `void *`?

Suppose a memory allocator only accepted integers.

``` c
int *allocateInt();
```

What about:

-   `double`
-   `char`
-   `struct`

We would need a different function for every type.

Instead:

``` c
void *malloc(size_t size);
```

One function works for every type.

------------------------------------------------------------------------

# Example

``` c
int value = 42;

void *ptr = &value;
```

Memory:

``` text
ptr
 │
 ▼
Address of value
```

`ptr` knows the address.

It does **not** know it's an integer.

------------------------------------------------------------------------

# Why Can't We Dereference It?

Wrong:

``` c
printf("%d\n", *ptr);
```

Compilation error.

Why?

Because the compiler doesn't know:

-   Is it an `int`?
-   A `char`?
-   A `double`?

It doesn't know how many bytes to read.

------------------------------------------------------------------------

# Type Casting

Convert the pointer to the correct type first.

``` c
printf("%d\n", *(int *)ptr);
```

Steps:

``` text
void *

↓

(int *)

↓

Dereference

↓

Value
```

------------------------------------------------------------------------

# `malloc()` Returns `void *`

``` c
int *numbers = malloc(5 * sizeof(int));
```

Conceptually:

``` text
malloc()

↓

void *

↓

int *
```

In C, the conversion is implicit.

------------------------------------------------------------------------

# Generic Function Example

``` c
void printInt(void *data)
{
    printf("%d\n", *(int *)data);
}
```

The function accepts any pointer.

Inside, we cast it to the expected type.

------------------------------------------------------------------------

# `qsort()` Example

The standard library function:

``` c
qsort()
```

Sorts any data type.

Comparison callback:

``` c
int compare(const void *a,
            const void *b)
```

The callback casts the pointers to the correct type before comparing.

This is one of the most common real-world uses of `void *`.

------------------------------------------------------------------------

# Limitations

A `void *`:

✅ Can store any address.

❌ Cannot be dereferenced directly.

❌ Cannot perform pointer arithmetic directly in standard C.

Convert it to a typed pointer first.

------------------------------------------------------------------------

# Common Mistakes

## Dereferencing Directly

Wrong:

``` c
*ptr;
```

Correct:

``` c
*(int *)ptr;
```

------------------------------------------------------------------------

## Wrong Type Cast

``` c
double value = 3.14;

void *ptr = &value;

printf("%d\n", *(int *)ptr);
```

This interprets the memory incorrectly and produces meaningless results.

Always cast to the correct type.

------------------------------------------------------------------------

# Real-World Uses

`void *` is used in:

-   `malloc()`
-   `calloc()`
-   `realloc()`
-   `qsort()`
-   POSIX threads (`pthread_create`)
-   Generic linked lists
-   Plugin systems
-   Linux kernel helper APIs

------------------------------------------------------------------------

# Hands-on Labs

1.  Store an `int` in a `void *` and print it.
2.  Repeat for a `double`.
3.  Allocate memory with `malloc()` for an integer array.
4.  Write a generic print function using `void *`.
5.  Read the documentation for `qsort()` and identify where `void *` is
    used.

------------------------------------------------------------------------

# Interview Questions

### What is a `void *`?

A generic pointer that can hold the address of any data type.

### Can you dereference a `void *` directly?

No.

It must first be cast to the correct pointer type.

### Why does `malloc()` return `void *`?

So it can allocate memory for any type.

### Where are `void *` pointers commonly used?

Generic libraries, memory allocators, sorting functions, POSIX APIs, and
plugin systems.

------------------------------------------------------------------------

# Summary

``` text
Object
   │
   ▼
Address
   │
   ▼
void *
   │
Type Cast
   │
   ▼
Typed Pointer
   │
Dereference
   │
   ▼
Value
```

Key Takeaways:

-   `void *` is a generic pointer.
-   It stores addresses, not type information.
-   Cast before dereferencing.
-   Generic C libraries rely heavily on `void *`.
-   Understanding `void *` is essential for memory allocation and
    systems programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 33 --- Function Pointers (Advanced)

You'll learn advanced function pointer patterns including dispatch
tables, virtual functions in C, plugin architectures, and Linux kernel
operations tables.
