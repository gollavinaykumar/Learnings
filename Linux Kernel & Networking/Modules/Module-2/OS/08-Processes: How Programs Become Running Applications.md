Most developers think:

> "When I open Chrome, Chrome starts running."

That's true—but **what actually starts running?**

The answer is:

**A Process.**

A process is one of the most important concepts in Operating Systems.

Without processes:

- Chrome cannot run
- Docker cannot start containers
- PostgreSQL cannot serve queries
- Java cannot execute applications
- Node.js cannot start servers

Even the Linux kernel itself manages everything through processes.

After this chapter, you'll understand exactly how a program becomes a running process and how Linux manages thousands of processes simultaneously.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 8 — Processes: How Programs Become Running Applications

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a Process
> - Program vs Process
> - Process Creation
> - Process Life Cycle
> - Process States
> - Process Control Block (PCB)
> - Process ID (PID)
> - Parent & Child Processes
> - fork()
> - exec()
> - wait()
> - Zombie Processes
> - Orphan Processes
> - Daemon Processes
> - Process Scheduling Overview

---

# 📖 What is a Program?

A **Program** is simply a file stored on disk.

Examples:

```
Google Chrome

↓

/usr/bin/google-chrome
```

```
Docker

↓

/usr/bin/docker
```

```
Node.js

↓

/usr/bin/node
```

Programs are **not running**.

They simply exist as files.

---

# What is a Process?

A **Process** is a running instance of a program.

Example:

```
Program

↓

Chrome
```

After opening it:

```
Chrome

↓

Running Process
```

Think of it like this.

Blueprint:

```
Program
```

Actual House:

```
Process
```

A program is just instructions.

A process is those instructions actively executing.

---

# Program vs Process

| Program | Process |
|---------|----------|
| Stored on SSD | Running in RAM |
| Passive | Active |
| Static File | Executing Program |
| Doesn't use CPU | Uses CPU |
| Doesn't use RAM | Uses RAM |

---

# How Does a Program Become a Process?

Suppose you double-click Chrome.

```
Chrome File

↓

Operating System

↓

Create Process

↓

Allocate Memory

↓

Assign PID

↓

Schedule CPU

↓

Chrome Starts
```

The Operating System performs all these steps automatically.

---

# Process Memory Layout

Every process receives its own memory.

```
+----------------------+

Stack

+----------------------+

Heap

+----------------------+

Data

+----------------------+

Program Code

+----------------------+
```

Each process has its own isolated memory.

---

# Multiple Processes

Suppose you open:

- Chrome
- VS Code
- Spotify

Linux creates:

```
Chrome Process

VS Code Process

Spotify Process
```

Each process has:

- Its own memory
- Its own stack
- Its own heap
- Its own registers

Processes cannot directly access each other's memory.

---

# Process ID (PID)

Every process gets a unique number.

Example:

```
Chrome

PID = 2451
```

```
VS Code

PID = 3012
```

```
Docker

PID = 712
```

Linux identifies processes using PIDs.

---

# Parent Process

Processes usually create other processes.

Example:

```
Terminal

↓

Runs Python
```

```
Terminal

↓

Python Process
```

Terminal becomes the **Parent Process**.

Python becomes the **Child Process**.

---

# Parent-Child Relationship

```
Terminal

PID 100

↓

Python

PID 200

↓

Flask

PID 300
```

Processes form a tree.

---

# Process Tree

Example:

```
systemd (PID 1)

│

├── sshd

│

├── Docker

│

├── PostgreSQL

│

└── Terminal

       │

       ├── Bash

       │

       └── Python
```

Everything ultimately starts from **PID 1**.

---

# Process States

A process doesn't always execute.

It moves between states.

```
New

↓

Ready

↓

Running

↓

Waiting

↓

Ready

↓

Running

↓

Terminated
```

---

# New State

```
Program

↓

Process Created
```

Resources are being allocated.

---

# Ready State

The process is ready.

It only waits for CPU time.

```
Ready Queue

↓

Waiting
```

---

# Running State

The CPU is executing the process.

```
CPU

↓

Running
```

Only processes currently assigned to a CPU core are in this state.

---

# Waiting (Blocked)

Sometimes a process waits for something.

Examples:

- File Read
- Network Response
- User Input

```
Running

↓

Waiting
```

During this time,

the CPU executes another process.

---

# Terminated

The program finishes.

```
Running

↓

Exit

↓

Process Destroyed
```

The Operating System releases its resources.

---

# Process Life Cycle

```
New

↓

Ready

↓

Running

↓

Waiting

↓

Running

↓

Terminated
```

This cycle repeats for every application.

---

# Process Control Block (PCB)

Linux stores information about every process.

This information is called the **Process Control Block**.

```
PCB

├── PID

├── Process State

├── CPU Registers

├── Memory Information

├── Scheduling Information

├── Open Files

├── Parent PID

├── User ID
```

