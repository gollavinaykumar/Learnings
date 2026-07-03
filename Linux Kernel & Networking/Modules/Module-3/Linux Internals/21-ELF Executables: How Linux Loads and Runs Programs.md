Most developers run programs like this:

```bash
./app
```

or

```bash
python app.py
```

or

```bash
node server.js
```

The program starts instantly.

But have you ever wondered:

- How does Linux know where execution begins?
- How does it load the program into memory?
- What are `.so` files?
- Why do programs depend on `libc.so`?
- What is dynamic linking?
- What is the ELF format?
- What does the kernel actually do after `exec()`?

The answer is:

**ELF (Executable and Linkable Format).**

Every compiled Linux program follows this format.

After this chapter, you'll understand exactly how Linux loads a program into memory and starts executing it.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 21 — ELF Executables: How Linux Loads and Runs Programs

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is ELF?
> - Why ELF exists
> - ELF Header
> - Sections
> - Segments
> - Program Loader
> - Static Linking
> - Dynamic Linking
> - Shared Libraries (`.so`)
> - Dynamic Linker (`ld-linux`)
> - Symbol Resolution
> - Relocation
> - Complete Program Execution Flow

---

# 📖 What Happens When You Run a Program?

Suppose you execute:

```bash
./hello
```

Most people imagine:

```
hello

↓

Running
```

In reality,

Linux performs many steps.

```
Executable File

↓

Kernel

↓

ELF Loader

↓

Memory Mapping

↓

Shared Libraries

↓

Program Starts
```

---

# What is ELF?

ELF stands for:

```
Executable

and

Linkable

Format
```

It is the standard executable format used by Linux and most Unix-like operating systems.

Examples:

```
Executable Programs

↓

ELF
```

```
Shared Libraries

↓

ELF
```

```
Object Files

↓

ELF
```

---

# Why Do We Need ELF?

Suppose Linux receives:

```
hello
```

How does it know:

- Where the code begins?
- Which libraries are required?
- Where global variables are stored?
- Which function executes first?

ELF stores all this information.

---

# ELF Architecture

```
ELF File

├── ELF Header

├── Program Headers

├── Sections

├── Symbol Table

├── String Table

└── Program Data
```

The kernel and linker read these structures.

---

# ELF Header

The ELF Header identifies the file.

Example:

```
ELF Header

↓

Architecture

↓

64-bit

↓

Entry Point

↓

Program Header Location
```

It tells Linux:

```
This is a valid ELF executable.
```

---

# Entry Point

Every executable has one address called:

```
Entry Point
```

Execution begins here.

```
Kernel

↓

Jump

↓

Entry Point

↓

Program Starts
```

---

# Sections

Sections organize information for the compiler and linker.

Examples:

```
.text

↓

Machine Code
```

```
.data

↓

Initialized Variables
```

```
.bss

↓

Uninitialized Variables
```

```
.rodata

↓

Read-Only Data
```

---

# Common ELF Sections

| Section | Purpose |
|----------|----------|
| `.text` | Executable Machine Code |
| `.data` | Initialized Global Variables |
| `.bss` | Uninitialized Variables |
| `.rodata` | Read-only Constants |
| `.symtab` | Symbol Table |
| `.strtab` | String Table |

---

# Sections vs Segments

This confuses many developers.

Sections are mainly for:

```
Compiler

↓

Linker
```

Segments are mainly for:

```
Operating System

↓

Program Loader
```

---

# Segments

When Linux loads a program,

it loads Segments,

not individual Sections.

Example:

```
Code Segment

↓

Memory
```

```
Data Segment

↓

Memory
```

```
Stack

↓

Memory
```

---

# Program Headers

Program Headers describe Segments.

```
Kernel

↓

Read Program Header

↓

Map Segment

↓

Continue
```

The kernel uses Program Headers during `exec()`.

---

# Program Loader

Suppose:

```bash
./hello
```

Linux performs:

```
Open ELF File

↓

Read ELF Header

↓

Read Program Headers

↓

Map Memory

↓

Load Libraries

↓

Jump to Entry Point
```

---

# Memory Layout After Loading

```
+----------------------+

Stack

+----------------------+

Heap

+----------------------+

Shared Libraries

+----------------------+

Data Segment

+----------------------+

Code Segment

+----------------------+
```

This becomes the process's virtual memory.

---

# Static Linking

Suppose your program needs:

```
printf()
```

Static Linking copies the implementation directly into the executable.

```
Program

+

Library

↓

Large Executable
```

Everything is self-contained.

---

# Advantages of Static Linking

✔ Runs independently

✔ No external libraries required

✔ Easy deployment

---

# Disadvantages of Static Linking

❌ Larger executable

❌ Duplicate library code

❌ Updating libraries requires rebuilding applications

---

# Dynamic Linking

Instead of copying libraries,

the executable stores references.

```
Program

↓

Needs libc.so
```

At runtime,

Linux loads the library.

---

