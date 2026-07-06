# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 50 — Memory Corruption

---

# Most beginners think:

> "If my program compiles successfully, my memory is safe."

Unfortunately, that's not true.

The compiler only checks **syntax and types**.

It **cannot guarantee** that your program accesses memory correctly.

One wrong pointer or one wrong array index can overwrite another part of memory.

This is called **Memory Corruption**.

Memory corruption is one of the biggest causes of:

- Program crashes
- Random bugs
- Data corruption
- Security vulnerabilities
- Operating system crashes
- Remote Code Execution (RCE)

Understanding memory corruption is one of the most important skills for a systems programmer.

---

# Learning Objectives

After this chapter you will understand:

- What memory corruption is
- Why memory corruption happens
- Buffer overflow
- Stack corruption
- Heap corruption
- Out-of-bounds access
- Stack smashing
- Security vulnerabilities
- How to detect corruption
- Best practices

---

# Imagine a Notebook

Suppose every page stores one student's record.

```
Page 1 → Student A

Page 2 → Student B

Page 3 → Student C
```

Now imagine writing Student A's data,

but instead of stopping at Page 1,

you continue writing onto Page 2.

```
Student A

↓↓↓↓↓↓↓↓↓

Student B
```

Student B's information becomes corrupted.

Memory corruption works exactly the same way.

---

# What is Memory Corruption?

Memory corruption happens when a program writes or reads memory **outside the area it owns**.

Example:

```
Program

↓

Allocated Memory

↓

Write Outside Block ❌

↓

Memory Corrupted
```

---

# Example 1 — Array Out of Bounds

```c
int numbers[5];

numbers[10] = 100;
```

Memory:

```
numbers

↓

0

1

2

3

4
```

Writing:

```
numbers[10]
```

goes outside the array.

The program overwrites unrelated memory.

---

# Example 2 — Buffer Overflow

```c
char name[5];

strcpy(name,"Vinay Kumar");
```

Memory:

```
name

↓

V

i

n

a

y
```

The string is much larger than the array.

Extra characters overwrite nearby memory.

This is called a **Buffer Overflow**.

---

# Buffer Overflow Visualization

Before:

```
+---------+
| Buffer  |
+---------+
```

After copying too much:

```
+---------+
| Buffer  |
+---------+
| Other   |
| Memory  |
+---------+
```

Neighboring memory gets overwritten.

---

# Stack Corruption

Suppose:

```c
void demo()
{
    char name[8];
}
```

Memory:

```
Stack Frame

----------------

Return Address

----------------

name[8]
```

If too much data is copied into `name`,

it may overwrite:

```
Return Address
```

Now the CPU no longer knows where to return.

Program crashes.

---

# Stack Smashing

Suppose:

```
Return Address

↓

0x12345678
```

Buffer overflow changes it:

```
0xDEADBEEF
```

Function returns.

CPU jumps to:

```
0xDEADBEEF
```

Crash.

Or worse,

malicious code.

This attack is called **Stack Smashing**.

---

# Heap Corruption

Heap corruption happens when heap memory is modified incorrectly.

Example:

```c
char *buf = malloc(10);

buf[100] = 'A';
```

Memory:

```
Allocated Block

↓

Overwrite Heap Metadata
```

The memory allocator may crash later.

Sometimes the crash occurs far away from the actual bug.

This makes heap corruption difficult to debug.

---

# Use-After-Free

```c
int *p = malloc(sizeof(int));

free(p);

*p = 10;
```

Memory:

```
Heap

↓

Released

↓

Still Accessed ❌
```

The memory no longer belongs to your program.

---

# Double Free

```c
free(p);

free(p);
```

The allocator becomes confused.

Modern allocators often detect this and terminate the program.

---

# Reading Uninitialized Memory

```c
int *p = malloc(sizeof(int));

printf("%d",*p);
```

The value is unknown.

Reading uninitialized memory is also undefined behavior.

---

# Memory Corruption in Real Systems

Memory corruption has caused:

- Browser exploits
- Linux kernel vulnerabilities
- Database crashes
- Banking software bugs
- Video game crashes
- Remote Code Execution attacks

Many security vulnerabilities begin with memory corruption.

---

# Common Types of Memory Corruption

