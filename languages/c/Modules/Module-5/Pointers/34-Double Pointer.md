# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 34 --- Double Pointer (Advanced)

------------------------------------------------------------------------

## Most developers think:

> "A double pointer is just a pointer to a pointer."

Technically that's true.

But in real systems, `T **` is used to solve problems that a normal
pointer cannot solve.

You'll see double pointers in:

-   `main(int argc, char **argv)`
-   `char **environ`
-   Dynamic 2D arrays
-   Linked lists
-   Trees
-   POSIX APIs
-   Memory allocation libraries

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why advanced code uses `T **`
-   Returning allocated memory
-   Dynamic arrays of pointers
-   Dynamic 2D arrays
-   Linked list insertion
-   Environment variables
-   Common mistakes

------------------------------------------------------------------------

# Review

``` c
int value = 10;
int *p = &value;
int **pp = &p;
```

``` text
pp
 │
 ▼
 p
 │
 ▼
value
```

------------------------------------------------------------------------

# Why Isn't `T *` Enough?

Suppose a library allocates memory.

``` c
void createBuffer(char *buf)
{
    buf = malloc(100);
}
```

The caller's pointer never changes because `buf` is only a copy.

------------------------------------------------------------------------

# Correct Solution

``` c
void createBuffer(char **buf)
{
    *buf = malloc(100);
}
```

Caller:

``` c
char *buffer = NULL;

createBuffer(&buffer);
```

Now:

``` text
buffer

↓

Allocated Memory
```

The function changed the caller's pointer.

------------------------------------------------------------------------

# Dynamic Array of Strings

``` c
char *names[3];
```

Each element is a pointer.

Memory:

``` text
names

 ├──► "Linux"
 ├──► "Kernel"
 └──► "Docker"
```

Sometimes this array itself is allocated dynamically.

``` c
char **names;
```

------------------------------------------------------------------------

# Dynamic 2D Array

Allocate rows:

``` c
int **matrix;
```

Step 1:

Allocate row pointers.

``` text
matrix

↓

Row0
Row1
Row2
```

Step 2:

Allocate each row separately.

``` text
matrix
 │
 ├──► 1 2 3
 ├──► 4 5 6
 └──► 7 8 9
```

This creates a dynamically sized table.

------------------------------------------------------------------------

# Linked List Example

Suppose the head may change.

Wrong:

``` c
void insert(Node *head)
```

The caller's head cannot be updated.

Correct:

``` c
void insert(Node **head)
```

Now the function can modify the head pointer itself.

------------------------------------------------------------------------

# Environment Variables

Many Unix systems expose environment variables as:

``` c
extern char **environ;
```

Conceptually:

``` text
environ

 ├──► PATH=...
 ├──► HOME=...
 └──► USER=...
```

Each entry is a pointer to a C string.

------------------------------------------------------------------------

# Pointer Levels

``` text
int value

↓

int *

↓

int **

↓

int ***

↓

...
```

Each `*` means:

> "Follow one more address."

------------------------------------------------------------------------

# Common Mistakes

## Forgetting `&`

Wrong:

``` c
createBuffer(buffer);
```

Correct:

``` c
createBuffer(&buffer);
```

------------------------------------------------------------------------

## Memory Leak

When allocating a dynamic 2D array:

Allocate:

``` text
Rows

↓

Columns
```

Free in reverse order:

``` text
Columns

↓

Rows
```

Otherwise memory leaks occur.

------------------------------------------------------------------------

# Real-World Uses

Double pointers appear in:

-   POSIX APIs
-   `argv`
-   `environ`
-   Dynamic matrices
-   Linked lists
-   Trees
-   Database engines
-   Memory allocators

------------------------------------------------------------------------

# Hands-on Labs

1.  Allocate memory inside a function using `char **`.
2.  Build a dynamic 3×3 matrix.
3.  Free every allocated row correctly.
4.  Implement linked-list insertion using `Node **head`.
5.  Print every command-line argument using `argv`.

------------------------------------------------------------------------

# Interview Questions

### Why use a double pointer?

To modify a pointer owned by the caller or manage collections of
pointers.

### Where are double pointers commonly used?

Dynamic memory allocation, linked lists, `argv`, environment variables,
and POSIX APIs.

### Why is `char **argv` a double pointer?

Because it points to an array of pointers, where each pointer references
a string.

### Why free rows before freeing the pointer array?

Because each row is a separate allocation.

------------------------------------------------------------------------

# Summary

``` text
Caller Pointer
      │
      ▼
Function (T **)
      │
      ▼
Modify Pointer
      │
      ▼
Caller Updated
```

Key Takeaways:

-   `T **` lets functions modify caller-owned pointers.
-   Double pointers are essential for dynamic memory allocation.
-   They represent arrays of pointers and dynamic 2D arrays.
-   Many Unix/Linux APIs expose data using double pointers.
-   Understanding double pointers is crucial for advanced C and systems
    programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 35 --- `const` Pointer vs Pointer to `const`

You'll learn the different meanings of:

-   `const int *p`
-   `int *const p`
-   `const int *const p`

and when each should be used.
