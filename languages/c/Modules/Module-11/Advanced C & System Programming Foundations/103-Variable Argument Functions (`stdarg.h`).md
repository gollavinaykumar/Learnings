# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 103 — Variable Argument Functions (`stdarg.h`)

---

# Most beginners think:

> "Every function must receive a fixed number of arguments."

Example:

```c
int add(int a, int b)
{
    return a + b;
}
```

Works for:

```c
add(10, 20);
```

But what about:

```c
printf("Hello");

printf("%d", 10);

printf("%d %d", 10, 20);

printf("%d %d %d", 10, 20, 30);
```

How can **one function** accept:

- 1 argument
- 2 arguments
- 5 arguments
- 100 arguments

The answer is:

```
Variable Argument Functions
```

---

# Learning Objectives

After this chapter you will understand:

- Variable arguments
- `stdarg.h`
- `va_list`
- `va_start`
- `va_arg`
- `va_end`
- How `printf()` works
- Building your own logging functions

---

# Fixed Arguments

Example:

```c
int sum(int a, int b);
```

Compiler knows:

```
Argument 1

↓

int

Argument 2

↓

int
```

Exactly two arguments.

No more.

No less.

---

# Variable Arguments

Example:

```c
printf(...)
```

Compiler knows:

```
First Argument

↓

Format String
```

Everything after that:

```
Unknown Number

↓

Unknown Types
```

The function figures them out at runtime.

---

# The Ellipsis (`...`)

General syntax:

```c
void func(
int count,
...);
```

The three dots mean:

```
Additional Arguments

↓

Unknown Quantity
```

At least one named parameter is required before `...`.

---

# Why Is the First Parameter Needed?

Suppose:

```c
func(...)
```

How would the function know:

- Where the variable arguments begin?
- How many arguments exist?
- What types they are?

It wouldn't.

The named parameter helps interpret the remaining arguments.

---

# Example

```c
sum(
3,
10,
20,
30);
```

Meaning:

```
count

↓

3

Arguments

↓

10

20

30
```

The function uses:

```
count

↓

Know How Many
```

---

# `stdarg.h`

Include:

```c
#include <stdarg.h>
```

Provides:

```
va_list

va_start

va_arg

va_end
```

These macros allow access to variable arguments.

---

# `va_list`

Example:

```c
va_list args;
```

Think of it conceptually as:

```
Iterator

↓

Arguments
```

It keeps track of the current position within the variable argument list.

---

# `va_start`

Example:

```c
va_start(args, count);
```

Meaning:

```
Begin Reading

↓

Arguments

↓

After count
```

The second parameter must be the last named parameter.

---

# `va_arg`

Example:

```c
int value =
va_arg(args,
int);
```

Conceptually:

```
Read Next Argument

↓

Interpret As int

↓

Advance
```

Each call retrieves one argument of the specified type.

---

# `va_end`

When finished:

```c
va_end(args);
```

Conceptually:

```
Cleanup

↓

Finished
```

Always pair `va_start` with `va_end`.

---

# Complete Example

```c
#include <stdio.h>
#include <stdarg.h>

int sum(int count, ...)
{
    va_list args;

    va_start(args, count);

    int total = 0;

    for(int i = 0; i < count; i++)
    {
        total +=
        va_arg(args, int);
    }

    va_end(args);

    return total;
}

int main()
{
    printf("%d\n",
           sum(4,
               10,
               20,
               30,
               40));

    return 0;
}
```

Output:

```
100
```

---

# Visualization

Call:

```c
sum(
4,
10,
20,
30,
40);
```

Memory (conceptual):

```
count

↓

4

10

20

30

40
```

`va_start`

↓

```
First Variable Argument
```

`va_arg`

↓

```
10

↓

20

↓

30

↓

40
```

---

# How `printf()` Works

Call:

```c
printf(
"%d %f",
10,
3.14);
```

First parameter:

```
"%d %f"
```

Parser sees:

```
%d

↓

Read int

----------------

%f

↓

Read double
```

Internally,

`printf()` repeatedly uses mechanisms similar to `va_arg()`.

---

# Default Argument Promotions