| Problem | Description |
|----------|-------------|
| Buffer Overflow | Writing beyond an array |
| Buffer Underflow | Writing before an array |
| Stack Corruption | Overwriting stack data |
| Heap Corruption | Damaging heap structures |
| Use-after-free | Using released memory |
| Double Free | Releasing memory twice |
| Wild Pointer | Using uninitialized pointers |
| Dangling Pointer | Using freed memory |
| Out-of-Bounds Access | Invalid array index |

---

# Detecting Memory Corruption

## AddressSanitizer

Compile:

```bash
gcc -fsanitize=address -g program.c
```

Run:

```
./a.out
```

Example output:

```
ERROR:

heap-buffer-overflow
```

AddressSanitizer reports:

- Line number
- Function
- Invalid memory access
- Stack trace

It is one of the best debugging tools available.

---

## Valgrind

Linux:

```bash
valgrind ./program
```

Valgrind detects:

- Invalid reads
- Invalid writes
- Memory leaks
- Use-after-free

---

## GDB

Run:

```bash
gdb ./program
```

If a segmentation fault occurs:

```bash
bt
```

View the call stack.

---

# How to Prevent Memory Corruption

Always:

✔ Check array bounds

✔ Initialize pointers

✔ Check `malloc()` results

✔ Call `free()` exactly once

✔ Set pointers to `NULL`

✔ Avoid unsafe functions

---

# Unsafe Functions

Avoid:

```c
gets()
```

Prefer:

```c
fgets()
```

Avoid:

```c
strcpy()
```

Prefer:

```c
strncpy()
```

Or safer alternatives provided by your platform or libraries.

---

# Real-World Uses

Memory safety is critical for:

- Linux Kernel
- PostgreSQL
- Redis
- SQLite
- Browsers
- Device Drivers
- Operating Systems
- Embedded Systems

Large companies continuously test for memory corruption.

---

# Hands-on Labs

## Lab 1

Create an array.

Access:

```c
numbers[100]
```

Compile with:

```bash
gcc -fsanitize=address -g program.c
```

Observe the error.

---

## Lab 2

Create a buffer overflow.

Replace:

```c
strcpy()
```

with:

```c
fgets()
```

---

## Lab 3

Perform a use-after-free.

Observe AddressSanitizer output.

---

## Lab 4

Run the program using:

```bash
valgrind
```

Observe invalid memory access reports.

---

## Lab 5

Debug a segmentation fault using:

```bash
gdb
```

Use:

```bash
bt
```

to inspect the call stack.

---

# Interview Questions

### What is memory corruption?

Memory corruption occurs when a program reads or writes memory outside the region it owns.

---

### What is a buffer overflow?

Writing more data into a buffer than it can hold.

---

### What is stack smashing?

Overwriting data on the stack, especially the return address.

---

### What tools detect memory corruption?

- AddressSanitizer
- Valgrind
- GDB

---

### Why is memory corruption dangerous?

It can cause:

- Crashes
- Data loss
- Security vulnerabilities
- Remote code execution

---

# Summary

```
Allocate Memory

↓

Use Correctly

↓

✔ Safe Program

---------------------

Allocate Memory

↓

Write Outside Bounds

↓

Memory Corruption

↓

Crash / Security Bug
```

## Key Takeaways

- Memory corruption means accessing memory incorrectly.
- Buffer overflows are one of the most common causes.
- Stack and heap corruption can lead to crashes and exploits.
- Use-after-free and double free are dangerous memory errors.
- AddressSanitizer and Valgrind are essential debugging tools.
- Safe memory handling is critical for operating systems, databases, browsers, and kernel development.

---

# 🎉 Module 6 Complete

Congratulations!

You now understand:

- ✅ Stack Memory
- ✅ Heap Memory
- ✅ Global Memory
- ✅ Static Memory
- ✅ `malloc()`
- ✅ `calloc()`
- ✅ `realloc()`
- ✅ `free()`
- ✅ Memory Leaks
- ✅ Memory Fragmentation
- ✅ Memory Alignment
- ✅ Memory Corruption

These concepts are the foundation of everything that comes next:

- Linux Kernel
- POSIX Programming
- Device Drivers
- Networking
- Databases
- Embedded Systems
- High-Performance Servers

---

# 🚀 Next Module

# Module 7 — Structures & Unions

You'll learn:

- `struct`
- Nested Structures
- Arrays of Structures
- Pointer to Structure
- `union`
- Bit Fields
- `typedef`
- Flexible Array Members

By the end of Module 7, you'll understand how complex data is represented in memory and how operating systems, network protocols, databases, and the Linux kernel organize and manipulate structured data.