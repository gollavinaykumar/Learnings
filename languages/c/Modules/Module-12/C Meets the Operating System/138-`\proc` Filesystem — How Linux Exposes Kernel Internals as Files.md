# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 138 — `/proc` Filesystem — How Linux Exposes Kernel Internals as Files

---

# Most Linux users think:

```
/proc

↓

Folder
```

Actually...

```
/proc

↓

Not A Real Folder
```

Nothing inside `/proc` exists on your SSD.

There are:

- No disk blocks
- No real files
- No stored text

Yet you can run:

```bash
cat /proc/cpuinfo
```

and instantly see information about your CPU.

How?

Linux generates everything **on demand**.

---

# Learning Objectives

After this chapter you will understand:

- What `/proc` is
- procfs
- Virtual files
- Process directories
- `/proc/self`
- `/proc/cpuinfo`
- `/proc/meminfo`
- `/proc/<pid>/maps`
- `/proc/<pid>/fd`
- Kernel information exposure

---

# The Problem

Applications need information about:

- CPU
- Memory
- Running processes
- Open files
- Kernel parameters

Should Linux create a new API for every piece of information?

That would become difficult to learn and maintain.

---

# Linux Philosophy

Linux says:

> **Everything should look like a file whenever possible.**

Instead of:

```
Get CPU API

↓

Get Memory API

↓

Get Process API
```

Linux exposes information as:

```
Files
```

---

# What is `/proc`?

Definition:

> **`/proc` is a virtual filesystem that exposes kernel and process information through files.**

Filesystem type:

```
procfs
```

---

# Visualization

```
Application

↓

read()

↓

VFS

↓

procfs

↓

Kernel Data
```

Notice:

```
No SSD
```

---

# Why is `/proc` Amazing?

You already know:

```bash
cat file.txt
```

uses:

```
open()

↓

read()

↓

close()
```

Guess what?

```bash
cat /proc/cpuinfo
```

uses **exactly the same system calls**.

---

# The Difference

Normal File

```
Disk

↓

read()

↓

Application
```

`/proc`

```
Kernel Memory

↓

Generate Text

↓

Application
```

---

# Listing `/proc`

Run:

```bash
ls /proc
```

Example:

```
1

23

456

789

cpuinfo

meminfo

uptime

version

self
```

Why are some names numbers?

---

# Process Directories

Each running process gets its own directory.

Example:

```
PID 1452
```

Creates:

```text
/proc/1452
```

---

# Visualization

```
Process

↓

PID

↓

/proc/PID
```

Every process has one.

---

# Example

Suppose:

```bash
sleep 100
```

PID:

```
6421
```

Linux creates:

```text
/proc/6421
```

---

# `/proc/self`

Special shortcut:

```text
/proc/self
```

Always means:

```
Current Process
```

Example:

Program:

```c
open("/proc/self/status");
```

Linux automatically redirects to the current PID.

---

# `/proc/cpuinfo`

Run:

```bash
cat /proc/cpuinfo
```

Output:

```
Processor

↓

Vendor

↓

Model

↓

Frequency

↓

Cache

↓

Flags
```

Linux generates this information dynamically.

---

# Visualization

```
CPU Registers

↓

Kernel

↓

Text

↓

cat
```

---

# `/proc/meminfo`

Run:

```bash
cat /proc/meminfo
```

Example:

```
MemTotal

↓

MemFree

↓

Cached

↓

Buffers

↓

Swap
```

Again:

```
Generated

↓

Not Stored
```

---

# `/proc/uptime`

Run:

```bash
cat /proc/uptime
```

Example:

```
10432.24

↓

How Long Linux Has Been Running
```

No file exists on disk.

---

# `/proc/version`

Shows:

```
Linux Version

↓

Compiler

↓

Build Information
```

Generated directly from kernel data.

---

# `/proc/<pid>/status`

Example:

```bash
cat /proc/1234/status
```

Shows:

```
PID

↓

Parent PID

↓

Memory Usage

↓

Threads

↓

UID

↓

State
```

---

# Visualization

```
Kernel Task Structure

↓

Format Text

↓

Return
```

---

# `/proc/<pid>/maps`

One of the most useful files.

Shows:

```
Virtual Memory Layout
```

Example:

```
.text

↓

Heap

↓

Stack

↓

Shared Libraries

↓

mmap()
```

Exactly what you learned in previous chapters.

---

# `/proc/<pid>/fd`

Contains:

```
Open File Descriptors
```

Example:

```
0

↓

stdin

--------------------

1

↓

stdout

--------------------

2

↓

stderr

--------------------

3

↓

notes.txt
```

---

# Visualization

