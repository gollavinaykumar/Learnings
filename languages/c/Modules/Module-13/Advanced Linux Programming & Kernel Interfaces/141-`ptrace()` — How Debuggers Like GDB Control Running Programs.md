# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 141 — `ptrace()` — How Debuggers Like GDB Control Running Programs

---

# Most developers use:

```bash
gdb

strace

lldb
```

Every day.

They can:

- Pause programs
- Read variables
- Change memory
- Change registers
- Execute one instruction at a time
- Set breakpoints

The obvious question is:

> **How can one process completely control another process?**

The answer is:

```
ptrace()
```

This is one of the most powerful system calls in Linux.

---

# Learning Objectives

After this chapter you will understand:

- What `ptrace()` is
- Tracer vs Tracee
- Process control
- Breakpoints
- Single stepping
- Reading memory
- Writing memory
- Register access
- How `gdb` works
- How `strace` works

---

# The Problem

Suppose you have:

```c
int x = 10;
```

While the program is running,

`gdb` shows:

```
(gdb)

print x

↓

10
```

Question:

How did GDB read your process memory?

Your program never sent it.

---

# Another Example

You type:

```
break main
```

Program stops exactly at:

```c
main()
```

How?

Linux normally executes instructions continuously.

Something must interrupt execution.

---

# Linux Solution

Linux provides:

```c
ptrace()
```

Meaning:

```
Process Trace
```

One process becomes the:

```
Tracer
```

The other becomes the:

```
Tracee
```

---

# Visualization

```
GDB

↓

ptrace()

↓

Your Program
```

---

# What is `ptrace()`?

Definition:

> **`ptrace()` allows one process to observe and control another process.**

It can:

- Pause execution
- Continue execution
- Read memory
- Write memory
- Read registers
- Modify registers
- Receive events

---

# Think About a Remote Controller

Imagine:

```
Robot

↓

Walking
```

Remote:

```
Pause

↓

Turn Left

↓

Resume

↓

Inspect Battery
```

`ptrace()` is that remote control.

---

# Tracer and Tracee

```
Tracer

↓

Controls

↓

Tracee
```

Example:

```
gdb

↓

hello
```

---

# Process States

Normal execution:

```
Running
```

Debugger attaches:

```
Stopped
```

Debugger examines:

- Memory
- Registers
- Variables

Then:

```
Continue
```

---

# Attaching

Debugger:

```
ptrace()

↓

Attach

↓

Process Stops
```

Now debugger has control.

---

# Visualization

```
Program

↓

Running

↓

Attach

↓

Paused
```

---

# Reading Memory

Suppose:

```
Variable

↓

Address

↓

0x7fff....
```

Debugger asks kernel:

```
Read Memory

↓

Return Bytes
```

Now GDB prints:

```
x = 10
```

---

# Writing Memory

Debugger can also modify memory.

Example:

```
x = 10

↓

Debugger

↓

x = 500
```

Program continues using the new value.

---

# Reading Registers

Debugger can inspect:

```
RIP

↓

RSP

↓

RAX

↓

RBX

↓

RCX
```

Entire CPU state.

---

# Visualization

```
CPU Registers

↓

Kernel

↓

Debugger
```

---

# Writing Registers

Debugger may change:

```
Instruction Pointer

↓

Continue
```

Execution resumes from a different location.

Very powerful.

---

# Single Stepping

Suppose:

```c
a++;

b++;
```

Debugger executes:

```
One Instruction

↓

Stop

↓

One Instruction

↓

Stop
```

Exactly how:

```
step

next
```

work.

---

# Breakpoints

How does:

```
break main
```

work?

Debugger replaces the first instruction with a special:

```
Trap Instruction
```

On x86:

```
INT3
```

---

# Visualization

Original:

```
Instruction

↓

Execute
```

Breakpoint:

```
INT3

↓

CPU Trap

↓

Kernel

↓

Debugger
```

Execution pauses.

---

# Restoring Instructions

Debugger:

```
Save Original Byte

↓

Insert INT3

↓

Breakpoint Hit

↓

Restore Byte

↓

Continue
```

You never notice this happening.

---

# System Call Tracing

`strace` uses `ptrace()` differently.

Instead of breakpoints,

it watches:

