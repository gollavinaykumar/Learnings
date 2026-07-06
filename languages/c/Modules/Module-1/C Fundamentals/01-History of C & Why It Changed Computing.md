# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 1 --- History of C & Why It Changed Computing

------------------------------------------------------------------------

## Most developers think:

> "C is just an old programming language."

But that's not true.

If C had never existed, much of today's computing infrastructure would
look very different.

Almost every layer of a modern computer depends on software written in
C.

``` text
Power Button
      │
      ▼
Firmware (BIOS / UEFI)
      │
      ▼
Bootloader
      │
      ▼
Linux Kernel
      │
      ▼
Device Drivers
      │
      ▼
System Libraries
      │
      ▼
Applications
```

Linux, Git, Redis, SQLite, PostgreSQL, Nginx, OpenSSL, CPython, Docker's
low-level components and countless embedded devices all rely heavily on
C.

Learning C is not just learning a programming language.

You are learning how computers actually work.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why C was created
-   Problems with Machine Code
-   Problems with Assembly Language
-   Why portability matters
-   What a compiler does
-   Why Unix was rewritten in C
-   Why Linux uses C
-   Why databases and servers use C
-   How C fits into your systems programming journey

------------------------------------------------------------------------

# Before C

Early programmers wrote instructions directly in binary.

``` text
01001010
11100101
10101010
```

This was extremely difficult to read and maintain.

Even one wrong bit could completely change program behavior.

------------------------------------------------------------------------

# Assembly Language

Assembly replaced binary with human-readable instructions.

``` asm
MOV AX, 10
ADD AX, 5
SUB AX, 2
```

This was much easier than binary, but Assembly had a major limitation.

Assembly is processor-specific.

Code written for Intel CPUs does not run on ARM CPUs.

``` text
Intel Assembly
      │
      ▼
Intel CPU ✔

ARM CPU ✘
PowerPC ✘
```

Developers often had to rewrite software for each processor family.

------------------------------------------------------------------------

# The Birth of C

In the early 1970s, Dennis Ritchie at Bell Labs created C while working
on Unix.

The goal was simple:

-   Keep the speed of Assembly.
-   Make programs easier to write.
-   Allow software to run on different processors with only a new
    compiler.

This idea transformed software engineering.

------------------------------------------------------------------------

# What is a Compiler?

A compiler translates C into machine code.

``` text
C Source Code
      │
      ▼
Compiler
      │
      ▼
Machine Code
      │
      ▼
CPU
```

The CPU understands only machine instructions.

Humans write C.

The compiler connects the two.

------------------------------------------------------------------------

# Why Unix Was Rewritten in C

Originally Unix was written mostly in Assembly.

Moving Unix to another processor required rewriting large amounts of
code.

After being rewritten in C:

``` text
Unix Source Code
      │
      ▼
Compiler
      │
      ▼
Intel CPU

or

ARM CPU

or

RISC-V CPU
```

The same source code became portable.

------------------------------------------------------------------------

# Why Linux Uses C

The Linux kernel must:

-   Manage memory
-   Schedule processes
-   Handle interrupts
-   Manage filesystems
-   Control hardware
-   Process network packets

These tasks require:

-   Direct memory access
-   Predictable execution
-   Minimal runtime overhead
-   Excellent performance

C provides all of these.

------------------------------------------------------------------------

# C vs High-Level Languages

Python:

``` python
numbers = [1,2,3]
```

Memory allocation is hidden.

C:

``` c
int numbers[3] = {1,2,3};
```

or

``` c
int *numbers = malloc(3 * sizeof(int));
```

You decide where memory comes from and when it is released.

This level of control is why operating systems prefer C.

------------------------------------------------------------------------

# Real Software Written in C

  Software       Why C?
  -------------- -----------------------------
  Linux Kernel   Hardware control
  Git            Speed
  Redis          Performance
  SQLite         Small footprint
  PostgreSQL     Efficient storage
  Nginx          High-performance networking
  OpenSSL        Cryptography
  CPython        Interpreter implementation

------------------------------------------------------------------------

# Where You'll Use C

``` text
C
│
├── Memory
├── Pointers
├── Processes
├── Threads
├── Networking
├── Linux Internals
├── Linux Kernel
├── Docker
├── Kubernetes
└── eBPF
```

Every advanced systems topic builds on C.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

``` bash
gcc --version
```

## Lab 2

``` bash
clang --version
```

## Lab 3

``` bash
which gcc
```

## Lab 4

``` bash
uname -m
```

Observe your compiler and CPU architecture.

------------------------------------------------------------------------

# Interview Questions

## Why was C created?

To provide Assembly-like performance while making large software
projects portable and easier to maintain.

## Who created C?

Dennis Ritchie at Bell Labs.

## Why does Linux use C?

Because it offers direct hardware access, predictable performance and
efficient memory management.

## What is a compiler?

Software that translates C source code into machine code.

------------------------------------------------------------------------

# Summary

``` text
Machine Code
      │
      ▼
Assembly
      │
      ▼
Problems with Portability
      │
      ▼
C Language
      │
      ▼
Unix
      │
      ▼
Linux
      │
      ▼
Modern Systems Software
```

Key Takeaways:

-   C solved the portability problem.
-   Compilers convert C into machine code.
-   Operating systems require low-level control.
-   Modern infrastructure still depends heavily on C.

------------------------------------------------------------------------

# Next Chapter

**Chapter 2 --- Installing GCC, Clang & Understanding the Compilation
Process**

You'll learn:

-   GCC
-   Clang
-   Compilation stages
-   Object files
-   Linking
-   Executables