Think of it as the process's identity card.

---

# Why PCB is Important

Suppose Chrome stops running for a moment.

The Operating System saves:

```
Registers

↓

Program Counter

↓

Stack Pointer

↓

Memory Info
```

Later,

Chrome resumes exactly where it stopped.

This information comes from the PCB.

---

# Process Creation

In Linux,

new processes are usually created using:

```
fork()
```

---

# fork()

Suppose Bash runs:

```bash
python app.py
```

Internally:

```
Bash

↓

fork()

↓

Child Process
```

The child is almost an exact copy of the parent.

---

# fork() Example

```
Parent

↓

fork()

↓

Parent

+

Child
```

Both continue running independently.

---

# exec()

After fork(),

the child usually replaces itself with another program.

```
fork()

↓

Child

↓

exec()

↓

Python
```

Now the child becomes a Python process.

---

# Complete Process Creation

```
Terminal

↓

fork()

↓

Child

↓

exec()

↓

Python Starts
```

This is how Linux starts most applications.

---

# wait()

Suppose Bash starts Python.

Bash waits until Python finishes.

```
Terminal

↓

Python

↓

wait()

↓

Continue
```

Without `wait()`,

the parent would continue immediately.

---

# Zombie Process

Suppose a child finishes.

But the parent never collects its exit status.

```
Child Ends

↓

Still Listed

↓

Zombie
```

Zombie processes consume very little memory,

but they still occupy a PID.

---

# Orphan Process

Suppose the parent exits first.

```
Parent Dies

↓

Child Continues
```

Linux automatically assigns the child to:

```
systemd

(PID 1)
```

This process becomes an **Orphan Process**.

---

# Daemon Process

A daemon runs in the background.

Examples:

```
Docker

sshd

systemd

cron

nginx
```

They usually start during boot and keep running.

---

# Real Example

You run:

```bash
python app.py
```

Internally:

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

Allocate Memory

↓

Assign PID

↓

Ready Queue

↓

CPU

↓

Running
```

Everything happens automatically.

---

# Why Processes Matter

Everything in Linux is based on processes.

Examples:

- Web Servers
- Databases
- Docker Containers
- Kubernetes Pods
- SSH Sessions

Even Docker containers ultimately run processes.

Later you'll learn:

```
Container

↓

Namespaces

↓

cgroups

↓

Linux Process
```

Containers are **not virtual machines**.

They are isolated Linux processes.

---

# Hands-on Lab

## Lab 1 — View Running Processes

```bash
ps -ef
```

Observe hundreds of running processes.

---

## Lab 2 — View Process Tree

```bash
pstree
```

Observe parent-child relationships.

---

## Lab 3 — View Current Shell PID

```bash
echo $$
```

---

## Lab 4 — Start a Background Process

```bash
sleep 60 &
```

View it:

```bash
ps -ef | grep sleep
```

Kill it:

```bash
kill <PID>
```

---

## Lab 5 — Observe Process IDs

```bash
ps -eo pid,ppid,cmd
```

Observe PID and PPID.

---

# Interview Questions

## What is the difference between a Program and a Process?

A program is a file stored on disk.

A process is a running instance of that program.

---

## What is PID?

PID (Process ID) is a unique identifier assigned by the Operating System to every running process.

---

## What does `fork()` do?

`fork()` creates a new child process by duplicating the calling process.

---

## Why is `exec()` used after `fork()`?

`exec()` replaces the child process with a new program, allowing it to execute different code.

---

## What is a Zombie Process?

A Zombie Process is a terminated process whose parent has not yet collected its exit status.

---

## What is an Orphan Process?

An Orphan Process is a child process whose parent has terminated. Linux reassigns it to `systemd` (PID 1).

---

## What is a Daemon Process?

A Daemon Process is a background service that runs continuously, often starting during system boot.

---

# Summary

A process is a running program managed by the Operating System.

```
Program

↓

fork()

↓

Process

↓

Allocate Memory

↓

Assign PID

↓

Ready

↓

Running

↓

Waiting

↓

Running

↓

Terminated
```

Key concepts:

- A program becomes a process when executed.
- Every process has a unique PID.
- Linux stores process information in the PCB.
- `fork()` creates child processes.
- `exec()` loads a new program into a process.
- `wait()` synchronizes parent and child processes.
- Zombie and Orphan processes occur due to parent-child lifecycle behavior.
- Every Docker container ultimately runs one or more Linux processes.

Understanding processes is essential before learning threads, CPU scheduling, namespaces, containers, and the Linux kernel.

---

# Next Chapter

## Chapter 9 — Threads: How One Process Performs Multiple Tasks

We'll explore:

- What is a Thread
- Process vs Thread
- User Threads
- Kernel Threads
- Thread Scheduling
- Thread Synchronization
- Race Conditions
- Mutexes
- Semaphores
- Deadlocks
- Multithreading in Modern Applications