# Shared Libraries

Linux uses files ending with:

```
.so

Shared Object
```

Examples:

```
libc.so

libpthread.so

libm.so

libssl.so
```

Multiple programs share the same library.

---

# Dynamic Linker

Who loads shared libraries?

```
ld-linux.so
```

The Dynamic Linker performs:

```
Locate Libraries

↓

Load Libraries

↓

Resolve Symbols

↓

Start Program
```

---

# Dynamic Linking Flow

```
Executable

↓

Kernel

↓

ld-linux

↓

Load Libraries

↓

Resolve Functions

↓

Program Starts
```

---

# Symbol Resolution

Suppose the program calls:

```c
printf()
```

Initially,

the executable only knows:

```
printf
```

The Dynamic Linker finds:

```
printf

↓

libc.so

↓

Memory Address
```

The call is now resolved.

---

# Relocation

Shared libraries may load at different addresses each time.

Relocation updates memory references accordingly.

```
Library

↓

Loaded

↓

Adjust Addresses

↓

Ready
```

---

# Lazy Binding

Linux doesn't always resolve every function immediately.

Instead:

```
Program Starts

↓

First printf()

↓

Resolve printf()

↓

Reuse Later
```

This speeds up program startup.

---

# Real Example

Suppose:

```bash
ls
```

Internally:

```
exec()

↓

Kernel

↓

Read ELF

↓

Load Segments

↓

Start ld-linux

↓

Load libc.so

↓

Resolve Symbols

↓

Entry Point

↓

main()

↓

Program Running
```

---

# Why Shared Libraries Matter

Suppose:

```
100 Programs
```

Need:

```
libc.so
```

Without shared libraries:

```
100 Copies
```

With shared libraries:

```
One Library

↓

Shared by Everyone
```

Huge memory savings.

---

# Why ELF Matters

Everything compiled on Linux depends on ELF.

Examples:

- Docker
- Kubernetes
- PostgreSQL
- Nginx
- Redis
- Java JVM
- Go Programs
- C/C++ Applications

Even the Linux kernel itself loads ELF user programs.

---

# Hands-on Lab

## Lab 1 — Identify an ELF File

```bash
file /bin/ls
```

Typical output:

```
ELF 64-bit executable
```

---

## Lab 2 — View ELF Header

```bash
readelf -h /bin/ls
```

Observe:

- Entry Point
- Machine Type
- ELF Class

---

## Lab 3 — View Program Headers

```bash
readelf -l /bin/ls
```

Observe memory segments.

---

## Lab 4 — View Sections

```bash
readelf -S /bin/ls
```

Observe:

```
.text

.data

.bss
```

---

## Lab 5 — View Shared Libraries

```bash
ldd /bin/ls
```

Observe:

```
libc.so

libselinux.so

...
```

---

## Lab 6 — View Symbols

```bash
nm /bin/ls
```

Observe exported and imported symbols.

---

# Interview Questions

## What is ELF?

ELF (Executable and Linkable Format) is the standard executable format used by Linux for executables, object files, and shared libraries.

---

## What is the ELF Header?

The ELF Header identifies the file and contains metadata such as architecture, entry point, and program header location.

---

## What is the difference between Sections and Segments?

Sections organize data for the compiler and linker.

Segments organize memory for the operating system loader.

---

## What is Dynamic Linking?

Dynamic Linking loads shared libraries at runtime instead of embedding them inside the executable.

---

## What is `ld-linux`?

`ld-linux` is the Linux Dynamic Linker responsible for loading shared libraries and resolving symbols before program execution.

---

## What is a Shared Library?

A Shared Library (`.so`) contains reusable code that multiple programs can load into memory simultaneously.

---

## Why are Shared Libraries useful?

They reduce executable size, save memory, and allow library updates without recompiling every application.

---

# Summary

Linux executes programs using the ELF format.

```
Executable

↓

Kernel

↓

ELF Loader

↓

Program Headers

↓

Memory Mapping

↓

Dynamic Linker

↓

Shared Libraries

↓

Entry Point

↓

Program Starts
```

Key concepts:

- ELF is the standard Linux executable format.
- The ELF Header describes the executable.
- Sections organize code and data for compilation.
- Segments organize memory for execution.
- The kernel loads segments into virtual memory.
- Dynamic Linking loads shared libraries at runtime.
- `ld-linux` resolves symbols and prepares the program.
- Shared Libraries reduce memory usage and executable size.

Understanding ELF is essential before learning system calls, kernel internals, container runtimes, and how Linux executes every application.

---

# Next Chapter

## Chapter 22 — System Calls: How User Space Talks to the Linux Kernel

We'll explore:

- What is a System Call?
- User Mode vs Kernel Mode
- CPU Privilege Levels (Ring 0 vs Ring 3)
- Trap Instructions
- System Call Table
- `syscall` Instruction
- Kernel Entry & Exit
- Common Linux System Calls
- How `strace` Works
- Complete System Call Flow