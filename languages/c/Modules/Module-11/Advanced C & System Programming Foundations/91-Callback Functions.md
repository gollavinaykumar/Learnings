# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 91 — Callback Functions

---

# Most beginners think:

> "A function always calls another function directly."

Example:

```c
main()
{
    hello();
}
```

Flow:

```
main()

↓

hello()
```

Simple.

But modern software rarely works this way.

Consider:

- A button click in a GUI
- A network packet arriving
- A timer expiring
- A file download finishing
- A keyboard key being pressed

How does the operating system know **which function** to call?

The answer is:

```
Callbacks
```

A callback is simply:

> **A function passed to another function so it can be called later.**

Callbacks are everywhere.

- Linux Kernel
- Web Browsers
- Game Engines
- Network Libraries
- Databases
- Operating Systems

---

# Learning Objectives

After this chapter you will understand:

- What callbacks are
- Why callbacks exist
- Passing functions as arguments
- Event-driven programming
- Callback execution flow
- Real-world callback systems

---

# What is a Callback?

Suppose we have:

```c
void hello()
{
    printf("Hello\n");
}
```

Instead of calling:

```c
hello();
```

directly,

we pass it to another function.

```
hello

↓

Another Function

↓

Calls It Later
```

That passed function is called a:

```
Callback
```

---

# First Callback Example

```c
#include <stdio.h>

void hello()
{
    printf("Hello\n");
}

void execute(void (*callback)())
{
    callback();
}

int main()
{
    execute(hello);

    return 0;
}
```

Output:

```
Hello
```

---

# Flow Visualization

```
main()

↓

execute()

↓

callback()

↓

hello()
```

Notice:

`execute()` doesn't know about `hello()`.

It simply receives **a function pointer**.

---

# Why Is This Useful?

Suppose:

Need different behavior.

Instead of:

```c
if(choice == 1)
    add();

if(choice == 2)
    subtract();
```

We can write:

```c
execute(add);

execute(subtract);
```

The same function can work with many behaviors.

---

# Another Example

```c
#include <stdio.h>

void add()
{
    printf("Add\n");
}

void multiply()
{
    printf("Multiply\n");
}

void run(void (*op)())
{
    op();
}

int main()
{
    run(add);

    run(multiply);

    return 0;
}
```

Output:

```
Add

Multiply
```

---

# Callback with Parameters

Suppose:

```c
int add(int a,
        int b)
{
    return a + b;
}
```

Callback:

```c
int calculate(
    int x,
    int y,
    int (*operation)(
        int,
        int))
{
    return operation(x, y);
}
```

Use:

```c
printf("%d",
calculate(
10,
20,
add));
```

Output:

```
30
```

---

# Visualization

```
calculate()

↓

operation

↓

add()

↓

Return Result
```

---

# Callback Memory

Suppose:

```
Function Pointer

↓

Stack

↓

Address

↓

Code Segment
```

Calling:

```
callback()

↓

Jump

↓

Execute Code
```

---

# Event-Driven Programming

Traditional programming:

```
Program

↓

Calls Function
```

Event-driven programming:

```
Event Happens

↓

Framework Calls Your Function
```

This is callback-based programming.

---

# Example — Button Click

GUI:

```
Button

↓

Click

↓

Framework

↓

Callback
```

You don't call the callback.

The framework does.

---

# Example — Timer

Program:

```
Start Timer
```

After:

```
5 Seconds
```

System:

```
Calls Callback
```

---

# Example — Network Server

Packet arrives.

```
Network Driver

↓

Callback

↓

Process Packet
```

The driver doesn't know your application logic.

It simply invokes the registered callback.

---

# Example — Linux Kernel

Device Driver:

Registers:

```
Read Callback

Write Callback

Open Callback

Close Callback
```

When a process performs:

```c
read()
```

the kernel invokes the driver's registered read callback.

---

# Example — `qsort()`

The C Standard Library provides:

```c
qsort()
```

Prototype (simplified):

```c
qsort(
    array,
    count,
    size,
    compare);
```

Notice:

```
compare

↓

Callback
```

`qsort()` doesn't know how to compare your data.

You provide the comparison function.

