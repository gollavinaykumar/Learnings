# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 3 --- Program Structure

------------------------------------------------------------------------

## Most beginners think:

> "A C program is just a few lines of code."

But every C program follows a structure that the compiler and operating
system expect.

Understanding that structure makes it much easier to read large
codebases like the Linux kernel.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a C source file is
-   What `main()` does
-   Why header files exist
-   What comments are
-   How execution begins
-   How the operating system starts your program

------------------------------------------------------------------------

# Your First Complete Program

``` c
#include <stdio.h>

int main(void)
{
    printf("Hello, C!\n");
    return 0;
}
```

Although this program is small, every line has a purpose.

------------------------------------------------------------------------

# Breaking It Down

``` text
#include <stdio.h>   → Include header

int main(void)       → Program entry point

{                     → Start block

printf(...)          → Function call

return 0;            → Exit status

}                     → End block
```

------------------------------------------------------------------------

# Source File

A file ending in `.c` is called a **source file**.

Examples:

``` text
main.c
server.c
memory.c
network.c
```

Large applications contain hundreds or thousands of source files.

------------------------------------------------------------------------

# Header Files

Header files usually end with `.h`.

Examples:

``` text
stdio.h
stdlib.h
string.h
unistd.h
```

Headers declare functions, types and constants that other files can use.

Think of a header as a **table of contents** for a library.

------------------------------------------------------------------------

# What Does #include Do?

``` c
#include <stdio.h>
```

During preprocessing, the compiler inserts the contents of the header
before compiling.

``` text
main.c
   │
   ▼
Preprocessor
   │
   ▼
main.c + stdio.h
```

------------------------------------------------------------------------

# The main() Function

Every normal C program starts from `main()`.

``` c
int main(void)
{
    return 0;
}
```

The operating system calls `main()` after loading your executable.

``` text
Executable
    │
    ▼
Operating System
    │
    ▼
main()
```

------------------------------------------------------------------------

# Why Return 0?

`return 0;` tells the operating system the program completed
successfully.

``` text
0   → Success

Non-zero → Error
```

Shell scripts often check this value.

------------------------------------------------------------------------

# Comments

Single-line:

``` c
// Print greeting
```

Multi-line:

``` c
/*
This program
prints Hello.
*/
```

Comments are ignored by the compiler.

They are for humans.

------------------------------------------------------------------------

# Blocks

Curly braces define a block.

``` c
if (x > 0)
{
    printf("Positive");
}
```

Variables created inside a block usually exist only inside that block.

------------------------------------------------------------------------

# Execution Flow

``` text
Program Starts
      │
      ▼
main()
      │
      ▼
Statement 1
      │
      ▼
Statement 2
      │
      ▼
return
      │
      ▼
Program Ends
```

Unless control statements change the flow, C executes statements from
top to bottom.

------------------------------------------------------------------------

# Real Project Structure

``` text
project/
│
├── include/
│   └── server.h
│
├── src/
│   ├── main.c
│   ├── server.c
│   └── network.c
│
└── Makefile
```

This layout is common in open-source C projects.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Create:

``` text
main.c
```

Compile:

``` bash
gcc main.c -o app
```

Run:

``` bash
./app
```

------------------------------------------------------------------------

## Lab 2

Remove:

``` c
return 0;
```

Compile and observe that modern compilers still allow `main()` to end
successfully.

------------------------------------------------------------------------

## Lab 3

Open the preprocessed output.

``` bash
gcc -E main.c
```

Notice how `stdio.h` is expanded.

------------------------------------------------------------------------

# Interview Questions

### Where does every C program begin?

At the `main()` function.

### What is a header file?

A file containing declarations that can be shared by multiple source
files.

### What does `return 0;` mean?

It tells the operating system that the program finished successfully.

### Are comments compiled?

No. The compiler ignores comments.

------------------------------------------------------------------------

# Summary

``` text
Source File (.c)
       │
       ▼
Header Files
       │
       ▼
Compilation
       │
       ▼
Executable
       │
       ▼
Operating System
       │
       ▼
main()
       │
       ▼
Program Executes
```

Key Takeaways:

-   Every C program starts from `main()`.
-   Header files provide declarations.
-   Source files contain implementations.
-   Comments improve readability.
-   The operating system transfers control to `main()` after loading the
    executable.

------------------------------------------------------------------------

# Next Chapter

## Chapter 4 --- Variables, Constants, Scope & Lifetime

You'll learn:

-   Variables
-   Constants
-   Keywords
-   Scope
-   Lifetime
-   Stack variables
-   Global variables
-   Static variables
