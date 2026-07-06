# 👑 C Mastery Roadmap

# Module 4 --- Arrays & Strings

# Chapter 21 --- Arrays

------------------------------------------------------------------------

## Most beginners think:

> "An array is just a list of values."

But an array is much more than that.

An array is a **contiguous block of memory**.

That single property makes arrays one of the fastest data structures in
computer science.

Operating systems, databases, network buffers, image processing, and the
Linux kernel all rely heavily on arrays.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What an array is
-   Why arrays exist
-   Array declaration
-   Initialization
-   Indexing
-   Memory layout
-   Traversing arrays
-   Array bounds
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Arrays?

Without arrays:

``` c
int s1 = 80;
int s2 = 75;
int s3 = 92;
int s4 = 88;
```

With arrays:

``` c
int scores[4] = {80, 75, 92, 88};
```

One variable represents many values.

------------------------------------------------------------------------

# What is an Array?

An array stores multiple values of the **same data type** in consecutive
memory locations.

``` text
scores

Index:   0    1    2    3

Value:  80   75   92   88
```

------------------------------------------------------------------------

# Declaring an Array

``` c
int numbers[5];
```

This reserves space for five integers.

------------------------------------------------------------------------

# Initializing an Array

``` c
int numbers[5] = {10,20,30,40,50};
```

The compiler stores each value in consecutive memory.

------------------------------------------------------------------------

# Memory Layout

Assume an `int` occupies 4 bytes.

``` text
Address      Value

0x1000  ---> 10
0x1004  ---> 20
0x1008  ---> 30
0x100C  ---> 40
0x1010  ---> 50
```

Elements are adjacent in memory.

------------------------------------------------------------------------

# Accessing Elements

``` c
printf("%d\n", numbers[0]);
printf("%d\n", numbers[3]);
```

Indexing starts at **0**.

The first element is always index `0`.

------------------------------------------------------------------------

# Updating Elements

``` c
numbers[2] = 99;
```

Only the third element changes.

------------------------------------------------------------------------

# Traversing an Array

``` c
for (int i = 0; i < 5; i++)
{
    printf("%d\n", numbers[i]);
}
```

Loops and arrays are commonly used together.

------------------------------------------------------------------------

# Array Size

``` c
int numbers[5];
```

Valid indexes:

``` text
0
1
2
3
4
```

Invalid:

``` text
5
6
-1
```

Accessing outside the array results in **undefined behavior**.

------------------------------------------------------------------------

# Common Mistakes

## Off-by-One Error

Wrong:

``` c
for (int i = 0; i <= 5; i++)
```

Correct:

``` c
for (int i = 0; i < 5; i++)
```

------------------------------------------------------------------------

## Array Out of Bounds

``` c
numbers[10] = 5;
```

The compiler usually cannot prevent this.

It may overwrite unrelated memory.

------------------------------------------------------------------------

# Real-World Uses

Arrays are used for:

-   Buffers
-   Packet storage
-   Images
-   Audio samples
-   Matrices
-   Lookup tables
-   CPU scheduling queues

------------------------------------------------------------------------

# Hands-on Labs

1.  Create an array of five integers.
2.  Print all elements using a loop.
3.  Find the largest value.
4.  Find the smallest value.
5.  Calculate the average.

------------------------------------------------------------------------

# Interview Questions

### What is an array?

A collection of elements of the same type stored in contiguous memory.

### Why do array indexes start at 0?

Because the index represents the offset from the first element.

### Can an array store different data types?

No. All elements must have the same type.

### What happens if you access an invalid index?

The behavior is undefined and may corrupt memory or crash the program.

------------------------------------------------------------------------

# Summary

``` text
Array
 │
 ▼
Contiguous Memory
 │
 ▼
Index
 │
 ▼
Element
```

Key Takeaways:

-   Arrays store multiple values of the same type.
-   Elements are stored contiguously.
-   Indexing begins at 0.
-   Arrays provide very fast random access.
-   Accessing outside array bounds is dangerous.

------------------------------------------------------------------------

# Next Chapter

## Chapter 22 --- Multidimensional Arrays

You'll learn how C stores tables, matrices, game boards, and images
using arrays of arrays.
