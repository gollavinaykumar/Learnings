Most developers think:

> "When I open Chrome, the CPU immediately runs Chrome."

But what if, at the same time, the computer is also running:

- VS Code
- Docker
- PostgreSQL
- Spotify
- Chrome
- Zoom
- Terminal

Which application gets the CPU first?

Who decides?

The answer is:

**The CPU Scheduler.**

The scheduler is one of the most important parts of every Operating System.

It decides **who runs, for how long, and when to switch**.

Without scheduling:

- Your computer would freeze.
- Some programs would never run.
- Interactive applications would feel extremely slow.

After this chapter, you'll understand how Linux schedules processes and why modern schedulers are incredibly sophisticated.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 10 — CPU Scheduling: How Linux Decides Which Process Runs Next

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why CPU Scheduling is needed
> - Scheduler Goals
> - Ready Queue
> - Dispatcher
> - CPU Burst & I/O Burst
> - Context Switching
> - FCFS Scheduling
> - Shortest Job First (SJF)
> - Shortest Remaining Time First (SRTF)
> - Round Robin (RR)
> - Priority Scheduling
> - Starvation
> - Aging
> - Multi-Level Queue
> - Multi-Level Feedback Queue (MLFQ)
> - Linux Completely Fair Scheduler (CFS)
> - Real-Time Scheduling
> - CPU Affinity

---

# 📖 Why Do We Need CPU Scheduling?

Imagine your laptop has:

```
8 CPU Cores
```

Now suppose:

```
500 Processes
```

are running.

Examples:

- Chrome
- VS Code
- Docker
- PostgreSQL
- Terminal
- Spotify
- Background Services

Can 500 processes run simultaneously?

No.

Only a limited number can execute at the same time.

The Operating System must decide:

```
Who runs?

Who waits?

Who runs next?
```

This decision is called **CPU Scheduling**.

---

# The Scheduler

The Scheduler is part of the Operating System Kernel.

```
Applications

↓

Ready Queue

↓

Scheduler

↓

CPU
```

Every process must pass through the scheduler before reaching the CPU.

---

# Scheduler Goals

A good scheduler tries to:

✔ Keep the CPU busy

✔ Minimize waiting time

✔ Improve response time

✔ Maximize throughput

✔ Prevent starvation

✔ Share CPU fairly

No scheduler is perfect.

Different workloads require different strategies.

---

# CPU Burst & I/O Burst

Processes don't always use the CPU.

Example:

```
Chrome

↓

CPU

↓

Wait for Network

↓

CPU

↓

Wait for User

↓

CPU
```

A process alternates between:

```
CPU Burst

↓

I/O Burst

↓

CPU Burst
```

During I/O,

another process can use the CPU.

---

# Ready Queue

Suppose five processes are ready.

```
Chrome

↓

VS Code

↓

Docker

↓

Spotify

↓

PostgreSQL
```

They wait inside the Ready Queue.

```
+---------------------+

Ready Queue

Chrome

VS Code

Docker

Spotify

PostgreSQL

+---------------------+
```

The scheduler chooses one.

---

# Dispatcher

The Dispatcher performs the actual switch.

```
Scheduler

↓

Dispatcher

↓

CPU
```

Responsibilities:

- Load process state
- Restore registers
- Start execution

---

# Context Switching

Suppose Chrome is running.

Then Docker needs CPU.

```
Chrome

↓

Save State

↓

Docker

↓

Load State

↓

Run
```

This switching is called:

```
Context Switch
```

---

# Scheduling Algorithms

Many algorithms exist.

Examples:

```
FCFS

SJF

Round Robin

Priority

CFS
```

Let's study each one.

---

# First Come First Serve (FCFS)

The simplest algorithm.

Rule:

```
Who arrives first,

runs first.
```

Example:

```
Arrival Order

Chrome

↓

Docker

↓

VS Code
```

Execution:

```
Chrome

↓

Docker

↓

VS Code
```

---

# FCFS Example

```
Chrome

10 seconds

↓

Docker

2 seconds

↓

VS Code

1 second
```

VS Code waits 12 seconds,

even though it only needs 1 second.

---

# FCFS Advantages

✔ Very simple

✔ Easy to implement

---

# FCFS Disadvantages

❌ Long jobs block short jobs

❌ Poor response time

❌ Interactive applications suffer

---

# Shortest Job First (SJF)

Rule:

```
Run the shortest job first.
```

Example:

```
Chrome

10 sec

VS Code

1 sec

Docker

2 sec
```

Execution:

```
VS Code

↓

Docker

↓

Chrome
```

Average waiting time improves.

---

# SJF Problem

How does the Operating System know:

```
How long a process will run?
```

Usually,

it doesn't.

Therefore,

pure SJF is rarely used.

---

# Shortest Remaining Time First (SRTF)

SRTF is the preemptive version of SJF.

Example:

```
Chrome

Running

↓

VS Code Arrives

↓

VS Code Shorter

↓

Switch Immediately
```

The scheduler interrupts Chrome.

---

# Round Robin (RR)

Round Robin gives every process equal CPU time.

Suppose:

```
Time Slice

=

100 ms
```

Execution:

```
Chrome

100 ms

↓

Docker

100 ms

↓

VS Code

100 ms

↓

Spotify

100 ms

↓

Repeat
```

Everyone gets a turn.

---

# Time Quantum

The time allocated to one process is called the:

```
Time Quantum
```

Example:

```
50 ms

100 ms

200 ms
```

Choosing the right quantum is important.

---

# Small Quantum

```
10 ms
```

Pros:

✔ Responsive

Cons:

❌ Too many context switches

---

# Large Quantum

```
1000 ms
```

Pros:

✔ Fewer context switches

Cons:

❌ Slow response

---

# Priority Scheduling

Processes receive priorities.

Example:

```
Priority 1

↓

Database
```

```
Priority 5

↓

Music Player
```

Higher-priority processes execute first.

---

# Starvation

Suppose:

High-priority jobs keep arriving.

```
High

↓

High

↓

High

↓

High
```

Low-priority process:

```
Never Runs
```

This is called:

```
Starvation
```

---

# Aging

To solve starvation,

the scheduler gradually increases waiting processes' priorities.

Example:

```
Priority

10

↓

9

↓

8

↓

7

↓

Eventually Runs
```

This technique is called:

```
Aging
```

---

# Multi-Level Queue

Processes are grouped.

Example:

```
Interactive

↓

Background

↓

System
```

Each queue has its own scheduling algorithm.

---

# Multi-Level Feedback Queue (MLFQ)

Unlike Multi-Level Queue,

processes can move between queues.

Example:

```
High Priority

↓

Uses Too Much CPU

↓

Move Lower
```

Interactive programs stay responsive.

CPU-intensive jobs move down.

---

# Linux Completely Fair Scheduler (CFS)

Modern Linux uses:

```
CFS

Completely Fair Scheduler
```

Instead of fixed time slices,

CFS tries to ensure:

```
Every Process

Gets Fair CPU Time
```

---

# How CFS Works

Imagine a timeline.

```
Chrome

Docker

VS Code

Spotify
```

The scheduler tracks:

```
Virtual Runtime
```

Processes that received less CPU time are scheduled sooner.

---

# Virtual Runtime

Suppose:

```
Chrome

100 ms
```

```
VS Code

20 ms
```

CFS prefers:

```
VS Code
```

because it has received less CPU time.

---

# Red-Black Tree

Linux stores runnable processes in a:

```
Red-Black Tree
```

The process with the smallest virtual runtime is chosen next.

We'll study Red-Black Trees later.

---

# Real-Time Scheduling

Some applications require strict timing.

Examples:

- Robotics
- Medical Devices
- Industrial Systems
- Audio Processing

