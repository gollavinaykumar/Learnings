# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 153 — eBPF Maps — The Shared Memory That Connects Kernel Programs and User Space

---

# In the previous chapter you learned:

```
Packet

↓

Hook

↓

eBPF Program

↓

Continue
```

A natural question appears:

> **Where does the eBPF program store its data?**

Suppose you want to count:

```
TCP Packets

↓

Millions Per Second
```

Can an eBPF program simply do:

```c
int packets = 0;
```

No.

Because every invocation would lose the previous value.

Instead,

Linux provides:

```
eBPF Maps
```

Maps are one of the most important concepts in eBPF.

Without maps,

most eBPF programs would be nearly useless.

---

# Learning Objectives

After this chapter you will understand:

- Why maps exist
- Hash maps
- Arrays
- Per-CPU maps
- LRU maps
- Ring Buffers
- Perf Event Arrays
- Queue Maps
- Stack Maps
- Pinning
- Kernel ↔ User communication

---

# The Problem

Suppose every packet runs:

```
eBPF Program
```

Each execution starts fresh.

How can we remember:

```
Packets Seen

↓

Connections

↓

Statistics

↓

Configuration
```

Between executions?

---

# Linux Solution

Kernel creates:

```
Persistent Objects

↓

Maps
```

Programs can:

```
Read

↓

Write

↓

Update
```

those maps.

---

# What is an eBPF Map?

Definition:

> **An eBPF map is a kernel-managed data structure used to share data between eBPF programs and user-space applications.**

Think of it as:

```
Shared Memory

↓

Managed By Kernel
```

---

# Visualization

```
User Program

↓

Map

↑

eBPF Program
```

Both sides access the same data structure.

---

# Why Kernel-Managed?

Kernel guarantees:

- Safety
- Lifetime management
- Synchronization rules
- Access control

Applications don't directly manipulate kernel memory.

---

# Hash Map

Most common map type.

Structure:

```
Key

↓

Value
```

Example:

```
PID

↓

Open Count
```

---

# Visualization

```
1001

↓

50

-------------------

1002

↓

17

-------------------

1003

↓

200
```

Each process has its own counter.

---

# Example

Every:

```
open()
```

executes:

```
PID

↓

Lookup

↓

Increment Counter
```

---

# Array Map

Simpler.

Index:

```
0

↓

10

----------------

1

↓

22

----------------

2

↓

35
```

Fast access.

Fixed size.

---

# Why Arrays?

When keys are sequential:

```
CPU Number

↓

Packet Count
```

Arrays are usually faster than hash maps.

---

# Per-CPU Maps

One of the smartest optimizations.

Instead of:

```
All CPUs

↓

Same Counter
```

Linux gives:

```
CPU0

↓

Own Counter

-------------------

CPU1

↓

Own Counter

-------------------

CPU2

↓

Own Counter
```

No lock contention.

---

# Visualization

Without Per-CPU

```
CPU0

↓

Shared Counter

↑

CPU1

↑

CPU2
```

Threads compete for the same memory.

---

# With Per-CPU

```
CPU0

↓

Counter

-------------------

CPU1

↓

Counter

-------------------

CPU2

↓

Counter
```

Much better scalability.

---

# LRU Hash Map

LRU means:

```
Least Recently Used
```

Suppose:

```
Map Full
```

Instead of failing,

kernel removes:

```
Oldest Unused Entry
```

Automatically.

---

# Example

Capacity:

```
100 Entries
```

Insert:

```
101st
```

Kernel evicts:

```
Least Recently Used
```

entry.

---

# Queue Map

Works like:

```
FIFO
```

Visualization:

```
A

↓

B

↓

C
```

Remove:

```
A
```

Useful for ordered processing.

---

# Stack Map

Works like:

```
LIFO
```

Visualization:

```
Top

↓

C

↓

B

↓

A
```

Pop:

```
C
```

---

# Ring Buffer

One of the most important map types.

Suppose eBPF wants to send:

```
Events

↓

User Space
```

Instead of:

```
Millions Of Copies
```

Linux uses:

```
Ring Buffer
```

---

# Visualization

```
Producer

↓

Ring Buffer

↓

Consumer
```

Circular memory.

Very efficient.

---

# Why Ring Buffer?

Oldest data disappears only when the buffer becomes full and the producer advances, depending on the application's consumption strategy.

Minimal allocations.

High throughput.

---

# Perf Event Array

Older tracing mechanism.

Used to send tracing data:

```
Kernel

↓

User Space
```

Modern applications often prefer ring buffers,

but perf event arrays remain widely used.

