# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 130 — Dynamic Linking (`.so`) — How Linux Loads Shared Libraries

---

# Most developers know shared libraries because of:

```bash
libc.so

libpthread.so

libm.so
```

But very few understand:

- Who loads them?
- When are they loaded?
- Why does every Linux program use them?
- What is the difference between static and dynamic linking?
- What are PLT and GOT?

After this chapter you'll understand how Linux actually starts almost every executable.

---

# Learning Objectives

After this chapter you will understand:

- Static linking
- Dynamic linking
- ELF shared libraries
- `.so` files
- Dynamic loader
- PLT
- GOT
- Lazy binding
- Runtime linking

---

# The Problem

Suppose you write:

```c
printf("Hello");
```

Question:

Where is:

```c
printf()
```

implemented?

Not inside your program.

It exists inside:

```
libc.so
```

So...

How does your executable find it?

---

# Before Linking

Source:

```c
printf("Hello");
```

Compiler:

```
main.o
```

Object file contains:

```
printf()

↓

Unknown
```

Address not yet known.

---

# What is Linking?

Linking means:

> **Connecting your compiled code with the code it depends on.**

Example:

```
main.o

↓

libc

↓

Executable
```

---

# Two Types of Linking

```
Static

↓

Copy Library

-------------------

Dynamic

↓

Load Later
```

---

# Static Linking

Imagine:

```
Program

↓

Copy libc

↓

Executable
```

Everything becomes part of one binary.

---

# Visualization

```
Executable

↓

Your Code

↓

printf()

↓

malloc()

↓

strlen()
```

Everything is copied.

---

# Advantages

- No external library dependency
- Easier deployment
- Runs even if shared libraries are unavailable

---

# Disadvantages

Huge executable.

Suppose:

```
100 Programs
```

Each contains:

```
libc
```

Memory and disk usage increase significantly.

---

# Dynamic Linking

Instead:

```
Executable

↓

Reference libc.so
```

Linux loads the shared library when the program starts.

---

# Visualization

Program A

```
↓

libc.so
```

Program B

```
↓

libc.so
```

Program C

```
↓

libc.so
```

One shared library.

Many programs.

---

# Shared Library

Linux shared libraries usually end with:

```
.so
```

Meaning:

```
Shared Object
```

Examples:

```
libc.so

libpthread.so

libm.so
```

---

# What is ELF?

Linux executables use:

```
ELF

Executable and Linkable Format
```

ELF stores:

- Code
- Data
- Symbols
- Required shared libraries
- Relocation information

---

# Visualization

ELF File

```
Headers

↓

Sections

↓

Symbols

↓

Libraries

↓

Machine Code
```

---

# Running a Program

Suppose:

```bash
./app
```

Kernel loads:

```
ELF
```

Then notices:

```
Needs libc.so
```

Who loads it?

---

# Dynamic Loader

Linux starts:

```
ld-linux

(or similar dynamic loader)
```

Responsibilities:

- Find shared libraries
- Map them into memory
- Resolve symbols
- Start your program

---

# Visualization

```
Kernel

↓

Dynamic Loader

↓

libc.so

↓

Your Program
```

---

# Mapping Libraries

Remember:

```
mmap()
```

Dynamic libraries are usually mapped into memory using memory mapping.

No large file copies are required.

---

# Symbol Resolution

Suppose:

```c
printf()
```

Loader asks:

```
Where Is printf?
```

Finds:

```
libc.so

↓

Address
```

Updates the executable so calls reach the correct function.

---

# Why Not Resolve Everything Immediately?

Imagine:

```
1000 Functions
```

Program only calls:

```
5
```

Resolving every symbol during startup wastes time.

---

# Lazy Binding

Linux often waits.

Program starts.

First time:

```c
printf()
```

is called,

the loader resolves its address.

Future calls go directly to the resolved location.

---

# Visualization

First Call

```
printf()

↓

Resolver

↓

Real Address
```

Later Calls

```
printf()

↓

Real Address
```

No additional resolution needed.

---

# What is PLT?

PLT stands for:

```
Procedure Linkage Table
```

Think of it as:

```
Function Gateway
```

Calls to external functions first pass through the PLT.

---

# Visualization

```
Your Code

↓

PLT

↓

printf()
```

Initially,

PLT may invoke the dynamic resolver.

Later,

it jumps directly to the actual function.

---

# What is GOT?

GOT stands for:

```
Global Offset Table
```

It stores addresses of:

- External functions
- Global objects

Example:

```
printf()

↓

GOT

↓

Actual Address
```

---

# PLT + GOT

Flow:

```
Program

↓

PLT

↓

GOT

↓

libc.so
```

Together,

