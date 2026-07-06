# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 14 --- Call Stack

------------------------------------------------------------------------

## Most developers think:

> "When I call a function, the CPU simply jumps to it."

That is only part of the story.

The operating system and CPU must remember:

-   Which function was running
-   Where to return after the call
-   The function's local variables
-   Parameters
-   Temporary values

This is managed using the **Call Stack**.

Without the call stack, functions would not work.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What the call stack is
-   Why it exists
-   Function call flow
-   Return addresses
-   Nested function calls
-   Recursive calls
-   Stack overflow
-   How debuggers use the stack

------------------------------------------------------------------------

# Why Do We Need a Call Stack?

Imagine:

``` c
main();
```

calls

``` c
login();
```

which calls

``` c
validateUser();
```

The CPU must know how to return.

``` text
main()
   │
   ▼
login()
   │
   ▼
validateUser()
```

After `validateUser()` finishes, execution must continue inside
`login()`, not restart the program.

------------------------------------------------------------------------

# What Is the Call Stack?

The call stack is a special memory region that stores information about
active function calls.

``` text
Top
┌────────────────────┐
│ validateUser()     │
├────────────────────┤
│ login()            │
├────────────────────┤
│ main()             │
└────────────────────┘
Bottom
```

The most recently called function is always on top.

------------------------------------------------------------------------

# Push and Pop

Calling a function:

``` text
Push Stack Frame
```

Returning from a function:

``` text
Pop Stack Frame
```

``` text
main()
  │
Push
  ▼
login()
  │
Push
  ▼
validateUser()
  │
Return
  ▼
Pop
  │
login()
```

------------------------------------------------------------------------

# Example

``` c
void c() {}

void b() {
    c();
}

void a() {
    b();
}

int main() {
    a();
}
```

Execution:

``` text
main
 │
 ▼
a
 │
 ▼
b
 │
 ▼
c
```

Stack growth:

``` text
┌────────┐
│ c()    │
├────────┤
│ b()    │
├────────┤
│ a()    │
├────────┤
│ main() │
└────────┘
```

Then the stack unwinds in reverse order.

------------------------------------------------------------------------

# Return Address

Every stack frame stores a return address.

``` text
login()

↓

Calls

↓

validateUser()

↓

Return Address

↓

Continue login()
```

This tells the CPU exactly where to continue execution.

------------------------------------------------------------------------

# Nested Calls

Multiple functions can call each other.

``` text
main
 │
 ▼
A
 │
 ▼
B
 │
 ▼
C
```

Every call creates another stack frame.

------------------------------------------------------------------------

# Recursion and the Stack

Recursive calls also create new stack frames.

``` text
factorial(3)
   │
factorial(2)
   │
factorial(1)
   │
factorial(0)
```

Each invocation has its own local variables.

------------------------------------------------------------------------

# Stack Overflow

If too many stack frames are created:

``` text
main
 │
 ▼
f()
 │
 ▼
f()
 │
 ▼
f()
 │
 ▼
...
```

Eventually the stack runs out of memory.

Result:

``` text
Stack Overflow
```

------------------------------------------------------------------------

# Debugging the Call Stack

When a program crashes, debuggers show a **stack trace**.

Example:

``` text
main()
 └── login()
      └── validateUser()
           └── crash()
```

This helps identify where the error occurred.

------------------------------------------------------------------------

# Common Mistakes

## Infinite Recursion

Missing a base case causes unlimited stack growth.

## Large Local Variables

Very large arrays inside functions consume stack memory quickly.

Prefer heap allocation for large dynamic data.

------------------------------------------------------------------------

# Hands-on Labs

1.  Write three functions that call each other.
2.  Trace the order of execution with `printf()`.
3.  Implement recursive factorial and visualize the stack.
4.  Use `gdb` and run `backtrace` (`bt`) after a breakpoint.
5.  Create intentional infinite recursion and observe the stack
    overflow.

------------------------------------------------------------------------

# Interview Questions

### What is the call stack?

A memory structure that stores active function calls.

### What is stored on the call stack?

Return addresses, parameters, local variables and other call
information.

### Why is it called a stack?

Because function calls follow Last-In, First-Out (LIFO) order.

### What causes stack overflow?

Too many nested function calls or excessive stack usage.

### What is a stack trace?

A list of active function calls at a particular moment, often shown
after a crash.

------------------------------------------------------------------------

# Summary

``` text
Function Call
      │
      ▼
Push Stack Frame
      │
      ▼
Execute Function
      │
      ▼
Return
      │
      ▼
Pop Stack Frame
      │
      ▼
Continue Caller
```

Key Takeaways:

-   Every function call creates a stack frame.
-   The call stack tracks active functions.
-   Functions return in reverse order of calls.
-   Recursive calls consume additional stack frames.
-   Understanding the call stack is essential for debugging and systems
    programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 15 --- Stack Frames

You'll explore the internal layout of a stack frame, including local
variables, parameters, saved registers, frame pointers, and how CPUs
access them.
