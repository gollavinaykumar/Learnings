# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 92 — Generic Programming with `void *`

---

# Most beginners think:

> "Every function should work with only one data type."

Example:

```c
void printInt(int x);
```

Works only for:

```
int
```

Need:

```
double
```

Write another function.

Need:

```
char
```

Write another function.

Need:

```
struct Student
```

Write another function.

Very quickly you end up writing almost identical code repeatedly.

Professional C programmers avoid this.

Instead,

they write **generic code**.

The secret is:

```
void *
```

---

# Learning Objectives

After this chapter you will understand:

- Why `void *` exists
- Generic programming
- Type-independent functions
- Type casting
- Generic swap
- Generic data structures
- Standard library usage

---

# Review

Normal pointer:

```c
int *p;
```

Means:

```
Pointer

↓

int
```

Another pointer:

```c
double *d;
```

Means:

```
Pointer

↓

double
```

Each pointer is tied to one specific type.

---

# What is `void *`?

`void *` means:

```
Pointer

↓

Unknown Type
```

The compiler knows:

```
Address Exists
```

But it does **not** know what data lives there.

---

# Visualization

Suppose:

```
0x1000
```

Memory:

```
10
```

Could be:

```
int
```

or

```
float
```

or

```
char
```

or

```
struct Student
```

`void *` simply stores:

```
Address
```

without assuming a data type.

---

# Example

```c
int x = 10;

void *p = &x;
```

Memory:

```
p

↓

0x1000
```

Compiler knows only:

```
Address
```

Not:

```
Integer
```

---

# Why Can't We Dereference It?

Wrong:

```c
printf("%d",
       *p);
```

Compiler error.

Why?

Because:

```
void *

↓

Unknown Type
```

Compiler doesn't know:

- Size
- Layout
- Meaning

---

# Type Casting

Tell the compiler:

```
Treat This As

↓

int *
```

Example:

```c
printf("%d",
*(int *)p);
```

Flow:

```
void *

↓

int *

↓

Dereference
```

Output:

```
10
```

---

# Another Example

```c
double pi = 3.14;

void *p = &pi;

printf("%.2f",
*(double *)p);
```

Output:

```
3.14
```

The same `void *` mechanism works for different data types.

---

# Why Does C Use `void *`?

Imagine writing:

```
Swap Integers
```

Need another function for:

```
Swap Doubles
```

Another for:

```
Swap Structures
```

Instead,

write one generic function.

---

# Generic Swap

```c
void swap(
    void *a,
    void *b,
    size_t size);
```

Now it works for:

- int
- float
- double
- char
- struct

Exactly the same function.

---

# How Does It Work?

Suppose:

```
Swap

↓

100 Bytes
```

Function copies:

```
Byte

↓

Byte

↓

Byte
```

It doesn't care what the bytes represent.

---

# Simplified Swap Example

```c
#include <string.h>

void swap(
    void *a,
    void *b,
    size_t size)
{
    char temp[size];

    memcpy(temp,
           a,
           size);

    memcpy(a,
           b,
           size);

    memcpy(b,
           temp,
           size);
}
```

The function works with any object type because it copies raw bytes.

---

# Generic Print?

Can we write:

```c
print(void *);
```

No.

Why?

Because the function doesn't know:

```
Integer?

Double?

Structure?

```

Generic functions usually require additional information,

such as:

- Size
- Callback function
- Type-specific handler

---

# `qsort()` Uses `void *`

Prototype:

```c
void qsort(
void *base,
size_t count,
size_t size,
int (*compare)(
const void *,
const void *));
```

Notice:

```
void *

↓

Array

Unknown Type
```

`qsort()` sorts:

- Integers
- Floats
- Structures
- Strings

using the same implementation.

---

# Comparison Callback

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

Inside:

```
void *

↓

int *

↓

Compare
```

---

# Generic Linked List

Suppose:

```
Node

↓

Data

↓

void *
```