```
System Call Entry

↓

System Call Exit
```

---

# Visualization

```
Program

↓

read()

↓

Kernel

↓

ptrace Event

↓

strace Prints
```

---

# Example

Run:

```bash
strace ls
```

Output:

```
open()

↓

read()

↓

close()

↓

write()
```

`strace` isn't reading your source code.

It watches the kernel events.

---

# Signals

Debugger also receives:

```
SIGSEGV

↓

SIGINT

↓

SIGTRAP
```

Before the application handles them.

---

# Relationship

```
Application

↓

Kernel

↓

ptrace

↓

Debugger
```

Kernel acts as the trusted intermediary.

---

# Security

Not every process may trace another.

Linux checks permissions.

Generally:

- Same user
- Appropriate capabilities
- Security policies

must allow tracing.

---

# Performance

Normal program:

```
Run
```

Debugger:

```
Instruction

↓

Kernel

↓

Debugger

↓

Continue
```

Much slower.

Debugging intentionally sacrifices performance.

---

# Real-World Example — GDB

Uses:

```
ptrace()

↓

Read Registers

↓

Read Memory

↓

Breakpoints
```

---

# Real-World Example — `strace`

Uses:

```
ptrace()

↓

Observe System Calls

↓

Print
```

---

# Real-World Example — Crash Analysis

Debugger:

```
Core Dump

↓

Registers

↓

Stack

↓

Memory
```

Many debugging tools rely on these capabilities.

---

# Real-World Example — Reverse Engineering

Many reverse-engineering tools use debugging interfaces to inspect program execution.

---

# Common Mistakes

---

## Thinking GDB Reads Variables Directly

The kernel provides controlled access through `ptrace()`.

---

## Thinking Breakpoints Are Magic

They are usually implemented using trap instructions inserted into executable code.

---

## Assuming Any Process Can Trace Any Other Process

Linux enforces permission and security checks.

---

## Forgetting Performance Costs

Single-stepping every instruction is naturally much slower than normal execution.

---

# Hands-on Labs

## Lab 1

Run:

```bash
gdb ./hello
```

Set a breakpoint at:

```text
main
```

Observe the pause.

---

## Lab 2

Run:

```bash
strace ls
```

Watch system calls in real time.

---

## Lab 3

Use GDB:

```
print variable
```

Observe memory inspection.

---

## Lab 4

Step through a program one instruction at a time.

Observe register changes.

---

## Lab 5

Read:

```bash
man 2 ptrace
```

Explore the available tracing requests.

---

# Interview Questions

### What is `ptrace()`?

A Linux system call allowing one process to observe and control another process.

---

### How does GDB pause a program?

It uses `ptrace()` together with trap instructions (breakpoints).

---

### How does `strace` work?

It traces system call entry and exit events using `ptrace()`.

---

### Can a debugger modify process memory?

Yes.

With appropriate permissions, it can read and write another process's memory and registers.

---

### Why is debugging slower than normal execution?

Because execution repeatedly transitions between the traced process, the kernel, and the debugger.

---

# Summary

Architecture:

```
Debugger

↓

ptrace()

↓

Kernel

↓

Target Process
```

Capabilities:

```
Pause

↓

Read Memory

↓

Write Memory

↓

Read Registers

↓

Write Registers

↓

Continue
```

Breakpoint Flow:

```
Original Instruction

↓

INT3

↓

Kernel Trap

↓

Debugger

↓

Restore

↓

Continue
```

## Key Takeaways

- `ptrace()` is the foundation of debugging on Linux.
- It enables controlled inspection and modification of another process.
- GDB uses `ptrace()` for breakpoints, stepping, and memory inspection.
- `strace` uses `ptrace()` to monitor system calls.
- Linux enforces security restrictions on tracing.
- Understanding `ptrace()` explains how debuggers work internally.

---

# Next Chapter

## Chapter 142 — Linux Capabilities — Splitting Root Privileges into Smaller Pieces

You'll learn:

- Why `root` is too powerful
- What Linux capabilities are
- `CAP_NET_ADMIN`
- `CAP_SYS_ADMIN`
- `CAP_SYS_PTRACE`
- File capabilities
- Process capabilities
- Ambient capabilities
- How modern Linux replaces the traditional all-or-nothing root permission model