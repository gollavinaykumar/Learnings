# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 30 --- Arrays vs Pointers

------------------------------------------------------------------------

## Most beginners think:

> "Arrays and pointers are exactly the same."

This is one of the biggest misconceptions in C.

Arrays and pointers are **closely related**, but they are **not the same
thing**.

An array is a block of memory.

A pointer is a variable that stores a memory address.

The confusion comes from the fact that arrays often **decay into
pointers**.

Understanding this difference is essential for writing correct C
programs.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Arrays vs pointers
-   Why people confuse them
-   Array decay
-   Memory layout
-   `sizeof()` differences
-   Can arrays be reassigned?
-   Can pointers be reassigned?
-   Function parameters
-   Common mistakes

------------------------------------------------------------------------

# What is an Array?

``` c
int numbers[4] = {10,20,30,40};
```

The compiler allocates one contiguous block of memory.

``` text
Address      Value

0x1000 ----> 10
0x1004 ----> 20
0x1008 ----> 30
0x100C ----> 40
```

The array **owns** this memory.

------------------------------------------------------------------------

# What is a Pointer?

``` c
int *p = numbers;
```

Memory:

``` text
numbers

0x1000
 │
 ▼
10 20 30 40

p

Value = 0x1000
```

`p` owns no array.

It only stores an address.

------------------------------------------------------------------------

# Why Do They Look Similar?

Both expressions work:

``` c
numbers[2]
```

``` c
p[2]
```

Both produce:

``` text
30
```

Because:

``` c
p[i]
```

is defined as:

``` c
*(p + i)
```

------------------------------------------------------------------------

# Array Decay

In most expressions, an array automatically converts to a pointer to its
first element.

``` c
int arr[4];

int *p = arr;
```

The compiler converts:

``` text
arr

↓

&arr[0]
```

This is called **array decay**.

------------------------------------------------------------------------

# Important Exception

The array itself is **not** a pointer.

``` c
sizeof(arr)
```

returns:

``` text
16
```

(assuming four `int`s)

But:

``` c
sizeof(p)
```

returns:

``` text
8
```

(on a typical 64-bit system)

------------------------------------------------------------------------

# Arrays Cannot Be Reassigned

Wrong:

``` c
arr = p;
```

Compilation error.

An array name is not a modifiable variable.

------------------------------------------------------------------------

# Pointers Can Be Reassigned

``` c
int a = 10;
int b = 20;

int *p = &a;

p = &b;
```

Now:

``` text
p

↓

b
```

Pointers can point somewhere else at any time.

------------------------------------------------------------------------

# Memory Comparison

``` text
Array

┌──────────────────────┐
│10│20│30│40│
└──────────────────────┘

Pointer

┌───────────┐
│ 0x1000    │
└───────────┘
```

An array stores data.

A pointer stores an address.

------------------------------------------------------------------------

# Arrays as Function Parameters

``` c
void print(int arr[])
```

is exactly the same as:

``` c
void print(int *arr)
```

Inside a function parameter list, arrays decay into pointers.

That means the function receives only the address of the first element.

------------------------------------------------------------------------

# Common Mistakes

## Thinking Arrays Are Pointers

They are related but fundamentally different.

------------------------------------------------------------------------

## Using `sizeof()` Incorrectly

``` c
int arr[10];

printf("%zu\n", sizeof(arr));
```

Returns the total size of the array.

Inside:

``` c
void f(int arr[])
```

`sizeof(arr)` returns the size of a pointer because `arr` is treated as
`int *`.

------------------------------------------------------------------------

## Trying to Assign an Array

``` c
arr = other;
```

Not allowed.

Copy elements instead.

------------------------------------------------------------------------

# Real-World Uses

Understanding arrays and pointers is critical for:

-   Linux kernel APIs
-   Buffer management
-   File I/O
-   Network packet parsing
-   Dynamic memory
-   High-performance libraries

------------------------------------------------------------------------

# Hands-on Labs

1.  Compare `sizeof(arr)` and `sizeof(pointer)`.
2.  Print `arr`, `&arr[0]`, and `pointer`.
3.  Reassign a pointer to another array.
4.  Attempt to assign one array to another and observe the compiler
    error.
5.  Write a function that receives an array parameter and inspect
    `sizeof()` inside it.

------------------------------------------------------------------------

# Interview Questions

### Are arrays and pointers the same?

No. Arrays own storage. Pointers store addresses.

### What is array decay?

The automatic conversion of an array to a pointer to its first element
in most expressions.

### Can an array be reassigned?

No.

### Can a pointer be reassigned?

Yes.

### Why does `sizeof(arr)` differ from `sizeof(pointer)`?

Because one measures the entire array while the other measures only the
pointer variable.

------------------------------------------------------------------------

# Summary

``` text
Array
 │
 ├── Owns Memory
 ├── Fixed Size
 └── Cannot Be Reassigned

Pointer
 │
 ├── Stores Address
 ├── Fixed Pointer Size
 └── Can Be Reassigned
```

Key Takeaways:

-   Arrays and pointers are closely related but different.
-   Arrays usually decay into pointers.
-   Arrays own memory; pointers only reference memory.
-   `sizeof()` behaves differently for arrays and pointers.
-   Understanding this distinction is essential before advanced pointer
    topics.

------------------------------------------------------------------------

# Next Chapter

## Chapter 31 --- Pointer to Pointer

You'll learn how pointers can point to other pointers, why double
pointers exist, and how they are used in dynamic memory allocation,
command-line arguments, and complex APIs.
