Most developers think:

> "The CPU executes my program."

That's true.

But here's the real question:

If your computer is running:

- Chrome
- Docker
- PostgreSQL
- VS Code
- Spotify
- Terminal
- Background Services

**Who decides which process gets the CPU?**

The CPU can execute only **one instruction per core at a time**.

So if thousands of processes want CPU time simultaneously,

who chooses?

The answer is:

**The Linux Scheduler.**

The scheduler is one of the most sophisticated parts of the Linux kernel.

It decides:

- Which process runs next
- How long it runs
- Which CPU core it runs on
- When it should stop
- How fairness is maintained

After this chapter, you'll understand how Linux efficiently shares CPU time among thousands of processes.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 25 — Linux Scheduling: How the Kernel Chooses the Next Process to Run

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Scheduling is needed
> - CPU Scheduler
> - Scheduling Classes
> - Completely Fair Scheduler (CFS)
> - Run Queue
> - Virtual Runtime (`vruntime`)
> - Time Slice
> - Context Switching
> - Nice Values
> - Real-Time Scheduling
> - CPU Affinity
> - Multi-Core Scheduling
> - Load Balancing

---

# 📖 Why Do We Need Scheduling?

Suppose your computer has:

```
1 CPU Core
```

Now imagine running:

```
Chrome

↓

Docker

↓

PostgreSQL

↓

VS Code

↓

Spotify

↓

Terminal
```

Can all of them execute simultaneously?

No.

A single CPU core executes only one instruction stream at a time.

The Operating System must decide:

```
Who Runs Next?
```

---

# What is the Scheduler?

The Scheduler is a kernel component that decides:

```
Ready Processes

↓

Choose One

↓

Assign CPU

↓

Switch Later

↓

Choose Next
```

It runs continuously.

---

# Scheduling Goal

The scheduler tries to balance:

✔ Fairness

✔ Performance

✔ Responsiveness

✔ Throughput

✔ Low Latency

No single process should monopolize the CPU.

---

# Process States

Remember:

```
Running

↓

Waiting

↓

Ready

↓

Running
```

Only **Ready** processes compete for CPU time.

---

# Run Queue

Every CPU maintains a queue of runnable processes.

```
CPU

↓

Run Queue

├── Chrome

├── Docker

├── PostgreSQL

└── VS Code
```

The scheduler selects one process from this queue.

---

# Time Slice

Linux does not allow one process to run forever.

Instead:

```
Process

↓

Runs

↓

Time Slice Ends

↓

Scheduler Runs

↓

Next Process
```

This creates the illusion that everything runs simultaneously.

---

# Context Switching

Suppose:

```
Chrome

↓

Docker
```

Switching requires saving Chrome's state and restoring Docker's state.

```
Save Registers

↓

Save Stack Pointer

↓

Save Program Counter

↓

Load Next Process

↓

Continue
```

This is called:

```
Context Switch
```

---

# Why Context Switching Is Expensive

A Context Switch requires:

- Saving CPU registers
- Loading another process's registers
- Switching memory mappings
- Updating scheduler data

Too many Context Switches reduce performance.

---

# Early Linux Scheduling

Older Linux versions used simple scheduling algorithms.

Problems:

❌ Some processes received too much CPU.

❌ Others waited too long.

Linux introduced a better scheduler.

---

# Completely Fair Scheduler (CFS)

Modern Linux uses:

```
CFS

Completely Fair Scheduler
```

Goal:

```
Every Process

↓

Fair CPU Share
```

Instead of fixed time slices,

CFS measures how much CPU each process has already received.

---

# Virtual Runtime (`vruntime`)

CFS tracks:

```
Virtual Runtime

(vruntime)
```

Meaning:

```
How Much CPU

Has This Process Used?
```

Smaller `vruntime` means:

```
Process Has Run Less

↓

Higher Priority
```

---

# Example

Suppose:

```
Chrome

vruntime = 5
```

```
Docker

vruntime = 2
```

Docker has received less CPU.

Therefore:

```
Docker Runs Next
```

---

# CFS Decision

Scheduler always chooses:

```
Smallest vruntime
```

This ensures fairness over time.

---

# Red-Black Tree

How does CFS quickly find the smallest `vruntime`?

It stores runnable processes in a:

```
Red-Black Tree
```

```
          20
         /  \
       10    40
      /      /
     5      30
```

The leftmost node has the smallest `vruntime`.

Finding the next process is efficient.

---

# CFS Flow

```
Ready Process

↓

Insert into Red-Black Tree

↓

Scheduler Picks Leftmost Node

↓

Process Runs

↓

Update vruntime

↓

Reinsert
```

This repeats continuously.

---

# Nice Values

Not all processes are equally important.

Linux allows adjusting priority using:

```
nice
```

Range:

```
-20

↓

Highest Priority
```

```
19

↓

Lowest Priority
```

Example:

```bash
nice -n 10 python app.py
```

---

# Effect of Nice Values

Lower nice value:

```
More CPU Time
```

Higher nice value:

```
Less CPU Time
```

CFS incorporates nice values into `vruntime` calculations.

---

# Scheduling Classes

Linux supports different scheduling policies.

```
Normal

↓

CFS
```

```
Real-Time

↓

FIFO

↓

Round Robin
```

Different workloads require different behavior.

---

# Real-Time Scheduling

