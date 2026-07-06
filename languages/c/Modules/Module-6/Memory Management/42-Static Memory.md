# 👑 C Mastery Roadmap

# Module 6 --- Memory Management

# Chapter 42 --- Static Memory

------------------------------------------------------------------------

## Most beginners think:

> "`static` means the variable can never change."

That's incorrect.

`static` **does not make a variable constant**.

Instead, `static` changes one (or both) of these:

-   **Lifetime** (how long the variable exists)
-   **Visibility** (who can access it)

Understanding `static` is essential because it is heavily used in:

-   Linux kernel
-   Embedded systems
-   Libraries
-   Device drivers
-   High-performance servers

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What `static` means
-   Static local variables
-   Static global variables
-   Lifetime vs scope
-   Static memory region
-   Why `static` exists
-   Real-world uses
-   Common mistakes

------------------------------------------------------------------------

# What Does `static` Mean?

`static` tells the compiler that the object should have **static storage
duration**.

That means it is created **before `main()` starts** and destroyed **when
the program exits**.

Unlike stack variables, it is **not recreated every time a function is
called**.

------------------------------------------------------------------------

# Static Local Variable

``` c
void counter(void)
{
    static int count = 0;

    count++;

    printf("%d\n", count);
}
```

Calling:

``` c
counter();
counter();
counter();
```

Output:

``` text
1
2
3
```

Why?

Because `count` is created only once.

------------------------------------------------------------------------

# Normal Local Variable

``` c
void counter(void)
{
    int count = 0;

    count++;

    printf("%d\n", count);
}
```

Output:

``` text
1
1
1
```

Each function call gets a brand-new stack variable.

------------------------------------------------------------------------

# Memory Comparison

Normal local variable:

``` text
Function Call

↓

Stack Frame Created

↓

Variable Exists

↓

Function Returns

↓

Variable Destroyed
```

Static local variable:

``` text
Program Starts

↓

Variable Created

↓

Function Uses It

↓

Function Returns

↓

Variable Still Exists

↓

Program Ends
```

------------------------------------------------------------------------

# Where Is a Static Variable Stored?

Static variables are **not stored on the stack**.

They are placed in the program's static storage area.

Conceptually:

``` text
+----------------------+
| Stack                |
+----------------------+
| Heap                 |
+----------------------+
| Static/Data/BSS      |
+----------------------+
| Code (.text)         |
+----------------------+
```

Initialized static variables usually go into `.data`.

Uninitialized static variables usually go into `.bss`.

------------------------------------------------------------------------

# Static Global Variable

``` c
static int total = 100;
```

This variable still exists for the entire program.

But now it can only be accessed **within the current source file**.

Other `.c` files cannot use it directly.

This is called **internal linkage**.

------------------------------------------------------------------------

# Lifetime vs Scope

These two concepts are often confused.

## Lifetime

How long the variable exists.

## Scope

Where the variable can be accessed.

Example:

``` c
void demo(void)
{
    static int x = 0;
}
```

-   Scope → only inside `demo()`
-   Lifetime → entire program

------------------------------------------------------------------------

# Why Use `static`?

Suppose a function needs to remember previous state.

``` c
int nextId(void)
{
    static int id = 0;
    return ++id;
}
```

Output:

``` text
1
2
3
4
```

Without `static`, it would always return `1`.

------------------------------------------------------------------------

# Real-World Uses

`static` is used for:

-   Function call counters
-   Caching results
-   State machines
-   Embedded firmware
-   Device driver state
-   File-private helper variables
-   Linux kernel modules

------------------------------------------------------------------------

# Advantages

-   Preserves values between function calls.
-   Hides file-private implementation details.
-   Reduces accidental access from other source files.

------------------------------------------------------------------------

# Common Mistakes

## Thinking `static` Means Constant

Wrong.

``` c
static int x = 5;

x++;
```

This is perfectly valid.

------------------------------------------------------------------------

## Confusing Lifetime with Scope

A static local variable has:

-   Local scope
-   Global lifetime

These are different concepts.

------------------------------------------------------------------------

# Hands-on Labs

1.  Compare a normal local variable with a static local variable.
2.  Build a function call counter.
3.  Create a `static` global variable and try to access it from another
    `.c` file.
4.  Inspect symbols using:

``` bash
nm program
```

Observe internal linkage.

------------------------------------------------------------------------

# Interview Questions

### What does `static` change?

It changes storage duration and, for global variables/functions,
linkage.

### Where are static variables stored?

In the program's static storage area (`.data` or `.bss`).

### Can a static local variable remember its previous value?

Yes.

### Can another source file access a static global variable?

No. It has internal linkage.

------------------------------------------------------------------------

# Summary

``` text
static
   │
   ├── Static Lifetime
   │
   ├── Local Scope (inside function)
   │
   └── Internal Linkage (for file-scope objects)
```

Key Takeaways:

-   `static` does **not** make data constant.
-   Static variables exist for the entire program.
-   Static local variables retain their values between calls.
-   Static global variables are private to one source file.
-   `static` is heavily used in systems programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 43 --- `malloc()`

You'll learn how dynamic memory allocation works internally, what
`malloc()` actually does, how it interacts with the heap, and why
checking its return value is essential.
