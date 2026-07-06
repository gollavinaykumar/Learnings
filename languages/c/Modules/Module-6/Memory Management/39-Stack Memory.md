# 👑 C Mastery Roadmap

# Module 6 --- Memory Management

# Chapter 39 --- Stack Memory

------------------------------------------------------------------------

## Most beginners think:

> "Memory is just one big block of RAM."

It isn't.

When your program starts, the operating system divides its virtual
memory into different regions.

One of the most important regions is the **Stack**.

Almost every function call you write uses the stack.

Without the stack:

-   Functions wouldn't work
-   Local variables couldn't exist
-   Recursion would be impossible
-   The CPU wouldn't know where to return after a function call

Understanding the stack is essential before learning the heap,
`malloc()`, and the Linux kernel.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What stack memory is
-   Why the stack exists
-   How the stack grows
-   Local variables
-   Function calls
-   Stack frames
-   Automatic memory management
-   Stack overflow
-   Stack vs Heap
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Stack Memory?

Consider this program:

``` c
int square(int x)
{
    int result = x * x;
    return result;
}

int main(void)
{
    int n = 5;
    int ans = square(n);
}
```

Questions:

-   Where is `n` stored?
-   Where is `x` stored?
-   Where is `result` stored?
-   How does the CPU know where to return after `square()`?

The answer is:

**The Stack.**

------------------------------------------------------------------------

# What is Stack Memory?

The stack is a region of memory used to store information for **active
function calls**.

Each function call gets its own **stack frame**.

``` text
Top of Stack
┌────────────────────┐
│ square()           │
├────────────────────┤
│ main()             │
└────────────────────┘
Bottom of Stack
```

When `square()` returns, its stack frame disappears automatically.

------------------------------------------------------------------------

# What Does a Stack Frame Contain?

A typical stack frame stores:

-   Function parameters
-   Local variables
-   Return address
-   Saved CPU registers
-   Bookkeeping information

``` text
Stack Frame

┌────────────────────┐
│ Local Variables    │
├────────────────────┤
│ Parameters         │
├────────────────────┤
│ Return Address     │
├────────────────────┤
│ Saved Registers    │
└────────────────────┘
```

------------------------------------------------------------------------

# Local Variables Live on the Stack

``` c
void demo(void)
{
    int age = 25;
}
```

Memory:

``` text
Stack

┌──────────────┐
│ age = 25     │
└──────────────┘
```

When the function returns:

``` text
Stack Frame Removed

↓

age no longer exists
```

------------------------------------------------------------------------

# Stack Growth

On most modern systems, the stack grows **toward lower memory
addresses**.

``` text
High Address

┌──────────────┐
│ Older Frame  │
├──────────────┤
│ Newer Frame  │
└──────────────┘

↓

Lower Address
```

The exact direction depends on the architecture, but downward growth is
the most common.

------------------------------------------------------------------------

# Push and Pop

Function call:

``` text
Push Stack Frame
```

Function return:

``` text
Pop Stack Frame
```

``` text
main()

↓

Push

↓

square()

↓

Return

↓

Pop

↓

main()
```

------------------------------------------------------------------------

# Automatic Memory Management

Stack memory is managed automatically.

You never write:

``` c
free(localVariable);
```

When a function ends, all its local variables are removed automatically.

This makes stack allocation extremely fast.

------------------------------------------------------------------------

# Stack Overflow

Every program has a limited stack size.

Example:

``` c
void recurse(void)
{
    recurse();
}
```

Every call creates another stack frame.

Eventually:

``` text
Stack

↓

Full

↓

Stack Overflow
```

Typical results:

-   Segmentation fault
-   Program termination

------------------------------------------------------------------------

# Stack vs Heap

  Stack               Heap
  ------------------- ------------------------
  Automatic           Manual (`malloc/free`)
  Very Fast           Slower
  Small               Much Larger
  Local Variables     Dynamic Memory
  Function Lifetime   Programmer Controlled

------------------------------------------------------------------------

# Why Is the Stack Fast?

Allocating stack memory usually means:

-   Adjust the stack pointer
-   Reserve space

No searching for free blocks.

No complex bookkeeping.

This makes stack allocation extremely efficient.

------------------------------------------------------------------------

# Common Mistakes

## Returning Local Variable Addresses

Wrong:

``` c
int *bad(void)
{
    int x = 10;
    return &x;
}
```

`x` disappears when the function returns.

------------------------------------------------------------------------

## Huge Local Arrays

``` c
char buffer[10000000];
```

Large local allocations can exhaust the stack.

Allocate large data on the heap instead.

------------------------------------------------------------------------

# Real-World Uses

Stack memory is used in:

-   Every C function
-   Linux kernel
-   Device drivers
-   Recursive algorithms
-   Compilers
-   Network servers
-   Database engines

------------------------------------------------------------------------

# Hands-on Labs

1.  Print addresses of local variables in different functions.
2.  Write a recursive function and observe stack growth using `gdb`.
3.  Create a large local array and observe stack overflow.
4.  Use `ulimit -s` (Linux/macOS) to view the stack size limit.

------------------------------------------------------------------------

# Interview Questions

### What is stack memory?

A memory region used for active function calls and local variables.

### Who manages the stack?

The compiler, CPU, and operating system automatically manage it.

### Why is stack allocation fast?

Because it usually requires only moving the stack pointer.

### What causes a stack overflow?

Too many nested function calls or excessive local memory usage.

### Can you return a pointer to a local variable?

No. The local variable is destroyed when the function returns.

------------------------------------------------------------------------

# Summary

``` text
Function Call
      │
      ▼
Push Stack Frame
      │
      ├── Parameters
      ├── Local Variables
      ├── Return Address
      └── Saved Registers
      │
      ▼
Execute Function
      │
      ▼
Return
      │
      ▼
Pop Stack Frame
```

Key Takeaways:

-   The stack stores active function calls.
-   Every function gets its own stack frame.
-   Local variables usually live on the stack.
-   Stack memory is allocated and freed automatically.
-   Stack allocation is extremely fast.
-   Excessive recursion or large local variables can cause stack
    overflow.

------------------------------------------------------------------------

# Next Chapter

## Chapter 40 --- Heap Memory

You'll learn:

-   What the heap is
-   Why dynamic memory exists
-   How heap allocation differs from stack allocation
-   Why `malloc()` uses the heap
-   How the operating system manages heap memory
