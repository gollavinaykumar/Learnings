# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 98 — Assertions (`assert.h`)

---

# Most beginners think:

> "If my program compiles successfully, it must be correct."

Unfortunately...

compilation only verifies:

- Syntax
- Types
- Language rules

It **cannot** verify your logic.

Consider:

```c
int divide(int a, int b)
{
    return a / b;
}
```

Compiles perfectly.

But what happens here?

```c
divide(10, 0);
```

```
Division By Zero

↓

Crash

or

Undefined Behavior
```

How can we detect mistakes **before** they become serious bugs?

Professional C programmers use:

```
Assertions
```

---

# Learning Objectives

After this chapter you will understand:

- What assertions are
- Why assertions exist
- `assert()`
- `NDEBUG`
- Debug vs Release builds
- Defensive programming
- Best practices

---

# What is an Assertion?

An assertion is a statement that says:

> **"This condition must always be true here."**

Example:

```c
assert(ptr != NULL);
```

Meaning:

```
Pointer

↓

Must Not Be NULL
```

If it is:

```
Stop Program Immediately
```

---

# Assertion Flow

```
Condition

↓

True

↓

Continue

-------------------

Condition

↓

False

↓

Print Error

↓

Abort Program
```

---

# Using `assert()`

Include:

```c
#include <assert.h>
```

Example:

```c
#include <assert.h>

int main()
{
    int x = 10;

    assert(x == 10);

    return 0;
}
```

Program runs normally.

---

# Failed Assertion

Example:

```c
#include <assert.h>

int main()
{
    int x = 5;

    assert(x == 10);

    return 0;
}
```

Typical output:

```
Assertion failed:

x == 10
```

The program terminates immediately.

---

# Why Is This Useful?

Suppose:

```
File Pointer

↓

Must Exist
```

Instead of:

```
Continue

↓

Crash Later
```

Use:

```c
assert(file != NULL);
```

The bug is detected immediately,

making debugging much easier.

---

# Assertion Example

```c
FILE *fp =
fopen("data.txt",
      "r");

assert(fp != NULL);
```

If opening the file unexpectedly fails,

the assertion reports the problem during development.

---

# Pointer Validation

Example:

```c
void print(
int *ptr)
{
    assert(ptr != NULL);

    printf("%d",
           *ptr);
}
```

Without assertion:

```
NULL

↓

Dereference

↓

Crash
```

With assertion:

```
NULL

↓

Detected Immediately
```

---

# Array Example

Suppose:

```
Index

↓

Must Be

0-9
```

Program:

```c
assert(index >= 0 &&
       index < 10);
```

This helps catch programming mistakes during testing.

---

# Function Preconditions

Suppose:

```c
double squareRoot(
double x)
```

Requirement:

```
x >= 0
```

Assertion:

```c
assert(x >= 0);
```

Documents an important assumption.

---

# Postconditions

Suppose:

Function returns:

```
Length

↓

Must Be Positive
```

Assertion:

```c
assert(length > 0);
```

Verifies that the function produced a valid result.

---

# Invariants

Suppose:

Queue:

```
size

↓

Never Negative
```

Assertion:

```c
assert(queue->size >= 0);
```

An invariant is something that should always remain true.

---

# What Happens Internally?

Conceptually:

```c
assert(x > 0);
```

Behaves like:

```
Condition True?

↓

Yes

↓

Continue

-----------------

No

↓

Print Diagnostic

↓

Abort
```

The exact implementation varies,

but this is the overall behavior.

---

# Assertion Message

Typical output:

```
Assertion failed:

ptr != NULL

File:

main.c

Line:

42
```

This information helps locate bugs quickly.

---

# Release Builds

Should assertions remain in production?

Often:

```
No
```

During release builds,

they are commonly disabled.

---

# `NDEBUG`

Before including:

```c
assert.h
```

Define:

```c
#define NDEBUG
```

Now:

```c
assert(x > 0);
```

Expands to:

```
Nothing
```

No runtime overhead.

Many build systems define `NDEBUG` automatically for release builds.

---

# Visualization

Debug Build:

```
assert()

↓

Check Condition
```

Release Build:

```
assert()

↓

Removed
```

---

# Assertions vs Error Handling

