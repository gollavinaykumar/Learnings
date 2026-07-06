# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 96 — Advanced Macros (`#define`)

---

# Most beginners think:

> "`#define` is just used to create constants."

Example:

```c
#define PI 3.14159
```

That's true...

but it is only about **5%** of what macros can do.

The C Preprocessor is much more powerful.

It can:

- Generate code
- Create generic functions
- Build logging systems
- Enable debugging
- Perform compile-time configuration
- Build entire APIs

The Linux kernel contains **thousands** of advanced macros.

Understanding macros is essential for reading professional C code.

---

# Learning Objectives

After this chapter you will understand:

- The C Preprocessor
- Object-like macros
- Function-like macros
- Multi-line macros
- Stringizing (`#`)
- Token pasting (`##`)
- Macro pitfalls
- Real-world usage

---

# What is the Preprocessor?

Before the compiler sees your code,

another program runs first.

```
Source Code

↓

Preprocessor

↓

Expanded Source Code

↓

Compiler

↓

Machine Code
```

The preprocessor performs simple text transformations.

It does **not** understand C syntax.

---

# Object-Like Macro

Example:

```c
#define PI 3.14159
```

Program:

```c
printf("%f", PI);
```

After preprocessing:

```c
printf("%f", 3.14159);
```

The compiler never sees:

```
PI
```

It only sees:

```
3.14159
```

---

# Another Example

```c
#define SIZE 100
```

Program:

```c
int arr[SIZE];
```

Becomes:

```c
int arr[100];
```

Pure text replacement.

---

# Function-Like Macros

Example:

```c
#define SQUARE(x) ((x) * (x))
```

Use:

```c
SQUARE(5)
```

Preprocessor output:

```c
((5) * (5))
```

Compiler then compiles the expanded code.

---

# Why Parentheses Matter

Wrong:

```c
#define SQUARE(x) x * x
```

Program:

```c
SQUARE(2 + 3)
```

Expands to:

```c
2 + 3 * 2 + 3
```

Result:

```
11
```

Expected:

```
25
```

---

# Correct Version

```c
#define SQUARE(x) \
((x) * (x))
```

Now:

```c
SQUARE(2 + 3)
```

Expands to:

```c
((2 + 3) *
 (2 + 3))
```

Result:

```
25
```

---

# Another Macro Problem

Example:

```c
SQUARE(i++)
```

Expands to:

```c
((i++) *
 (i++))
```

`i++` is evaluated twice.

This leads to undefined behavior because the variable is modified multiple times without sequencing.

Avoid passing expressions with side effects to macros like this.

---

# Multi-Line Macros

Use:

```
\
```

Example:

```c
#define PRINT(x) \
printf("Value: "); \
printf("%d\n", x);
```

The backslash tells the preprocessor:

```
Continue On Next Line
```

---

# Safer Multi-Line Macro

Professional style:

```c
#define PRINT(x)        \
do {                    \
    printf("%d\n", x);  \
} while(0)
```

Why?

It behaves like a single statement.

---

# Why `do { } while(0)`?

Suppose:

```c
if(flag)
    PRINT(x);
else
    ...
```

Without the wrapper,

multi-statement macros can break `if/else` syntax.

With:

```c
do {
    ...
} while(0)
```

the macro behaves safely as one statement.

---

# Stringizing (`#`)

Suppose:

```c
#define SHOW(x) \
printf(#x);
```

Use:

```c
SHOW(hello)
```

Expands to:

```c
printf("hello");
```

Notice:

```
Identifier

↓

String
```

Automatically.

---

# Another Example

```c
#define DEBUG(x) \
printf(#x " = %d\n", x);
```

Program:

```c
int age = 21;

DEBUG(age);
```

Output:

```
age = 21
```

Very useful for debugging.

---

# Token Pasting (`##`)

Suppose:

```c
#define MAKE(name) \
int var_##name
```

Use:

```c
MAKE(count);
```

Expands to:

```c
int var_count;
```

