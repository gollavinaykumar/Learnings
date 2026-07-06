# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 76 — Process Memory Layout (Stack vs Heap)

---

# Most beginners think:

> "When I declare a variable, it is simply stored in RAM."

That's true...

but **where** in RAM?

A process doesn't receive one large chunk of memory.

Instead,

the Operating System organizes memory into **multiple regions**.

Every running program has a memory layout similar to this:

```
High Memory
+----------------------+
|        Stack         |
+----------------------+
|                      |
|      Free Space      |
|                      |
+----------------------+
|        Heap          |
+----------------------+
|   BSS Segment        |
+----------------------+
|  Initialized Data    |
+----------------------+
|     Code (Text)      |
+----------------------+
Low Memory
```

Understanding this layout is essential for:

- Linux Kernel
- Operating Systems
- Debugging
- Memory Leaks
- Buffer Overflows
- Dynamic Memory Allocation
- Performance Optimization

---

# Learning Objectives

After this chapter you will understand:

- How a process is loaded into memory
- Process memory layout
- Code Segment
- Data Segment
- BSS Segment
- Heap
- Stack
- Memory growth directions
- Real-world examples

---

# When You Run a Program

Suppose:

```bash
./program
```

The Operating System performs several steps.

```
Executable File

↓

Linux Loader

↓

Virtual Memory

↓

Running Process
```

The loader does **not** place everything randomly.

It organizes memory into regions.

---

# Complete Process Memory Layout

```
Higher Addresses

+----------------------+
|       Stack          |
+----------------------+
|                      |
|     Free Space       |
|                      |
+----------------------+
|        Heap          |
+----------------------+
|        BSS           |
+----------------------+
| Initialized Data     |
+----------------------+
|   Code (Text)        |
+----------------------+

Lower Addresses
```

Each region has a specific purpose.

---

# 1. Code Segment (Text Segment)

Contains:

```
Functions

Instructions

Machine Code
```

Example:

```c
int add(int a, int b)
{
    return a + b;
}
```

The compiled machine instructions are stored here.

Characteristics:

- Read-only (typically)
- Shared between processes running the same executable
- Loaded from the executable file

---

# Visualization

```
Code Segment

↓

main()

↓

add()

↓

printf()

↓

Compiled Instructions
```

---

# 2. Initialized Data Segment

Contains:

Global and static variables that are initialized.

Example:

```c
int age = 21;

static int count = 10;
```

Stored here.

Memory already contains:

```
21

10
```

before `main()` begins.

---

# Example

```c
int x = 100;
```

Memory:

```
Initialized Data

↓

100
```

---

# 3. BSS Segment

BSS stands for:

```
Block Started by Symbol
```

Contains:

Global and static variables that are **not explicitly initialized**.

Example:

```c
int count;

static int total;
```

The C language guarantees these start as zero.

Conceptually:

```
count

↓

0
```

```
total

↓

0
```

---

# Why Separate BSS?

Suppose:

```c
int numbers[1000000];
```

If stored in the executable,

the executable would become huge.

Instead,

the executable only records:

```
Reserve Memory

↓

Initialize to Zero
```

Much smaller executable.

---

# 4. Heap

The Heap stores **dynamically allocated memory**.

Created using:

```c
malloc()

calloc()

realloc()
```

Example:

```c
int *p =
malloc(sizeof(int));
```

Memory:

```
Heap

↓

Allocated Integer
```

The heap usually grows toward higher memory addresses.

---

# Heap Characteristics

- Runtime allocation
- Programmer controls lifetime
- Must call:

```c
free()
```

- Used for dynamic data structures

Examples:

- Linked Lists
- Trees
- Hash Tables
- Graphs

---

# 5. Stack

The Stack stores:

- Local variables
- Function parameters
- Return addresses
- Saved registers

Example:

```c
void test()
{
    int x = 10;
}
```

Variable:

```
x
```

lives on the stack.

---

# Stack Example

```c
int add(int a, int b)
{
    int sum = a + b;

    return sum;
}
```

Stack contains:

```
Return Address

↓

a

↓

b

↓

sum
```

When the function returns,

everything is removed automatically.

---

# Stack Growth

Most modern systems grow the stack:

```
High Address

↓

Low Address
```

Visualization:

```
Stack

↓

↓

↓

```

