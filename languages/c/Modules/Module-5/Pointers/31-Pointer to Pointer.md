# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 31 --- Pointer to Pointer

------------------------------------------------------------------------

## Most beginners think:

> "A pointer can only point to a normal variable."

Not true.

A pointer can point to **another pointer**.

That means one pointer stores the address of another pointer, which in
turn stores the address of the actual data.

This is called a **pointer to a pointer** or **double pointer**.

Double pointers are heavily used in:

-   Dynamic memory allocation
-   Command-line arguments (`argv`)
-   Linked data structures
-   Linux/POSIX APIs
-   Library functions that need to modify pointers

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a pointer to pointer is
-   Why double pointers exist
-   How to declare them
-   How dereferencing works
-   Memory layout
-   Modifying pointers through functions
-   Real-world uses
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need a Pointer to a Pointer?

Suppose:

``` c
int value = 100;
int *p = &value;
```

Now imagine another variable needs to know **where `p` is stored**.

That variable must store the address of `p`.

------------------------------------------------------------------------

# Declaring a Pointer to Pointer

``` c
int value = 100;

int *p = &value;

int **pp = &p;
```

Read it as:

-   `p` → pointer to `int`
-   `pp` → pointer to pointer to `int`

------------------------------------------------------------------------

# Memory Layout

``` text
value

Address: 0x1000
Value  : 100

        ▲
        │

p

Address: 0x2000
Value  : 0x1000

        ▲
        │

pp

Address: 0x3000
Value  : 0x2000
```

Each variable has its own address.

------------------------------------------------------------------------

# Dereferencing

``` c
printf("%d\n", value);
```

Output:

``` text
100
```

Using one pointer:

``` c
printf("%d\n", *p);
```

Output:

``` text
100
```

Using a double pointer:

``` c
printf("%d\n", **pp);
```

Output:

``` text
100
```

Explanation:

``` text
pp
 │
 ▼
p
 │
 ▼
value
```

`**pp` follows both pointers to reach the original value.

------------------------------------------------------------------------

# Modifying the Original Value

``` c
**pp = 500;
```

Now:

``` text
value = 500
```

The original integer changes.

------------------------------------------------------------------------

# Why Not Just Use a Pointer?

Sometimes a function must change **where a pointer points**, not just
the data it points to.

Example:

``` c
void allocate(int **ptr)
{
    /* assign a new address to *ptr */
}
```

If only `int *ptr` were passed, the function would receive a copy of the
pointer and couldn't change the caller's pointer itself.

------------------------------------------------------------------------

# Real Example --- Swapping Pointers

``` c
void setNull(int **p)
{
    *p = NULL;
}
```

Caller:

``` c
int value = 10;
int *ptr = &value;

setNull(&ptr);
```

After the call:

``` text
ptr = NULL
```

The function modified the caller's pointer.

------------------------------------------------------------------------

# Command-Line Arguments

Every C program can start like this:

``` c
int main(int argc, char *argv[])
```

This is equivalent to:

``` c
int main(int argc, char **argv)
```

`argv` is a pointer to pointers.

Each element points to a command-line string.

``` text
argv
 │
 ├──► "program"
 ├──► "file.txt"
 └──► "--help"
```

------------------------------------------------------------------------

# Dynamic Memory Allocation

Some APIs allocate memory and return it through a double pointer.

``` c
char *buffer = NULL;

/* library allocates memory */

allocateBuffer(&buffer);
```

The function changes the caller's pointer to point to newly allocated
memory.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting One `*`

``` c
printf("%d\n", *pp);
```

This prints the value stored in `p` (an address), not the integer.

To access the integer:

``` c
printf("%d\n", **pp);
```

------------------------------------------------------------------------

## Passing the Wrong Argument

Wrong:

``` c
setNull(ptr);
```

Correct:

``` c
setNull(&ptr);
```

Because the function expects the address of the pointer.

------------------------------------------------------------------------

# Real-World Uses

Double pointers appear in:

-   `main(int argc, char **argv)`
-   Memory allocators
-   Linked list insertion
-   Trees
-   POSIX APIs
-   Plugin systems
-   Database engines

------------------------------------------------------------------------

# Hands-on Labs

1.  Create an integer, pointer, and double pointer.
2.  Print the value using `value`, `*p`, and `**pp`.
3.  Modify the integer using `**pp`.
4.  Write a function that sets a pointer to `NULL` using a double
    pointer.
5.  Print all command-line arguments using `argv`.

------------------------------------------------------------------------

# Interview Questions

### What is a pointer to a pointer?

A pointer that stores the address of another pointer.

### How do you declare a double pointer?

``` c
int **pp;
```

### What does `**pp` mean?

Dereference twice to access the original object.

### Why are double pointers useful?

They allow functions to modify the caller's pointer and are widely used
in allocation and complex data structures.

------------------------------------------------------------------------

# Summary

``` text
value
  ▲
  │
 *p
  ▲
  │
**pp
```

Key Takeaways:

-   A pointer can point to another pointer.
-   `**pp` accesses the original value.
-   Double pointers let functions modify pointers owned by the caller.
-   They are common in POSIX APIs, memory allocation, and command-line
    processing.

------------------------------------------------------------------------

# Next Chapter

## Chapter 32 --- Void Pointer

You'll learn how `void *` can point to any data type, why generic
libraries use it, and how C implements type-independent programming.