---

# `qsort()` Comparator

Example:

```c
int compare(
const void *a,
const void *b)
{
    return
    *(int *)a -
    *(int *)b;
}
```

Then:

```c
qsort(
arr,
n,
sizeof(int),
compare);
```

The sorting algorithm repeatedly calls your callback.

---

# Callback Flow

```
qsort()

↓

compare()

↓

compare()

↓

compare()

↓

Sorted
```

The library controls **when** and **how often** your function is called.

---

# Advantages

Callbacks provide:

- Flexibility
- Code reuse
- Extensibility
- Event-driven programming
- Plugin support

The framework and the user code remain loosely coupled.

---

# Real-World Example — HTTP Server

Request:

```
GET /users
```

Framework:

```
Find Handler

↓

Call Callback
```

Different URLs map to different callback functions.

---

# Real-World Example — Database

Database engine:

```
Query Finished

↓

Callback

↓

Return Results
```

Applications register functions to process results.

---

# Real-World Example — Game Engine

Game Loop:

```
Collision

↓

Callback

↓

Play Sound

↓

Reduce Health
```

The engine triggers callbacks when events occur.

---

# Real-World Example — Operating System

Operating systems rely on callbacks for:

- Interrupt handlers
- Signal handlers
- Device drivers
- Timers

The hardware or kernel decides **when** the callback runs.

---

# Common Mistakes

---

## Wrong Function Signature

Suppose callback expects:

```c
int (*)(int,int)
```

Passing:

```c
void hello()
```

is incorrect.

The function signature must match.

---

## Calling an Uninitialized Callback

Wrong:

```c
void (*cb)();

cb();
```

Undefined behavior.

Always initialize callback pointers.

---

## Confusing Function with Function Pointer

These are equivalent in many contexts:

```c
execute(hello);
```

and

```c
execute(&hello);
```

The function name usually converts to a pointer automatically.

---

## Forgetting the Framework Owns the Call

With callbacks,

your function is often **called by someone else**.

You don't control exactly when it executes.

---

# Complete Example

```c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int calculate(
    int x,
    int y,
    int (*operation)(
        int,
        int))
{
    return operation(x, y);
}

int main()
{
    printf("%d\n",
           calculate(
               5,
               7,
               add));

    return 0;
}
```

Output:

```
12
```

---

# Hands-on Labs

## Lab 1

Pass a function as an argument.

Call it inside another function.

---

## Lab 2

Create callbacks for:

- Addition
- Subtraction
- Multiplication
- Division

Implement a generic calculator.

---

## Lab 3

Use `qsort()` with a custom comparison callback.

Sort integers in ascending and descending order.

---

## Lab 4

Create a simple event system where different callbacks handle different events.

---

## Lab 5

Simulate a button click by storing and invoking a callback function.

---

# Interview Questions

### What is a callback?

A function passed to another function so it can be called later.

---

### How are callbacks implemented in C?

Using function pointers.

---

### Why are callbacks useful?

They allow flexible, reusable, event-driven program design.

---

### Give an example of a callback in the C standard library.

`qsort()` uses a comparison callback.

---

### Where are callbacks commonly used?

- GUI frameworks
- Operating systems
- Device drivers
- Networking
- Timers
- Event loops
- Libraries

---

# Summary

```
Function

↓

Function Pointer

↓

Pass To Another Function

↓

Called Later

↓

Callback
```

Event Flow:

```
Event Happens

↓

Framework

↓

Callback

↓

Your Code Executes
```

## Key Takeaways

- A callback is a function invoked by another function at a later time.
- Callbacks are implemented using function pointers.
- They enable event-driven programming and highly reusable code.
- The callback's signature must match what the caller expects.
- The C standard library (`qsort`) uses callbacks extensively.
- Callbacks are fundamental in operating systems, networking, GUIs, databases, and modern software frameworks.

---

# Next Chapter

## Chapter 92 — Generic Programming with `void *`

You'll learn:

- Why `void *` exists
- Generic data handling
- Type-independent functions
- Building reusable libraries
- Generic swap
- Generic linked lists
- How the C standard library implements functions like `qsort()` and `bsearch()`