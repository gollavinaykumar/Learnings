# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 131 — ELF (Executable and Linkable Format) Deep Dive — How Linux Loads an Executable

---

# Most developers know ELF because of:

```bash
file myprogram
```

Output:

```
ELF 64-bit executable
```

Most people stop there.

Very few understand:

- What is inside an ELF file?
- How does Linux know where `main()` is?
- Why are there `.text`, `.data`, `.bss` sections?
- What are program headers?
- What are section headers?
- What actually happens after you type:

```bash
./myprogram
```

This chapter answers all of those questions.

---

# Learning Objectives

After this chapter you will understand:

- What ELF is
- ELF Header
- Program Header Table
- Section Header Table
- `.text`
- `.data`
- `.bss`
- `.rodata`
- `.plt`
- `.got`
- Symbol Table
- Relocation Table
- Linux executable loading

---

# What is ELF?

ELF stands for:

```
Executable

↓

Linkable

↓

Format
```

It is the standard executable format used by Linux and many Unix-like systems.

ELF is used for:

- Executables
- Shared libraries
- Object files
- Core dumps

---

# Think About a Shipping Container

Imagine:

```
Container

↓

Documents

↓

Food

↓

Electronics

↓

Clothes
```

Everything has its own compartment.

ELF works the same way.

```
Headers

↓

Code

↓

Data

↓

Symbols

↓

Debug Info
```

Everything has its own section.

---

# High-Level ELF Layout

```
+----------------------+
| ELF Header           |
+----------------------+
| Program Headers      |
+----------------------+
| Section Headers      |
+----------------------+
| Sections             |
+----------------------+
```

Each part has a different responsibility.

---

# ELF Header

The ELF header is the first structure in every ELF file.

It answers questions like:

- Is this really an ELF file?
- 32-bit or 64-bit?
- ARM or x86-64?
- Entry point?
- Where are the program headers?
- Where are the section headers?

---

# Visualization

```
ELF Header

↓

Magic Number

↓

Architecture

↓

Entry Point

↓

Offsets
```

---

# Magic Number

Every ELF file starts with:

```
7F 45 4C 46
```

ASCII:

```
7F

E

L

F
```

Linux checks this first.

If it doesn't match,

the kernel rejects the file.

---

# Entry Point

One important field:

```
Entry Address
```

Example:

```
0x401000
```

This is **not always `main()`**.

Instead,

execution usually begins at:

```
_start
```

---

# Why Not `main()`?

Actual startup:

```
Kernel

↓

_start

↓

Runtime Initialization

↓

main()
```

The C runtime prepares the environment before your code begins.

---

# Program Header Table

The kernel uses:

```
Program Headers
```

to decide:

```
Which Parts

↓

Load Into Memory
```

---

# Visualization

```
Disk

↓

Program Header

↓

Memory Mapping
```

Without program headers,

the kernel wouldn't know what to load.

---

# Section Header Table

Used mostly by:

- Linkers
- Debuggers
- Development tools

Not by the kernel when launching a program.

---

# Visualization

```
Compiler

↓

Linker

↓

Debugger

↓

Section Headers
```

---

# Important Sections

Let's explore the most common ELF sections.

---

# `.text`

Contains:

```
Machine Instructions
```

Example:

```c
printf("Hello");
```

Compiled instructions live here.

---

# Visualization

```
.text

↓

CPU Instructions
```

Usually:

```
Read

↓

Execute
```

Not writable.

---

# `.data`

Contains:

```
Initialized Global Variables
```

Example:

```c
int x = 10;
```

Stored inside:

```
.data
```

---

# Visualization

```
.data

↓

Initialized Variables
```

---

# `.bss`

Contains:

```
Uninitialized Globals
```

Example:

```c
int counter;
```

The executable does **not** store zeros.

Instead:

```
Size Recorded

↓

Kernel Allocates

↓

Zero Fills
```

Saves disk space.

---

# Example

```c
int a = 5;
int b;
```

Memory layout:

```
.data

↓

a

---------------------

.bss

↓

b
```

---

# `.rodata`

Contains:

```
Read-Only Data
```

Example:

```c
"Hello World"
```

String literals live here.

---

# Visualization

```
.rodata

↓

Strings

↓

Constants
```

Read-only.

---

# `.plt`

You learned earlier:

```
PLT

↓

Procedure Linkage Table
```

Purpose:

```
Jump To

↓

Shared Library Function
```

---

# `.got`

Contains:

```
Resolved Addresses
```

For dynamically linked functions and objects.

---

# `.symtab`

Stores:

```
Function Names

↓

Variable Names

↓

Addresses
```

Useful for:

- Linkers
- Debuggers

---

# Example

```
main

↓

0x401050

---------------------

printf

↓

External Symbol
```

---

# `.strtab`

Symbol names are stored separately.

Example:

```
main

printf

malloc
```

The symbol table references these strings.

---

# Relocation Sections

Suppose:

```c
printf()
```

Compiler doesn't yet know its final address.

Relocation entries tell the linker:

```
Fix This Later
```

---

# Visualization

```
printf()

↓

Relocation

↓

Linker

↓

Real Address
```

---

# Loading an ELF File

Complete flow:

```
./app

↓

Kernel

↓

Read ELF Header

↓

Program Headers

↓

Map Sections

↓

Load Libraries

↓

_start

↓

main()
```