Some applications cannot tolerate delays.

Examples:

- Robotics
- Medical Devices
- Industrial Control
- Audio Processing

Linux provides Real-Time Scheduling.

---

# FIFO Scheduler

```
First In

↓

First Out
```

The running process continues until:

- It blocks
- It exits
- A higher-priority real-time process arrives

No fairness like CFS.

---

# Round Robin Scheduler

Similar to FIFO,

but each process receives a fixed time quantum.

```
Process A

↓

Process B

↓

Process C

↓

Repeat
```

---

# CPU Affinity

Suppose your machine has:

```
8 CPU Cores
```

Linux may move processes between cores.

Sometimes you want:

```
Always Core 3
```

This is called:

```
CPU Affinity
```

---

# Why CPU Affinity?

Keeping a process on the same CPU improves:

✔ Cache Performance

✔ Predictable Latency

✔ Reduced Context Switching

---

# Multi-Core Scheduling

Modern CPUs have many cores.

Example:

```
Core 0

↓

Chrome
```

```
Core 1

↓

Docker
```

```
Core 2

↓

PostgreSQL
```

Each core has its own run queue.

---

# Load Balancing

Suppose:

```
Core 0

↓

20 Processes
```

```
Core 1

↓

2 Processes
```

Linux balances work.

```
Move Process

↓

Less Busy Core
```

This maximizes CPU utilization.

---

# Scheduler Tick

The kernel periodically checks:

```
Continue Running?

or

Switch Process?
```

Modern Linux also supports **tickless scheduling** in many situations to reduce unnecessary interrupts.

---

# Real Example

Suppose you're using:

- Chrome
- Docker
- PostgreSQL
- VS Code

Internally:

```
Run Queue

↓

CFS

↓

Choose Lowest vruntime

↓

CPU Executes

↓

Update vruntime

↓

Repeat
```

Thousands of times every second.

---

# Why Scheduling Matters

Everything depends on the scheduler.

Examples:

- Docker Containers
- Kubernetes Pods
- Java JVM
- Go Runtime
- Databases
- Web Servers

Efficient scheduling improves responsiveness and throughput.

---

# Hands-on Lab

## Lab 1 — View Running Processes

```bash
top
```

Observe CPU usage.

---

## Lab 2 — View Per-Core Usage

```bash
htop
```

Observe separate CPU cores and process placement.

---

## Lab 3 — Start a Low-Priority Process

```bash
nice -n 10 yes > /dev/null
```

Observe its scheduling behavior.

---

## Lab 4 — Change Process Priority

```bash
renice 5 <PID>
```

Adjust the nice value of a running process.

---

## Lab 5 — View CPU Affinity

```bash
taskset -p <PID>
```

---

## Lab 6 — Run on a Specific Core

```bash
taskset -c 2 ./app
```

Run the program only on CPU core 2.

---

# Interview Questions

## Why is Scheduling needed?

Because many processes compete for a limited number of CPU cores.

---

## What is the Completely Fair Scheduler (CFS)?

CFS is Linux's default scheduler for normal processes. It aims to distribute CPU time fairly by tracking each process's virtual runtime.

---

## What is `vruntime`?

`vruntime` (Virtual Runtime) represents how much CPU time a process has effectively received. CFS prefers the process with the smallest `vruntime`.

---

## Why does CFS use a Red-Black Tree?

A Red-Black Tree efficiently keeps processes sorted by `vruntime`, allowing the scheduler to quickly select the process that should run next.

---

## What is a Context Switch?

A Context Switch saves the state of one process and restores the state of another so the CPU can continue execution.

---

## What are Nice Values?

Nice values adjust the scheduling priority of normal processes, ranging from -20 (highest priority) to 19 (lowest priority).

---

## What is CPU Affinity?

CPU Affinity restricts a process to run on specific CPU cores, improving cache locality and predictability.

---

## What is the difference between CFS and FIFO scheduling?

CFS focuses on fairness for normal workloads, while FIFO is a real-time scheduling policy where a process runs until it blocks, exits, or is preempted by a higher-priority real-time task.

---

# Summary

The Linux Scheduler fairly distributes CPU time among runnable processes.

```
Ready Processes

↓

Run Queue

↓

CFS

↓

Lowest vruntime

↓

CPU

↓

Update vruntime

↓

Repeat
```

Key concepts:

- The scheduler decides which process runs next.
- Each CPU maintains a run queue.
- Context Switching enables multitasking.
- CFS is Linux's default scheduler for normal processes.
- `vruntime` measures a process's effective CPU usage.
- A Red-Black Tree allows efficient scheduling decisions.
- Nice values influence scheduling priority.
- Real-Time Scheduling supports latency-sensitive applications.
- CPU Affinity improves cache locality.
- Load Balancing distributes work across multiple CPU cores.

Understanding the scheduler is essential before studying memory management, networking, containers, and Linux kernel performance optimization.

---

# Next Chapter

## Chapter 26 — Linux Memory Management: Virtual Memory, Paging, Page Tables & Copy-on-Write

We'll explore:

- Physical vs Virtual Memory
- Address Spaces
- Page Tables
- Memory Mapping
- Paging
- Demand Paging
- Page Faults
- Copy-on-Write (COW)
- `mmap()`
- Huge Pages
- Swapping
- Out-of-Memory (OOM) Killer
- How Linux Efficiently Manages Memory