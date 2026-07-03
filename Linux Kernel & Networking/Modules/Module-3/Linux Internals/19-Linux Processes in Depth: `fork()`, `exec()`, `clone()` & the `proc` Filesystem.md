Most developers think creating a process is simple.

```bash
python app.py
```

Done.

But internally, Linux performs dozens of operations.

Questions you may have:

- What actually happens when you run a program?
- Why is `fork()` so important?
- What is `exec()`?
- Why doesn't Docker use `fork()` alone?
- What is `clone()`?
- What is `task_struct`?
- Why does Linux have a `/proc` directory?
- Where does `ps` get its information?

The answer lies inside the Linux kernel.

Every running process in Linux is represented by a kernel data structure and managed through system calls.

After this chapter, you'll understand exactly how Linux creates, manages, and monitors processes internally.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 19 — Linux Processes in Depth: `fork()`, `exec()`, `clone()` & the `/proc` Filesystem

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Linux Process Model
> - Process Creation
> - `fork()`
> - `vfork()`
> - `exec()`
> - `clone()`
> - `task_struct`
> - Process Descriptors
> - Process States in Linux
> - Context Switching
> - `/proc` Filesystem
> - Process Information
> - How Docker Creates Processes

---

# 📖 Every Program Starts as a File

Suppose you have:

```bash
/usr/bin/python3
```

This is just a file stored on disk.

```
SSD

↓

python3
```

Nothing is running yet.

A running process exists only after the kernel creates it.

---

# From Program to Process

Suppose you execute:

```bash
python app.py
```

Linux performs:

```
Program File

↓

fork()

↓

Child Process

↓

exec()

↓

Python Running
```

This happens every time a program starts.

---

# Linux Process Model

Every process is represented inside the kernel.

```
Application

↓

Linux Kernel

↓

Process Object
```

The kernel tracks every running process.

---

# The Process Descriptor

Linux stores process information inside a structure called:

```
task_struct
```

Think of it as the kernel's "record" for a process.

---

# What is `task_struct`?

Every running process has one.

```
task_struct

├── PID

├── Parent PID

├── State

├── CPU Registers

├── Scheduling Info

├── Memory Info

├── Open Files

├── Credentials

├── Signal Handlers

└── Threads
```

The kernel constantly updates this structure.

---

# Where is `task_struct` Stored?

```
Kernel Memory

↓

Process List

↓

task_struct

↓

task_struct

↓

task_struct
```

One per process.

---

# Process Creation

Most Linux processes begin with:

```
fork()
```

---

# What Does `fork()` Do?

Suppose Bash executes:

```bash
python app.py
```

Internally:

```
Bash

↓

fork()

↓

Parent

+

Child
```

The child is initially almost identical to the parent.

---

# After `fork()`

Initially:

```
Parent

↓

Memory

↓

Open Files

↓

Registers
```

Child receives copies (using Copy-on-Write where possible).

Both continue executing independently.

---

# Copy-on-Write Reminder

Linux does **not** immediately duplicate memory.

Instead:

```
Parent

↓

Shared Pages

↑

Child
```

Only modified pages are copied.

This makes `fork()` fast.

---

# What is `exec()`?

After `fork()`,

the child usually replaces itself with another program.

```
Child

↓

exec()

↓

Python
```

Everything except the PID is replaced.

---

# What Changes During `exec()`?

Replaced:

✔ Program Code

✔ Heap

✔ Stack

✔ Data

Preserved:

✔ PID

✔ Some Open File Descriptors

The process becomes a completely different program.

---

# Complete Flow

```
Terminal

↓

fork()

↓

Child

↓

exec()

↓

Python

↓

Running Process
```

---

# Why Use `fork()` + `exec()`?

This allows the parent to continue running.

Example:

```
Bash

↓

fork()

↓

Child

↓

exec(ls)
```

Bash remains alive while `ls` executes.

---

# What is `vfork()`?

`vfork()` is an optimized version of `fork()`.

Instead of creating a separate address space immediately,

the child temporarily shares the parent's memory.

```
Parent

↓

Shared Memory

↑

Child
```

The parent is paused until the child calls:

```
exec()

or

exit()
```

Useful when the child immediately executes another program.

---

# What is `clone()`?

`clone()` is one of the most important Linux system calls.

Unlike `fork()`,

it allows selective sharing.

Example:

```
Share Memory?

Yes

No
```

```
Share Files?

Yes

No
```

```
Share Network?

Yes

No
```

The caller chooses.

---

# Why Does Docker Use `clone()`?

Docker doesn't want a normal child process.

It wants:

✔ Separate Process IDs

✔ Separate Network

✔ Separate Mounts

✔ Separate Hostname

✔ Separate Users

`clone()` makes this possible.

---

# clone() Example

```
clone()

↓

New Process

+

New Namespace

+

Shared or Separate Resources
```

This flexibility powers Linux containers.

---

# Linux Process States

A Linux process moves through several states.

```
Running

↓

Interruptible Sleep

↓

Running

↓

Stopped

↓

Zombie

↓

Exited
```

The kernel tracks these inside `task_struct`.

---

# Running State

The process is currently executing,

or waiting for CPU time.

```
Ready

↓

CPU

↓

Running
```

---

# Interruptible Sleep

Suppose a process waits for disk I/O.

```
Process

↓

Sleep

↓

Data Arrives

↓

Wake Up
```

