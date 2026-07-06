# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 151 — eBPF — Running Safe Programs Inside the Linux Kernel

---

# Most developers think:

```
Application

↓

Runs In User Space
```

or

```
Kernel Module

↓

Runs In Kernel Space
```

Only two choices.

Actually...

Modern Linux has a third option:

```
eBPF Program

↓

Runs Inside Kernel

↓

Without Writing A Kernel Module
```

This is one of the biggest innovations in Linux during the last decade.

Companies like:

- Meta
- Google
- Cloudflare
- Netflix
- Isovalent
- Cilium

use eBPF extensively for:

- Networking
- Observability
- Security
- Performance monitoring

---

# Learning Objectives

After this chapter you will understand:

- What eBPF is
- Why it was created
- BPF vs eBPF
- eBPF Virtual Machine
- Verifier
- Maps
- Helper Functions
- Hook Points
- Why eBPF is safe

---

# The Problem

Suppose you want to:

```
Count

↓

Every TCP Packet
```

Old solution:

```
Modify Linux Kernel

↓

Recompile

↓

Reboot
```

Very risky.

---

# Another Problem

Suppose you want to monitor:

```
Every

↓

open()

↓

read()

↓

write()
```

Old solution:

```
Kernel Module
```

Problems:

- Difficult to write
- Easy to crash Linux
- Hard to maintain

---

# Linux Solution

Instead of:

```
Kernel Module
```

Linux allows:

```
Tiny Verified Programs

↓

Run Inside Kernel
```

These are:

```
eBPF Programs
```

---

# What is eBPF?

Definition:

> **eBPF (Extended Berkeley Packet Filter) is a technology that allows verified programs to run safely inside the Linux kernel.**

Think of it as:

```
Mini Applications

↓

Inside Kernel
```

---

# Why "Extended"?

Originally Linux had:

```
BPF

↓

Packet Filtering
```

Only networking.

Later Linux extended it.

Now eBPF works for:

- Networking
- Tracing
- Security
- Performance
- Filesystems
- Scheduling

Much more than packets.

---

# Think About Plugins

Instead of modifying:

```
Photoshop Source Code
```

You install:

```
Plugin
```

eBPF works similarly.

Instead of changing the kernel,

you load a small verified program.

---

# Architecture

```
User Program

↓

Load eBPF

↓

Verifier

↓

Kernel

↓

Hook Point
```

---

# The Verifier

This is the secret.

Before execution,

Linux checks:

```
Program Safe?

↓

Yes

↓

Load

--------------------

No

↓

Reject
```

---

# Why Verify?

Kernel crashes are unacceptable.

The verifier prevents programs from:

- Infinite loops (historically; modern kernels support bounded loops after verification)
- Invalid memory access
- Unsafe pointer operations
- Corrupting kernel memory

---

# Visualization

```
Program

↓

Verifier

↓

Approved

↓

Kernel
```

---

# eBPF Virtual Machine

Programs do **not** execute as raw native machine code immediately.

They first execute within the eBPF execution model.

Kernel understands:

```
eBPF Instructions
```

These instructions are then interpreted or JIT-compiled depending on kernel configuration.

---

# Hook Points

An eBPF program must attach somewhere.

Examples:

```
Network Packet

↓

System Call

↓

Function Entry

↓

Function Exit

↓

Tracepoint
```

When that event occurs,

the kernel executes the eBPF program.

---

# Visualization

```
TCP Packet

↓

Kernel

↓

Hook

↓

eBPF Program
```

---

# Example

Suppose:

```
Every

↓

open()
```

Kernel executes:

```
Your eBPF Program
```

before or after the event,

depending on the attachment point.

---

# eBPF Maps

Programs need storage.

Kernel provides:

```
Maps
```

Think of them as:

```
Shared Hash Tables
```

Used by:

```
Kernel

↓

User Space
```

Both can access the data.

---

# Visualization

```
User

↓

Map

↑

eBPF Program
```

---

# Example Map

```
PID

↓

Open Count
```

Each:

```
open()
```

increments the counter.

User application later reads the map.

---

# Helper Functions

eBPF programs cannot call arbitrary kernel functions.

Instead,

Linux provides approved helper functions.

Examples include:

```
Get Current PID

↓

Get Current Time

↓

Print Debug Message

↓

Access Maps
```

Only safe operations are allowed.

---

# Program Flow

```
System Call

↓

Kernel

↓

Hook

↓

eBPF Program

↓

Map Update

↓

Return
```

---

# JIT Compilation

For performance,

