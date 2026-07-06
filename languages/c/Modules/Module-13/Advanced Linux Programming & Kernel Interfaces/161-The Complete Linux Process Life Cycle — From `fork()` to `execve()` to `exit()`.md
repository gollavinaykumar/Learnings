# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 161 — The Complete Linux Process Life Cycle — From `fork()` to `execve()` to `exit()`

---

# This is one of the most important chapters in Linux.

Every process you've ever started—

- Chrome
- VS Code
- Docker
- PostgreSQL
- Node.js
- Python
- Java
- Bash

—all follow nearly the same life cycle.

```
Created

↓

Runs

↓

Executes Program

↓

Uses CPU

↓

Terminates
```

But...

**What actually happens inside the Linux kernel?**

This chapter follows one process from birth until death.

---

# Learning Objectives

After this chapter you will understand:

- Process creation
- `fork()`
- Copy-on-Write (CoW)
- `execve()`
- ELF loading
- Dynamic linker
- Scheduler
- Signals
- `exit()`
- Zombie processes
- Process cleanup

---

# The Big Picture

Every Linux process follows this journey:

```
fork()

↓

Child Process

↓

execve()

↓

Program Loaded

↓

Running

↓

exit()

↓

Zombie

↓

wait()

↓

Destroyed
```

Let's study every step.

---

# Step 1 — Parent Process

Suppose:

```
bash
```

is running.

Process Tree:

```
systemd

↓

bash
```

Now you type:

```bash
ls
```

Question:

Does Bash become `ls`?

No.

---

# Linux Solution

Bash creates:

```
Child Process
```

using:

```c
fork();
```

---

# Step 2 — `fork()`

Definition:

> **`fork()` creates a new child process by duplicating the calling process.**

Important:

```
Parent

↓

Still Exists
```

New child is created.

---

# Visualization

Before:

```
bash
```

After:

```
bash

↓

Child
```

Two processes now exist.

---

# Step 3 — Process Control Block

Kernel allocates:

```
task_struct
```

for the child.

Contains:

- PID
- State
- Registers
- Scheduling info
- Memory references
- File descriptors

Everything required to manage the process.

---

# Step 4 — Copy-on-Write

Question:

Does Linux copy:

```
Entire Memory?
```

No.

That would be slow.

Instead:

```
Parent Pages

↓

Shared

↓

Read Only
```

---

# Visualization

```
Parent

↓

RAM

↑

Child
```

Initially,

both processes share the same physical pages.

---

# Step 5 — Copy-on-Write Trigger

Suppose child writes:

```c
x = 10;
```

Kernel notices:

```
Shared Page

↓

Write Attempt
```

Now:

```
Allocate New Page

↓

Copy Data

↓

Continue
```

Only modified pages are copied.

---

# Visualization

Before Write:

```
Parent

↓

Page

↑

Child
```

After Write:

```
Parent

↓

Page A

Child

↓

Page B
```

---

# Step 6 — `fork()` Return Value

Parent receives:

```
Child PID
```

Child receives:

```
0
```

This allows:

```c
if (fork() == 0)
```

to distinguish parent and child execution.

---

# Step 7 — `execve()`

Child now calls:

```c
execve("/bin/ls", ...)
```

This is the most misunderstood step.

---

# Important

`execve()` does **not** create a process.

It replaces:

```
Current Program
```

inside the existing process.

PID remains the same.

---

# Visualization

Before:

```
PID 500

↓

bash
```

After:

```
PID 500

↓

ls
```

Same process.

Different program.

---

# Step 8 — Old Address Space Removed

Kernel destroys:

```
Old Code

↓

Old Heap

↓

Old Stack

↓

Old Data
```

Process becomes empty.

---

# Step 9 — ELF Loader

Kernel reads:

```
ELF File
```

Example:

```
/bin/ls
```

Parses:

- ELF header
- Program headers
- Memory layout

---

# Step 10 — Memory Mapping

Kernel creates:

```
Text Segment

↓

Data Segment

↓

BSS

↓

Heap

↓

Stack
```

New virtual address space.

---

# Visualization

```
Text

↓

Data

↓

BSS

↓

Heap

↓

Stack
```

---

# Step 11 — Dynamic Linker

Suppose binary is dynamically linked.

Kernel starts:

```
ld-linux
```

Dynamic linker loads:

```
libc

↓

libpthread

↓

Other Shared Libraries
```

---

# Step 12 — Program Entry

Eventually execution reaches:

```
_start
```

Not:

```
main()
```

`_start` prepares the runtime.

Only then:

```
main()
```

begins.

---

# Step 13 — Running

Process now enters:

```
Scheduler
```

Linux repeatedly performs:

```
Run

↓

Pause

↓

Run

↓

Pause
```

Until program finishes.

---

# Step 14 — Signals

During execution,

process may receive:

```
SIGINT

↓

SIGTERM

↓

SIGUSR1

↓

SIGCHLD
```

Kernel delivers them according to signal rules.

---

# Step 15 — File Operations

Program may call:

```
open()

↓

read()

↓

write()

↓

close()
```

Kernel performs system calls.

---

# Step 16 — Memory Allocation

Program calls:

```c
malloc()
```

Eventually kernel may expand memory using mechanisms such as:

```
brk()

↓

mmap()
```

depending on allocator behavior.

---

# Step 17 — Exit

Program finishes:

```c
return 0;
```

or

```c
exit(0);
```

Kernel begins termination.

---

# Step 18 — Cleanup

Kernel releases:

- Memory mappings
- Page tables
- Open files (reference counts updated)
- Signal handlers
- Timers
- Other process resources

---

# Step 19 — Zombie

Important.

Process is **not immediately destroyed**.