Every function call pushes a new stack frame.

---

# Heap Growth

Heap usually grows:

```
Low Address

↓

High Address
```

Visualization:

```
Heap

↑

↑

↑
```

---

# Why Opposite Directions?

Between stack and heap lies:

```
Free Space
```

As programs need more stack or heap memory,

they grow toward each other.

```
Stack

↓

↓

Free Space

↑

↑

Heap
```

If they collide,

memory allocation fails.

---

# Stack vs Heap

| Stack | Heap |
|--------|------|
| Automatic | Manual |
| Fast | Slightly Slower |
| Function Variables | Dynamic Objects |
| Freed Automatically | Must Call `free()` |
| Smaller | Usually Much Larger |

---

# Example Program

```c
#include <stdio.h>
#include <stdlib.h>

int global = 10;

int main()
{
    int local = 20;

    int *heap =
        malloc(sizeof(int));

    *heap = 30;

    printf("%p\n", (void *)&global);

    printf("%p\n", (void *)&local);

    printf("%p\n", (void *)heap);

    free(heap);

    return 0;
}
```

Observe that the addresses belong to different memory regions.

---

# Memory Visualization

```
Code

↓

main()

↓

add()

------------------

Initialized Data

↓

global = 10

------------------

Stack

↓

local = 20

------------------

Heap

↓

30
```

---

# Real-World Example — Web Browser

Google Chrome creates:

- Many processes
- Many threads

Each thread gets its own:

```
Stack
```

while dynamically allocated objects are stored in the process heap.

---

# Real-World Example — PostgreSQL

Database:

```
Heap

↓

Query Buffers

↓

Caches

↓

Indexes
```

Large data structures live in dynamically allocated memory.

---

# Real-World Example — Linux Kernel

The kernel manages:

- Process stacks
- Kernel stacks
- Dynamic kernel memory
- Memory mappings

Understanding process memory layout is essential for kernel development.

---

# Common Mistakes

---

## Returning Address of a Local Variable

Wrong:

```c
int *test()
{
    int x = 10;

    return &x;
}
```

`x` lives on the stack.

After the function returns,

that memory is no longer valid.

---

## Forgetting `free()`

Heap memory is **not** released automatically.

You must call:

```c
free()
```

when finished.

---

## Large Arrays on the Stack

Example:

```c
int arr[10000000];
```

This may cause a **stack overflow**.

Large allocations often belong on the heap instead.

---

# Hands-on Labs

## Lab 1

Declare:

- Global variable
- Static variable
- Local variable
- Heap variable

Print their addresses.

---

## Lab 2

Call a function several times.

Observe how local variable addresses behave.

---

## Lab 3

Allocate several heap objects using:

```c
malloc()
```

Print their addresses.

---

## Lab 4

Write a recursive function.

Observe how each recursive call creates a new stack frame.

---

## Lab 5

Use a debugger like `gdb` to inspect the process memory layout.

---

# Interview Questions

### What are the major memory segments of a process?

- Code (Text)
- Initialized Data
- BSS
- Heap
- Stack

---

### Where are local variables stored?

On the stack.

---

### Where does `malloc()` allocate memory?

On the heap.

---

### What is stored in the BSS segment?

Global and static variables that are not explicitly initialized.

---

### Why is the heap used?

To allocate memory dynamically during program execution.

---

# Summary

```
Executable

↓

Operating System

↓

Process Memory

↓

+----------------------+
| Stack               |
+----------------------+
| Free Space          |
+----------------------+
| Heap                |
+----------------------+
| BSS                 |
+----------------------+
| Initialized Data    |
+----------------------+
| Code                |
+----------------------+
```

## Key Takeaways

- Every process has a well-defined memory layout.
- Code, global variables, heap, and stack each occupy separate regions.
- The stack stores automatic variables and function call information.
- The heap stores dynamically allocated memory.
- The heap typically grows upward; the stack typically grows downward.
- Understanding process memory layout is fundamental for debugging, operating systems, dynamic memory allocation, and systems programming.

---

# Next Chapter

## Chapter 77 — `malloc()`

You'll learn:

- What `malloc()` is
- How heap allocation works
- Memory allocation algorithms
- What happens inside the C library
- How the Operating System provides heap memory
- Why `malloc()` does **not** initialize memory