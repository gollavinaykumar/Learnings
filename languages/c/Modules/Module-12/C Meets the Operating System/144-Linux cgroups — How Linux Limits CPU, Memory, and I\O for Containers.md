# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 144 — Linux cgroups — How Linux Limits CPU, Memory, and I/O

---

# Most developers think:

```
Docker Container

↓

Uses Only Assigned Memory
```

But Linux doesn't magically know:

- How much CPU a container may use
- How much RAM it may consume
- How many processes it may create
- How much disk I/O it may perform

Without limits,

one process could consume:

```
100% CPU

100% RAM

Entire Machine
```

Linux solves this using:

```
Control Groups

↓

cgroups
```

---

# Learning Objectives

After this chapter you will understand:

- What cgroups are
- Why they exist
- cgroup v1 vs v2
- CPU controller
- Memory controller
- I/O controller
- PID controller
- OOM behavior
- How Docker and Kubernetes use cgroups

---

# The Problem

Imagine a server:

```
64 GB RAM
```

Running:

```
Database

↓

Web Server

↓

Redis

↓

Docker

↓

AI Model
```

Now imagine:

```
AI Model

↓

Consumes

↓

63 GB RAM
```

Everything else becomes slow or crashes.

---

# Another Problem

One process:

```
while(1){
}
```

Consumes:

```
100% CPU
```

Other applications become unresponsive.

---

# Linux Solution

Linux groups processes.

Each group gets:

```
CPU Limit

↓

Memory Limit

↓

Disk Limit

↓

Process Limit
```

This group is called:

```
cgroup
```

---

# What is a cgroup?

Definition:

> **A control group (cgroup) is a Linux kernel feature that groups processes and manages their resource usage.**

Think of it as:

```
Resource Container
```

---

# Visualization

```
Processes

↓

cgroup

↓

Resource Limits
```

---

# Namespaces vs cgroups

Many beginners confuse them.

Namespaces:

```
Isolation
```

cgroups:

```
Resource Control
```

---

# Comparison

```
Namespaces

↓

"What Can I See?"

------------------------

cgroups

↓

"How Much Can I Use?"
```

---

# Example

Container:

```
Namespaces

↓

Own Network

↓

Own PID

↓

Own Filesystem
```

cgroups:

```
CPU

↓

2 Cores

↓

RAM

↓

2 GB
```

Both technologies work together.

---

# cgroup Hierarchy

Processes belong to a tree.

Example:

```
Root

↓

Docker

↓

Container A

↓

Container B

↓

Container C
```

Each node can have different limits.

---

# cgroup Controllers

A controller manages one resource.

Examples:

```
CPU

↓

Memory

↓

I/O

↓

PIDs

↓

cpuset
```

---

# CPU Controller

Suppose:

```
Container

↓

Limit

↓

2 CPUs
```

Scheduler ensures:

```
Never Uses

↓

More Than Allowed
```

---

# Visualization

```
4 CPU Machine

↓

Container A

↓

2 CPUs

--------------------

Container B

↓

2 CPUs
```

---

# CPU Shares

Not always hard limits.

Example:

```
Container A

↓

1024 Shares

---------------------

Container B

↓

512 Shares
```

A gets roughly twice the CPU time when both compete.

---

# CPU Quota

Hard limit.

Example:

```
100 ms Period

↓

Container

↓

50 ms CPU
```

Maximum:

```
50%
```

CPU usage.

---

# Memory Controller

Suppose:

```
Limit

↓

512 MB
```

Program allocates:

```
600 MB
```

Kernel intervenes.

---

# Memory Limit

```
Program

↓

malloc()

↓

Beyond Limit

↓

Denied or OOM
```

---

# OOM Killer

OOM:

```
Out Of Memory
```

When memory cannot be allocated,

Linux may terminate one or more processes.

Within constrained environments,

the offending process or container is often selected.

---

# Visualization

```
Memory Full

↓

OOM

↓

Kill Process
```

---

# Memory Statistics

Each cgroup tracks:

```
Current Usage

↓

Maximum Usage

↓

Limit

↓

Faults
```

Useful for monitoring.

---

# PID Controller

Suppose:

```
Fork Bomb
```

Example:

```
fork()

↓

fork()

↓

fork()
```

Thousands of processes.

Without limits,

the machine may become unusable.

---

# PID Limit

Set:

```
Maximum

↓

100 Processes
```

After that:

```
fork()

↓

Fails
```

Protects the system.

---

# I/O Controller

Storage devices are shared.

One process:

```
Heavy Writes
```

Could slow everyone else.

The I/O controller regulates storage access.

---

# Example

Container:

```
Maximum

↓

50 MB/s
```