The preprocessor joins tokens together.

---

# Another Example

```c
#define FUNC(x) \
void func_##x()
```

Use:

```c
FUNC(test)
```

Expands to:

```c
void func_test()
```

Useful for generating repetitive code.

---

# Macro Expansion

Example:

```c
#define A 10

#define B A + 5
```

Program:

```c
B
```

Expansion:

```
B

↓

A + 5

↓

10 + 5
```

Macros can expand into other macros.

---

# Macros vs Functions

Macro:

```
Text Replacement
```

Function:

```
Real Function

↓

Call

↓

Return
```

Macros have no runtime call overhead,

but they also lack type checking.

---

# Real-World Example — Linux Kernel

Kernel logging:

```c
pr_info(...)
```

is implemented using macros that expand differently depending on configuration.

---

# Real-World Example — Assertions

```c
assert(x > 0);
```

is typically implemented using macros.

In release builds,

it may expand to nothing.

---

# Real-World Example — Debug Logging

```c
LOG(value);
```

May expand to:

```c
printf(...)
```

during development,

and to nothing in production.

---

# Real-World Example — Generic Containers

Macros generate repetitive code for:

- Lists
- Trees
- Hash tables

without requiring manual duplication.

---

# Common Mistakes

---

## Missing Parentheses

Wrong:

```c
#define MUL(a,b) a*b
```

Correct:

```c
#define MUL(a,b) \
((a) * (b))
```

---

## Side Effects

Wrong:

```c
MAX(i++, j++)
```

Arguments may be evaluated multiple times.

Prefer inline functions when possible.

---

## Missing `do { } while(0)`

Multi-statement macros should behave like a single statement.

---

## Expecting Type Safety

Macros perform text replacement.

They do not check types.

---

# Complete Example

```c
#include <stdio.h>

#define MAX(a,b) \
((a) > (b) ? (a) : (b))

int main()
{
    printf("%d\n",
           MAX(10,20));

    return 0;
}
```

Output:

```
20
```

---

# Hands-on Labs

## Lab 1

Implement macros for:

- MAX
- MIN
- ABS

Use proper parentheses.

---

## Lab 2

Create a debug macro using stringizing (`#`).

---

## Lab 3

Generate variable names using token pasting (`##`).

---

## Lab 4

Write a safe multi-line logging macro using:

```c
do {
} while(0)
```

---

## Lab 5

Run:

```bash
gcc -E main.c
```

Observe the preprocessor output and identify macro expansions.

---

# Interview Questions

### What is the C preprocessor?

A tool that performs text transformations before compilation.

---

### What is a function-like macro?

A macro that accepts parameters and expands into code.

---

### Why should macro parameters be wrapped in parentheses?

To avoid operator precedence problems after expansion.

---

### What does `#` do in a macro?

It converts a macro argument into a string literal.

---

### What does `##` do?

It concatenates two preprocessing tokens.

---

### Why use `do { } while(0)`?

To make multi-statement macros behave like a single statement in all contexts.

---

# Summary

```
Source Code

↓

Preprocessor

↓

Expanded Source

↓

Compiler
```

Useful Macro Features:

```
Constants

↓

Function-Like Macros

↓

Stringizing (#)

↓

Token Pasting (##)

↓

Multi-Line Macros
```

## Key Takeaways

- Macros are expanded before compilation.
- The preprocessor performs text replacement, not semantic analysis.
- Parentheses are essential in function-like macros.
- `#` converts arguments to strings.
- `##` joins tokens together.
- `do { } while(0)` makes multi-line macros safe.
- Advanced macros are heavily used in the Linux kernel and other large C projects.

---

# Next Chapter

## Chapter 97 — Conditional Compilation (`#if`, `#ifdef`, `#ifndef`)

You'll learn:

- Compile-time decisions
- Feature flags
- Debug vs Release builds
- Include guards
- Platform-specific code
- Cross-platform development
- How large projects support Windows, Linux, and macOS from one codebase