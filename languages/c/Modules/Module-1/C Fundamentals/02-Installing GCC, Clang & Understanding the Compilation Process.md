# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 2 --- Installing GCC, Clang & Understanding the Compilation Process

------------------------------------------------------------------------

## Most beginners think:

> "I write C code and my computer runs it."

But that's not what actually happens.

Your computer **cannot understand C**.

It only understands **machine code (binary instructions)**.

So when you write:

``` c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

The CPU cannot execute it directly.

Something must translate it first.

That translator is called a **compiler**.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What GCC is
-   What Clang is
-   Installing compilers
-   Source files
-   Header files
-   Compilation stages
-   Object files
-   Linking
-   Executables
-   Why compilation errors happen

------------------------------------------------------------------------

# Why Do We Need a Compiler?

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

The compiler converts your human-readable program into instructions your
processor can execute.

------------------------------------------------------------------------

# GCC

**GCC** stands for **GNU Compiler Collection**.

Originally it compiled only C.

Today it supports:

-   C
-   C++
-   Objective-C
-   Go
-   Fortran
-   Ada

GCC is the default compiler on many Linux systems.

------------------------------------------------------------------------

# Clang

Clang is part of the LLVM project.

Advantages:

-   Faster error messages
-   Excellent diagnostics
-   Modern architecture
-   Widely used on macOS

On macOS, `clang` is the default C compiler.

------------------------------------------------------------------------

# Installing GCC

## Ubuntu

``` bash
sudo apt update
sudo apt install build-essential
```

Verify:

``` bash
gcc --version
```

------------------------------------------------------------------------

## Fedora

``` bash
sudo dnf install gcc
```

------------------------------------------------------------------------

## Arch Linux

``` bash
sudo pacman -S gcc
```

------------------------------------------------------------------------

## macOS

Install Xcode Command Line Tools:

``` bash
xcode-select --install
```

Verify:

``` bash
clang --version
```

------------------------------------------------------------------------

# Your First Program

Create a file named:

``` text
hello.c
```

Contents:

``` c
#include <stdio.h>

int main() {
    printf("Hello, C!\n");
    return 0;
}
```

------------------------------------------------------------------------

# Compiling

``` bash
gcc hello.c -o hello
```

Meaning:

``` text
gcc
 │
 ├── hello.c  → input source file
 │
 └── -o hello → output executable
```

Run:

Linux/macOS

``` bash
./hello
```

Output

``` text
Hello, C!
```

------------------------------------------------------------------------

# Compilation Stages

Many developers think compilation is one step.

Actually it has four major stages.

``` text
Source Code
      │
      ▼
Preprocessor
      │
      ▼
Compiler
      │
      ▼
Assembler
      │
      ▼
Linker
      │
      ▼
Executable
```

------------------------------------------------------------------------

# Stage 1 --- Preprocessor

Processes:

-   `#include`
-   `#define`
-   Conditional compilation

Example:

``` c
#include <stdio.h>
```

The header contents are inserted before compilation.

Generate only the preprocessed output:

``` bash
gcc -E hello.c
```

------------------------------------------------------------------------

# Stage 2 --- Compiler

The compiler converts C into assembly.

``` bash
gcc -S hello.c
```

Produces:

``` text
hello.s
```

------------------------------------------------------------------------

# Stage 3 --- Assembler

Assembly becomes object code.

``` bash
gcc -c hello.c
```

Produces:

``` text
hello.o
```

Object files contain machine code but cannot yet run.

------------------------------------------------------------------------

# Stage 4 --- Linker

The linker combines:

-   Your object files
-   Standard libraries

``` text
hello.o
     │
     ▼
Linker
     │
     ▼
Executable
```

Without linking, functions like `printf()` cannot be resolved.

------------------------------------------------------------------------

# What is an Executable?

An executable is a file the operating system can load into memory and
run.

Linux:

``` text
./hello
```

Windows:

``` text
hello.exe
```

------------------------------------------------------------------------

# Common Errors

## Missing Semicolon

``` c
int a = 10
```

Compiler:

``` text
expected ';'
```

------------------------------------------------------------------------

## Missing Header

``` c
printf("Hello");
```

without

``` c
#include <stdio.h>
```

The compiler cannot properly recognize `printf()`.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Check GCC

``` bash
gcc --version
```

## Lab 2

Generate Assembly

``` bash
gcc -S hello.c
```

Inspect `hello.s`.

## Lab 3

Generate Object File

``` bash
gcc -c hello.c
```

List files:

``` bash
ls
```

Observe `hello.o`.

## Lab 4

Compile and Run

``` bash
gcc hello.c -o hello
./hello
```

------------------------------------------------------------------------

# Interview Questions

### What is GCC?

GNU Compiler Collection used to compile C and several other languages.

### What is Clang?

A modern compiler built on LLVM with excellent diagnostics.

### Name the compilation stages.

1.  Preprocessor
2.  Compiler
3.  Assembler
4.  Linker

### What is an object file?

Machine code produced by the assembler before linking.

### Why is linking required?

To combine object files with required libraries and produce a runnable
executable.

------------------------------------------------------------------------

# Summary

``` text
hello.c
   │
   ▼
Preprocessor
   │
   ▼
Compiler
   │
   ▼
Assembler
   │
   ▼
Object File
   │
   ▼
Linker
   │
   ▼
Executable
   │
   ▼
CPU
```

Key Takeaways:

-   CPUs execute machine code, not C.
-   GCC and Clang translate C into executable programs.
-   Compilation has four distinct stages.
-   Object files are intermediate outputs.
-   Linking creates the final executable.

------------------------------------------------------------------------

# Next Chapter

## Chapter 3 --- Program Structure

You'll learn:

-   `main()`
-   Header files
-   Comments
-   Source files
-   Program execution flow
-   How the operating system starts a C program
