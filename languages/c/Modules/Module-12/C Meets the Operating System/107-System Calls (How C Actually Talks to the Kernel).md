# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 107 — System Calls (How C Actually Talks to the Kernel)

---

# Most beginners think:

> "`printf()` prints text on the screen."

Actually...

`printf()` **doesn't know how to control your monitor**.

It doesn't know:

- How your GPU works
- How your terminal works
- How your keyboard works
- How your SSD works

Instead,

it asks the operating system to do those things.

The communication channel between a program and the kernel is called a:

```
System Call
```

Every operating system service begins here.

---

# Learning Objectives

After this chapter you will understand:

- What system calls are
- Why they exist
- libc wrappers
- Kernel entry
- Kernel exit
- Trap instructions
- `read()`
- `write()`
- `open()`
- `close()`

---

# The Big Picture

Suppose your program wants to:

```
Read File
```

Can it talk directly to the SSD?

No.

Instead:

```
Application

↓

System Call

↓

Kernel

↓

Filesystem

↓

Disk Driver

↓

SSD
```

---

# What is a System Call?

A system call is:

> **A controlled request from a user-space program asking the kernel to perform a privileged operation.**

Examples:

```
Open File

↓

Read File

↓

Write File

↓

Allocate Memory

↓

Create Process

↓

Create Socket
```

All of these require kernel involvement.

---

# Why System Calls Exist

Imagine:

```
Browser

↓

Erase Disk
```

Or:

```
Game

↓

Disable Firewall
```

Without system calls,

applications could perform dangerous operations directly.

Instead:

```
Application

↓

Ask Kernel

↓

Kernel Checks Permission

↓

Perform Operation
```

---

# User Space → Kernel Space

Suppose:

```c
write(fd,
      buffer,
      length);
```

Flow:

```
User Mode

↓

System Call

↓

Kernel Mode

↓

Device Driver

↓

Hardware

↓

Return

↓

User Mode
```

This transition happens thousands of times every second.

---

# Visualization

```
+----------------------+
| User Space           |
| Your Program         |
+----------------------+
          │
          │ System Call
          ▼
+----------------------+
| Kernel Space         |
| Operating System     |
+----------------------+
          │
          ▼
+----------------------+
| Hardware             |
+----------------------+
```

---

# `printf()` Isn't a System Call

Many beginners believe:

```
printf()

↓

Kernel
```

Not quite.

Actually:

```
printf()

↓

C Standard Library

↓

write()

↓

System Call

↓

Kernel
```

`printf()` is a library function.

`write()` is the operating system interface.

---

# libc Wrappers

Your code:

```c
printf("Hello\n");
```

Library:

```
Format String

↓

Create Buffer

↓

write()
```

Kernel:

```
Display Output
```

The C library provides a convenient wrapper around system calls.

---

# Example

```c
#include <unistd.h>

int main()
{
    write(
        1,
        "Hello\n",
        6);

    return 0;
}
```

Output:

```
Hello
```

Here:

```
1

↓

Standard Output
```

No `printf()` involved.

---

# Why Use `printf()` Then?

Because it provides:

- Formatting
- Buffering
- `%d`
- `%f`
- `%s`

Eventually,

it still calls:

```
write()
```

---

# Common System Calls

Some important ones:

| System Call | Purpose |
|-------------|----------|
| `read()` | Read data |
| `write()` | Write data |
| `open()` | Open a file |
| `close()` | Close a file |
| `fork()` | Create a process |
| `execve()` | Execute a new program |
| `wait()` | Wait for a child process |
| `mmap()` | Map memory |
| `socket()` | Create a network socket |

These form the foundation of operating system programming.

---

# Example — `open()`

Program:

```c
open(
"notes.txt",
O_RDONLY);
```

Flow:

```
Application

↓

Kernel

↓

Filesystem

↓

Disk

↓

File Descriptor
```

---

# Example — `read()`

Program:

```c
read(
fd,
buffer,
100);
```

Flow:

```
Application

↓

Kernel

↓

Disk

↓

Memory Buffer

↓

Application
```

---

# Example — `close()`

Program:

```c
close(fd);
```

Kernel:

```
Release Resources

↓

Descriptor Closed
```

---

# How Does the CPU Enter the Kernel?

The CPU provides a special instruction

(such as `syscall`, `sysenter`, or another architecture-specific mechanism)

that safely transfers control to the kernel.

Conceptually:

```
Application

↓

Trap Instruction

↓

Kernel Entry

↓

Execute

↓

Return
```

---

# Kernel Entry

Conceptually:

```
User Mode

↓

Save CPU State

↓

Kernel Mode

↓

Run Requested Service
```