---

# Configuration Maps

Maps aren't only for statistics.

Example:

```
Blocked IPs

↓

Hash Map
```

User application updates:

```
New IP
```

eBPF immediately starts blocking it.

No reload required.

---

# Pinning Maps

Normally:

```
Program Exits

↓

Map Removed
```

Sometimes you want:

```
Map Survives
```

Linux allows:

```
Pin

↓

Map

↓

bpffs
```

Now another application can reuse it.

---

# Visualization

```
Kernel

↓

Pinned Map

↓

bpffs

↓

Another Program
```

---

# Relationship with Hook Points

Every hook can use maps.

Example:

```
Packet

↓

XDP

↓

Map Update

↓

Continue
```

---

# Relationship with User Space

Flow:

```
eBPF

↓

Map

↓

User Reads

↓

Dashboard
```

Very common.

---

# Relationship with the Verifier

The verifier checks:

```
Map Access

↓

Valid?

↓

Allow
```

Unsafe access is rejected.

---

# Relationship with Networking

Packet:

```
Source IP

↓

Lookup

↓

Map

↓

Blocked?

↓

Drop
```

No userspace round-trip.

---

# Relationship with Containers

Container ID:

```
Lookup

↓

Policy Map

↓

Decision
```

Many container networking platforms work this way.

---

# Real-World Example — Cilium

Maps store:

- Endpoints
- Services
- Policies
- Connection tracking

eBPF programs consult them for every packet.

---

# Real-World Example — `bpftrace`

Every event:

```
Map

↓

Counter

↓

Print Summary
```

---

# Real-World Example — Observability

System call:

```
PID

↓

Map

↓

Latency

↓

Dashboard
```

---

# Real-World Example — Security

Blocked Processes:

```
PID

↓

Policy Map

↓

Allow/Deny
```

---

# Common Mistakes

---

## Thinking Maps Are Ordinary Variables

Maps persist across multiple eBPF program executions.

---

## Ignoring Per-CPU Maps

Shared counters can become bottlenecks on multi-core systems.

---

## Using Hash Maps for Everything

Arrays, ring buffers, queues, and other map types may be more appropriate depending on the workload.

---

## Forgetting About Pinning

Pinned maps allow independent applications to share long-lived kernel state.

---

# Hands-on Labs

## Lab 1

Run:

```bash
bpftool map show
```

Observe existing maps.

---

## Lab 2

Inspect:

```bash
bpftool map dump
```

View map contents.

---

## Lab 3

Study the available eBPF map types in the kernel documentation.

---

## Lab 4

Compare:

- Hash Map
- Array
- Per-CPU Array

Understand when each is appropriate.

---

## Lab 5

Research how Cilium stores networking information inside eBPF maps.

---

# Interview Questions

### What is an eBPF map?

A kernel-managed data structure shared between eBPF programs and user-space applications.

---

### Why are maps necessary?

They provide persistent storage and communication across multiple eBPF program executions.

---

### What is a Per-CPU map?

A map that maintains separate values for each CPU to reduce contention.

---

### What is an LRU map?

A map that automatically evicts the least recently used entries when it reaches capacity.

---

### Why are ring buffers important?

They efficiently transfer event data from kernel-space eBPF programs to user-space applications.

---

# Summary

Architecture:

```
User Space

↓

Map

↑

eBPF Program
```

Common Map Types:

| Map Type | Purpose |
|-----------|---------|
| Hash Map | Key-value storage |
| Array | Fixed indexed storage |
| Per-CPU Map | Scalable counters |
| LRU Map | Automatic eviction |
| Ring Buffer | Event streaming |
| Perf Event Array | Tracing events |
| Queue | FIFO storage |
| Stack | LIFO storage |

Typical Flow:

```
Packet

↓

Hook

↓

Lookup Map

↓

Update

↓

Continue
```

## Key Takeaways

- Maps are the primary storage mechanism for eBPF programs.
- They enable communication between kernel-space eBPF programs and user-space applications.
- Different workloads benefit from different map types.
- Per-CPU maps significantly improve scalability on multi-core systems.
- Ring buffers are optimized for high-performance event delivery.
- Mastering eBPF maps is essential for building practical eBPF applications.

---

# Next Chapter

## Chapter 154 — The eBPF Verifier — How Linux Proves Your Kernel Program Is Safe Before It Runs

You'll learn:

- Why the verifier exists
- Program analysis
- Control-flow verification
- Memory safety
- Pointer tracking
- Register tracking
- Bounded loops
- Why unsafe eBPF programs are rejected before they ever execute