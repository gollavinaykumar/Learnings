# 👑 C Mastery Roadmap

# Module 4 --- Arrays & Strings

# Chapter 26 --- Memory Representation

------------------------------------------------------------------------

## Most beginners think:

> "Variables just exist somewhere in memory."

But the CPU only understands **bytes and addresses**.

When you write:

``` c
int x = 10;
```

the compiler reserves bytes in memory, stores the binary representation
of `10`, and gives your program a way to access it.

Understanding memory representation is the bridge between arrays and
pointers.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   How data is stored in memory
-   Bytes and addresses
-   Memory representation of integers
-   Memory representation of arrays
-   Memory representation of strings
-   Endianness preview
-   `sizeof()`
-   Common mistakes

------------------------------------------------------------------------

# What is Memory?

Memory (RAM) is a huge collection of bytes.

Each byte has a unique address.

``` text
Address      Value

0x1000  --->  ??
0x1001  --->  ??
0x1002  --->  ??
0x1003  --->  ??
```

Every variable occupies one or more of these bytes.

------------------------------------------------------------------------

# What is a Byte?

A byte contains **8 bits**.

``` text
1 Byte

+---+---+---+---+---+---+---+---+
| 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |
+---+---+---+---+---+---+---+---+
```

Every value in your program is ultimately stored as bytes.

------------------------------------------------------------------------

# Integer Memory Representation

``` c
int number = 10;
```

Suppose an `int` occupies 4 bytes.

Memory might look like:

``` text
Address

0x1000
0x1001
0x1002
0x1003
```

Those four bytes together represent the integer.

The exact byte order depends on the CPU architecture.

------------------------------------------------------------------------

# Character Representation

``` c
char c = 'A';
```

ASCII value:

``` text
'A'

↓

65

↓

01000001
```

Only one byte is required.

------------------------------------------------------------------------

# Array Representation

``` c
int numbers[4] = {10,20,30,40};
```

Memory:

``` text
Address      Value

0x1000 ---> 10
0x1004 ---> 20
0x1008 ---> 30
0x100C ---> 40
```

Arrays are stored in **contiguous memory**.

This is why random access is so fast.

------------------------------------------------------------------------

# String Representation

``` c
char name[] = "Linux";
```

Memory:

``` text
Address      Value

0x2000 ---> L
0x2001 ---> i
0x2002 ---> n
0x2003 ---> u
0x2004 ---> x
0x2005 ---> \0
```

Every character occupies one byte.

The null terminator marks the end.

------------------------------------------------------------------------

# Why Addresses Matter

Imagine the CPU wants the third element.

``` c
numbers[2]
```

The compiler calculates:

``` text
Base Address

+

(Index × Element Size)

=

Actual Address
```

Example:

``` text
0x1000

+

2 × 4

=

0x1008
```

No searching is required.

This is why arrays provide **O(1)** access.

------------------------------------------------------------------------

# `sizeof()`

The `sizeof` operator returns the size of a type or object in bytes.

``` c
printf("%zu\n", sizeof(int));
printf("%zu\n", sizeof(double));
printf("%zu\n", sizeof(numbers));
```

Example output (varies by platform):

``` text
4
8
16
```

`numbers` contains four integers:

``` text
4 × 4 = 16 bytes
```

------------------------------------------------------------------------

# Endianness (Preview)

Multi-byte values can be stored in different byte orders.

Example:

``` text
0x12345678
```

Some CPUs store:

``` text
12 34 56 78
```

Others store:

``` text
78 56 34 12
```

We'll study endianness in detail in the Bit Manipulation module.

------------------------------------------------------------------------

# Real-World Uses

Understanding memory representation is essential for:

-   Pointer arithmetic
-   Operating systems
-   Network protocols
-   Binary file formats
-   Device drivers
-   Embedded systems
-   Database storage engines

------------------------------------------------------------------------

# Common Mistakes

## Assuming Every Type Has the Same Size

``` text
char   != int != double
```

Use `sizeof()` instead of guessing.

------------------------------------------------------------------------

## Ignoring Alignment

Some CPUs perform better when data is aligned correctly.

We'll revisit this in the Memory Management module.

------------------------------------------------------------------------

# Hands-on Labs

1.  Print the size of all primitive data types using `sizeof()`.
2.  Print the addresses of array elements using `%p`.
3.  Observe that integer array addresses increase by `sizeof(int)`.
4.  Print every character and address in a string.
5.  Compare memory usage of `char[20]` and `int[20]`.

------------------------------------------------------------------------

# Interview Questions

### What is memory?

A collection of addressable bytes used to store program data.

### Why are arrays fast?

Because elements are stored contiguously and their addresses can be
calculated directly.

### What does `sizeof()` return?

The size of a type or object in bytes.

### Why does a string end with `'\0'`?

So library functions know where the text ends.

### Why is memory representation important?

It helps you understand pointers, performance, debugging, file formats,
and low-level programming.

------------------------------------------------------------------------

# Summary

``` text
Variable
     │
     ▼
Bytes
     │
     ▼
Memory Address
     │
     ▼
RAM
     │
     ▼
CPU Access
```

Key Takeaways:

-   Every variable occupies bytes in memory.
-   Each byte has a unique address.
-   Arrays are stored contiguously.
-   Strings are character arrays ending with `'\0'`.
-   `sizeof()` reports object sizes in bytes.
-   Memory representation is the foundation for understanding pointers.

------------------------------------------------------------------------

# 🎉 Module 4 Complete

You have learned:

-   Arrays
-   Multidimensional Arrays
-   Character Arrays
-   Strings
-   String Library Functions
-   Memory Representation

------------------------------------------------------------------------

# Next Module

## Module 5 --- Pointers

You'll learn one of the most important topics in C:

-   Memory addresses
-   Pointer basics
-   Pointer arithmetic
-   Arrays vs pointers
-   Double pointers
-   Void pointers
-   Const pointers
-   Function pointers (advanced usage)
-   Dangling, wild, and NULL pointers

Pointers are the heart of systems programming, Linux kernel development,
networking, and memory management.
