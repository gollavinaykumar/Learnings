# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 143 — Linux Namespaces — How Containers Create Isolated Worlds

---

# Most developers think:

```
Docker

↓

Virtual Machine
```

Actually...

```
Docker

↓

NOT A Virtual Machine
```

Docker does **not** create a new Linux kernel.

It creates the **illusion** that a process has:

- Its own processes
- Its own network
- Its own hostname
- Its own filesystem
- Its own users

The technology behind this illusion is called:

```
Linux Namespaces
```

Without namespaces,

Docker, Kubernetes, Podman, LXC, and many modern container platforms would not exist.

---

# Learning Objectives

After this chapter you will understand:

- What namespaces are
- PID namespace
- Mount namespace
- Network namespace
- IPC namespace
- UTS namespace
- User namespace
- Time namespace
- How containers are isolated

---

# The Problem

Suppose two applications run on the same Linux system.

Application A:

```
PID

↓

1001
```

Application B:

```
PID

↓

1002
```

Now imagine you want Application B to believe:

```
It Is The Only Application
```

How?

---

# Another Problem

Application A:

```
Hostname

↓

server-1
```

Application B wants:

```
Hostname

↓

database
```

On the same machine.

Normally,

Linux has only one hostname.

---

# Another Problem

Application A:

```
eth0

↓

10.0.0.5
```

Application B wants:

```
eth0

↓

192.168.1.2
```

Again,

same machine.

---

# Linux Solution

Create:

```
Separate Views

↓

For Each Process Group
```

These separate views are:

```
Namespaces
```

---

# What is a Namespace?

Definition:

> **A namespace isolates a particular type of system resource so that a group of processes sees its own independent view.**

Think of it as:

```
Private Universe
```

---

# Visualization

```
Linux Kernel

↓

Namespace A

↓

Processes

----------------------

Namespace B

↓

Processes
```

Same kernel.

Different worlds.

---

# One Kernel

Important concept:

```
Container A

↓

Kernel

↑

Container B

↑

Container C
```

Everyone shares:

```
One Linux Kernel
```

Unlike virtual machines.

---

# Types of Namespaces

Linux currently supports several namespace types.

Major ones:

```
PID

Mount

Network

IPC

UTS

User

Time

Cgroup
```

We'll study each.

---

# PID Namespace

Without namespaces:

```
System

↓

PID 1

↓

systemd
```

Container:

```
Sees

↓

PID 1

↓

Its Own Process
```

Even though the host has many processes.

---

# Visualization

Host:

```
PID 1

↓

systemd
```

Container:

```
PID 1

↓

nginx
```

Both are correct,

depending on the namespace.

---

# Why?

The kernel maintains separate PID mappings for each namespace.

Same process,

different visible PID.

---

# Mount Namespace

Normally:

```
/

↓

Same Filesystem
```

Containers need different views.

Container A:

```
/

↓

Ubuntu
```

Container B:

```
/

↓

Alpine
```

Same host.

Different filesystem view.

---

# Visualization

```
Host

↓

Real Filesystem

--------------------

Container

↓

Private Mount Tree
```

---

# Network Namespace

Each namespace gets:

- Network interfaces
- Routing table
- Firewall rules
- Ports

Example:

Container:

```
eth0

↓

172.18.0.2
```

Host:

```
eth0

↓

192.168.1.100
```

Independent networking stacks.

---

# Visualization

```
Host Network

--------------------

Container Network
```

Applications believe they own the network.

---

# IPC Namespace

IPC means:

```
Inter-Process Communication
```

Examples:

- Shared memory
- Semaphores
- Message queues

Namespaces prevent one container from accidentally accessing another container's IPC objects.

---

# UTS Namespace

UTS stands for:

```
UNIX Time-sharing System
```

It isolates:

- Hostname
- Domain name

Example:

Container:

```bash
hostname
```

Output:

```
web-server
```

Host:

```
production-host
```

---

# User Namespace

One of the most important security features.

Container:

```
UID 0

↓

root
```

Host:

```
UID 100000
```

Container thinks:

```
I Am Root
```

Host knows:

```
Just An Unprivileged User
```

Huge security improvement.

---

# Time Namespace

Allows processes to observe different notions of time for certain clocks.

Useful for:

- Testing
- Checkpoint/restore
- Specialized workloads

---

# Cgroup Namespace

Hides details about:

```
CPU Limits

↓

Memory Limits

↓

Resource Hierarchy
```

Processes see their own cgroup view.

---

# How Docker Starts a Container

Simplified flow:

```
docker run

↓

containerd

↓

runc

↓

clone()

↓

Namespaces

↓

cgroups

↓

execve()

↓

Container Process
```

