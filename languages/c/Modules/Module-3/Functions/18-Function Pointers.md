# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 18 --- Function Pointers

------------------------------------------------------------------------

## Most developers think:

> "Pointers can only point to variables."

But in C, pointers can also point to **functions**.

This allows programs to decide **which function to execute at runtime**.

Function pointers are used throughout:

-   Linux Kernel
-   POSIX APIs
-   Device Drivers
-   GUI Frameworks
-   Event Systems
-   Network Servers
-   Plugin Architectures

Without function pointers, callbacks would not exist.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a function pointer is
-   Why function pointers exist
-   Function pointer syntax
-   Calling functions through pointers
-   Arrays of function pointers
-   Practical uses
-   Common mistakes

------------------------------------------------------------------------

# What is a Function Pointer?

A function pointer stores the **address of a function**.

Just like:

``` c
int *p;
```

stores the address of an integer,

a function pointer stores the address of executable code.

------------------------------------------------------------------------

# Why Do We Need Function Pointers?

Normally:

``` text
Caller
  │
  ▼
Known Function
```

With function pointers:

``` text
Caller
  │
  ▼
Function Pointer
  │
  ▼
Chosen Function
```

The function can be selected while the program is running.

------------------------------------------------------------------------

# Declaring a Function

``` c
int add(int a, int b)
{
    return a + b;
}
```

------------------------------------------------------------------------

# Declaring a Function Pointer

``` c
int (*operation)(int, int);
```

Meaning:

-   returns `int`
-   points to a function
-   accepts two `int` parameters

------------------------------------------------------------------------

# Assigning a Function

``` c
operation = add;
```

or

``` c
operation = &add;
```

Both are valid.

------------------------------------------------------------------------

# Calling Through the Pointer

``` c
int result = operation(5, 3);
```

or

``` c
int result = (*operation)(5, 3);
```

Both call `add()`.

------------------------------------------------------------------------

# Memory View

``` text
Code Segment

add()
Address: 0x401200

      ▲
      │

operation

0x401200
```

The pointer stores the function's address.

------------------------------------------------------------------------

# Example

``` c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    int (*fp)(int,int) = add;

    printf("%d\n", fp(10,20));
}
```

Output:

``` text
30
```

------------------------------------------------------------------------

# Array of Function Pointers

``` c
int add(int,int);
int sub(int,int);

int (*ops[])(int,int) =
{
    add,
    sub
};
```

Select one:

``` c
ops[0](5,2);
ops[1](5,2);
```

Useful for menus and command dispatchers.

------------------------------------------------------------------------

# Real-World Uses

Function pointers power:

-   Callback APIs
-   Interrupt handlers
-   Event loops
-   Virtual tables (object-oriented behavior in C)
-   Linux kernel operations tables
-   POSIX signal handlers

------------------------------------------------------------------------

# Common Mistakes

## Wrong Signature

The function pointer type must exactly match the function.

Wrong:

``` c
void (*fp)(void);

fp = add;
```

Compiler error or warning.

------------------------------------------------------------------------

## Forgetting Parentheses

Wrong:

``` c
int *fp(int,int);
```

This declares a function, not a function pointer.

Correct:

``` c
int (*fp)(int,int);
```

------------------------------------------------------------------------

# Hands-on Labs

1.  Create `add`, `sub`, `mul`, and `div` functions.
2.  Store them in function pointers.
3.  Build a simple calculator using an array of function pointers.
4.  Print function addresses using `%p`.

------------------------------------------------------------------------

# Interview Questions

### What is a function pointer?

A pointer that stores the address of a function.

### Why are function pointers useful?

They enable runtime selection of behavior, callbacks, and modular
designs.

### Can functions be called through pointers?

Yes. Calling `fp()` executes the function whose address is stored in
`fp`.

### Where are function pointers used?

Operating systems, event systems, device drivers, GUI frameworks,
networking libraries, and plugin systems.

------------------------------------------------------------------------

# Summary

``` text
Function
    │
    ▼
Address
    │
    ▼
Function Pointer
    │
    ▼
Function Call
```

Key Takeaways:

-   Function pointers store function addresses.
-   They allow runtime selection of functions.
-   Their declaration syntax is unique but follows the same pointer
    principles.
-   Function pointers are the foundation for callbacks and many systems
    programming APIs.

------------------------------------------------------------------------

# Next Chapter

## Chapter 19 --- Callback Functions

You'll learn how programs pass function pointers into other functions to
customize behavior, enabling event-driven programming and reusable
libraries.