This is an important concept.

Suppose:

```c
char c = 'A';

float f = 3.5f;
```

Passed through `...`

They become:

```
char

↓

int

----------------

float

↓

double
```

These are called:

```
Default Argument Promotions
```

Therefore:

```c
va_arg(args, char);
```

is wrong.

Use:

```c
va_arg(args, int);
```

Similarly,

retrieve `float` arguments as `double`.

---

# Why Type Safety Is Lost

Suppose:

```c
sum(
2,
10,
20);
```

Program expects:

```
Integers
```

But:

```c
sum(
2,
10,
3.14);
```

Nothing stops you.

The compiler cannot verify the types inside `...`.

Variable argument functions sacrifice some type safety.

---

# Real-World Example — Logging

Function:

```c
log(
"User %s logged in",
name);
```

Logging framework forwards arguments using `stdarg.h`.

---

# Real-World Example — Error Reporting

Example:

```c
error(
"File %s not found",
filename);
```

One function supports many message formats.

---

# Real-World Example — Database

Database API:

```
Execute Query

↓

Variable Parameters
```

Internally,

some database libraries use variadic interfaces for convenience.

---

# Real-World Example — Linux Kernel

The Linux kernel implements formatted printing functions such as:

```
printk()
```

which process variable argument lists similarly to `printf()`.

---

# Common Mistakes

---

## Forgetting `va_end`

Always call:

```c
va_end(args);
```

after finishing with a variable argument list.

---

## Reading the Wrong Type

Wrong:

```c
va_arg(args, float);
```

Because of default promotions,

retrieve floating-point arguments as:

```c
double
```

---

## Reading Too Many Arguments

Suppose:

```
count

↓

3
```

Reading:

```
4 Arguments
```

Results in undefined behavior.

---

## Expecting Type Checking

The compiler cannot verify the types inside `...`.

The caller and callee must agree on the argument types.

---

# Building a Logger

Example:

```c
void logMessage(
const char *fmt,
...)
```

Flow:

```
Format String

↓

Read Arguments

↓

Print

↓

Cleanup
```

This is the same general idea used by `printf()`.

---

# Hands-on Labs

## Lab 1

Implement a function that sums a variable number of integers.

---

## Lab 2

Build a simple logging function that forwards arguments to `vprintf()`.

---

## Lab 3

Experiment with passing:

- `char`
- `short`
- `float`

Observe default argument promotions.

---

## Lab 4

Deliberately read the wrong type with `va_arg()`.

Observe the incorrect behavior.

---

## Lab 5

Study the documentation for:

- `printf`
- `vprintf`
- `vsnprintf`

Understand how variadic functions and `va_list` interact.

---

# Interview Questions

### What does `...` mean in a function declaration?

The function accepts a variable number of arguments.

---

### What is `va_list`?

A type used to access variable arguments.

---

### What does `va_start` do?

Initializes access to the variable argument list.

---

### What does `va_arg` do?

Retrieves the next argument of the specified type.

---

### Why is `va_end` required?

It completes the processing of the variable argument list and should always be paired with `va_start`.

---

### Why is `printf()` a variadic function?

Because it accepts different numbers and types of arguments based on the format string.

---

# Summary

```
Function

↓

Named Parameters

↓

...

↓

Variable Arguments
```

Processing Flow:

```
va_list

↓

va_start

↓

va_arg

↓

va_arg

↓

...

↓

va_end
```

## Key Takeaways

- Variadic functions accept a variable number of arguments.
- `stdarg.h` provides the macros needed to process them.
- `va_start`, `va_arg`, and `va_end` work together to access arguments.
- `printf()` is the classic example of a variadic function.
- Default argument promotions affect how arguments must be retrieved.
- Variadic functions are powerful but require careful design because they lack compile-time type checking.

---

# Next Chapter

## Chapter 104 — Building Generic Libraries in C

You'll learn:

- Combining `void *`, callbacks, and `typedef`
- Designing reusable APIs
- Opaque data types
- Public vs private headers
- Library architecture
- Versioning
- How professional C libraries like OpenSSL, SQLite, and libcurl are designed