Namespaces provide isolation.

cgroups provide resource control.

---

# Relationship with `clone()`

The Linux system call:

```
clone()
```

can create new namespaces.

Example flags:

```
CLONE_NEWNS

CLONE_NEWPID

CLONE_NEWNET

CLONE_NEWUSER
```

Each requests a different namespace.

---

# Process View

Host:

```
PID

↓

4231
```

Inside container:

```
PID

↓

1
```

Same process.

Different namespace.

---

# Filesystem View

Host:

```
/

↓

Entire System
```

Container:

```
/

↓

Container Root Filesystem
```

Applications cannot normally see outside their mount namespace.

---

# Networking View

Host:

```
eth0

↓

192.168.1.100
```

Container:

```
eth0

↓

172.17.0.2
```

Different virtual network stacks.

---

# Relationship with VFS

Mount namespaces create independent filesystem views,

while the underlying VFS still serves filesystem operations.

---

# Relationship with Capabilities

User namespaces and capabilities work together.

A process may appear to be root inside a namespace,

while remaining restricted on the host.

---

# Real-World Example — Docker

Every container gets:

- PID namespace
- Mount namespace
- Network namespace
- UTS namespace
- IPC namespace

Often also:

- User namespace
- Cgroup namespace

depending on configuration.

---

# Real-World Example — Kubernetes

Each Pod relies on Linux namespaces to isolate workloads running on the same node.

---

# Real-World Example — Podman

Like Docker,

Podman uses Linux namespaces to isolate containers without requiring a virtual machine.

---

# Virtual Machine vs Namespace

Virtual Machine:

```
Guest Kernel

↓

Guest OS

↓

Applications
```

Container:

```
Shared Kernel

↓

Namespaces

↓

Applications
```

Containers are much lighter because they reuse the host kernel.

---

# Common Mistakes

---

## Thinking Containers Have Their Own Kernel

They share the host Linux kernel.

---

## Thinking Namespaces Provide Resource Limits

Namespaces isolate.

cgroups limit resources.

These are different technologies.

---

## Confusing Namespaces with Virtual Machines

Virtual machines emulate complete operating systems.

Namespaces isolate processes within one kernel.

---

## Assuming User Namespace Gives Host Root Access

Container root can be mapped to an unprivileged host user.

---

# Hands-on Labs

## Lab 1

Inspect namespace links:

```bash
ls -l /proc/self/ns
```

Observe different namespace types.

---

## Lab 2

Run a Docker container.

Compare:

```bash
hostname
```

inside and outside the container.

---

## Lab 3

Compare:

```bash
ps -ef
```

on the host and inside a container.

Observe PID isolation.

---

## Lab 4

Inspect network interfaces:

```bash
ip addr
```

inside and outside a container.

---

## Lab 5

Research the `clone()` flags used to create new namespaces.

---

# Interview Questions

### What is a Linux namespace?

A kernel feature that isolates a particular type of system resource for a group of processes.

---

### Why are namespaces important?

They provide the isolation that makes Linux containers possible.

---

### Do Docker containers have their own kernel?

No.

They share the host Linux kernel.

---

### What is a PID namespace?

A namespace providing an isolated process ID view.

Processes inside it can have their own PID 1.

---

### What is the difference between namespaces and cgroups?

Namespaces isolate resources.

cgroups control and limit resource usage.

---

# Summary

Architecture:

```
Linux Kernel

↓

Namespaces

↓

Container Processes
```

Major Namespace Types:

| Namespace | Isolates |
|-----------|----------|
| PID | Process IDs |
| Mount | Filesystem view |
| Network | Network stack |
| IPC | IPC objects |
| UTS | Hostname/domain name |
| User | User/group IDs |
| Time | Certain clocks |
| Cgroup | Resource hierarchy view |

Container Startup:

```
clone()

↓

Create Namespaces

↓

Configure Isolation

↓

execve()

↓

Application
```

## Key Takeaways

- Namespaces create isolated views of system resources.
- Containers share one Linux kernel while remaining isolated.
- Different namespace types isolate different kernel resources.
- `clone()` is commonly used to create namespaces.
- Namespaces provide isolation; cgroups provide resource control.
- Understanding namespaces explains how Docker and Kubernetes work internally.

---

# Next Chapter

## Chapter 144 — Linux cgroups — How Linux Limits CPU, Memory, and I/O for Containers

You'll learn:

- What cgroups are
- cgroup v1 vs v2
- CPU limits
- Memory limits
- I/O limits
- Process limits
- OOM handling
- How Docker and Kubernetes prevent one container from consuming the entire machine