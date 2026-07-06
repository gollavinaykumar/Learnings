# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 104 — Building Generic Libraries in C

---

# Most beginners think:

> "A C program is just one `.c` file."

Professional software isn't built this way.

Examples:

- SQLite
- OpenSSL
- libcurl
- zlib
- OpenSSL
- Linux Kernel Subsystems

They are built as:

```
Reusable Libraries
```

A library should work for:

- Thousands of programs
- Millions of users
- Different operating systems
- Different compilers

Today we'll learn how professional C libraries are designed.

---

# Learning Objectives

After this chapter you will understand:

- Library architecture
- Public vs private APIs
- Header files
- Source files
- Generic programming
- Opaque data types
- Callbacks
- Versioning
- API design

---

# What is a Library?

Suppose you wrote:

```c
int add(int a, int b)
{
    return a + b;
}
```

Instead of copying this code into every project,

you package it.

```
Source Code

↓

Library

↓

Reuse Everywhere
```

---

# Library Structure

Typical layout:

```
mylib/

│

├── include/

│      mylib.h

│

├── src/

│      mylib.c

│

├── examples/

│

├── tests/

│

└── README.md
```

This structure is common in professional projects.

---

# Public API

Users should only see:

```
include/

↓

mylib.h
```

Example:

```c
int add(
int,
int);
```

This is the public interface.

---

# Private Implementation

Hidden inside:

```
src/

↓

mylib.c
```

Example:

```c
int add(
int a,
int b)
{
    return a+b;
}
```

Users don't need to know how it works.

---

# API vs Implementation

Think of a TV.

You know:

```
Power Button

Volume

Channels
```

You don't need to know:

```
Circuit Board

↓

Transistors

↓

Wiring
```

Libraries work the same way.

---

# Header File

Example:

```c
#ifndef MYLIB_H

#define MYLIB_H

int add(
int,
int);

#endif
```

Contains:

- Function declarations
- Types
- Constants
- Documentation

---

# Source File

```c
#include "mylib.h"

int add(
int a,
int b)
{
    return a+b;
}
```

Contains:

```
Implementation
```

---

# Opaque Data Types

Professional libraries hide internal structures.

Header:

```c
typedef struct Vector
Vector;
```

Notice:

```
Structure

↓

Incomplete
```

User cannot access internal fields.

---

# Hidden Implementation

Inside:

```c
struct Vector
{
    int *data;

    size_t size;

    size_t capacity;
};
```

Only the library knows this.

---

# Why Hide Structures?

Suppose today:

```
Vector

↓

Array
```

Tomorrow:

```
Linked List
```

User code still works.

Because the implementation is hidden.

---

# Public Functions

Instead of:

```c
vector->size
```

Provide:

```c
size_t
vector_size(
Vector *);
```

Benefits:

- Encapsulation
- Easier maintenance
- Safer API

---

# Generic Programming

Instead of:

```
Integer Vector

Float Vector

String Vector
```

Professional libraries use:

```c
void *
```

One implementation.

Many data types.

---

# Generic Example

```c
void vector_push(
Vector *,
void *);
```

Can store:

- Integers
- Structures
- Strings
- Anything

---

# Callback Example

Suppose:

Need sorting.

Instead of hardcoding comparison:

```
Library

↓

User Callback
```

Example:

```c
sort(
data,
compare);
```

The library becomes flexible.

---

# Memory Ownership

One of the most important library concepts.

Question:

Who frees memory?

```
Library?

or

User?
```

Professional APIs clearly define ownership.

---

# Example

Library allocates:

```c
Vector *v =
vector_create();
```

User destroys:

```c
vector_destroy(v);
```

Ownership is explicit.

---

# Error Handling

Never silently fail.

Return:

```c
NULL
```

or

```c
Error Code
```

Example:

```c
Vector *

↓

NULL

↓

Allocation Failed
```

---

# Documentation

Professional APIs explain:

- Parameters
- Return values
- Ownership
- Errors
- Thread safety

Without documentation,

a library becomes difficult to use correctly.

---

# Versioning

Suppose:

Version 1

↓

```c
add()
```

Version 2

↓

Different parameters

Old programs may break.

Professional libraries maintain:

```
Backward Compatibility
```

whenever possible.

---

# Static Library

Build:

```
Source

↓

Object Files

↓

Archive

↓

libmylib.a
```

Program links against the archive at build time.

---

# Dynamic Library

Build:

```
libmylib.so

Linux

----------------

mylib.dll

Windows

----------------

libmylib.dylib

macOS
```

