Most developers know that modern laptops have:

- 4 Cores
- 8 Cores
- 16 Cores

They also hear terms like:

- Threads
- Hyper-Threading
- Context Switching
- Multi-Core Processing

But many developers think:

> "One CPU runs one program."

That's not true.

Your computer may be running **hundreds of processes simultaneously** while you watch YouTube, write code in VS Code, build Docker images, and listen to Spotify.

How?

The answer lies in CPU architecture and operating system scheduling.

After this chapter, you'll understand how modern CPUs execute billions of instructions every second and how operating systems efficiently share CPU time among thousands of tasks.

---

# 👑 Linux Kernel & Networking Mastery

# Module 1 — Computer Fundamentals

# Chapter 4 — CPU Architecture: Cores, Threads, Pipelines & Context Switching

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a CPU
> - CPU Core
> - CPU Thread
> - Single-Core vs Multi-Core
> - Hyper-Threading / SMT
> - CPU Registers
> - Arithmetic Logic Unit (ALU)
> - Control Unit (CU)
> - Clock Cycle
> - CPU Pipeline
> - Branch Prediction
> - Instruction-Level Parallelism
> - Context Switching
> - CPU Scheduling Basics
> - CPU Affinity
> - Why modern CPUs are so fast

---

# 📖 What Does the CPU Actually Do?

The CPU executes instructions.

Everything eventually becomes CPU instructions.

Example:

```java
int total = price + tax;
```

The CPU doesn't understand Java.

It only understands machine instructions.

```
Java Code

↓

Compiler

↓

Machine Instructions

↓

CPU Executes
```

---

# CPU Components

A simplified CPU looks like this.

```
               CPU
        +----------------+
        | Control Unit   |
        |                |
        | Registers      |
        |                |
        | ALU            |
        |                |
        | Cache          |
        +----------------+
```

Each component has a different responsibility.

---

# Control Unit (CU)

The Control Unit is like a manager.

It decides:

- Which instruction to execute
- When to fetch data
- Which register to use
- Where results should go

Think of it as the traffic controller inside the CPU.

---

# Arithmetic Logic Unit (ALU)

The ALU performs calculations.

Examples:

```
10 + 20

50 - 30

40 × 5

100 / 10

A > B

A == B
```

Every arithmetic and logical operation eventually reaches the ALU.

---

# Registers

Registers are tiny memory locations inside the CPU.

Example:

```
RAM

↓

Registers

↓

ALU
```

Suppose:

```java
int a = 5;
int b = 8;
```

Execution:

```
RAM

↓

Register A

↓

Register B

↓

ALU

↓

Register Result

↓

RAM
```

Registers are the fastest memory in the entire computer.

---

# CPU Clock

CPUs work according to a clock.

Example:

```
3.6 GHz
```

Means approximately:

```
3.6 Billion Clock Cycles

Every Second
```

Each instruction requires one or more clock cycles.

---

# Clock Cycle

Imagine a drum beating.

```
Tick

Tick

Tick

Tick
```

Every tick allows the CPU to advance its work.

```
Fetch

↓

Decode

↓

Execute

↓

Store
```

This process repeats billions of times every second.

---

# Fetch → Decode → Execute Cycle

Every instruction follows this cycle.

```
Instruction

↓

Fetch

↓

Decode

↓

Execute

↓

Write Result
```

Example:

```
5 + 3
```

Step 1

```
Fetch

Read instruction from memory.
```

Step 2

```
Decode

Understand the operation.
```

Step 3

```
Execute

ALU performs addition.
```

Step 4

```
Store

Save result.
```

---

# CPU Core

A CPU Core is an independent processing unit.

Example:

```
8-Core CPU

Core 1

Core 2

Core 3

Core 4

Core 5

Core 6

Core 7

Core 8
```

Each core can execute instructions independently.

---

# Single-Core CPU

Older computers often had one core.

```
Program A

↓

CPU

↓

Program B

↓

CPU

↓

Program C
```

Only one task executes at a time.

---

# Multi-Core CPU

Modern CPUs contain multiple cores.

```
Core 1 → Chrome

Core 2 → VS Code

Core 3 → Docker

Core 4 → Spotify
```

Many tasks execute simultaneously.

---

# What is a CPU Thread?

A hardware thread is an execution path inside a CPU core.

Example:

```
1 Core

↓

2 Threads
```

One core can appear as two logical CPUs.

---

# Hyper-Threading (Intel) / SMT (AMD)

Suppose one core is waiting for memory.

Without Hyper-Threading:

```
Core Waiting

↓

Nothing Happens
```

With Hyper-Threading:

```
Core

├── Thread 1

└── Thread 2
```

If Thread 1 is waiting,

Thread 2 can execute.

This improves CPU utilization.

---

# Example

Laptop:

```
8 Cores

16 Threads
```

Operating system sees:

```
16 Logical CPUs
```

Not just 8.

---

# CPU Pipeline

Imagine a car factory.

Instead of building one car completely,

multiple cars are built simultaneously at different stages.

```
Fetch

↓

Decode

↓

Execute

↓

Write Back
```

While one instruction executes,

another is decoding,

and another is being fetched.

This is called pipelining.

---