The scheduler can wake it at any time.

---

# Uninterruptible Sleep

Sometimes a process waits for critical I/O.

State:

```
D
```

Usually:

```
Disk

↓

Storage Driver

↓

Wake Up
```

Signals generally do not interrupt this state.

---

# Zombie Process

Suppose:

```
Child Ends
```

But:

```
Parent

↓

Didn't Call wait()
```

The child becomes:

```
Zombie
```

Only its exit status remains.

---

# Orphan Process

Suppose:

```
Parent Dies

↓

Child Continues
```

Linux automatically reassigns the child to:

```
systemd

PID 1
```

---

# Context Switching

Suppose:

```
Chrome

↓

Docker

↓

Python
```

The scheduler switches between them.

Linux saves:

```
Registers

↓

Program Counter

↓

Stack Pointer
```

from one process,

then restores another.

---

# The `/proc` Filesystem

Linux exposes process information using:

```
/proc
```

This is **not** a real disk filesystem.

It is generated dynamically by the kernel.

---

# Example

```
/proc

├── 1

├── 230

├── 821

├── 1574
```

Each directory is a PID.

---

# Process Information

Suppose:

```
PID = 2450
```

View:

```bash
ls /proc/2450
```

Files include:

```
cmdline

status

maps

fd

environ

stat

limits
```

---

# `/proc/<PID>/status`

Example:

```bash
cat /proc/2450/status
```

Shows:

```
PID

PPID

Threads

Memory

State

UID

GID
```

---

# `/proc/<PID>/maps`

Displays:

```
Virtual Memory Layout
```

Example:

```
Code

↓

Heap

↓

Libraries

↓

Stack
```

Useful for debugging.

---

# `/proc/<PID>/fd`

Shows open file descriptors.

Example:

```
0

↓

stdin
```

```
1

↓

stdout
```

```
2

↓

stderr
```

Plus open files,

pipes,

and sockets.

---

# Where Does `ps` Get Information?

When you execute:

```bash
ps -ef
```

It simply reads:

```
/proc
```

The kernel generates the data dynamically.

---

# Real Example

Suppose you run:

```bash
docker run nginx
```

Internally:

```
dockerd

↓

clone()

↓

New Process

↓

Namespaces

↓

cgroups

↓

exec()

↓

nginx
```

The container is ultimately just a Linux process with additional isolation.

---

# Why This Matters

Everything depends on Linux processes.

Examples:

- Java JVM
- Go Runtime
- Docker
- Kubernetes
- PostgreSQL
- Redis
- Nginx

Even containers begin as Linux processes.

---

# Hands-on Lab

## Lab 1 — View Running Processes

```bash
ps -ef
```

---

## Lab 2 — View Process Tree

```bash
pstree
```

---

## Lab 3 — View `/proc`

```bash
ls /proc
```

Observe numeric directories representing process IDs.

---

## Lab 4 — View Process Status

```bash
cat /proc/$$/status
```

`$$` represents the current shell's PID.

---

## Lab 5 — View Memory Map

```bash
cat /proc/$$/maps
```

Observe the process's virtual memory layout.

---

## Lab 6 — View Open File Descriptors

```bash
ls -l /proc/$$/fd
```

Observe stdin, stdout, stderr, and any additional open files.

---

# Interview Questions

## What is `task_struct`?

`task_struct` is the Linux kernel data structure that stores information about a running process.

---

## Why are `fork()` and `exec()` usually used together?

`fork()` creates a child process, and `exec()` replaces that child with a new program.

---

## What is `vfork()`?

`vfork()` is an optimized version of `fork()` that temporarily shares the parent's address space until the child calls `exec()` or exits.

---

## Why does Docker use `clone()` instead of only `fork()`?

Because `clone()` allows Docker to selectively share or isolate resources such as process IDs, networking, mounts, and user namespaces.

---

## What is the `/proc` filesystem?

`/proc` is a virtual filesystem generated by the kernel that exposes information about running processes and the system.

---

## Where does the `ps` command get process information?

`ps` reads process information from the `/proc` filesystem.

---

## Why are Zombie Processes created?

A Zombie Process exists when a child has exited but its parent has not yet collected its exit status using `wait()`.

---

# Summary

Linux represents every running program as a kernel-managed process.

```
Program File

↓

fork()

↓

Child Process

↓

exec()

↓

task_struct

↓

Running Process

↓

/proc
```

Key concepts:

- Programs become processes through `fork()` and `exec()`.
- Every process has a `task_struct` inside the kernel.
- Copy-on-Write makes `fork()` efficient.
- `clone()` provides fine-grained control over shared resources.
- `/proc` exposes process and kernel information dynamically.
- Docker containers are created using `clone()` along with namespaces and cgroups.

Understanding Linux process internals is the foundation for learning namespaces, cgroups, containers, and the Linux kernel's scheduling and isolation mechanisms.

---

# Next Chapter

## Chapter 20 — The `/proc` & `/sys` Filesystems: Inspecting the Linux Kernel at Runtime

We'll explore:

- Why `/proc` exists
- Virtual Filesystems
- `/proc` vs `/sys`
- Kernel Parameters
- `sysctl`
- Process Information
- Memory Information
- CPU Information
- Device Information
- Runtime Kernel Tuning
- Why Linux says "Everything is a File"