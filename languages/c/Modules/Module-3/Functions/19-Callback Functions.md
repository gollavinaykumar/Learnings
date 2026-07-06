# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 19 --- Callback Functions

------------------------------------------------------------------------

## Most developers think:

> "A function always decides everything by itself."

But many libraries don't know what **your application** wants to do.

Instead of hardcoding behavior, they ask **you** to provide a function.

That function is called a **callback**.

Callbacks are built using **function pointers**.

They are everywhere:

-   Linux kernel
-   POSIX APIs
-   GUI toolkits
-   Networking libraries
-   Sorting functions
-   Event systems

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a callback is
-   Why callbacks exist
-   Relationship between callbacks and function pointers
-   Passing functions as arguments
-   Runtime behavior
-   Real-world examples
-   Common mistakes

------------------------------------------------------------------------

# What is a Callback?

A callback is a function that is **passed to another function** and
executed later.

``` text
Your Function
      │
      ▼
Library Function
      │
      ▼
Callback Executes
```

The library decides **when** to call it.

------------------------------------------------------------------------

# Why Do We Need Callbacks?

Imagine a sorting library.

The library knows **how** to sort.

It doesn't know **how to compare** your custom data.

So you provide a comparison function.

``` text
Sort Library
     │
     ▼
Compare Function (Callback)
```

------------------------------------------------------------------------

# Simple Example

``` c
#include <stdio.h>

void greet(void)
{
    printf("Hello!\n");
}

void execute(void (*cb)(void))
{
    cb();
}

int main(void)
{
    execute(greet);
}
```

Output:

``` text
Hello!
```

------------------------------------------------------------------------

# How It Works

``` text
main()
   │
   ▼
execute(greet)
   │
   ▼
Store Function Pointer
   │
   ▼
cb()
   │
   ▼
greet()
```

The callback runs through the function pointer.

------------------------------------------------------------------------

# Callback Parameters

Callbacks can receive arguments too.

``` c
void printNumber(int n)
{
    printf("%d\n", n);
}

void repeat(void (*cb)(int))
{
    cb(10);
}
```

------------------------------------------------------------------------

# Real-World Example: Event System

``` text
Button Click
      │
      ▼
GUI Library
      │
      ▼
Your Callback
      │
      ▼
Open Window
```

The GUI framework doesn't know your application's logic.

It simply calls your callback.

------------------------------------------------------------------------

# Real-World Example: POSIX Signals

``` text
Signal
   │
   ▼
Kernel
   │
   ▼
Signal Handler (Callback)
```

You register a handler.

The operating system calls it when the signal occurs.

------------------------------------------------------------------------

# Real-World Example: Linux Kernel

Device drivers register callback functions in operation tables.

``` text
Kernel
  │
  ▼
read()
write()
open()
close()
```

The kernel calls the appropriate driver function when needed.

------------------------------------------------------------------------

# Common Mistakes

## Wrong Function Signature

The callback type must exactly match the expected signature.

------------------------------------------------------------------------

## Calling a NULL Callback

``` c
void (*cb)(void) = NULL;

cb();
```

Undefined behavior.

Always validate pointers when appropriate.

------------------------------------------------------------------------

# Hands-on Labs

1.  Write a function that accepts a callback and executes it.
2.  Pass different callbacks to perform different actions.
3.  Build a tiny menu system using callbacks.
4.  Print callback addresses using `%p`.

------------------------------------------------------------------------

# Interview Questions

### What is a callback?

A function passed as an argument and executed later by another function.

### How are callbacks implemented in C?

Using function pointers.

### Why are callbacks useful?

They make libraries reusable and allow behavior to be customized without
modifying library code.

### Where are callbacks commonly used?

GUI frameworks, operating systems, networking libraries, sorting APIs,
event loops, and device drivers.

------------------------------------------------------------------------

# Summary

``` text
Your Function
      │
      ▼
Function Pointer
      │
      ▼
Library
      │
      ▼
Callback Execution
```

Key Takeaways:

-   Callbacks are functions executed by other functions.
-   They are implemented using function pointers.
-   Callbacks enable flexible and reusable APIs.
-   Many operating system and networking APIs rely heavily on callbacks.

------------------------------------------------------------------------

# Next Chapter

## Chapter 20 --- Inline Functions

You'll learn how inline functions reduce function-call overhead, when
the compiler inlines code, and when `inline` should (and should not) be
used.