Assertion:

```
Programmer Mistake
```

Example:

```c
assert(ptr != NULL);
```

Error handling:

```
External Failure
```

Example:

```
Disk Full

↓

Network Timeout

↓

File Missing
```

Assertions are **not** a replacement for proper error handling.

---

# Wrong Use

Example:

```c
assert(
fopen(...) != NULL);
```

Opening a file can fail because:

- File missing
- Permission denied
- Disk unavailable

These are runtime conditions,

not necessarily programmer errors.

Use normal error handling instead.

---

# Correct Use

Example:

```c
assert(node != NULL);
```

If your own code guarantees:

```
Node

↓

Must Exist
```

an assertion is appropriate.

---

# Real-World Example — Linux Kernel

The Linux kernel uses internal checking mechanisms (such as `BUG_ON()` and `WARN_ON()`),

which are conceptually similar to assertions,

to detect impossible conditions during development and debugging.

---

# Real-World Example — Database

Database engine:

```
Tree Root

↓

Must Exist
```

Assertion detects corrupted internal state immediately.

---

# Real-World Example — Game Engine

Physics engine:

```
Mass

↓

Must Be Positive
```

Assertion catches invalid object initialization.

---

# Real-World Example — Networking

Packet parser:

```
Header Length

↓

Must Match

Packet Size
```

Assertions verify internal assumptions while developing the parser.

---

# Common Mistakes

---

## Using Assertions for User Input

Wrong:

```c
assert(age >= 0);
```

if `age` comes directly from user input.

Users can make mistakes.

Handle invalid input gracefully.

---

## Ignoring Assertion Failures

An assertion failure indicates a bug or violated assumption.

Investigate and fix it.

---

## Putting Side Effects Inside Assertions

Wrong:

```c
assert(i++);
```

When assertions are disabled,

`i++` is not executed.

Avoid expressions with side effects.

---

## Assuming Assertions Replace Error Handling

Assertions validate internal assumptions.

They do not replace runtime error handling.

---

# Complete Example

```c
#include <assert.h>
#include <stdio.h>

void printValue(int *ptr)
{
    assert(ptr != NULL);

    printf("%d\n", *ptr);
}

int main()
{
    int x = 100;

    printValue(&x);

    return 0;
}
```

Output:

```
100
```

---

# Hands-on Labs

## Lab 1

Add assertions to validate:

- Non-null pointers
- Valid array indexes

---

## Lab 2

Create a function with a documented precondition.

Enforce it using `assert()`.

---

## Lab 3

Compile with and without:

```c
#define NDEBUG
```

Observe the behavior.

---

## Lab 4

Add assertions to a linked list implementation to verify internal consistency.

---

## Lab 5

Read the implementation of `assert` on your platform and compare it with the conceptual behavior shown here.

---

# Interview Questions

### What is an assertion?

A runtime check that verifies an internal assumption made by the programmer.

---

### What happens when an assertion fails?

The program prints diagnostic information and terminates.

---

### What does `NDEBUG` do?

It disables assertions by expanding `assert()` to nothing.

---

### Should assertions validate user input?

Generally, no.

User input requires normal error handling.

---

### What is the difference between assertions and error handling?

Assertions detect programmer mistakes.

Error handling manages expected runtime failures.

---

# Summary

```
Program

↓

assert(condition)

↓

Condition True?

↓

Continue

-----------------------

Condition False?

↓

Diagnostic Message

↓

Abort Program
```

Build Modes:

```
Debug

↓

Assertions Enabled

-----------------------

Release

↓

Assertions Disabled
```

## Key Takeaways

- Assertions verify assumptions that should always hold true.
- They are valuable for catching bugs early during development.
- `assert()` is typically enabled in debug builds and disabled in release builds using `NDEBUG`.
- Assertions should not replace proper runtime error handling.
- Avoid side effects inside assertion expressions.
- Well-placed assertions improve code quality, documentation, and maintainability.

---

# Next Chapter

## Chapter 99 — `volatile` Keyword

You'll learn:

- What `volatile` really means
- Compiler optimizations
- Memory-mapped I/O
- Hardware registers
- Signal handlers
- Multithreading misconceptions
- When `volatile` should and should not be used