---

# Memory Layout

After loading:

```
Code (.text)

↓

Read-Only (.rodata)

↓

Initialized Data (.data)

↓

BSS (.bss)

↓

Heap

↓

Memory Mapping

↓

Stack
```

Looks familiar?

This is your process's virtual address space.

---

# Relationship with `mmap()`

Linux loads ELF segments using:

```
mmap()
```

Executable pages are mapped directly into virtual memory.

---

# Shared Libraries

Libraries are also ELF files.

Example:

```
libc.so

↓

ELF

↓

Mapped

↓

Program
```

---

# Tools

Show ELF header:

```bash
readelf -h app
```

Program headers:

```bash
readelf -l app
```

Sections:

```bash
readelf -S app
```

Symbols:

```bash
readelf -s app
```

---

# Useful Commands

Display all ELF information:

```bash
readelf -a app
```

Disassemble:

```bash
objdump -d app
```

View symbols:

```bash
nm app
```

---

# Real-World Example — GCC

Compilation:

```
Source

↓

Object Files

↓

ELF

↓

Executable
```

---

# Real-World Example — GDB

Debugger:

```
Reads

↓

.symtab

↓

.strtab
```

Uses symbol information to display function names and variables.

---

# Real-World Example — Kernel

When you execute:

```bash
./program
```

The kernel parses:

```
ELF Header

↓

Program Headers

↓

Maps Segments

↓

Transfers Control
```

---

# Real-World Example — Shared Libraries

```
Executable

↓

libc.so

↓

libpthread.so

↓

libm.so
```

Each is its own ELF file.

---

# Common Mistakes

---

## Thinking Sections Are Loaded Directly

The kernel loads **program segments**, not individual sections.

Multiple sections may belong to the same segment.

---

## Thinking `main()` Is the Entry Point

Execution begins at:

```
_start
```

The runtime later calls:

```
main()
```

---

## Confusing Sections and Segments

Sections are mainly for linking and debugging.

Segments are for loading into memory.

---

## Assuming `.bss` Occupies File Space

Only its size is stored.

The kernel allocates zero-filled memory when loading the program.

---

# Hands-on Labs

## Lab 1

Run:

```bash
readelf -h app
```

Study the ELF header.

---

## Lab 2

Inspect program headers:

```bash
readelf -l app
```

---

## Lab 3

List sections:

```bash
readelf -S app
```

Identify:

- `.text`
- `.data`
- `.bss`
- `.rodata`

---

## Lab 4

Use:

```bash
objdump -d app
```

Find the `_start` symbol.

---

## Lab 5

Run:

```bash
nm app
```

Observe exported and undefined symbols.

---

# Interview Questions

### What is ELF?

The Executable and Linkable Format used for executables, object files, and shared libraries on Linux.

---

### What is stored in the ELF header?

Information such as architecture, entry point, and offsets to other ELF structures.

---

### What is the difference between `.data` and `.bss`?

`.data` stores initialized global variables.

`.bss` stores uninitialized globals and is zero-initialized at runtime.

---

### Why does Linux use `_start` instead of calling `main()` directly?

The runtime must initialize the process environment before calling `main()`.

---

### What is the difference between program headers and section headers?

Program headers describe what the kernel loads into memory.

Section headers organize the file for linkers, debuggers, and development tools.

---

# Summary

ELF Structure:

```
ELF Header

↓

Program Headers

↓

Sections

↓

Symbols
```

Important Sections:

| Section | Purpose |
|----------|----------|
| `.text` | Machine code |
| `.data` | Initialized globals |
| `.bss` | Uninitialized globals |
| `.rodata` | Read-only constants |
| `.plt` | Dynamic linking stubs |
| `.got` | Resolved addresses |
| `.symtab` | Symbols |
| `.strtab` | Symbol names |

Program Startup:

```
Kernel

↓

Read ELF

↓

Map Segments

↓

Dynamic Loader

↓

_start

↓

main()
```

## Key Takeaways

- ELF is the standard executable format on Linux.
- The ELF header describes the executable.
- Program headers guide the kernel during loading.
- Section headers organize information for development tools.
- The kernel begins execution at `_start`, not `main()`.
- Understanding ELF explains how Linux transforms a file on disk into a running process.

---

# 🎉 Linux Executable Format Mastered!

You now understand:

- ✅ ELF Header
- ✅ Program Headers
- ✅ Section Headers
- ✅ `.text`
- ✅ `.data`
- ✅ `.bss`
- ✅ `.rodata`
- ✅ `.plt`
- ✅ `.got`
- ✅ Symbol tables
- ✅ Relocations
- ✅ Linux executable loading

You now know exactly what happens after running:

```bash
./program
```

until your `main()` function begins executing.

---

# Next Chapter

## Chapter 132 — From Power Button to `main()` — The Complete Linux Boot and Program Execution Journey

You'll learn the complete lifecycle:

- Power button pressed
- CPU reset
- BIOS/UEFI
- Bootloader (GRUB)
- Linux kernel startup
- Kernel initialization
- `init` / `systemd`
- User login
- Shell startup
- Executing a command
- `fork()`
- `execve()`
- ELF loading
- Dynamic linker
- `_start`
- `main()`

This chapter combines everything you've learned into one complete end-to-end picture.