The node can now store:

- Integer
- Structure
- String
- Anything else

Very flexible.

---

# Visualization

```
Node

↓

void *

↓

Student

----------------

Node

↓

void *

↓

Integer

----------------

Node

↓

void *

↓

String
```

One data structure,

many data types.

---

# Generic Stack

Instead of:

```
Stack Of Integers
```

Use:

```
void *
```

Now the same implementation supports:

- Files
- Network Packets
- Trees
- Structures

---

# Why Doesn't C Have Generics?

Modern languages provide:

```
Java

↓

Generics

C++

↓

Templates

Rust

↓

Generics
```

C predates these language features.

Instead,

it achieves generic programming using:

```
void *
```

plus callbacks and careful type casting.

---

# Real-World Example — Linux Kernel

Kernel data structures often store:

```
void *
```

allowing them to work with many object types.

---

# Real-World Example — GUI Toolkit

Widget:

```
Button

↓

User Data

↓

void *
```

The toolkit doesn't need to know what the user stores there.

---

# Real-World Example — Database

Database engines often attach:

```
void *

↓

Context
```

to callbacks and execution state.

The generic pointer can reference any internal structure.

---

# Real-World Example — Networking

Libraries frequently pass:

```
Connection Context

↓

void *
```

through callback APIs.

Applications decide what the pointer represents.

---

# Common Mistakes

---

## Dereferencing `void *`

Wrong:

```c
*p;
```

`void *` must first be converted to the correct pointer type.

---

## Casting to the Wrong Type

Suppose:

```
double
```

Memory.

Cast to:

```c
(int *)
```

The result is meaningless and leads to undefined behavior when accessed.

Always cast back to the original type.

---

## Forgetting the Object Size

Generic memory functions often require:

```c
sizeof(...)
```

Without the correct size,

memory copying becomes incorrect.

---

## Assuming `void *` Knows the Type

It doesn't.

`void *` stores only an address.

Type information must come from elsewhere.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    int x = 42;

    void *p = &x;

    printf("%d\n",
           *(int *)p);

    return 0;
}
```

Output:

```
42
```

---

# Hands-on Labs

## Lab 1

Store an:

- int
- double
- char

inside a `void *` (one at a time).

Recover each using the correct cast.

---

## Lab 2

Implement a generic swap function.

Use it to swap:

- Integers
- Doubles
- Structures

---

## Lab 3

Use `qsort()` to sort:

- Integers
- Structures

with custom comparison callbacks.

---

## Lab 4

Implement a generic linked list using:

```c
void *
```

for node data.

---

## Lab 5

Build a generic stack capable of storing different object types.

---

# Interview Questions

### What is `void *`?

A generic pointer type that can hold the address of any object.

---

### Can you dereference a `void *` directly?

No.

It must first be cast to the correct pointer type.

---

### Why does `qsort()` use `void *`?

To sort arrays of any data type using one implementation.

---

### What is generic programming in C?

Writing reusable code that works with multiple data types using techniques such as `void *` and callbacks.

---

### Does `void *` store type information?

No.

It stores only an address.

---

# Summary

```
void *

↓

Generic Address

↓

Cast

↓

Correct Pointer Type

↓

Use Data
```

Generic Function:

```
void *

↓

Any Object

↓

One Implementation

↓

Many Data Types
```

## Key Takeaways

- `void *` is C's generic pointer type.
- It can store the address of any object type.
- You must cast it back before dereferencing.
- Generic programming in C relies heavily on `void *` and callbacks.
- The C standard library (`qsort`, `bsearch`, etc.) uses `void *` extensively.
- `void *` enables reusable libraries, generic data structures, and flexible APIs.

---

# Next Chapter

## Chapter 93 — `typedef` (Advanced Usage)

You'll learn:

- Why `typedef` exists
- Simplifying complex declarations
- Function pointer typedefs
- Structure aliases
- API design
- Platform-independent types
- Best practices used in large C codebases