Loaded dynamically at runtime.

---

# Example Project

```
Calculator

↓

Library

↓

Many Applications

↓

Reuse Same Code
```

No duplication.

---

# Real-World Example — SQLite

Application uses:

```c
sqlite3_open();
```

Application never sees:

```
Internal B-Tree

↓

Pager

↓

Cache
```

Implementation is hidden.

---

# Real-World Example — OpenSSL

Public API:

```
SSL_connect()

SSL_read()

SSL_write()
```

Internal cryptographic structures remain private.

---

# Real-World Example — libcurl

Simple API:

```c
curl_easy_perform();
```

Internally:

- HTTP
- HTTPS
- FTP
- DNS
- TLS

All hidden behind a clean interface.

---

# Real-World Example — Linux Kernel

Kernel subsystems expose stable interfaces,

while hiding complex internal implementations.

Drivers interact with APIs,

not internal data structures.

---

# Common Mistakes

---

## Exposing Internal Structures

Wrong:

```c
struct Vector
{
...
};
```

inside the public header,

unless users genuinely need direct access.

---

## Mixing Public and Private Code

Keep:

```
include/

↓

Public

----------------

src/

↓

Private
```

Clearly separated.

---

## Forgetting Ownership Rules

Always document:

```
Who Allocates?

↓

Who Frees?
```

---

## No Error Reporting

Every public API should document failure conditions.

---

# Complete Example

### mylib.h

```c
#ifndef MYLIB_H
#define MYLIB_H

typedef struct Vector
Vector;

Vector *
vector_create(void);

void
vector_destroy(
Vector *);

#endif
```

---

### mylib.c

```c
struct Vector
{
    int *data;

    size_t size;
};

Vector *
vector_create(void)
{
    ...
}
```

User cannot access:

```
size

↓

Hidden
```

---

# Hands-on Labs

## Lab 1

Create a small reusable math library.

Separate:

- Header
- Source

---

## Lab 2

Implement an opaque `Stack` type.

Expose only public functions.

---

## Lab 3

Create a generic vector using:

```c
void *
```

---

## Lab 4

Add callback support for sorting stored elements.

---

## Lab 5

Package the library as:

- Static library
- Shared library

Link it from another program.

---

# Interview Questions

### What is a library?

A reusable collection of functions and data types packaged for use by multiple programs.

---

### Why separate header and source files?

Headers expose the public API.

Source files contain the implementation.

---

### What is an opaque data type?

A type whose internal representation is hidden from library users.

---

### Why use callbacks in libraries?

To allow user-defined behavior without changing the library implementation.

---

### Why is ownership documentation important?

It prevents memory leaks, double frees, and API misuse.

---

# Summary

```
Application

↓

Library API

↓

Library Implementation

↓

Operating System
```

Professional Library Layout:

```
include/

↓

Public API

-------------------

src/

↓

Private Code

-------------------

tests/

-------------------

examples/
```

## Key Takeaways

- Professional C libraries separate interfaces from implementations.
- Public headers define the API; source files implement it.
- Opaque data types hide internal details and improve maintainability.
- `void *`, callbacks, and `typedef` enable generic, reusable libraries.
- Clearly documenting ownership and errors is essential for robust APIs.
- This design philosophy is used by SQLite, OpenSSL, libcurl, and many other widely used C libraries.

---

# 🎉 Module 11 Complete!

You have now mastered the core advanced features of C, including:

- ✅ Function Pointers
- ✅ Callback Functions
- ✅ Generic Programming (`void *`)
- ✅ Advanced `typedef`
- ✅ Bit Manipulation
- ✅ Bit Fields
- ✅ Advanced Macros
- ✅ Conditional Compilation
- ✅ Assertions
- ✅ `volatile`
- ✅ `const`
- ✅ `restrict`
- ✅ Inline Functions
- ✅ Variable Argument Functions
- ✅ Professional Library Design

These concepts form the foundation for reading and writing production-quality systems software.

---

# 🚀 Next Module

# Module 12 — C Meets the Operating System

You'll learn how C interacts directly with the operating system, including:

- Process Creation (`fork`)
- Program Execution (`exec`)
- Process Termination (`exit`)
- Waiting for Child Processes (`wait`, `waitpid`)
- Process IDs (PID, PPID)
- Environment Variables
- Signals
- Pipes
- File Descriptors
- System Calls
- Process Memory
- Shell Internals
- Building a Mini Shell

This is the point where C transitions from being just a programming language to becoming the language used to communicate directly with the operating system.