# 👑 C Mastery Roadmap

# Module 6 --- Memory Management

# Chapter 41 --- Global Memory

------------------------------------------------------------------------

## Most beginners think:

> "Global variables are stored on the stack."

They are not.

Global variables live in a separate memory region that exists for the
**entire lifetime of the program**.

Unlike stack variables, they are created **before `main()` starts** and
destroyed only when the process exits.

Operating systems load global data when the executable starts.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What global memory is
-   Why global memory exists
-   Global variable lifetime
-   Global scope
-   `.data` section
-   `.bss` section
-   Program startup
-   Global vs stack vs heap

------------------------------------------------------------------------

# Why Do We Need Global Memory?

Suppose every function needs the same configuration.

``` c
int maxConnections = 100;
```

Instead of passing it to every function, a global variable can be
shared.

------------------------------------------------------------------------

# What is a Global Variable?

A variable declared **outside every function**.

``` c
int counter = 0;

int main(void)
{
    counter++;
}
```

`counter` exists before `main()` begins.

------------------------------------------------------------------------

# Memory Layout

A typical process looks like:

``` text
High Address

+----------------------+
| Stack                |
+----------------------+
|                      |
| Heap                 |
|                      |
+----------------------+
| BSS (.bss)           |
+----------------------+
| Initialized Data     |
| (.data)              |
+----------------------+
| Code (.text)         |
+----------------------+

Low Address
```

Global variables are stored in `.data` or `.bss`.

------------------------------------------------------------------------

# Initialized Global Variables (.data)

``` c
int count = 10;
char grade = 'A';
```

Stored in:

``` text
.data
```

The executable file already contains these initial values.

------------------------------------------------------------------------

# Uninitialized Global Variables (.bss)

``` c
int total;
char buffer[1024];
```

Stored in:

``` text
.bss
```

The operating system initializes them to zero before `main()`.

------------------------------------------------------------------------

# Program Startup

Execution flow:

``` text
Executable
      │
      ▼
Operating System Loader
      │
      ├── Load Code
      ├── Load .data
      ├── Zero .bss
      ├── Create Heap
      └── Create Stack
      │
      ▼
main()
```

This happens automatically.

------------------------------------------------------------------------

# Lifetime

``` c
int global = 5;
```

Timeline:

``` text
Program Starts
      │
global exists
      │
Program Ends
```

Unlike local variables, globals do not disappear after a function
returns.

------------------------------------------------------------------------

# Scope

A global variable can usually be accessed by functions in the same
source file.

``` c
int counter = 0;

void inc(void)
{
    counter++;
}
```

We'll later learn `extern` and `static` to control visibility.

------------------------------------------------------------------------

# Global vs Stack vs Heap

  Feature       Global           Stack             Heap
  ------------- ---------------- ----------------- ----------------
  Lifetime      Entire program   Function call     Until `free()`
  Managed by    Loader/OS        Compiler/CPU      Programmer
  Typical use   Shared state     Local variables   Dynamic data

------------------------------------------------------------------------

# Advantages

-   Shared across functions
-   Exists for the whole program
-   No allocation required

------------------------------------------------------------------------

# Disadvantages

-   Hidden dependencies
-   Harder to test
-   Can introduce race conditions in multithreaded programs

Prefer passing data explicitly unless global state is truly needed.

------------------------------------------------------------------------

# Common Mistakes

## Assuming Globals Are on the Stack

They are stored in dedicated program sections.

## Excessive Global State

Too many globals make code difficult to maintain.

------------------------------------------------------------------------

# Real-World Uses

Global memory is used for:

-   Configuration
-   Lookup tables
-   Device driver state
-   Embedded firmware
-   Kernel subsystems
-   Read-only constants (with `const`)

------------------------------------------------------------------------

# Hands-on Labs

1.  Create initialized and uninitialized global variables.
2.  Print their addresses.
3.  Compare with local variable addresses.
4.  Use `nm` or `objdump -t` to inspect `.data` and `.bss`.

------------------------------------------------------------------------

# Interview Questions

### Where are initialized global variables stored?

In the `.data` section.

### Where are uninitialized global variables stored?

In the `.bss` section.

### When are global variables created?

Before `main()` starts.

### Who initializes `.bss`?

The program loader/operating system sets it to zero.

------------------------------------------------------------------------

# Summary

``` text
Program Starts
      │
Load .text
      │
Load .data
      │
Zero .bss
      │
Create Heap & Stack
      │
Run main()
```

Key Takeaways:

-   Global variables exist for the program's lifetime.
-   Initialized globals are stored in `.data`.
-   Uninitialized globals are stored in `.bss`.
-   The operating system prepares these regions before `main()`.
-   Global state should be used carefully.

------------------------------------------------------------------------

# Next Chapter

## Chapter 42 --- Static Memory

You'll learn how `static` changes a variable's lifetime and visibility,
and why it is widely used in embedded systems, libraries, and the Linux
kernel.
