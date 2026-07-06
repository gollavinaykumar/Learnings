# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 97 — Conditional Compilation (`#if`, `#ifdef`, `#ifndef`)

---

# Most beginners think:

> "An `if` statement decides whether code runs."

Example:

```c
if (x > 10)
{
    printf("Hello");
}
```

Correct.

But that decision happens:

```
Program

↓

Already Compiled

↓

Running
```

Sometimes we don't even want the compiler to **see** certain code.

Example:

```
Linux Code

↓

Compile On Linux Only
```

or

```
Debug Code

↓

Compile Only During Development
```

This is called:

```
Conditional Compilation
```

The decision happens **before compilation**.

---

# Learning Objectives

After this chapter you will understand:

- Compile-time decisions
- `#if`
- `#ifdef`
- `#ifndef`
- `#elif`
- `#else`
- Include guards
- Cross-platform programming

---

# Runtime vs Compile Time

Runtime:

```c
if (debug)
{
    printf("Debug");
}
```

Flow:

```
Compile

↓

Run

↓

Check Condition

↓

Execute
```

Compile-time:

```c
#ifdef DEBUG
printf("Debug");
#endif
```

Flow:

```
Preprocessor

↓

Keep Or Remove Code

↓

Compiler

↓

Executable
```

The executable may not even contain the debug code.

---

# `#ifdef`

Means:

```
If Defined
```

Example:

```c
#define DEBUG

#ifdef DEBUG
printf("Debug Mode\n");
#endif
```

After preprocessing:

```c
printf("Debug Mode\n");
```

---

# Without Definition

Program:

```c
#ifdef DEBUG
printf("Debug");
#endif
```

If:

```
DEBUG

↓

Not Defined
```

Result:

```
Entire Block Removed
```

Compiler never sees it.

---

# Visualization

Before:

```
Source Code

↓

#ifdef DEBUG

↓

Code

↓

#endif
```

If DEBUG exists:

```
Code Remains
```

Otherwise:

```
Code Removed
```

---

# `#ifndef`

Means:

```
If Not Defined
```

Example:

```c
#ifndef DEBUG
printf("Release Build\n");
#endif
```

If:

```
DEBUG

↓

Missing
```

The code is included.

---

# `#if`

Allows expressions.

Example:

```c
#define VERSION 2

#if VERSION >= 2
printf("New Version");
#endif
```

Preprocessor evaluates:

```
VERSION >= 2

↓

True
```

The block remains.

---

# `#elif`

Example:

```c
#if VERSION == 1

printf("V1");

#elif VERSION == 2

printf("V2");

#else

printf("Unknown");

#endif
```

Similar to:

```
if

else if

else
```

But evaluated by the preprocessor.

---

# `#else`

Example:

```c
#ifdef DEBUG

printf("Debug");

#else

printf("Release");

#endif
```

Only one branch is compiled.

---

# Include Guards

Suppose:

```
main.c

↓

Includes

student.h

↓

Includes

student.h Again
```

Without protection,

the compiler sees the same declarations multiple times.

---

# Problem

```c
struct Student
{
    int age;
};
```

Included twice:

```
Structure Defined

↓

Again

↓

Compilation Error
```

---

# Solution

```c
#ifndef STUDENT_H

#define STUDENT_H

struct Student
{
    int age;
};

#endif
```

Flow:

First include:

```
STUDENT_H

↓

Not Defined

↓

Define It

↓

Include Content
```

Second include:

```
STUDENT_H

↓

Already Defined

↓

Skip Content
```

---

# Visualization

```
First Include

↓

Header Loaded

↓

Macro Defined

----------------

Second Include

↓

Macro Exists

↓

Ignore Header
```

---

# Platform-Specific Code

Example:

```c
#ifdef _WIN32

/* Windows */

#endif

#ifdef __linux__

/* Linux */

#endif

#ifdef __APPLE__

/* macOS */

#endif
```

One source file,

multiple operating systems.

---

# Debug Builds

Example:

```c
#ifdef DEBUG

printf("x=%d\n", x);

#endif
```

Development:

```
Compile With DEBUG

↓

Extra Logs
```

Production:

```
No DEBUG

↓

Logs Removed
```

No runtime overhead.

---

# Compile-Time Feature Flags

Suppose:

```
Application

↓

Database

↓

Logging

↓

Encryption
```

