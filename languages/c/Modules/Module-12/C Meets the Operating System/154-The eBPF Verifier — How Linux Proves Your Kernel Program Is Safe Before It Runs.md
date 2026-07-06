# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 154 — The eBPF Verifier — How Linux Proves Your Kernel Program Is Safe Before It Runs

---

# Imagine this situation:

You write an eBPF program.

It runs:

```
Inside

↓

The Linux Kernel
```

Question:

> **What happens if your program has a bug?**

A normal user-space program may crash.

But...

A buggy kernel program could:

- Crash the operating system
- Corrupt kernel memory
- Freeze every process
- Cause data loss

Linux cannot allow that.

Instead of trusting your code,

Linux verifies it **before it ever runs.**

That verification system is called:

```
The eBPF Verifier
```

It is one of the most sophisticated static analysis engines inside the Linux kernel.

---

# Learning Objectives

After this chapter you will understand:

- Why the verifier exists
- Static analysis
- Register tracking
- Pointer tracking
- Memory safety
- Control-flow analysis
- Loop verification
- Program rejection
- Why eBPF is considered safe

---

# The Problem

Suppose someone loads:

```c
while (1) {
}
```

inside the kernel.

Result:

```
CPU

↓

100%

↓

Kernel Never Returns
```

Entire system becomes unstable.

Linux refuses to execute it.

---

# Another Problem

Suppose a program does:

```c
*(char *)0 = 1;
```

Meaning:

```
Write

↓

Random Memory
```

Kernel memory corruption.

Again:

```
Rejected
```

---

# Another Problem

Imagine:

```
Pointer

↓

Unknown Address

↓

Read Memory
```

The verifier asks:

```
How Do You Know

↓

That Pointer Is Safe?
```

If it cannot prove safety,

the program is rejected.

---

# Linux Solution

Before loading:

```
Program

↓

Verifier

↓

Safe?

↓

Yes

↓

Run

-------------------------

No

↓

Reject
```

No exceptions.

---

# What is the Verifier?

Definition:

> **The eBPF verifier is a kernel component that statically analyzes every eBPF program and rejects any program that cannot be proven safe.**

Notice:

```
Proven Safe
```

Not:

```
Probably Safe
```

---

# Think About Airport Security

Passenger:

```
Arrives
```

Security:

```
Scan

↓

Check

↓

Verify

↓

Allow Boarding
```

Unsafe passenger:

```
Rejected
```

The verifier performs the same role for kernel programs.

---

# Static Analysis

The verifier:

```
Does NOT

↓

Execute Program
```

Instead,

it reads the instructions and reasons about them.

---

# Visualization

```
Program

↓

Instruction 1

↓

Instruction 2

↓

Instruction 3

↓

Analysis
```

---

# Control Flow Analysis

Suppose:

```c
if (x)
```

Verifier explores:

```
True Path

↓

False Path
```

Both must be safe.

---

# Visualization

```
if

↓

True

↓

Safe?

↑

False

↓

Safe?
```

Every execution path matters.

---

# Register Tracking

The eBPF virtual machine provides registers.

Example:

```
R0

↓

Return Value

------------------

R1

↓

Argument

------------------

R2

↓

Argument
```

The verifier tracks:

```
Current Value

↓

Possible Range

↓

Pointer Type
```

for every register.

---

# Example

Suppose:

```
R1

↓

Packet Pointer
```

Verifier knows:

```
Points To Packet
```

Not:

```
Random Memory
```

---

# Pointer Tracking

Suppose:

```
Packet Start

↓

+20 Bytes
```

Verifier checks:

```
Still Inside Packet?
```

If yes:

```
Allow
```

Otherwise:

```
Reject
```

---

# Visualization

```
Packet

↓

Valid

↓

Pointer

↓

Safe
```

---

# Bounds Checking

Imagine packet size:

```
100 Bytes
```

Program reads:

```
Byte 150
```

Verifier detects:

```
Out Of Bounds

↓

Reject
```

Before execution.

---

# Memory Safety

Verifier ensures:

```
Every Read

↓

Valid

-------------------

Every Write

↓

Valid
```

Invalid accesses never reach runtime.

---

# Stack Verification

eBPF programs have a limited stack.

Verifier tracks:

```
Allocated

↓

Initialized

↓

Accessed
```

Reading uninitialized stack memory is rejected.

---

# Function Calls

Modern eBPF supports helper functions and BPF-to-BPF function calls.

Verifier checks:

- Arguments
- Return values
- Call depth
- Stack usage

Everything must remain safe.

---

# Helper Function Validation

Example:

```
Lookup Map

↓

Return Pointer
```

Verifier tracks:

```
Pointer

↓

Nullable?

↓

Checked?

↓

Safe?
```

Programs must verify returned pointers before dereferencing them.

---

# Loop Verification

Originally:

```
Loops

↓

Forbidden
```

Reason:

Verifier couldn't prove termination.

