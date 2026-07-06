# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 106 — User Space vs Kernel Space (The Two Worlds)

---

# Most beginners think:

> "Every program has full access to the computer."

If that were true,

one buggy program could:

- Delete the operating system
- Read your passwords
- Modify another application's memory
- Crash the entire computer

Modern operating systems prevent this by creating **two completely different worlds**:

```
User Space

and

Kernel Space
```

Understanding this concept is one of the biggest milestones in becoming a systems programmer.

---

# Learning Objectives

After this chapter you will understand:

- CPU privilege levels
- User Space
- Kernel Space
- User Mode vs Kernel Mode
- Memory protection
- Context switching
- Why segmentation faults happen
- System call transitions

---

# Imagine a Bank

Customers can:

- Deposit money
- Withdraw money
- Check balance

But customers cannot:

- Open the vault
- Print money
- Modify bank databases

Instead:

```
Customer

↓

Bank Employee

↓

Vault
```

The bank employee acts as the trusted intermediary.

The operating system works the same way.

---

# Two Worlds

```
+------------------------+
| User Space             |
| Your Applications      |
+------------------------+
| Kernel Space           |
| Operating System       |
+------------------------+
```

Applications live here:

```
Chrome

VS Code

Docker

Git

Your C Program
```

Kernel lives below them.

---

# CPU Privilege Levels

Modern CPUs support different privilege levels.

Conceptually:

```
Highest Privilege

↓

Kernel Mode

---------------------

Lowest Privilege

↓

User Mode
```

Most operating systems primarily use these two levels.

---

# User Mode

Programs running here:

```
Cannot

↓

Access Hardware

Cannot

↓

Modify Kernel Memory

Cannot

↓

Execute Privileged Instructions
```

This protects the entire system.

---

# Kernel Mode

Kernel code can:

```
Read Any Memory

Write Any Memory

Control CPU

Access Devices

Schedule Processes

Handle Interrupts
```

The kernel has full control.

---

# Visualization

```
Application

↓

User Mode

↓

System Call

↓

Kernel Mode

↓

Hardware
```

Applications temporarily enter kernel mode only through controlled mechanisms.

---

# Why Is User Mode Necessary?

Imagine:

```
Game

↓

Overwrite Browser Memory
```

Or:

```
Music Player

↓

Disable Firewall
```

Without isolation,

every application could interfere with every other application.

---

# Memory Protection

Suppose:

```
Process A

↓

Memory A

-------------------

Process B

↓

Memory B
```

Process A cannot access Memory B directly.

The CPU and operating system enforce this isolation.

---

# Example

Program:

```c
int *p = (int *)0x12345678;

*p = 100;
```

Question:

Can you write to any address?

No.

If the address isn't mapped into your process,

the CPU raises an exception.

Result:

```
Segmentation Fault
```

---

# Why Does Segmentation Fault Happen?

Conceptually:

```
Program

↓

Access Invalid Memory

↓

CPU Detects

↓

Kernel Handles Exception

↓

Program Terminated
```

The kernel protects memory.

---

# Example

```c
int *p = NULL;

*p = 10;
```

Flow:

```
NULL

↓

Invalid Address

↓

CPU Fault

↓

Kernel

↓

SIGSEGV

↓

Program Ends
```

---

# What Is a System Call?

Applications cannot directly perform privileged work.

Instead:

```
Application

↓

System Call

↓

Kernel

↓

Hardware
```

The system call is the controlled entrance into kernel mode.

---

# Mode Switching

Suppose:

```c
write(...)
```

Execution:

```
User Mode

↓

CPU Trap Instruction

↓

Kernel Mode

↓

Kernel Executes

↓

Return

↓

User Mode
```

This transition is called a:

```
Mode Switch
```

---

# Context Switch vs Mode Switch

Many beginners confuse these.

### Mode Switch

```
Same Process

↓

User

↓

Kernel

↓

User
```

The process remains the same.

---

### Context Switch

```
Process A

↓

Scheduler

↓

Process B
```

The CPU changes from one process to another.

Different concept.

---

# Example

Program:

```c
printf("Hello");
```

Internally:

```
User Mode

↓

printf()

↓

write()

↓

System Call

↓

Kernel Mode

↓

Terminal Driver

↓

Return

↓

User Mode
```

---

# Privileged Instructions

Some CPU instructions are reserved for the kernel.

Examples include operations that:

- Configure memory management hardware
- Enable or disable interrupts
- Access certain processor control registers

If a user program attempts such instructions,