# Pipeline Example

Without pipeline:

```
Instruction 1

↓

Instruction 2

↓

Instruction 3
```

With pipeline:

```
Fetch I3

Decode I2

Execute I1
```

Multiple instructions are processed together.

---

# Branch Prediction

Suppose the CPU encounters:

```java
if (age > 18)
```

The CPU predicts which path is most likely.

```
Prediction

↓

Execute Speculatively
```

If correct:

Great.

If wrong:

Discard work and execute the correct path.

Branch prediction greatly improves performance.

---

# Instruction-Level Parallelism

Modern CPUs try to execute independent instructions simultaneously.

Example:

```java
a = b + c;

x = y + z;
```

Since these are independent,

the CPU may execute them together.

---

# Why Isn't Every Program Instantly Fast?

Sometimes the CPU waits for:

- RAM
- SSD
- Network
- User Input

During these waits,

the operating system schedules another task.

---

# What is Context Switching?

Suppose one CPU must run three programs.

```
Chrome

VS Code

Docker
```

The operating system quickly switches between them.

```
Chrome

↓

VS Code

↓

Docker

↓

Chrome

↓

VS Code
```

This rapid switching creates the illusion that all programs run simultaneously.

---

# What Happens During Context Switching?

The operating system saves the current program's state.

```
Registers

Program Counter

Stack Pointer
```

Then loads another program's state.

```
CPU

↓

Program B Continues
```

The previous program resumes later exactly where it stopped.

---

# Why Context Switching Has a Cost

Switching isn't free.

The operating system must:

- Save registers
- Load new registers
- Update memory mappings
- Flush some CPU state

Too many context switches reduce performance.

---

# CPU Scheduling (Basic Idea)

Suppose there are four programs.

```
Chrome

VS Code

Docker

Spotify
```

The scheduler decides:

```
Who gets CPU?

For how long?

Who waits?
```

We'll study scheduling algorithms in the Operating Systems module.

---

# CPU Affinity

Sometimes we want a program to stay on one CPU core.

Example:

```
Program

↓

Core 4 Only
```

Benefits:

- Better cache usage
- Lower latency
- Predictable performance

Used in:

- Databases
- High-frequency trading
- Network servers

---

# Real Example — Opening Google Chrome

```
Click Chrome

↓

Operating System Creates Process

↓

Scheduler Assigns CPU Core

↓

Instructions Loaded

↓

Pipeline Executes Instructions

↓

GPU Draws Window

↓

Chrome Opens
```

The CPU continuously executes millions of instructions during this process.

---

# Why This Matters

Understanding CPU architecture helps explain:

- Why multi-threading improves performance
- Why Docker containers share CPUs
- Why Kubernetes limits CPU usage
- Why databases pin threads to CPU cores
- Why context switching affects scalability

Every operating system builds upon these CPU concepts.

---

# Hands-on Lab

## Lab 1 — View CPU Information

### Linux

```bash
lscpu
```

### macOS

```bash
sysctl -n machdep.cpu.brand_string
sysctl hw.physicalcpu
sysctl hw.logicalcpu
```

---

## Lab 2 — Monitor CPU Usage

### Linux

```bash
top
```

### macOS

```bash
top
```

Observe multiple processes sharing CPU time.

---

## Lab 3 — View Number of Logical CPUs

### Linux

```bash
nproc
```

### macOS

```bash
sysctl hw.logicalcpu
```

---

## Lab 4 — Observe Context Switching

### Linux

```bash
vmstat 1
```

Watch the `cs` (context switches) column increase as the system runs.

---

# Interview Questions

## What is a CPU Core?

A CPU Core is an independent processing unit capable of executing instructions.

---

## What is Hyper-Threading?

Hyper-Threading allows one physical core to execute two hardware threads, improving CPU utilization.

---

## Why are registers faster than RAM?

Registers are located inside the CPU, eliminating the need to access external memory.

---

## What is Context Switching?

Context Switching is the operating system saving one process's execution state and loading another's so multiple processes can share the CPU.

---

## Why does excessive context switching hurt performance?

Because saving and restoring execution state consumes CPU time and reduces cache efficiency.

---

# Summary

Modern CPUs execute billions of instructions every second using multiple hardware optimizations.

```
Program

↓

Operating System

↓

CPU Scheduler

↓

CPU Core

↓

Registers

↓

ALU

↓

Pipeline

↓

Result
```

Key concepts:

- CPU executes machine instructions
- Cores provide parallel execution
- Hardware threads improve utilization
- Registers are the fastest memory
- Pipelines execute multiple instruction stages simultaneously
- Branch prediction reduces waiting
- Context switching allows many programs to share CPUs
- CPU affinity improves performance for latency-sensitive applications

Understanding CPU architecture is essential before learning operating systems, Linux scheduling, multithreading, containers, and kernel internals.

---

# Next Chapter

## Chapter 5 — Assembly Language & Machine Code Fundamentals

We'll explore:

- Why CPUs cannot understand Java, Go, or JavaScript
- Machine code
- Assembly language
- CPU instruction set
- Registers in practice
- Function calls
- Stack frames
- Push and Pop instructions
- Calling conventions
- How high-level code becomes machine instructions