Kernel keeps:

```
PID

↓

Exit Status

↓

Accounting Information
```

State:

```
Zombie
```

---

# Why?

Parent still needs to know:

```
Did Child

↓

Succeed?
```

---

# Visualization

```
Child

↓

Finished

↓

Zombie

↓

wait()
```

---

# Step 20 — `wait()`

Parent calls:

```c
wait()
```

Kernel returns:

```
Exit Status
```

Now zombie disappears.

---

# Final Step

Kernel removes:

```
task_struct

↓

PID

↓

Scheduler Entry
```

Process no longer exists.

---

# Complete Life Cycle

```
fork()

↓

Child

↓

Copy-on-Write

↓

execve()

↓

ELF Loader

↓

Dynamic Linker

↓

_start

↓

main()

↓

Scheduler

↓

System Calls

↓

Signals

↓

exit()

↓

Zombie

↓

wait()

↓

Destroyed
```

---

# Relationship with the Scheduler

Process alternates between:

```
Running

↓

Runnable

↓

Sleeping

↓

Running
```

Until exit.

---

# Relationship with Virtual Memory

Every process receives:

```
Own

↓

Virtual Address Space
```

Even if physical pages are shared initially through Copy-on-Write.

---

# Relationship with File Descriptors

After `fork()`:

```
Parent

↓

FD Table

↑

Child
```

File descriptions are shared,

while descriptor tables are duplicated.

---

# Relationship with `execve()`

`execve()` replaces:

- Code
- Data
- Stack
- Heap

while preserving:

- PID
- Some process attributes
- Open file descriptors (unless marked close-on-exec)

---

# Relationship with Signals

Signals may interrupt the process at various points during its lifetime.

---

# Real-World Example — Bash

```
bash

↓

fork()

↓

Child

↓

execve()

↓

ls

↓

exit()

↓

wait()

↓

Continue
```

---

# Real-World Example — Web Server

Master:

```
fork()

↓

Workers

↓

Accept Requests
```

---

# Real-World Example — Docker

Container runtime:

```
fork()

↓

Namespaces

↓

execve()

↓

Application
```

---

# Real-World Example — systemd

```
systemd

↓

fork()

↓

execve()

↓

Daemon
```

---

# Common Mistakes

---

## Thinking `fork()` Loads a New Program

It only creates a child process.

`execve()` loads the new program.

---

## Thinking `execve()` Creates a Process

It replaces the current program inside an existing process.

---

## Thinking Zombies Consume CPU

Zombie processes have already finished execution.

They only retain minimal process information until the parent collects their status.

---

## Forgetting Copy-on-Write

Linux does not immediately duplicate all process memory during `fork()`.

---

# Hands-on Labs

## Lab 1

Run:

```bash
ps -ef
```

Observe parent-child relationships.

---

## Lab 2

Read:

```bash
man 2 fork
```

---

## Lab 3

Read:

```bash
man 2 execve
```

---

## Lab 4

Read:

```bash
man 2 wait
```

---

## Lab 5

Write a simple C program using:

- `fork()`
- `execve()`
- `wait()`

Trace it with:

```bash
strace
```

Observe the complete life cycle.

---

# Interview Questions

### What does `fork()` do?

Creates a new child process by duplicating the calling process.

---

### Does `execve()` create a new process?

No.

It replaces the current program inside the existing process.

---

### Why is Copy-on-Write important?

It avoids copying memory until either process modifies a shared page.

---

### What is a zombie process?

A terminated process whose exit status has not yet been collected by its parent.

---

### Why does the parent call `wait()`?

To collect the child's termination status and allow the kernel to fully remove the process.

---

# Summary

Complete Process Life Cycle:

```
Parent

↓

fork()

↓

Child

↓

Copy-on-Write

↓

execve()

↓

ELF

↓

Dynamic Linker

↓

_start

↓

main()

↓

Scheduler

↓

System Calls

↓

Signals

↓

exit()

↓

Zombie

↓

wait()

↓

Destroyed
```

Major Components:

| Stage | Purpose |
|--------|---------|
| `fork()` | Create child process |
| Copy-on-Write | Efficient memory sharing |
| `execve()` | Load new program |
| ELF Loader | Load executable |
| Dynamic Linker | Load shared libraries |
| Scheduler | Execute process |
| `exit()` | Begin termination |
| Zombie | Preserve exit status |
| `wait()` | Final cleanup |

## Key Takeaways

- Every Linux process follows a predictable life cycle.
- `fork()` creates a new process; `execve()` replaces the program inside it.
- Copy-on-Write makes `fork()` efficient.
- The ELF loader and dynamic linker prepare the process before `main()` executes.
- Zombie processes exist until the parent collects their exit status.
- Understanding the complete process life cycle is fundamental to Linux systems programming, debugging, and kernel development.

---

# 🎉 Major Milestone

You have now completed **161 chapters**.

At this point you understand:

- Linux process management
- Virtual memory
- System calls
- ELF loading
- Scheduling
- Networking
- Storage I/O
- Containers
- eBPF
- XDP
- AF_XDP
- io_uring
- Complete packet flow
- Complete file I/O flow
- Complete process life cycle

You are now entering territory typically studied by **Linux kernel developers, performance engineers, database engineers, and cloud infrastructure engineers**.

---

# Next Chapter

## Chapter 162 — The Complete Linux Boot Process — From Power Button to `main()`

You'll learn:

- BIOS vs UEFI
- Firmware initialization
- Bootloader (GRUB)
- Linux kernel decompression
- Early kernel initialization
- initramfs
- Mounting the root filesystem
- Starting `systemd`
- Launching login services
- How your program finally becomes the first user application after boot