the CPU raises an exception.

---

# Interrupts

Suppose:

Keyboard pressed.

Flow:

```
Keyboard

↓

Interrupt

↓

CPU

↓

Kernel

↓

Driver

↓

Application
```

Interrupt handlers execute in kernel mode.

---

# Process Isolation

Suppose:

```
Chrome

↓

Memory

------------------

VS Code

↓

Memory

------------------

Docker

↓

Memory
```

Each process has its own virtual address space.

They cannot directly modify one another.

---

# Why Doesn't `malloc()` Need Kernel Mode Every Time?

Suppose:

```c
malloc(64);
```

Usually:

```
Allocator

↓

Existing Heap

↓

Return Memory
```

No kernel transition required.

Only when the allocator needs additional memory does it request it from the kernel.

---

# Real-World Example — Linux

Linux separates:

```
User Space

↓

Applications

--------------------

Kernel Space

↓

Drivers

↓

Scheduler

↓

Networking

↓

Filesystem
```

This separation is fundamental to Linux security.

---

# Real-World Example — Windows

Applications execute in user mode.

The Windows kernel performs privileged operations on their behalf.

---

# Real-World Example — Browser Sandbox

Modern browsers isolate:

```
Tab 1

↓

Separate Process

----------------

Tab 2

↓

Separate Process
```

If one tab crashes,

others usually continue running.

---

# Real-World Example — Docker

Containers:

```
Container Process

↓

User Space

↓

Shared Linux Kernel
```

Containers do not bypass the kernel.

---

# Common Mistakes

---

## Thinking User Programs Access Hardware Directly

Normally they don't.

They use system calls.

---

## Confusing Kernel Mode with Root User

Root is a user account with elevated permissions.

Kernel mode is a CPU execution mode.

They are different concepts.

---

## Thinking Segmentation Fault Is a Compiler Error

It is a runtime fault caused by invalid memory access.

---

## Confusing Mode Switch with Context Switch

Mode switch:

```
User

↓

Kernel

↓

Same Process
```

Context switch:

```
Process A

↓

Process B
```

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    printf("Hello Kernel!\n");

    return 0;
}
```

Conceptual Flow:

```
main()

↓

printf()

↓

write()

↓

System Call

↓

Kernel Mode

↓

Display Output

↓

Return To User Mode
```

---

# Hands-on Labs

## Lab 1

Run:

```bash
strace ./program
```

Observe where system calls occur.

---

## Lab 2

Trigger a segmentation fault intentionally:

```c
int *p = NULL;
*p = 1;
```

Observe the operating system's response.

---

## Lab 3

Read about CPU privilege rings and understand why most operating systems primarily use Ring 0 (kernel) and Ring 3 (user).

---

## Lab 4

Use:

```bash
cat /proc/self/maps
```

(on Linux)

Inspect your process's virtual memory layout.

---

## Lab 5

Draw the path of:

```c
printf("Hello");
```

through:

- User Mode
- Kernel Mode
- Terminal
- Back to User Mode

---

# Interview Questions

### What is user space?

The protected execution environment where normal applications run.

---

### What is kernel space?

The privileged execution environment where the operating system kernel runs.

---

### Why can't applications access hardware directly?

Because the operating system and CPU enforce protection and resource management.

---

### What causes a segmentation fault?

Attempting to access memory that the process is not permitted to access.

---

### What is the difference between a mode switch and a context switch?

A mode switch changes CPU privilege level within the same process.

A context switch changes execution from one process or thread to another.

---

# Summary

```
Application

↓

User Mode

↓

System Call

↓

Kernel Mode

↓

Hardware

↓

Kernel Mode

↓

User Mode
```

Memory Protection:

```
Process A

↓

Own Memory

---------------------

Process B

↓

Own Memory
```

## Key Takeaways

- Modern operating systems separate execution into user space and kernel space.
- User programs execute with limited privileges.
- The kernel executes with full hardware access.
- System calls provide the controlled bridge between user mode and kernel mode.
- Segmentation faults occur when the CPU detects illegal memory access.
- Understanding user space and kernel space is essential for learning processes, files, networking, device drivers, and Linux kernel internals.

---

# Next Chapter

## Chapter 107 — System Calls (How C Actually Talks to the Kernel)

You'll learn:

- What system calls are
- Trap instructions
- Kernel entry and exit
- `read()`, `write()`, `open()`, `close()`
- libc wrappers vs raw system calls
- Why `printf()` eventually becomes `write()`
- How every operating system service begins with a system call