```
Kernel FD Table

↓

procfs

↓

Directory Entries
```

---

# `/proc/<pid>/cmdline`

Shows:

```
Program Arguments
```

Example:

```bash
python server.py
```

---

# `/proc/<pid>/exe`

Interesting file.

It's a symbolic link pointing to:

```
Running Executable
```

Example:

```bash
ls -l /proc/1234/exe
```

---

# `/proc/<pid>/cwd`

Shows:

```
Current Working Directory
```

---

# `/proc/sys`

Contains:

```
Kernel Parameters
```

Example:

```bash
/proc/sys/net
```

Networking configuration.

Many values can be changed.

---

# Example

Read:

```bash
cat /proc/sys/kernel/hostname
```

Change:

```bash
echo myhost > /proc/sys/kernel/hostname
```

(Requires appropriate privileges.)

---

# Relationship with VFS

Remember:

```
Application

↓

VFS

↓

procfs

↓

Kernel Structures
```

Applications never talk directly to kernel structures.

---

# How procfs Works

Suppose:

```bash
cat /proc/meminfo
```

Kernel performs:

```
Read Request

↓

Generate Current Values

↓

Return Text
```

Nothing is stored permanently.

---

# Real-World Example — `ps`

Command:

```bash
ps
```

Internally reads:

```
/proc/<pid>
```

for every running process.

---

# Real-World Example — `top`

Refreshes:

- CPU usage
- Memory usage
- Process states

by repeatedly reading `/proc`.

---

# Real-World Example — Docker

Docker inspects:

```
/proc

↓

Namespaces

↓

Mounts

↓

Processes
```

to understand container state.

---

# Real-World Example — Monitoring Tools

Utilities like:

- `htop`
- `vmstat`
- `free`

obtain much of their information from `/proc`.

---

# Common Mistakes

---

## Thinking `/proc` Exists on Disk

It doesn't.

It is generated dynamically.

---

## Thinking `/proc` Uses Special APIs

Applications simply use:

```
open()

↓

read()

↓

close()
```

---

## Assuming `/proc` Never Changes

Its contents may change every time you read them,

because they reflect the current kernel state.

---

## Forgetting Process Directories Disappear

When a process exits,

its `/proc/<pid>` directory disappears.

---

# Hands-on Labs

## Lab 1

Run:

```bash
ls /proc
```

Observe numbered process directories.

---

## Lab 2

Read:

```bash
cat /proc/cpuinfo
```

---

## Lab 3

Inspect:

```bash
cat /proc/meminfo
```

---

## Lab 4

Run:

```bash
ls -l /proc/self/fd
```

Observe open file descriptors.

---

## Lab 5

Inspect your process memory layout:

```bash
cat /proc/self/maps
```

Relate it to previous chapters on virtual memory.

---

# Interview Questions

### What is `/proc`?

A virtual filesystem exposing kernel and process information as files.

---

### Does `/proc` store real files?

No.

Its contents are generated dynamically by the kernel.

---

### What is `/proc/self`?

A symbolic reference to the current running process.

---

### What does `/proc/<pid>/fd` contain?

Information about the process's open file descriptors.

---

### Why does Linux expose kernel information through files?

It provides a simple, consistent interface that applications can access using ordinary file operations.

---

# Summary

Architecture:

```
Application

↓

open()

↓

read()

↓

VFS

↓

procfs

↓

Kernel Structures
```

Useful Files:

| Path | Purpose |
|------|----------|
| `/proc/cpuinfo` | CPU details |
| `/proc/meminfo` | Memory statistics |
| `/proc/uptime` | System uptime |
| `/proc/version` | Kernel version |
| `/proc/self` | Current process |
| `/proc/<pid>/status` | Process information |
| `/proc/<pid>/maps` | Virtual memory layout |
| `/proc/<pid>/fd` | Open file descriptors |
| `/proc/sys` | Kernel parameters |

## Key Takeaways

- `/proc` is a virtual filesystem implemented by `procfs`.
- Its contents are generated dynamically from kernel data structures.
- Applications access `/proc` using ordinary file APIs.
- Every running process has its own directory under `/proc`.
- `/proc` is the foundation of many Linux monitoring and administration tools.
- Understanding `/proc` provides deep insight into how Linux exposes internal state.

---

# Next Chapter

## Chapter 139 — `/sys` Filesystem (`sysfs`) — How Linux Represents Hardware as Files

You'll learn:

- What `sysfs` is
- The Linux device model
- `/sys/class`
- `/sys/block`
- `/sys/devices`
- `/sys/bus`
- Kernel objects (`kobjects`)
- Device drivers
- Hotplug events
- How Linux exposes hardware through a virtual filesystem