Linux provides:

```
SCHED_FIFO

SCHED_RR
```

for real-time workloads.

---

# CPU Affinity

Normally,

the scheduler may move a process between CPU cores.

Sometimes we want:

```
Database

↓

Always Core 4
```

This is called:

```
CPU Affinity
```

Benefits:

- Better cache usage
- Lower latency
- Predictable performance

---

# Real Example

Suppose your laptop is running:

```
Chrome

↓

VS Code

↓

Docker

↓

Spotify

↓

Zoom
```

Every few milliseconds,

Linux decides:

```
Who Runs?

↓

Context Switch

↓

Who Runs Next?
```

This happens thousands of times every second.

You never notice it.

---

# Why Scheduling Matters

Scheduling directly affects:

- Gaming
- Web Servers
- Databases
- Docker
- Kubernetes
- Cloud Computing

Good scheduling makes systems feel fast.

Poor scheduling causes lag and reduced throughput.

---

# Hands-on Lab

## Lab 1 — View CPU Usage

```bash
top
```

Observe multiple processes sharing CPU time.

---

## Lab 2 — View Per-Core Usage

Linux:

```bash
mpstat -P ALL
```

---

## Lab 3 — View Process Priority

```bash
ps -eo pid,ni,comm
```

`NI` is the nice value used to influence scheduling priority.

---

## Lab 4 — Change Priority

```bash
nice -n 10 sleep 60
```

Observe the process priority.

---

## Lab 5 — Set CPU Affinity

```bash
taskset -c 0 sleep 60
```

This pins the process to CPU Core 0.

---

# Interview Questions

## Why is CPU Scheduling needed?

Because the number of runnable processes is usually much larger than the number of CPU cores.

---

## What is the Ready Queue?

The Ready Queue contains processes that are ready to execute but are waiting for CPU time.

---

## What is a Context Switch?

A Context Switch is the process of saving one process's execution state and loading another's so the CPU can switch between them.

---

## What is Round Robin Scheduling?

Round Robin gives each process a fixed time quantum, ensuring fair CPU sharing.

---

## What is Starvation?

Starvation occurs when a process waits indefinitely because higher-priority processes keep being scheduled.

---

## How does Aging solve Starvation?

Aging gradually increases the priority of waiting processes until they eventually receive CPU time.

---

## What scheduling algorithm does modern Linux use?

Modern Linux uses the **Completely Fair Scheduler (CFS)** for most normal processes.

---

## What is CPU Affinity?

CPU Affinity binds a process or thread to specific CPU cores to improve cache efficiency and performance.

---

# Summary

The CPU Scheduler decides which process executes next.

```
Processes

↓

Ready Queue

↓

Scheduler

↓

Dispatcher

↓

CPU

↓

Context Switch

↓

Next Process
```

Key concepts:

- Scheduling allows many processes to share limited CPU cores.
- The Ready Queue stores runnable processes.
- The Dispatcher performs context switches.
- FCFS is simple but inefficient.
- SJF minimizes average waiting time but requires predicting job lengths.
- Round Robin provides fairness using time quanta.
- Priority Scheduling can cause starvation, which Aging helps prevent.
- Linux uses the Completely Fair Scheduler (CFS) to distribute CPU time fairly.
- CPU Affinity keeps processes on specific CPU cores for better performance.

Understanding scheduling is essential before studying memory management, synchronization, Linux kernel internals, containers, and high-performance systems.

---

# Next Chapter

## Chapter 11 — Memory Management: Virtual Memory, Paging & Address Translation

We'll explore:

- Why Virtual Memory exists
- Physical vs Virtual Memory
- Address Spaces
- Page Tables
- Paging
- Page Faults
- Translation Lookaside Buffer (TLB)
- Memory Mapping (`mmap`)
- Demand Paging
- Copy-on-Write (CoW)
- Linux Memory Manager Internals