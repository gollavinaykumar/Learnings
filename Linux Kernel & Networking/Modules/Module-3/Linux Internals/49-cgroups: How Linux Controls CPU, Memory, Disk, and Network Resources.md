Up until now, we've learned how Linux isolates **what a process can see** using Namespaces.

A Docker container gets its own:

- Network
- PID space
- Mount points
- Hostname
- Users

But here's another question.

Suppose you start a container like this:

```bash
docker run ubuntu
```

What prevents it from doing this?

```
while(true){
    allocate_memory();
}
```

or

```
while(true){
    use_cpu();
}
```

Could one container consume:

- 100% CPU?
- All RAM?
- Entire Disk I/O?
- Entire Network Bandwidth?

Without protection,

**one process could crash the whole machine.**

The answer is:

**cgroups (Control Groups).**

Namespaces provide **isolation**.

cgroups provide **resource control**.

Together,

they form the two biggest foundations of modern containers.

After this chapter, you'll understand how Linux limits CPU, memory, disk, and other resources for every container.

---

# 👑 Linux Kernel & Networking Mastery

# Module 6 — Linux Resource Isolation (cgroups)

# Chapter 49 — cgroups: How Linux Controls CPU, Memory, Disk, and Network Resources

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why cgroups exist
> - What is a Control Group?
> - cgroup Hierarchy
> - CPU Controller
> - Memory Controller
> - Disk I/O Controller
> - PID Controller
> - OOM Killer
> - cgroup v1 vs cgroup v2
> - How Docker Uses cgroups

---

# 📖 Why Do We Need cgroups?

Suppose one server runs:

```
Container A
```

```
Container B
```

```
Container C
```

Container A starts consuming:

```
100% CPU

+

All Memory
```

Result:

```
Entire Server

↓

Slow

↓

Crash
```

Linux needs a way to limit resource usage.

That mechanism is:

```
cgroups
```

---

# Before cgroups

Normal Linux:

```
Process A

↓

Unlimited CPU
```

```
Process B

↓

Unlimited Memory
```

Every process competes for the same resources.

---

# What is a cgroup?

A **Control Group (cgroup)** is a Linux kernel feature that groups one or more processes and applies resource limits to them.

Think of it as:

```
Process Group

+

Resource Rules
```

---

# Namespaces vs cgroups

Namespaces answer:

```
What Can The Process See?
```

Examples:

- Network
- Files
- Processes

---

cgroups answer:

```
How Much Can The Process Use?
```

Examples:

- CPU
- Memory
- Disk I/O
- Number of Processes

---

# Together They Build Containers

```
Container

↓

Namespaces

↓

Isolation
```

```
Container

↓

cgroups

↓

Resource Limits
```

Docker uses both.

---

# cgroup Hierarchy

Linux organizes cgroups in a hierarchy.

```
Root

├── system

├── docker

│     ├── container1

│     ├── container2

│     └── container3

└── user
```

Each group has its own limits.

---

# Processes Belong to a cgroup

Suppose:

```
nginx

↓

PID 2001
```

Linux assigns it to:

```
docker/container1
```

All resource accounting happens at the cgroup level.

---

# CPU Controller

Suppose:

```
4 CPU Cores
```

Container:

```
Limit

↓

1 Core
```

Even if the application tries to use all CPUs,

Linux schedules it according to the configured limits.

---

# CPU Shares

Example:

Container A:

```
1024 Shares
```

Container B:

```
512 Shares
```

If CPU contention exists,

Container A receives roughly twice the CPU time of Container B.

CPU shares are **relative weights**, not hard limits.

---

# CPU Quota

Hard limit example:

```
100 ms

↓

Only 50 ms CPU Allowed
```

After reaching its quota,

the process is throttled until the next scheduling period.

---

# Docker CPU Limit

Example:

```bash
docker run \
--cpus=2 \
nginx
```

Container can use approximately:

```
2 CPU Cores
```

---

# Memory Controller

Suppose:

Container:

```
Limit

↓

512 MB
```

Application requests:

```
2 GB
```

Linux refuses further allocation beyond the configured limit.

---

# Docker Memory Limit

Example:

```bash
docker run \
-m 512m \
nginx
```

Maximum memory:

```
512 MB
```

---

# What Happens When Memory Is Exhausted?

Suppose:

```
Limit

↓

512 MB
```

Application allocates:

```
600 MB
```

Linux triggers an Out-Of-Memory (OOM) event for that cgroup.

Typically,

the kernel terminates one or more processes inside the cgroup.

---

# OOM Killer

Linux includes:

```
OOM Killer
```

Its job:

```
Memory Exhausted

↓

Select Process

↓

Terminate
```

This protects the rest of the system.

---

# PID Controller

Suppose malware creates:

```
1 Million Processes
```

Without limits,

the machine could become unusable.

Linux can limit:

```
Maximum Processes
```

per cgroup.

---

# Docker Example

```bash
docker run \
--pids-limit 100 \
nginx
```

Maximum:

```
100 Processes
```

---

# Block I/O Controller

Suppose:

```
Database
```

and

```
Backup Job
```

share the same disk.

Without limits,

the backup could consume all disk bandwidth.

The Block I/O controller limits disk usage.

---

# Docker Example

Docker supports options for tuning block I/O weight and, on supported systems, read/write bandwidth or IOPS limits.

This helps prevent one container from monopolizing storage performance.

---

# Network Resource Control

Traditional cgroups focus on CPU, memory, I/O, and processes.

Network bandwidth control is typically achieved by combining cgroups with Linux traffic control (`tc`) and other networking features.

Docker networking solutions may use these Linux capabilities depending on configuration.

---

# Resource Accounting

cgroups continuously record:

```
CPU Time

Memory Used

Disk I/O

Processes
```

Administrators can inspect current usage.

---

# Viewing cgroups

Modern Linux:

```bash
ls /sys/fs/cgroup
```

Observe controller files and resource statistics.

---

# cgroup v1

Earlier Linux versions:

```
Separate Controllers
```

Example:

```
cpu

memory

blkio
```

Each controller had its own hierarchy.

---

# cgroup v2

Modern Linux uses:

```
Unified Hierarchy
```

Benefits:

✔ Simpler

✔ Better Accounting

✔ Easier Management

Most modern distributions default to cgroup v2.

---

# Docker and cgroups

Suppose:

```bash
docker run \
-m 1g \
--cpus=2 \
nginx
```

Docker translates these options into cgroup configuration.

Linux enforces the limits.

Docker itself does **not** schedule CPU or manage memory.

The kernel does.

---

# Complete Architecture

```
Application

↓

Process

↓

cgroup

↓

Linux Scheduler

↓

CPU

Memory

Disk

Network
```

---

# Why cgroups Matter

Every container platform depends on them.

Examples:

- Docker
- Kubernetes
- Podman
- containerd
- CRI-O
- systemd

Without cgroups,

containers could not safely share the same machine.

---

# Hands-on Lab

## Lab 1 — View cgroup Version

```bash
mount | grep cgroup
```

Observe whether the system uses cgroup v1 or v2.

---

## Lab 2 — View cgroup Files

```bash
ls /sys/fs/cgroup
```

---

## Lab 3 — Run a Memory-Limited Container

```bash
docker run \
-m 256m \
nginx
```

---

## Lab 4 — Run a CPU-Limited Container

```bash
docker run \
--cpus=1 \
nginx
```

---

## Lab 5 — Inspect Container Limits

```bash
docker inspect <container-id>
```

Observe configured resource limits.

---

## Lab 6 — View Container Statistics

```bash
docker stats
```

Observe:

- CPU Usage
- Memory Usage
- Network I/O
- Block I/O

---

# Interview Questions

## What are cgroups?

cgroups (Control Groups) are a Linux kernel feature that limits, prioritizes, and accounts for resource usage by groups of processes.

---

## What is the difference between Namespaces and cgroups?

Namespaces isolate what a process can see.

cgroups control how much of a resource a process can use.

---

## Why are cgroups required for containers?

Without cgroups, one container could consume excessive CPU, memory, or other resources and negatively affect the host and other containers.

---

## What happens when a container exceeds its memory limit?

The kernel typically triggers an OOM event and may terminate one or more processes in that cgroup.

---

## Does Docker enforce CPU and memory limits?

Docker configures cgroups.

The Linux kernel enforces the limits.

---

## What is the difference between cgroup v1 and v2?

cgroup v1 uses separate hierarchies for different controllers.

cgroup v2 provides a unified hierarchy with improved management and accounting.

---

# Summary

cgroups provide resource control for Linux processes.

```
Container

↓

Processes

↓

cgroup

↓

CPU

Memory

Disk

PID

↓

Linux Kernel
```

Key concepts:

- cgroups group processes for resource management.
- They limit CPU, memory, disk I/O, and process counts.
- The OOM Killer protects the system during memory exhaustion.
- Docker uses cgroups to enforce container limits.
- Modern Linux systems primarily use cgroup v2.
- cgroups and Namespaces together form the core of container technology.

At this point, you've learned the second major pillar of containers:

- **Namespaces → Isolation**
- **cgroups → Resource Control**

Together, they enable multiple applications to safely share the same Linux kernel.

---

# Next Chapter

## Chapter 50 — PID Namespace: How Containers Get Their Own Process Tree

We'll explore:

- What is a PID Namespace?
- Why Containers Have Their Own PID 1
- Process Trees
- Parent and Child Processes
- Zombie Processes
- Init Processes
- Signal Handling
- Docker PID Namespace
- `ps` Inside Containers
- Complete Process Isolation