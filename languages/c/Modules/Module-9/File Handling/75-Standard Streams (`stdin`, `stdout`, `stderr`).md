# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 75 — Standard Streams (`stdin`, `stdout`, `stderr`)

---

# Most beginners think:

> "`printf()` prints directly to the screen."

It doesn't.

This is one of the most important concepts in Linux and Unix.

When you write:

```c
printf("Hello");
```

your program is **not talking to the monitor**.

Instead, it writes to something called:

```
stdout
```

Similarly,

when you use:

```c
scanf()
```

your program is **not reading directly from the keyboard**.

It reads from:

```
stdin
```

And when an error occurs,

it should be written to:

```
stderr
```

These three streams are the foundation of every Unix-like operating system.

Without them,

there would be:

- No Linux Terminal
- No Pipes (`|`)
- No Redirection (`>`)
- No Shell Scripting
- No Docker Logs
- No CI/CD Terminal Output

---

# Learning Objectives

After this chapter you will understand:

- What Standard Streams are
- `stdin`
- `stdout`
- `stderr`
- Why they exist
- Linux file descriptors
- Redirection
- Pipes
- Real-world examples

---

# Every Program Starts With Three Streams

When a program starts,

the Operating System automatically creates three streams.

```
Program

↓

stdin

stdout

stderr
```

Every C program already has access to them.

You don't need to create them.

---

# The Three Standard Streams

| Stream | Purpose |
|---------|---------|
| `stdin` | Input |
| `stdout` | Normal Output |
| `stderr` | Error Output |

---

# `stdin`

```
Standard Input
```

Default source:

```
Keyboard
```

Example:

```c
int age;

scanf("%d", &age);
```

Flow:

```
Keyboard

↓

stdin

↓

scanf()

↓

Program
```

Notice:

`scanf()` reads from:

```
stdin
```

not directly from the keyboard.

---

# `stdout`

```
Standard Output
```

Default destination:

```
Terminal
```

Example:

```c
printf("Hello");
```

Flow:

```
Program

↓

printf()

↓

stdout

↓

Terminal
```

Again,

the program doesn't know anything about monitors.

It simply writes to:

```
stdout
```

---

# `stderr`

```
Standard Error
```

Used for:

- Error Messages
- Warnings
- Debug Information

Example:

```c
fprintf(stderr,
        "File not found\n");
```

Flow:

```
Program

↓

stderr

↓

Terminal
```

---

# Why Separate `stdout` and `stderr`?

Imagine running:

```bash
myprogram
```

Output:

```
Loading...

Loading...

Loading...

ERROR

Loading...
```

Suppose you want to save only the normal output.

If errors and normal output were mixed together,

that would be difficult.

Having separate streams solves this problem.

---

# Linux File Descriptors

Internally,

Linux identifies these streams using file descriptors.

| File Descriptor | Stream |
|----------------|--------|
| 0 | `stdin` |
| 1 | `stdout` |
| 2 | `stderr` |

These are created automatically when your process starts.

---

# Visualization

```
Process

↓

0

↓

stdin

----------------

1

↓

stdout

----------------

2

↓

stderr
```

Later,

we'll learn system calls like:

```c
read()

write()
```

which operate directly on these file descriptors.

---

# Standard Streams Are Just Files

This is one of the greatest Unix ideas.

Everything is treated like a file.

Examples:

```
Keyboard

↓

File
```

```
Terminal

↓

File
```

```
Printer

↓

File
```

```
Socket

↓

File
```

```
Pipe

↓

File
```

This design makes Unix extremely powerful.

---

# Output Redirection

Suppose:

```c
printf("Hello");
```

Normally:

```
stdout

↓

Terminal
```

But Linux allows:

```bash
./program > output.txt
```

Now:

```
stdout

↓

output.txt
```

Without changing your C program.

---

# Input Redirection

Suppose:

Program:

```c
scanf("%d",
      &age);
```

Normally:

```
Keyboard

↓

stdin
```

Now:

```bash
./program < input.txt
```

Flow:

```
input.txt

↓

stdin

↓

scanf()
```

The program thinks it's reading from the keyboard,

but it's actually reading from a file.

---

# Error Redirection

Suppose:

```c
fprintf(stderr,
        "Error\n");
```

Normally:

```
stderr

↓

Terminal
```

Redirect:

```bash
./program 2> errors.txt
```

Now:

```
stderr

↓

errors.txt
```

Normal output still appears on the screen.

---

# Redirect Everything

Command:

```bash
./program > output.txt 2> errors.txt
```

Result:

```
stdout

↓

output.txt

----------------

stderr

↓

errors.txt
```

Very common in production systems.

---

# Pipes

This is where Unix becomes amazing.

Suppose:

```bash
cat data.txt
```

Normally:

```
stdout

↓

Terminal
```

Now:

```bash
cat data.txt | sort
```

Flow:

```
cat

↓

stdout

↓

Pipe

↓

stdin

↓

sort

↓

stdout

↓

Terminal
```

