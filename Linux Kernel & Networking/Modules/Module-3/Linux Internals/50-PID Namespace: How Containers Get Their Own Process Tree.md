Most developers have run this command inside a container:

```bash
ps -ef
```

Output:

```text
PID  CMD

1    nginx
```

Only **one process**?

But on the host machine:

```bash
ps -ef
```

shows:

```
Thousands of Processes
```

Question:

**Where did all the host processes go?**

Even more interesting...

Inside every Docker container:

```
PID 1
```

exists.

How can **every container have its own PID 1**?

A Linux machine can only have one process with PID 1...

Right?

Actually,

it can't.

Unless Linux creates:

**PID Namespaces.**

PID Namespaces are another core Linux kernel feature that powers containers.

After this chapter, you'll understand why every container believes it's running its own operating system.

---

# 👑 Linux Kernel & Networking Mastery

# Module 7 — Linux Process Isolation (PID Namespace)

# Chapter 50 — PID Namespace: How Containers Get Their Own Process Tree

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why PID Namespaces exist
> - What is a PID Namespace?
> - Process IDs (PIDs)
> - Process Trees
> - Parent & Child Processes
> - PID 1
> - Zombie Processes
> - Init Process
> - Signal Handling
> - Docker PID Namespace

---

# 📖 Why Do We Need PID Namespaces?

Suppose your Linux server runs:

```
PostgreSQL

↓

PID 2401
```

```
Redis

↓

PID 3210
```

```
Docker

↓

PID 5010
```

Now Docker starts:

```
Nginx Container
```

Without PID Namespaces,

the container would see:

```
Every Process

Running

On The Host
```

This is:

❌ Not secure

❌ Not isolated

Containers should only see their own processes.

---

# Normal Linux

One system.

One process tree.

```
PID 1

↓

systemd

↓

sshd

↓

nginx

↓

redis

↓

postgres
```

Every process belongs to the same hierarchy.

---

# Problem

Suppose inside the container:

```bash
kill 2401
```

If the container could see host processes,

it might terminate critical host services.

That must never happen.

---

# PID Namespace Solution

Linux creates:

```
Namespace A

↓

Own Process Tree
```

```
Namespace B

↓

Own Process Tree
```

Processes become isolated.

---

# What is a PID Namespace?

A PID Namespace gives processes:

✔ Their own PID numbering

✔ Their own process tree

✔ Their own PID 1

Processes outside the namespace remain hidden.

---

# Visual Representation

```
Host Namespace

PID 1

↓

systemd

↓

Docker
```

Container:

```
PID Namespace

↓

PID 1

↓

nginx
```

Same Linux kernel.

Different process view.

---

# PID Numbers

Host:

```
PID 8450

↓

nginx
```

Inside container:

```
PID 1

↓

nginx
```

The same process has:

```
Two PID Values
```

depending on who is looking.

---

# Namespace Mapping

```
Host PID

↓

8450
```

Container PID:

```
1
```

Linux maintains this mapping internally.

---

# Why PID 1?

Every Linux process tree begins with:

```
PID 1
```

Inside a container,

the first process becomes:

```
PID 1
```

Even though it isn't the host's PID 1.

---

# Host vs Container

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

Both exist simultaneously.

Because they belong to different PID Namespaces.

---

# Process Tree

Inside container:

```bash
pstree
```

Example:

```
nginx

├── worker

├── worker

└── worker
```

Host processes remain invisible.

---

# Parent & Child Processes

Suppose:

```
nginx

↓

Worker Process
```

The worker inherits:

```
Parent PID

↓

1
```

inside the namespace.

---

# Process Visibility

Container:

```bash
ps -ef
```

Shows:

```
Container Processes
```

Host:

```bash
ps -ef
```

Shows:

```
Everything
```

Host can see container processes.

Container cannot normally see host processes.

---

# Namespace Hierarchy

PID Namespaces are hierarchical.

```
Host Namespace

↓

Container Namespace

↓

Nested Namespace
```

Parent namespaces can observe child namespace processes.

The reverse is not true.

---

# Zombie Processes

Suppose:

```
Child Process

↓

Exits
```

If the parent doesn't collect its exit status,

the process becomes a:

```
Zombie
```

Zombie processes remain until they are reaped.

