# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 20 --- Inline Functions

------------------------------------------------------------------------

## Most beginners think:

> "Every function call costs nothing."

Every function call has overhead.

The CPU must:

-   Pass arguments
-   Save registers
-   Create a stack frame
-   Jump to another address
-   Return to the caller

For tiny functions, that overhead can be larger than the actual work.

Inline functions help reduce that cost.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What an inline function is
-   Why inline exists
-   Function call overhead
-   How the compiler handles inline
-   Advantages
-   Limitations
-   When to use inline
-   Common mistakes

------------------------------------------------------------------------

# Normal Function Call

``` c
int square(int x)
{
    return x * x;
}

int main(void)
{
    int y = square(5);
}
```

Execution:

``` text
main()
   │
   ▼
Call square()
   │
Create Stack Frame
   │
Return
   │
Continue main()
```

------------------------------------------------------------------------

# What is an Inline Function?

``` c
inline int square(int x)
{
    return x * x;
}
```

`inline` is a request to the compiler:

> "If it makes sense, replace the function call with the function's
> code."

------------------------------------------------------------------------

# Before Inlining

``` c
int y = square(5);
```

CPU performs a function call.

------------------------------------------------------------------------

# After Inlining

The compiler may transform it into:

``` c
int y = 5 * 5;
```

No function call is required.

------------------------------------------------------------------------

# Why is This Faster?

Without a function call:

-   No stack frame
-   No return address
-   No call/return instructions

This can improve performance for very small functions.

------------------------------------------------------------------------

# Is Inlining Guaranteed?

No.

`inline` is only a **hint**.

The compiler decides whether to inline based on:

-   Optimization level
-   Function size
-   Recursion
-   Complexity

------------------------------------------------------------------------

# When Should You Use Inline?

Good candidates:

-   Small mathematical helpers
-   Getter functions
-   Tiny utility functions

Example:

``` c
inline int max(int a, int b)
{
    return (a > b) ? a : b;
}
```

------------------------------------------------------------------------

# When Should You Avoid Inline?

Avoid large functions.

``` c
inline void processDatabase(void)
{
    /* hundreds of lines */
}
```

Inlining large functions can increase executable size and reduce
instruction-cache efficiency.

------------------------------------------------------------------------

# Advantages

-   Reduces function-call overhead
-   May improve performance
-   Useful in performance-critical code

------------------------------------------------------------------------

# Disadvantages

-   Larger executable if overused
-   Compiler may ignore the request
-   Not suitable for complex functions

------------------------------------------------------------------------

# Common Mistakes

## Assuming `inline` Always Happens

The compiler is free to ignore the request.

------------------------------------------------------------------------

## Inlining Large Functions

Large inline functions often hurt performance rather than improve it.

------------------------------------------------------------------------

# Hands-on Labs

1.  Create a normal `square()` function.
2.  Convert it to `inline`.
3.  Compile with optimization:

``` bash
gcc -O2 program.c
```

4.  Generate assembly:

``` bash
gcc -S -O2 program.c
```

Observe whether the call disappeared.

------------------------------------------------------------------------

# Interview Questions

### What is an inline function?

A function that the compiler may expand directly at the call site.

### Is `inline` guaranteed?

No. It is only a compiler hint.

### Why use inline?

To reduce function-call overhead for very small functions.

### Can inline increase executable size?

Yes. Replacing many calls with copied code can increase binary size.

------------------------------------------------------------------------

# Summary

``` text
Normal Function

Call
 │
 ▼
Execute
 │
 ▼
Return

Inline Function

Replace Call
 │
 ▼
Execute Directly
```

Key Takeaways:

-   Function calls have overhead.
-   `inline` requests the compiler to remove that overhead.
-   The compiler decides whether to inline.
-   Use `inline` for small, frequently called functions.
-   Avoid overusing it on large functions.

------------------------------------------------------------------------

# Module 3 Complete ✅

You have learned:

-   Function declarations
-   Function definitions
-   Function calls
-   Call stack
-   Stack frames
-   Pass by value
-   Simulating pass by reference
-   Function pointers
-   Callback functions
-   Inline functions

Next:

# Module 4 --- Arrays & Strings

You'll learn arrays, multidimensional arrays, strings, memory layout,
and how C stores text in memory.