The operating system saves enough state so execution can safely resume later.

---

# Kernel Exit

After finishing:

```
Kernel

↓

Restore CPU State

↓

Return Value

↓

User Mode
```

Your program continues executing.

---

# Return Values

Suppose:

```c
fd = open(...);
```

Possible:

```
Success

↓

Positive File Descriptor

------------------------

Failure

↓

-1
```

The kernel communicates success or failure through return values (and related error mechanisms).

---

# Cost of a System Call

A normal function call:

```
Program

↓

Function

↓

Return
```

A system call:

```
User Mode

↓

Kernel Mode

↓

Security Checks

↓

Kernel Work

↓

Return
```

System calls are generally more expensive than ordinary function calls.

---

# Why Not Make Everything a System Call?

Suppose:

```c
x = y + z;
```

If every addition required entering the kernel,

computers would become extremely slow.

Only privileged operations require system calls.

Normal computation stays in user space.

---

# Real-World Example — Reading a File

Program:

```c
fread(...)
```

Internally:

```
fread()

↓

read()

↓

Kernel

↓

Filesystem

↓

SSD
```

---

# Real-World Example — Web Server

HTTP request arrives.

Server:

```
socket()

↓

accept()

↓

read()

↓

write()

↓

close()
```

Almost every networking operation relies on system calls.

---

# Real-World Example — Docker

Docker itself eventually invokes system calls such as:

- `clone()`
- `mount()`
- `setns()`

to create and manage containers.

---

# Real-World Example — Database

Database:

```
Read Page

↓

read()

↓

Kernel

↓

Disk
```

The operating system performs the actual I/O.

---

# Common Mistakes

---

## Thinking `printf()` Is a System Call

It is a C library function.

Eventually,

it uses system calls like `write()`.

---

## Ignoring Return Values

Always check:

```c
if(fd == -1)
```

System calls can fail.

---

## Confusing Library Functions with Kernel Services

Examples:

```
printf()

malloc()

fopen()
```

are library functions.

They may internally invoke one or more system calls.

---

## Assuming System Calls Are Cheap

They require:

- Mode switch
- Kernel execution
- Return to user mode

Minimizing unnecessary system calls can improve performance.

---

# Complete Example

```c
#include <unistd.h>

int main()
{
    write(
        1,
        "Hello Kernel\n",
        13);

    return 0;
}
```

Flow:

```
main()

↓

write()

↓

System Call

↓

Kernel

↓

Terminal

↓

Display
```

---

# Hands-on Labs

## Lab 1

Replace:

```c
printf()
```

with:

```c
write()
```

Observe the difference.

---

## Lab 2

Run:

```bash
strace ./program
```

(on Linux)

Identify:

- `write`
- `open`
- `close`

system calls.

---

## Lab 3

Open a file using:

```c
open()
```

Read it using:

```c
read()
```

Close it using:

```c
close()
```

---

## Lab 4

Observe the return values of failed system calls.

Print appropriate error messages.

---

## Lab 5

Read the manual pages:

```bash
man 2 write

man 2 read

man 2 open
```

Notice they are in **Section 2**, which documents system calls.

---

# Interview Questions

### What is a system call?

A controlled interface through which a user-space program requests services from the operating system kernel.

---

### Why are system calls needed?

They provide secure access to privileged operations such as file I/O, networking, process management, and memory management.

---

### Is `printf()` a system call?

No.

It is a C library function that eventually uses system calls such as `write()`.

---

### Why are system calls slower than normal function calls?

Because they involve switching from user mode to kernel mode, performing kernel work, and returning to user mode.

---

### Give five common Linux system calls.

- `read()`
- `write()`
- `open()`
- `close()`
- `fork()`

---

# Summary

```
Application

↓

Library Function

↓

System Call

↓

Kernel

↓

Hardware

↓

Kernel

↓

Application
```

Examples:

```
printf()

↓

write()

--------------------

fopen()

↓

open()

--------------------

fread()

↓

read()
```

## Key Takeaways

- System calls are the gateway between user programs and the operating system kernel.
- Library functions often provide higher-level abstractions over system calls.
- Every privileged operation ultimately requires kernel involvement.
- System calls involve a mode switch and are more expensive than ordinary function calls.
- Understanding system calls is fundamental to systems programming, networking, process management, and operating system internals.

---

# Next Chapter

## Chapter 108 — File Descriptors (Everything Is a File)

You'll learn:

- What file descriptors are
- Standard input, output, and error
- How the kernel tracks open files
- Why sockets, pipes, terminals, and files all use file descriptors
- File descriptor tables
- `dup()` and `dup2()`
- The Unix philosophy: **"Everything is a file."**