---

# What Does PID 1 Do?

PID 1 has special responsibilities.

It should:

✔ Start child processes

✔ Reap zombie processes

✔ Handle important signals

Without proper handling,

zombies may accumulate.

---

# Why PID 1 Is Special

Unlike ordinary processes,

PID 1 has special behavior in the Linux kernel.

If it ignores certain signals,

they may not terminate the container as expected.

This is why proper init behavior matters.

---

# Docker Example

Suppose:

```bash
docker run nginx
```

Inside:

```bash
ps
```

Output:

```
PID 1

↓

nginx
```

Nginx becomes the container's init process.

---

# Signal Handling

Suppose:

```bash
docker stop
```

Docker sends a termination signal to the container's PID 1.

If PID 1 handles the signal correctly:

```
Graceful Shutdown
```

Otherwise,

Docker eventually forces termination.

---

# Why Tiny Init Processes Exist

Many images use:

```
tini
```

or another lightweight init process.

Purpose:

✔ Forward Signals

✔ Reap Zombie Processes

✔ Improve Container Behavior

---

# Example

Instead of:

```
PID 1

↓

Java
```

Some images run:

```
PID 1

↓

tini

↓

Java
```

This improves signal handling and process cleanup.

---

# Docker Inspect

Host:

```bash
ps -ef
```

Observe:

```
Container Process

↓

Host PID
```

Inside container:

```bash
ps
```

Observe:

```
PID 1
```

Same process.

Different namespaces.

---

# Kubernetes Example

Every Pod gets:

```
PID Namespace
```

By default,

containers in the same Pod have separate PID namespaces unless configured otherwise.

Kubernetes also supports sharing a PID namespace within a Pod when explicitly enabled.

---

# Why PID Namespaces Matter

Without PID Namespaces:

- Containers could inspect host processes.
- Containers could interfere with each other.
- Process isolation would not exist.

They are essential for secure multi-tenant systems.

---

# Hands-on Lab

## Lab 1 — Start a Container

```bash
docker run -it alpine sh
```

---

## Lab 2 — View Processes

Inside:

```bash
ps
```

Observe:

```
PID 1
```

---

## Lab 3 — View Host Processes

On the host:

```bash
ps -ef
```

Compare the difference.

---

## Lab 4 — Find Container PID on Host

```bash
docker inspect <container-id>
```

Locate:

```
State

↓

Pid
```

This is the host PID.

---

## Lab 5 — Compare PIDs

Inside:

```bash
ps
```

Outside:

```bash
ps -fp <host-pid>
```

Observe that the same process has different PID values.

---

# Interview Questions

## What is a PID Namespace?

A PID Namespace isolates process IDs so that processes see only the processes within their own namespace.

---

## Why does every container have PID 1?

Because each container typically runs in its own PID Namespace, where the first process is assigned PID 1.

---

## Can the host see container processes?

Yes.

The host namespace can see processes in child PID namespaces.

---

## Can a container see host processes?

Not normally.

A container sees only the processes in its own PID Namespace.

---

## Why is PID 1 special?

PID 1 is responsible for signal handling and reaping orphaned child processes within its namespace.

---

## Why is `tini` often used?

`tini` acts as a lightweight init process that forwards signals and reaps zombie processes.

---

# Summary

PID Namespaces isolate process IDs and process trees.

```
Host

↓

PID Namespace

↓

PID 1

↓

Application

↓

Child Processes
```

Key concepts:

- PID Namespaces isolate process visibility.
- Every namespace has its own PID 1.
- The same process can have different PID values inside and outside a namespace.
- PID 1 has special responsibilities for signal handling and child process cleanup.
- Docker containers typically run with their own PID Namespace.
- Lightweight init systems such as `tini` improve container behavior.

Understanding PID Namespaces explains why containers behave like independent Linux systems while sharing the same kernel.

---

# Next Chapter

## Chapter 51 — Mount Namespace: How Containers Get Their Own Filesystem View

We'll explore:

- What is a Mount Namespace?
- Filesystem Isolation
- Mount Points
- Root Filesystem
- Bind Mounts
- OverlayFS Integration
- Volumes
- Read-only Mounts
- Docker Filesystem Isolation
- Complete Filesystem Architecture