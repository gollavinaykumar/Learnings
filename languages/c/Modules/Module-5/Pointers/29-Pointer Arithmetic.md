# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 29 --- Pointer Arithmetic

------------------------------------------------------------------------

## Most beginners think:

> "If I add `1` to a pointer, it moves one byte."

That is **not** how pointer arithmetic works.

When you add `1` to a pointer, the compiler moves it by the size of the
data type it points to.

This allows pointers to move correctly through arrays without you
calculating byte offsets manually.

Pointer arithmetic is one of the features that makes C powerful---and
dangerous if misunderstood.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What pointer arithmetic is
-   Why it exists
-   How `+` and `-` work with pointers
-   Pointer increment and decrement
-   Arrays and pointer arithmetic
-   Pointer subtraction
-   Why scaling happens automatically
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Pointer Arithmetic?

Suppose we have:

``` c
int numbers[4] = {10, 20, 30, 40};
```

Memory:

``` text
Address      Value

0x1000 ----> 10
0x1004 ----> 20
0x1008 ----> 30
0x100C ----> 40
```

If a pointer points to the first element:

``` c
int *p = numbers;
```

How can it move to the next integer?

Pointer arithmetic solves this.

------------------------------------------------------------------------

# Adding 1 to a Pointer

``` c
int *p = numbers;

p = p + 1;
```

Result:

``` text
Before

p ─────► 0x1000

After

p ─────► 0x1004
```

Notice:

It moved **4 bytes**, not 1 byte.

Because `sizeof(int)` is usually 4 bytes.

------------------------------------------------------------------------

# Different Data Types

``` c
char *cp;
int *ip;
double *dp;
```

Typical movement on a 64-bit machine:

  Pointer Type   `p + 1` Moves
  -------------- ---------------
  `char *`       1 byte
  `int *`        4 bytes
  `double *`     8 bytes

The compiler automatically multiplies by `sizeof(type)`.

------------------------------------------------------------------------

# Pointer Increment

``` c
p++;
```

is equivalent to:

``` c
p = p + 1;
```

Likewise:

``` c
p--;
```

moves to the previous element.

------------------------------------------------------------------------

# Arrays and Pointers

``` c
int arr[] = {10,20,30};
int *p = arr;
```

These are equivalent:

``` c
arr[2]
```

and

``` c
*(p + 2)
```

How?

The compiler translates:

``` text
Base Address
      +
(Index × sizeof(int))
```

then dereferences the result.

------------------------------------------------------------------------

# Visualizing Pointer Arithmetic

``` text
Array

Index

0      1      2      3

Value

10     20     30     40

Address

1000   1004   1008   1012

Pointer

p
│
▼

1000

p+1

▼

1004

p+2

▼

1008
```

------------------------------------------------------------------------

# Pointer Subtraction

Pointers to elements of the **same array** can be subtracted.

``` c
int *a = &numbers[0];
int *b = &numbers[3];

printf("%td\n", b - a);
```

Output:

``` text
3
```

The result is the number of elements between them, **not** the number of
bytes.

------------------------------------------------------------------------

# Pointer Comparisons

Pointers into the same array can be compared.

``` c
if (p < end)
{
    /* continue */
}
```

This is common in high-performance libraries.

------------------------------------------------------------------------

# Common Mistakes

## Assuming Byte Arithmetic

Wrong expectation:

``` text
int *

1000

+

1

=

1001
```

Actual:

``` text
1004
```

------------------------------------------------------------------------

## Going Outside the Array

``` c
p = p + 100;
```

If `p` no longer points within the same array (or one past the end),
dereferencing it is undefined behavior.

------------------------------------------------------------------------

## Mixing Different Arrays

Subtracting pointers from unrelated arrays is undefined behavior.

------------------------------------------------------------------------

# Real-World Uses

Pointer arithmetic is used in:

-   Array traversal
-   Memory allocators
-   Network packet parsing
-   Image processing
-   File systems
-   Linux kernel
-   Database engines

------------------------------------------------------------------------

# Hands-on Labs

1.  Print the addresses of every element in an integer array.
2.  Traverse an array using only a pointer.
3.  Rewrite array indexing using `*(p + i)`.
4.  Subtract two pointers and observe the result.
5.  Compare pointer movement for `char`, `int`, and `double`.

------------------------------------------------------------------------

# Interview Questions

### What is pointer arithmetic?

Arithmetic performed on pointers where movement is scaled by the size of
the pointed-to type.

### Why doesn't `p + 1` always move one byte?

Because the compiler multiplies the offset by `sizeof(*p)`.

### What does `*(p + i)` mean?

Access the element `i` positions after the address stored in `p`.

### What does subtracting two pointers return?

The number of elements between them (when both point into the same
array).

------------------------------------------------------------------------

# Summary

``` text
Pointer
   │
   ▼
Base Address
   │
+ (Index × sizeof(Type))
   │
   ▼
Target Address
   │
Dereference (*)
   │
   ▼
Array Element
```

Key Takeaways:

-   Pointer arithmetic works in units of elements, not bytes.
-   The compiler automatically scales pointer movement.
-   Array indexing is built on pointer arithmetic.
-   Pointer subtraction returns an element count.
-   Staying within array bounds is essential.

------------------------------------------------------------------------

# Next Chapter

## Chapter 30 --- Arrays vs Pointers

You'll learn why arrays and pointers are related but **not the same
thing**, one of the most misunderstood topics in C.
