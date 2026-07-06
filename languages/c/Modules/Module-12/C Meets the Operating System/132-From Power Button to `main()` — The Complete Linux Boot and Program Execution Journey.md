# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 132 — From Power Button to `main()` — The Complete Linux Boot and Program Execution Journey

---

# Most developers know:

```bash
./app
```

Program starts.

That's it.

But what actually happened?

How did electricity become a running process?

How did your code reach:

```c
int main()
{
}
```

This chapter connects **everything** you've learned so far into one complete journey.

---

# Learning Objectives

After this chapter you will understand:

- What happens when you press the power button
- CPU reset
- BIOS / UEFI
- Bootloader
- Linux Kernel startup
- Kernel initialization
- `systemd`
- Login process
- Shell startup
- `fork()`
- `execve()`
- ELF loading
- Dynamic linker
- `_start`
- `main()`

---

# The Complete Journey

```
Power Button

↓

CPU Reset

↓

BIOS / UEFI

↓

Bootloader (GRUB)

↓

Linux Kernel

↓

Kernel Initialization

↓

systemd

↓

Login

↓

Shell

↓

./program

↓

fork()

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
```

Every Linux program follows this path.

---

# Step 1 — Power Button

You press:

```
Power
```

Motherboard supplies power to:

- CPU
- RAM
- SSD
- GPU
- Devices

CPU immediately resets.

---

# Step 2 — CPU Reset

After reset,

the CPU knows almost nothing.

Its instruction pointer is set to a predefined address.

Example:

```
Reset Vector
```

CPU begins executing firmware.

---

# Visualization

```
Power

↓

CPU Reset

↓

Firmware
```

---

# Step 3 — BIOS / UEFI

Firmware initializes hardware.

Checks:

- RAM
- Keyboard
- Disk
- USB
- PCI Devices

This is called:

```
POST

Power-On Self Test
```

---

# Firmware Responsibilities

```
Detect Hardware

↓

Initialize CPU

↓

Initialize RAM

↓

Find Boot Device
```

---

# Step 4 — Bootloader

Firmware loads:

```
GRUB

(or another bootloader)
```

Bootloader's job:

```
Load Linux Kernel

↓

Load initramfs

↓

Transfer Control
```

---

# Visualization

```
UEFI

↓

GRUB

↓

Linux Kernel
```

---

# Step 5 — Linux Kernel Starts

Kernel is now executing.

Kernel initializes:

- Memory manager
- Scheduler
- Interrupts
- Device drivers
- Filesystems
- Networking

Linux is becoming a complete operating system.

---

# Step 6 — Kernel Initialization

Kernel creates important subsystems.

Example:

```
Memory

↓

Scheduler

↓

Virtual Memory

↓

IPC

↓

Networking
```

---

# First Process

Kernel creates:

```
PID 1
```

Historically:

```
init
```

Today:

```
systemd
```

---

# Why PID 1 Matters

Every other user process ultimately descends from PID 1.

Visualization:

```
systemd

↓

Services

↓

Login

↓

Shell

↓

Applications
```

---

# Step 7 — `systemd`

Responsibilities:

- Start services
- Mount filesystems
- Configure networking
- Launch login managers
- Manage daemons

Examples:

```
SSH

↓

Docker

↓

Cron

↓

NetworkManager
```

---

# Step 8 — Login

User enters:

```
Username

Password
```

Authentication succeeds.

Linux starts:

```
User Shell
```

Example:

```
bash

zsh

fish
```

---

# Step 9 — Shell

Prompt:

```bash
vinay@linux:~$
```

Shell is itself:

```
A Process
```

Waiting for commands.

---

# You Type

```bash
./hello
```

Shell begins execution.

---

# Step 10 — `fork()`

Shell creates a child process.

```
Shell

↓

fork()

↓

Child
```

Parent shell continues existing.

---

# Why `fork()`?

Because the shell should remain alive after the program exits.

Visualization:

```
Shell

↓

fork()

↓

Shell

+

Child
```

---

# Step 11 — `execve()`

Child process executes:

```c
execve()
```

Purpose:

```
Replace Current Program

↓

With

↓

hello
```

Everything except the process identity is replaced.

---

# Visualization

Before:

```
Child

↓

bash
```

After:

```
Child

↓

hello
```

Same PID.

Different program.

---

# Step 12 — Kernel Reads ELF

Kernel:

```
Open File

↓

Read ELF Header

↓

Read Program Headers
```

Checks:

- Valid executable
- Correct architecture
- Permissions

---

# Step 13 — Memory Mapping

Kernel creates process memory.

```
.text

↓

.rodata

↓

.data

↓

.bss

↓

Heap

↓

Stack
```

Most executable segments are mapped using:

```
mmap()
```

---

# Step 14 — Shared Libraries

Executable requires:

```
libc.so

↓

libpthread.so

↓

libm.so
```

Kernel starts the dynamic loader.

---

# Step 15 — Dynamic Linker

Loader:

```
Find Libraries

↓

Map Libraries

↓

Resolve Symbols

↓

Prepare Process
```

---

# Step 16 — `_start`

Execution begins at:

```
_start
```

Not:

```
main()
```

---

# `_start` Responsibilities

```
Setup Stack

↓

Initialize C Runtime

↓

Environment Variables

↓

Arguments

↓

Constructors

↓

Call main()
```

---

# Finally...

Execution reaches:

```c
int main()
{
    printf("Hello");
}
```

Your program finally starts.

---

# What Happens During `printf()`?

```
main()

↓

printf()

↓

PLT

↓

GOT

↓

libc.so

↓

write()

↓

Kernel

↓

Terminal
```