Modern Linux supports:

```
Bounded Loops
```

The verifier must prove that every loop finishes.

---

# Example

Good:

```c
for (i = 0; i < 10; i++)
```

Maximum iterations:

```
10
```

Verifier approves.

---

# Bad Example

```c
while (1)
```

Termination:

```
Unknown
```

Verifier rejects it.

---

# Program Size

The verifier also checks program complexity and instruction limits.

Very large or excessively complex programs may be rejected depending on kernel limits.

---

# State Explosion

Suppose:

```
if

↓

if

↓

if

↓

if
```

Every branch doubles possible execution states.

Verifier carefully manages state exploration to remain practical.

---

# Why Verification Takes Time

Loading an eBPF program may take milliseconds because:

```
Analyze

↓

Registers

↓

Pointers

↓

Control Flow

↓

Loops

↓

Helpers

↓

Maps
```

Execution afterwards is very fast.

---

# Relationship with Maps

Suppose:

```
Map Lookup

↓

Pointer Returned
```

Verifier asks:

```
Did You

↓

Check For NULL?
```

If not:

```
Reject
```

---

# Relationship with Hook Points

Every hook:

```
XDP

↓

Tracepoint

↓

LSM

↓

Kprobe
```

Uses the same verifier.

Safety rules remain consistent.

---

# Relationship with JIT

Flow:

```
Program

↓

Verifier

↓

Approved

↓

JIT Compiler

↓

Native Code
```

Verification always happens before JIT compilation.

---

# Real-World Example — Cilium

Every networking policy program:

```
Verifier

↓

Approved

↓

Loaded
```

No verifier,

no execution.

---

# Real-World Example — `bpftrace`

Every tracing program must successfully pass verification before attaching to the requested hook.

---

# Real-World Example — Security

An unsafe security program cannot accidentally crash the kernel.

The verifier prevents it from loading.

---

# Common Reasons Programs Are Rejected

```
Invalid Pointer

↓

Out-of-Bounds Access

↓

Unchecked NULL

↓

Unknown Loop Bound

↓

Stack Misuse

↓

Illegal Helper Usage
```

---

# Common Mistakes

---

## Thinking the Verifier Executes the Program

It analyzes instructions without running the program.

---

## Ignoring Pointer Checks

Every pointer must be proven valid.

---

## Assuming Any Loop Is Allowed

Only loops with provable bounded execution are accepted.

---

## Forgetting NULL Checks

Map lookups often require explicit NULL validation before use.

---

# Hands-on Labs

## Lab 1

Install:

```bash
bpftool
```

Attempt to load example programs and observe verifier output.

---

## Lab 2

Read kernel documentation about verifier internals.

---

## Lab 3

Use:

```bash
bpftool prog load
```

and inspect verifier log messages.

---

## Lab 4

Experiment with an invalid map lookup and study why the verifier rejects it.

---

## Lab 5

Research how the verifier tracks register and pointer state through different execution paths.

---

# Interview Questions

### What is the eBPF verifier?

A kernel component that statically analyzes every eBPF program before it is allowed to run.

---

### Why is the verifier necessary?

Because eBPF programs execute inside the kernel, and unsafe programs could otherwise crash or corrupt the operating system.

---

### Does the verifier execute the program?

No.

It analyzes the program's instructions and possible execution paths.

---

### Why were infinite loops historically forbidden?

Because the verifier could not guarantee termination.

Modern kernels allow loops only when they can prove they are bounded.

---

### What kinds of errors does the verifier detect?

Invalid memory access, unsafe pointers, unchecked NULL values, illegal helper usage, excessive complexity, and other safety violations.

---

# Summary

Verification Flow:

```
eBPF Program

↓

Verifier

↓

Static Analysis

↓

Safe?

↓

Load

↓

Run
```

Checks Performed:

```
Registers

↓

Pointers

↓

Memory

↓

Control Flow

↓

Loops

↓

Helpers

↓

Maps
```

Program Lifecycle:

```
Compile

↓

Load

↓

Verify

↓

JIT

↓

Execute
```

## Key Takeaways

- The verifier is the foundation of eBPF safety.
- Every eBPF program is analyzed before execution.
- The verifier proves memory safety, pointer safety, and control-flow correctness.
- Only bounded loops are accepted.
- Verification enables eBPF to run inside the kernel without requiring trusted kernel modules.
- Understanding the verifier is essential for writing successful eBPF programs.

---

# Next Chapter

## Chapter 155 — XDP (Express Data Path) — Processing Network Packets Before the Linux Network Stack

You'll learn:

- What XDP is
- Why it is the fastest packet-processing framework in Linux
- NIC drivers
- Packet flow
- XDP actions (`PASS`, `DROP`, `TX`, `REDIRECT`, `ABORTED`)
- DDoS protection
- Load balancing
- How Cloudflare, Meta, and Cilium process millions of packets per second with XDP