One program's output becomes another program's input.

No temporary file required.

---

# C Doesn't Know About Pipes

Program:

```c
printf("Hello");
```

The program simply writes to:

```
stdout
```

Whether `stdout` is:

- Terminal
- File
- Pipe
- Network Connection

is decided by the Operating System.

The C program doesn't need to change.

---

# Using `fprintf()` with Standard Streams

Instead of:

```c
printf("Hello");
```

You can write:

```c
fprintf(stdout,
        "Hello");
```

Exactly the same result.

Similarly:

```c
fprintf(stderr,
        "Error");
```

---

# Real-World Example — Docker

Suppose a container prints:

```c
printf("Server Started");
```

Docker captures:

```
stdout
```

and stores it as container logs.

Error messages printed to:

```
stderr
```

are also captured separately.

---

# Real-World Example — GCC Compiler

Compile:

```bash
gcc main.c
```

Normal messages:

```
stdout
```

Compilation errors:

```
stderr
```

This separation allows build tools to process errors independently.

---

# Real-World Example — Linux Shell

Suppose:

```bash
ls
```

Output:

```
stdout
```

Now:

```bash
ls > files.txt
```

Without changing `ls`,

its output is redirected into a file.

---

# Common Mistakes

---

## Printing Errors with `printf()`

Wrong:

```c
printf("File not found");
```

Better:

```c
fprintf(stderr,
        "File not found\n");
```

Errors belong on `stderr`.

---

## Assuming `stdin` Means Keyboard

Normally yes,

but `stdin` could also come from:

- File
- Pipe
- Network
- Another Program

---

## Thinking `stdout` Always Means Screen

Not true.

It could point to:

- Terminal
- File
- Pipe
- Socket
- Log Collector

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    fprintf(stdout,
            "Program Started\n");

    fprintf(stderr,
            "Warning: Test Error\n");

    return 0;
}
```

Run normally:

```
Program Started

Warning: Test Error
```

Redirect:

```bash
./program > output.txt 2> errors.txt
```

Results:

```
output.txt

↓

Program Started
```

```
errors.txt

↓

Warning: Test Error
```

---

# Hands-on Labs

## Lab 1

Write a program using:

```c
fprintf(stdout,...)
```

instead of:

```c
printf()
```

---

## Lab 2

Print warnings using:

```c
stderr
```

Redirect them to a file.

---

## Lab 3

Read input using:

```bash
./program < input.txt
```

Observe that `scanf()` still works.

---

## Lab 4

Redirect normal output and errors into separate files.

---

## Lab 5

Use a pipe:

```bash
./program | sort
```

Observe how one program feeds another.

---

# Interview Questions

### What are the three standard streams?

- `stdin`
- `stdout`
- `stderr`

---

### Which file descriptor belongs to `stdout`?

```
1
```

---

### Which file descriptor belongs to `stderr`?

```
2
```

---

### Why is `stderr` separate from `stdout`?

So that errors can be handled or redirected independently of normal program output.

---

### Why are standard streams important?

They make input/output flexible by allowing programs to work with terminals, files, pipes, sockets, and other resources without changing the program logic.

---

# Summary

```
Program

↓

stdin

↓

Input

----------------

Program

↓

stdout

↓

Normal Output

----------------

Program

↓

stderr

↓

Errors
```

Linux File Descriptors:

```
0

↓

stdin

----------------

1

↓

stdout

----------------

2

↓

stderr
```

## Key Takeaways

- Every C program starts with three standard streams.
- `stdin` is used for input.
- `stdout` is used for normal output.
- `stderr` is used for errors.
- These streams are represented by file descriptors 0, 1, and 2.
- Redirection and pipes work because programs communicate through standard streams instead of directly with hardware.
- Standard streams are fundamental to Unix, Linux, Docker, shells, CI/CD pipelines, and system programming.

---

# 🎉 Module 9 Complete

Congratulations!

You now understand:

- ✅ `fopen()`
- ✅ `fread()`
- ✅ `fwrite()`
- ✅ `fprintf()`
- ✅ `fclose()`
- ✅ Text Files vs Binary Files
- ✅ File Position (`ftell()`, `fseek()`, `rewind()`)
- ✅ Standard Streams (`stdin`, `stdout`, `stderr`)

You now know how C programs communicate with the operating system to read files, write files, navigate within files, and interact with the terminal.

These concepts are the foundation for:

- Linux System Programming
- Database Engines
- Log Processing
- Shell Utilities
- File Systems
- Web Servers
- Compilers

---

# 🚀 Next Module

# Module 10 — Dynamic Memory Management

You'll learn:

- Stack vs Heap
- Memory Layout of a Process
- `malloc()`
- `calloc()`
- `realloc()`
- `free()`
- Memory Leaks
- Dangling Pointers
- Double Free
- Heap Internals
- How Linux manages process memory

By the end of Module 10, you'll understand one of the most important topics in systems programming and be ready to build data structures like linked lists, trees, hash tables, and memory-efficient applications.