Everything you've learned connects together.

---

# Program Exit

Suppose:

```c
return 0;
```

Flow:

```
main()

↓

exit()

↓

Kernel

↓

Release Memory

↓

Close Files

↓

Wake Parent

↓

Shell Continues
```

---

# Complete Timeline

```
Power

↓

Firmware

↓

Bootloader

↓

Kernel

↓

systemd

↓

Shell

↓

fork()

↓

execve()

↓

ELF

↓

Dynamic Loader

↓

_start

↓

main()

↓

exit()
```

---

# Real-World Example — Chrome

Power On

↓

Linux

↓

systemd

↓

Desktop

↓

Chrome

↓

Thousands of Processes

↓

Thousands of Threads

↓

Millions of System Calls

---

# Real-World Example — Docker

```
Docker CLI

↓

fork()

↓

execve()

↓

Container Runtime

↓

Namespaces

↓

cgroups

↓

Container Process
```

All built on the same Linux process model.

---

# Real-World Example — VS Code

```
Desktop

↓

Electron

↓

Node.js

↓

libuv

↓

epoll

↓

Kernel
```

Every application follows the same fundamental execution path.

---

# Complete Architecture

```
Power

↓

Firmware

↓

Bootloader

↓

Linux Kernel

↓

PID 1 (systemd)

↓

Shell

↓

fork()

↓

execve()

↓

ELF

↓

Dynamic Linker

↓

main()
```

---

# Common Mistakes

---

## Thinking `main()` Starts the Program

Execution actually begins at:

```
_start
```

---

## Thinking the Kernel Calls `main()`

The C runtime startup code calls `main()` after initialization.

---

## Forgetting the Dynamic Linker

Shared libraries are resolved before normal program execution begins.

---

## Thinking `fork()` Starts a New Program

`fork()` duplicates the current process.

`execve()` replaces the duplicated process with a new program.

---

# Hands-on Labs

## Lab 1

Compile a simple program.

Run:

```bash
readelf -h hello
```

Identify the entry point.

---

## Lab 2

Use:

```bash
objdump -d hello
```

Locate:

```
_start
```

---

## Lab 3

Run:

```bash
strace ./hello
```

Observe:

- `execve()`
- library loading
- system calls

---

## Lab 4

Run:

```bash
ps -ef
```

Observe:

```
systemd

↓

Shell

↓

Program
```

---

## Lab 5

Draw the complete execution timeline from power button to `main()` without referring to notes.

---

# Interview Questions

### What happens after pressing the power button?

Firmware initializes hardware, loads the bootloader, which loads the Linux kernel.

---

### Which process is PID 1?

Typically:

```
systemd
```

---

### Why does the shell call `fork()`?

To create a child process while keeping the shell itself running.

---

### What does `execve()` do?

Replaces the current process image with a new executable.

---

### Does Linux jump directly to `main()`?

No.

Execution begins at `_start`, which initializes the runtime before calling `main()`.

---

# Summary

Complete Journey:

```
Power Button

↓

CPU Reset

↓

BIOS / UEFI

↓

GRUB

↓

Linux Kernel

↓

systemd

↓

Shell

↓

fork()

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

exit()
```

Major Concepts Connected:

| Stage | Concept |
|--------|----------|
| Boot | Firmware, Bootloader |
| Kernel | Memory, Scheduler, Drivers |
| User Space | systemd, Shell |
| Process | `fork()`, `execve()` |
| Executable | ELF |
| Runtime | Dynamic Linker |
| Startup | `_start` |
| Your Code | `main()` |

## Key Takeaways

- A Linux program's journey begins long before `main()`.
- Firmware, bootloader, and kernel cooperate to create the operating system.
- `systemd` starts user-space services and login sessions.
- The shell uses `fork()` and `execve()` to launch programs.
- The kernel loads ELF executables and the dynamic linker resolves shared libraries.
- `_start` initializes the runtime before invoking your `main()` function.

---

# 🎉 Module 12 Complete — C Meets the Operating System

You now have a deep understanding of:

### Process Management
- ✅ Processes
- ✅ `fork()`
- ✅ `execve()`
- ✅ Signals
- ✅ Scheduling

### Inter-Process Communication (IPC)
- ✅ Pipes
- ✅ FIFOs
- ✅ Unix Domain Sockets
- ✅ Shared Memory

### Synchronization
- ✅ Semaphores
- ✅ Mutexes
- ✅ Condition Variables
- ✅ Read-Write Locks

### Concurrency
- ✅ Threads
- ✅ Thread Pools

### High-Performance Linux
- ✅ `epoll`
- ✅ Event Loops
- ✅ `io_uring`
- ✅ `mmap`

### Executables
- ✅ ELF
- ✅ Shared Libraries
- ✅ Dynamic Linking
- ✅ PLT / GOT

### System Startup
- ✅ Power Button → `main()`

You now understand how Linux executes programs from the hardware level all the way to your C code.

---

# Next Module

# Module 13 — Advanced Linux Programming & Kernel Interfaces

You'll move beyond application programming and learn how user-space interacts with the Linux kernel in depth.

Topics include:

- System calls in detail
- Virtual File System (VFS)
- `/proc` and `/sys`
- `ioctl()`
- `ptrace()`
- Capabilities
- Namespaces
- cgroups
- `seccomp`
- `inotify`
- `eventfd`
- `timerfd`
- `signalfd`
- Netlink sockets
- eBPF fundamentals
- Advanced kernel-user communication

This module bridges expert C programming with Linux kernel internals.