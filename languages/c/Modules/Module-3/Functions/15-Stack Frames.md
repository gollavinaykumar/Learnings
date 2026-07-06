# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 15 --- Stack Frames

------------------------------------------------------------------------

## Most developers think:

> "The call stack only stores function names."

Not quite.

Every function call creates a **stack frame** containing everything
needed for that specific invocation.

Without stack frames, functions could not have local variables,
parameters, or know where to return.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a stack frame is
-   Why stack frames exist
-   Stack frame layout
-   Parameters
-   Local variables
-   Return address
-   Frame pointer
-   Stack pointer
-   Saved registers
-   Function prologue & epilogue

------------------------------------------------------------------------

# What is a Stack Frame?

A stack frame is the block of memory allocated for one function call.

``` text
Call Stack

┌────────────────────┐
│ Stack Frame (foo)  │
├────────────────────┤
│ Stack Frame (main) │
└────────────────────┘
```

Every active function has its own frame.

------------------------------------------------------------------------

# Why Does Each Function Need One?

Consider:

``` c
void printSum(int a, int b)
{
    int total = a + b;
}
```

The function needs memory for:

-   Parameters (`a`, `b`)
-   Local variable (`total`)
-   Return address
-   Saved CPU state

------------------------------------------------------------------------

# Typical Stack Frame Layout

``` text
High Memory
┌──────────────────────┐
│ Function Arguments   │
├──────────────────────┤
│ Return Address       │
├──────────────────────┤
│ Saved Frame Pointer  │
├──────────────────────┤
│ Local Variables      │
├──────────────────────┤
│ Temporary Data       │
└──────────────────────┘
Low Memory
```

The exact layout depends on the CPU architecture and compiler.

------------------------------------------------------------------------

# Stack Pointer (SP)

The **Stack Pointer** always points to the current top of the stack.

``` text
SP
 │
 ▼
┌────────────┐
│ Current    │
│ StackFrame │
└────────────┘
```

As functions are called, the stack pointer moves.

------------------------------------------------------------------------

# Frame Pointer (FP)

The **Frame Pointer** provides a stable reference point inside the
current frame.

It allows the compiler to easily access:

-   Parameters
-   Local variables

Even if the stack pointer changes during execution.

------------------------------------------------------------------------

# Function Prologue

Before executing a function body, the compiler usually:

1.  Saves the previous frame pointer.
2.  Creates a new stack frame.
3.  Allocates space for local variables.

``` text
Call Function
      │
      ▼
Create Stack Frame
      │
      ▼
Execute Code
```

------------------------------------------------------------------------

# Function Epilogue

When returning:

1.  Local variables are discarded.
2.  Previous frame pointer is restored.
3.  Return address is used.
4.  Control goes back to the caller.

``` text
Return
   │
   ▼
Destroy Frame
   │
   ▼
Continue Caller
```

------------------------------------------------------------------------

# Example

``` c
int square(int x)
{
    int result = x * x;
    return result;
}
```

Stack frame:

``` text
┌──────────────────┐
│ x = 5            │
├──────────────────┤
│ result = 25      │
├──────────────────┤
│ Return Address   │
└──────────────────┘
```

When `square()` finishes, the entire frame disappears.

------------------------------------------------------------------------

# Stack Frames During Nested Calls

``` text
main()
  │
  ▼
login()
  │
  ▼
authenticate()
```

Stack:

``` text
┌──────────────────┐
│ authenticate()   │
├──────────────────┤
│ login()          │
├──────────────────┤
│ main()           │
└──────────────────┘
```

Each function has its own independent local variables.

------------------------------------------------------------------------

# Common Mistakes

## Returning Address of Local Variable

``` c
int* bad()
{
    int x = 10;
    return &x;
}
```

`x` disappears when the stack frame is destroyed.

The returned pointer becomes invalid.

------------------------------------------------------------------------

## Large Local Arrays

``` c
char buffer[1000000];
```

Very large local arrays may exhaust stack memory.

Allocate large dynamic objects on the heap instead.

------------------------------------------------------------------------

# Hands-on Labs

1.  Write two functions and print addresses of local variables.
2.  Observe how each function gets different stack memory.
3.  Step through function calls using `gdb`.
4.  Inspect the backtrace with `bt`.
5.  Compare recursive and non-recursive stack growth.

------------------------------------------------------------------------

# Interview Questions

### What is a stack frame?

A block of stack memory allocated for one function call.

### What does a stack frame contain?

Parameters, local variables, return address, saved registers, and
bookkeeping information.

### What is the stack pointer?

A CPU register that points to the top of the current stack.

### What is the frame pointer?

A stable reference used to access variables inside the current stack
frame.

### Why can't we return a pointer to a local variable?

Because the local variable is destroyed when the function returns.

------------------------------------------------------------------------

# Summary

``` text
Function Call
      │
      ▼
Create Stack Frame
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
Destroy Frame
      │
      ▼
Return to Caller
```

Key Takeaways:

-   Every function call gets its own stack frame.
-   Stack frames isolate local variables between function calls.
-   The stack pointer and frame pointer help manage execution.
-   Returning pointers to local variables is unsafe.
-   Understanding stack frames is essential for debugging, memory
    management, and systems programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 16 --- Pass by Value

You'll learn how arguments are copied into function parameters, why
changes inside a function usually don't affect the caller, and how this
leads to pass-by-reference techniques.