they support efficient dynamic linking.

---

# First Function Call

```
Program

↓

PLT

↓

Dynamic Loader

↓

Resolve Address

↓

Update GOT
```

---

# Later Function Calls

```
Program

↓

PLT

↓

GOT

↓

Direct Jump
```

Fast.

---

# Why Dynamic Linking Matters

Suppose:

Security bug found in:

```
libc.so
```

Administrator updates:

```
libc.so
```

Every dynamically linked program automatically benefits,

without recompilation (provided compatibility is maintained).

---

# Static vs Dynamic

Static:

```
Program

↓

Everything Inside
```

Dynamic:

```
Program

↓

Shared Libraries
```

---

# Real-World Example — `printf()`

Your source:

```c
printf("Hello");
```

Actual flow:

```
Program

↓

PLT

↓

GOT

↓

libc.so

↓

printf()
```

---

# Real-World Example — Chrome

Chrome loads many shared libraries:

```
Graphics

↓

Audio

↓

Networking

↓

JavaScript Engine
```

Each can be updated independently.

---

# Real-World Example — Python

Python loads:

- Standard libraries
- Extension modules
- Native shared libraries

using dynamic loading techniques.

---

# Real-World Example — Docker

Containers share common system libraries provided by the container image,

reducing duplication compared with statically embedding everything.

---

# Common Mistakes

---

## Thinking Libraries Are Copied at Runtime

Dynamic libraries are typically mapped into memory,

not copied into the executable.

---

## Confusing `.a` and `.so`

```
.a

↓

Static Library

---------------------

.so

↓

Shared Library
```

---

## Thinking PLT Executes Functions

PLT is a jump mechanism,

not the function implementation itself.

---

## Ignoring the Dynamic Loader

The dynamic loader is responsible for resolving shared library dependencies before your program runs normally.

---

# Useful Commands

Show shared library dependencies:

```bash
ldd ./app
```

Inspect ELF information:

```bash
readelf -a ./app
```

View symbols:

```bash
nm ./app
```

---

# Hands-on Labs

## Lab 1

Compile a simple C program.

Run:

```bash
ldd
```

Observe its shared library dependencies.

---

## Lab 2

Use:

```bash
readelf
```

Inspect the ELF headers.

---

## Lab 3

Run:

```bash
nm
```

Locate unresolved external symbols.

---

## Lab 4

Compare:

- Dynamically linked executable
- Statically linked executable

Observe file sizes.

---

## Lab 5

Research PLT and GOT entries using:

```bash
objdump -d
```

---

# Interview Questions

### What is dynamic linking?

Loading and resolving shared libraries when a program starts (and in some cases lazily during execution).

---

### What is a shared library?

A reusable library stored as a `.so` file that can be shared among multiple programs.

---

### What is the difference between static and dynamic linking?

Static linking copies library code into the executable.

Dynamic linking loads shared libraries at runtime.

---

### What is the PLT?

The Procedure Linkage Table used to dispatch calls to dynamically linked functions.

---

### What is the GOT?

The Global Offset Table storing resolved addresses used by dynamically linked code.

---

# Summary

Static Linking:

```
Executable

↓

Everything Included
```

Dynamic Linking:

```
Executable

↓

Dynamic Loader

↓

Shared Libraries
```

Runtime Resolution:

```
Program

↓

PLT

↓

GOT

↓

Shared Library
```

Comparison:

| Static | Dynamic |
|---------|----------|
| Larger executable | Smaller executable |
| Library code copied | Shared libraries loaded |
| Self-contained | Depends on shared libraries |

## Key Takeaways

- Linking connects compiled code with the libraries it depends on.
- Dynamic linking reduces duplication by sharing libraries between programs.
- Linux executables use the ELF format.
- The dynamic loader maps shared libraries and resolves symbols.
- PLT and GOT support efficient runtime resolution of external functions.
- Dynamic linking is a core part of how modern Linux applications start and run.

---

# 🎉 Linux Executable Internals Complete!

You now understand:

- ✅ ELF format
- ✅ Shared libraries (`.so`)
- ✅ Static linking
- ✅ Dynamic linking
- ✅ Dynamic loader
- ✅ PLT
- ✅ GOT
- ✅ Lazy binding

You now know what actually happens between:

```bash
./program
```

and:

```text
main()
```

---

# Next Chapter

## Chapter 131 — ELF (Executable and Linkable Format) Deep Dive

You'll learn:

- ELF headers
- Program headers
- Section headers
- `.text`
- `.data`
- `.bss`
- `.rodata`
- `.plt`
- `.got`
- Symbol tables
- Relocation entries
- How the Linux kernel loads an executable from disk into memory