Need different builds.

Example:

```c
#ifdef ENABLE_LOGGING
```

Compile:

```
With Logging
```

or

```
Without Logging
```

using the same source code.

---

# Command-Line Definitions

You don't need:

```c
#define DEBUG
```

inside the source file.

Compile:

```bash
gcc -DDEBUG main.c
```

Equivalent to:

```c
#define DEBUG
```

before preprocessing.

---

# Removing Code Completely

Consider:

```c
#ifdef DEBUG

expensive_check();

#endif
```

Release build:

```
expensive_check()

↓

Removed
```

No CPU time,

no binary size,

no runtime cost.

---

# Real-World Example — Linux Kernel

Kernel supports:

- Thousands of hardware devices
- Multiple CPU architectures
- Optional debugging
- Different filesystems

Conditional compilation selects only the required code.

---

# Real-World Example — SQLite

SQLite enables or disables optional features at compile time using preprocessor macros.

One codebase,

many build configurations.

---

# Real-World Example — Embedded Systems

Small microcontrollers may disable:

- Logging
- Floating-point support
- Debugging

to reduce program size.

---

# Real-World Example — Networking Library

Compile with:

```
IPv4 Only

↓

or

IPv6

↓

or

Both
```

using compile-time feature flags.

---

# Common Mistakes

---

## Confusing `if` and `#if`

Runtime:

```c
if(...)
```

Compile-time:

```c
#if ...
```

Different stages of program execution.

---

## Forgetting Include Guards

Header included multiple times:

```
Compilation Errors
```

Always protect reusable headers.

---

## Using Undefined Macros Incorrectly

Prefer:

```c
#ifdef FEATURE
```

instead of assuming a macro exists.

---

## Placing `#define` in Many Files

Shared configuration should usually live in common headers or build-system definitions,

not be duplicated throughout the project.

---

# Complete Example

```c
#include <stdio.h>

#define DEBUG

int main()
{
#ifdef DEBUG
    printf("Debug Build\n");
#else
    printf("Release Build\n");
#endif

    return 0;
}
```

Output:

```
Debug Build
```

If `DEBUG` is removed:

```
Release Build
```

---

# Hands-on Labs

## Lab 1

Build the same program with and without:

```bash
-DDEBUG
```

Observe the difference.

---

## Lab 2

Create a header file with include guards.

Include it multiple times.

Verify compilation succeeds.

---

## Lab 3

Create platform-specific messages using:

- `_WIN32`
- `__linux__`
- `__APPLE__`

---

## Lab 4

Enable and disable a logging system using conditional compilation.

---

## Lab 5

Use:

```bash
gcc -E
```

to inspect the preprocessor output.

Observe which code blocks remain.

---

# Interview Questions

### What is conditional compilation?

Selecting code to include or exclude before compilation.

---

### What does `#ifdef` do?

Compiles the following block only if the specified macro is defined.

---

### What is an include guard?

A preprocessor technique that prevents a header from being processed multiple times.

---

### What is the difference between `if` and `#if`?

`if` is evaluated at runtime.

`#if` is evaluated by the preprocessor before compilation.

---

### Why is conditional compilation useful?

It enables:

- Platform-specific code
- Debug builds
- Feature flags
- Smaller binaries
- Configurable software

---

# Summary

```
Source Code

↓

Preprocessor

↓

Conditional Compilation

↓

Compiler

↓

Executable
```

Common Directives:

```
#ifdef

#ifndef

#if

#elif

#else

#endif
```

Typical Uses:

```
Include Guards

↓

Debug Builds

↓

Platform Support

↓

Feature Flags
```

## Key Takeaways

- Conditional compilation happens before the compiler processes the code.
- `#ifdef`, `#ifndef`, and `#if` control which code is compiled.
- Include guards prevent duplicate header inclusion.
- Compile-time configuration is widely used for portability and feature management.
- Large systems like the Linux kernel rely heavily on conditional compilation.
- Understanding the preprocessor is essential for reading and maintaining professional C projects.

---

# Next Chapter

## Chapter 98 — Assertions (`assert.h`)

You'll learn:

- What assertions are
- Runtime sanity checks
- Detecting programmer errors
- Debug vs Release behavior
- `NDEBUG`
- Defensive programming
- Best practices for using assertions in production-quality software