many Linux systems translate verified eBPF bytecode into native CPU instructions.

Benefits:

```
Near Native Speed
```

---

# Relationship with System Calls

Suppose:

```
read()
```

Kernel enters:

```
Hook

↓

eBPF

↓

Continue
```

Applications never notice.

---

# Relationship with Networking

Incoming packet:

```
NIC

↓

Kernel

↓

eBPF

↓

Firewall

↓

TCP
```

Packets can be inspected,

modified,

or redirected,

depending on the hook and permissions.

---

# Relationship with Security

eBPF can:

```
Observe

↓

System Calls

↓

File Access

↓

Network Events
```

Many modern security tools rely on it.

---

# Relationship with Observability

Instead of adding:

```
printf()
```

everywhere,

eBPF observes the kernel dynamically.

No reboot required.

---

# Relationship with Containers

Container networking platforms such as Cilium use eBPF to implement:

- Networking
- Load balancing
- Security policies

directly inside the Linux kernel.

---

# Real-World Example — Cilium

Network packet:

```
Kernel

↓

eBPF

↓

Policy

↓

Forward
```

Very high performance.

---

# Real-World Example — `bpftrace`

Command:

```bash
bpftrace
```

Loads tracing programs into the kernel using eBPF.

---

# Real-World Example — Performance Profiling

Engineers can count:

- System calls
- Context switches
- Cache misses (through appropriate tracing mechanisms)
- Network events

without modifying applications.

---

# Real-World Example — Cloudflare

Large-scale networking platforms use eBPF for efficient packet processing and observability.

---

# Why Not Kernel Modules?

Kernel Module:

```
Bug

↓

Kernel Panic
```

eBPF:

```
Verifier

↓

Reject Unsafe Program
```

Much safer.

---

# Common Mistakes

---

## Thinking eBPF Can Do Anything

It runs inside carefully enforced limits.

The verifier and helper APIs restrict what it can do.

---

## Thinking eBPF Replaces Kernel Modules

Many tasks still require traditional kernel modules.

eBPF complements them.

---

## Forgetting About the Verifier

Every eBPF program must pass verification before it can run.

---

## Assuming eBPF Is Only for Networking

Modern eBPF is used for networking, tracing, security, observability, and more.

---

# Hands-on Labs

## Lab 1

Install:

```bash
bpftool
```

Explore eBPF objects on your system.

---

## Lab 2

Install:

```bash
bpftrace
```

Run a simple tracing example.

---

## Lab 3

Read:

```bash
man bpf
```

Understand the `bpf()` system call.

---

## Lab 4

Inspect loaded eBPF programs using:

```bash
bpftool prog
```

---

## Lab 5

Research the different eBPF attachment points available in modern Linux.

---

# Interview Questions

### What is eBPF?

A Linux technology that allows verified programs to run safely inside the kernel.

---

### Why is the verifier important?

It prevents unsafe programs from crashing or corrupting the kernel.

---

### What are eBPF maps?

Kernel-managed data structures used to exchange information between eBPF programs and user-space applications.

---

### Why is eBPF faster than many traditional monitoring approaches?

It executes directly inside the kernel, reducing unnecessary context switches and data copying.

---

### Is eBPF only for networking?

No.

It is widely used for networking, tracing, security, observability, and performance analysis.

---

# Summary

Architecture:

```
User Program

↓

Load eBPF

↓

Verifier

↓

Kernel

↓

Hook Point
```

Program Flow:

```
Event

↓

Hook

↓

eBPF

↓

Map

↓

Continue
```

Major Components:

| Component | Purpose |
|-----------|---------|
| Verifier | Ensures safety |
| Virtual Machine | Executes eBPF bytecode |
| Maps | Shared data storage |
| Helper Functions | Safe kernel APIs |
| Hook Points | Event attachment locations |

## Key Takeaways

- eBPF allows safe programs to execute inside the Linux kernel.
- The verifier is the foundation of eBPF safety.
- Programs attach to kernel events through hook points.
- Maps enable communication between kernel-space eBPF programs and user-space applications.
- JIT compilation provides excellent performance.
- eBPF has become one of the most important technologies in modern Linux infrastructure.

---

# Next Chapter

## Chapter 152 — eBPF Hook Points — Where Exactly Does an eBPF Program Run?

You'll learn:

- XDP
- TC (Traffic Control)
- Kprobes
- Kretprobes
- Tracepoints
- Uprobes
- USDT
- LSM Hooks
- Cgroup Hooks
- Socket Hooks
- The complete map of where eBPF programs can attach inside the Linux kernel