Even if the SSD is faster,

the cgroup enforces the configured policy.

---

# cpuset Controller

Restricts execution to specific CPUs.

Example:

```
Machine

↓

8 CPUs
```

Container:

```
CPU 2

↓

CPU 3
```

Useful for performance tuning.

---

# cgroup v1

Older design.

Separate hierarchies:

```
CPU

↓

Memory

↓

I/O
```

Each controller managed independently.

---

# cgroup v2

Modern design.

Unified hierarchy:

```
Root

↓

Controllers Together
```

Simpler and more consistent.

Most modern Linux distributions prefer cgroup v2.

---

# Docker Example

Command:

```bash
docker run \
--memory=1g \
--cpus=2
```

Docker configures the appropriate cgroup settings.

Kernel enforces the limits.

---

# Kubernetes Example

Pod:

```
Requests

↓

Limits
```

Kubernetes translates these into Linux cgroup settings.

---

# Relationship with Scheduler

CPU controller cooperates with:

```
Linux Scheduler
```

The scheduler decides:

```
Who Runs

↓

How Long
```

while respecting cgroup constraints.

---

# Relationship with OOM Killer

If a cgroup exceeds memory limits,

Linux may trigger an OOM event within that constrained environment.

---

# Relationship with Namespaces

Complete container:

```
Namespaces

↓

Isolation

+

cgroups

↓

Limits
```

Together they form the foundation of containers.

---

# Inspecting cgroups

Modern systems expose cgroup information through:

```text
/sys/fs/cgroup
```

Because cgroups are managed by the kernel.

---

# Real-World Example — Docker

Each container:

```
Own Namespaces

↓

Own cgroup

↓

Own Limits
```

---

# Real-World Example — Kubernetes

Every Pod receives CPU and memory limits implemented using cgroups.

---

# Real-World Example — System Services

System services can also be placed into cgroups,

not just containers.

---

# Common Mistakes

---

## Thinking Namespaces Limit CPU

They do not.

cgroups enforce resource limits.

---

## Thinking cgroups Create Isolation

They primarily manage and account for resource usage.

Namespaces provide isolation.

---

## Assuming Memory Limits Prevent All OOM Events

Processes can still encounter out-of-memory conditions within their configured limits.

---

## Forgetting cgroup v2

Modern Linux systems increasingly use the unified cgroup v2 hierarchy.

---

# Hands-on Labs

## Lab 1

Inspect:

```bash
ls /sys/fs/cgroup
```

Observe available cgroup files.

---

## Lab 2

Run a Docker container with:

```bash
--memory=256m
```

Observe its memory limit.

---

## Lab 3

Inspect memory statistics inside:

```text
/sys/fs/cgroup
```

---

## Lab 4

Compare cgroup v1 and cgroup v2 documentation.

---

## Lab 5

Research how Kubernetes translates resource limits into Linux cgroups.

---

# Interview Questions

### What is a cgroup?

A Linux kernel feature that groups processes and manages their resource usage.

---

### What is the difference between namespaces and cgroups?

Namespaces isolate system resources.

cgroups limit and account for resource consumption.

---

### What does the CPU controller do?

Controls CPU allocation and scheduling policies for processes within a cgroup.

---

### What happens when a process exceeds its memory limit?

The kernel may deny allocations and can trigger an out-of-memory event, often terminating processes within that cgroup.

---

### Why is cgroup v2 preferred?

It provides a unified hierarchy and more consistent resource management.

---

# Summary

Architecture:

```
Processes

↓

cgroup

↓

CPU

↓

Memory

↓

I/O

↓

PID Limits
```

Container Model:

```
Namespaces

↓

Isolation

+

cgroups

↓

Resource Limits
```

Common Controllers:

| Controller | Purpose |
|------------|---------|
| CPU | CPU scheduling and quotas |
| Memory | RAM limits and accounting |
| I/O | Block device throughput |
| PIDs | Maximum number of processes |
| cpuset | CPU affinity |

## Key Takeaways

- cgroups control and account for process resource usage.
- They complement namespaces, which provide isolation.
- CPU, memory, storage, and process limits are enforced by kernel controllers.
- Docker and Kubernetes rely heavily on cgroups.
- cgroup v2 is the modern unified implementation.
- Understanding cgroups is essential for understanding Linux containers and resource management.

---

# Next Chapter

## Chapter 145 — `seccomp` — How Linux Prevents Programs from Making Dangerous System Calls

You'll learn:

- What `seccomp` is
- System call filtering
- Allow lists vs deny lists
- BPF-based filters
- Docker seccomp profiles
- Sandboxing applications
- Why browsers and containers rely on `seccomp` for security