# 👑 C Mastery Roadmap

# Module 2 --- Control Flow

# Chapter 12 --- Recursion

------------------------------------------------------------------------

## Most beginners think:

> "A function should never call itself."

It sounds strange at first.

How can a function keep calling itself without running forever?

The answer is:

A recursive function must know **when to stop**.

Recursion is one of the most powerful ideas in computer science. Many
algorithms, compilers, file systems, and tree structures rely on it.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What recursion is
-   Base case
-   Recursive case
-   How the call stack works
-   Stack frames
-   Recursion vs iteration
-   Common recursive algorithms
-   Common mistakes

------------------------------------------------------------------------

# What is Recursion?

Recursion is when a function calls itself.

``` c
void greet(int n)
{
    if (n == 0)
        return;

    printf("Hello\n");

    greet(n - 1);
}
```

Each call creates a new function invocation.

------------------------------------------------------------------------

# Why Recursion Works

Every recursive function has two parts.

## Base Case

The stopping condition.

``` c
if (n == 0)
    return;
```

## Recursive Case

The function calls itself with a smaller problem.

``` c
greet(n - 1);
```

Without a base case, recursion never stops.

------------------------------------------------------------------------

# Visualizing Recursion

Calling:

``` text
greet(3)
```

creates:

``` text
greet(3)
   │
   ▼
greet(2)
   │
   ▼
greet(1)
   │
   ▼
greet(0)
```

Now the calls return in reverse order.

------------------------------------------------------------------------

# The Call Stack

Every function call is placed on the call stack.

``` text
Top
┌────────────┐
│ greet(0)   │
├────────────┤
│ greet(1)   │
├────────────┤
│ greet(2)   │
├────────────┤
│ greet(3)   │
└────────────┘
Bottom
```

When `greet(0)` returns, its stack frame is removed.

------------------------------------------------------------------------

# Example --- Factorial

``` c
int factorial(int n)
{
    if (n == 0)
        return 1;

    return n * factorial(n - 1);
}
```

Calling:

``` text
factorial(4)
```

Expands to:

``` text
4 × factorial(3)
      │
3 × factorial(2)
      │
2 × factorial(1)
      │
1 × factorial(0)
      │
1
```

Result:

``` text
24
```

------------------------------------------------------------------------

# Stack Frames

Every function call has its own stack frame.

A stack frame contains:

-   Parameters
-   Local variables
-   Return address
-   Saved registers

``` text
Stack

┌──────────────┐
│ factorial(0) │
├──────────────┤
│ factorial(1) │
├──────────────┤
│ factorial(2) │
├──────────────┤
│ factorial(3) │
└──────────────┘
```

------------------------------------------------------------------------

# Recursion vs Iteration

Iteration:

``` c
int result = 1;

for (int i = 1; i <= n; i++)
    result *= i;
```

Recursion:

``` c
return n * factorial(n - 1);
```

  Recursion           Iteration
  ------------------- ------------------------------
  Elegant             Often faster
  Uses call stack     Uses loops
  Simpler for trees   Better for simple repetition

------------------------------------------------------------------------

# Common Uses

Recursion is widely used for:

-   Factorial
-   Fibonacci
-   Tree traversal
-   Directory traversal
-   Graph algorithms
-   Divide and conquer algorithms
-   Backtracking

------------------------------------------------------------------------

# Common Mistakes

## Missing Base Case

``` c
void f()
{
    f();
}
```

This never stops.

Eventually:

``` text
Stack Overflow
```

------------------------------------------------------------------------

## Too Much Recursion

Very deep recursion may exhaust stack memory.

Sometimes an iterative solution is more appropriate.

------------------------------------------------------------------------

# Hands-on Labs

1.  Write a recursive factorial function.
2.  Print numbers from `n` to `1` recursively.
3.  Implement recursive Fibonacci.
4.  Compare recursive and iterative factorial performance.
5.  Observe stack growth using a debugger.

------------------------------------------------------------------------

# Interview Questions

### What is recursion?

A technique where a function calls itself to solve a smaller version of
the same problem.

### What is the base case?

The condition that stops recursion.

### What happens if there is no base case?

The function continues calling itself until the program crashes with a
stack overflow.

### What is a stack frame?

The memory allocated for one function call on the call stack.

### When should recursion be preferred?

When the problem is naturally recursive, such as trees, graphs,
divide-and-conquer, or backtracking.

------------------------------------------------------------------------

# Summary

``` text
Function Call
      │
      ▼
Stack Frame
      │
      ▼
Recursive Call
      │
      ▼
Base Case
      │
      ▼
Return
      │
      ▼
Stack Unwinds
```

Key Takeaways:

-   Recursion solves problems by reducing them into smaller subproblems.
-   Every recursive function needs a base case.
-   Each call creates a new stack frame.
-   Deep recursion can cause stack overflow.
-   Recursion is fundamental to many advanced algorithms and data
    structures.

------------------------------------------------------------------------

# Module 2 Complete ✅

You have completed:

-   if / else / switch
-   for / while / do...while
-   break / continue / goto
-   Recursion

Next:

# Module 3 --- Functions

You'll explore function declarations, definitions, stack frames,
parameter passing, function pointers, callbacks, and inline functions.
