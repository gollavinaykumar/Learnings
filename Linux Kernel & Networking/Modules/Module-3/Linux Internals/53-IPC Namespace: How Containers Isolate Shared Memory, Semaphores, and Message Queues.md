Most developers know processes communicate using:

- HTTP
- TCP
- WebSockets
- Files

But Linux provides something much faster.

Processes running on the **same machine** can communicate directly through the kernel without using the network.

Examples:

- Shared Memory
- Semaphores
- Message Queues

These are called:

```
IPC

Inter-Process Communication
```

Now imagine two Docker containers.

Should one container be able to access another container's shared memory?

Absolutely not.

That would completely break container isolation.

The solution is:

**IPC Namespaces.**

IPC Namespaces isolate Linux IPC resources so that every container gets its own private communication environment.

After this chapter, you'll understand how Linux isolates shared memory, semaphores, and message queues for containers.

---

# 👑 Linux Kernel & Networking Mastery

# Module 7 — Linux Process & Filesystem Isolation

# Chapter 53 — IPC Namespace: How Containers Isolate Shared Memory, Semaphores, and Message Queues

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why IPC exists
> - What is an IPC Namespace?
> - Shared Memory
> - Message Queues
> - Semaphores
> - POSIX IPC
> - System V IPC
> - Docker IPC Modes
> - Kubernetes IPC Sharing
> - IPC Namespace Architecture

---

# 📖 Why Do We Need IPC?

Suppose two processes run on the same Linux machine.

```
Process A
```

```
Process B
```

Instead of sending data through:

```
TCP

↓

Kernel

↓

TCP

↓

Process
```

they can communicate directly using kernel IPC mechanisms.

This is much faster.

---

# What is IPC?

IPC stands for:

```
Inter-Process Communication
```

It allows processes on the same machine to exchange data or synchronize with one another.

Linux provides several IPC mechanisms.

---

# Common IPC Mechanisms

Linux supports:

✔ Shared Memory

✔ Message Queues

✔ Semaphores

These resources are managed by the kernel.

---

# Shared Memory

Fastest IPC mechanism.

```
Process A

↓

Shared Memory

↓

Process B
```

Both processes access the same memory region.

No network required.

---

# Why Shared Memory Is Fast

Normally:

```
Process A

↓

Kernel

↓

Socket

↓

Kernel

↓

Process B
```

Shared memory:

```
Process A

↓

Shared Memory

↓

Process B
```

Much less overhead.

---

# Message Queue

A Message Queue works like a mailbox.

```
Process A

↓

Queue

↓

Process B
```

Messages remain in the queue until they are read.

---

# Semaphore

A Semaphore is used for synchronization.

Suppose two processes update the same shared data.

Without coordination:

```
Race Condition
```

Semaphores help ensure that only the intended number of processes access a resource at a given time.

---

# Why IPC Isolation Matters

Suppose:

```
Container A

↓

Shared Memory
```

Container B:

```
Reads

↓

Container A Memory
```

This would be a serious security issue.

Containers must have isolated IPC resources.

---

# What is an IPC Namespace?

An IPC Namespace isolates:

✔ Shared Memory

✔ Message Queues

✔ Semaphores

Processes inside one namespace cannot normally access IPC objects from another namespace.

---

# Visual Representation

```
Linux Kernel

├── IPC Namespace A

│      Shared Memory

│      Message Queues

│      Semaphores

│

├── IPC Namespace B

│      Shared Memory

│      Message Queues

│      Semaphores
```

Each namespace has independent IPC resources.

---

# Shared Memory Isolation

Container A:

```
Shared Memory

↓

Segment 101
```

Container B:

```
Cannot See

Segment 101
```

Linux hides it.

---

# Message Queue Isolation

Container A:

```
Queue A
```

Container B:

```
Queue B
```

Each namespace maintains its own queues.

---

# Semaphore Isolation

Container A:

```
Semaphore Set
```

Container B:

```
Different Semaphore Set
```

Synchronization remains private to the namespace.

---

# System V IPC

Traditional Linux IPC includes:

- Shared Memory
- Message Queues
- Semaphores

Collectively known as:

```
System V IPC
```

These kernel objects are namespace-aware.

---

# POSIX IPC

Linux also supports:

- POSIX Shared Memory
- POSIX Semaphores
- POSIX Message Queues

These are modern standardized IPC APIs.

IPC Namespaces help isolate these resources as well.

---

# Viewing IPC Objects

Commands:

```bash
ipcs
```

Displays:

- Shared Memory
- Message Queues
- Semaphores

Within the current IPC Namespace.

---

# Removing IPC Objects

Example:

```bash
ipcrm
```

Removes selected IPC resources.

Useful for cleanup during development.

---

# Docker Default Behavior

Normally:

```bash
docker run ubuntu
```

Docker creates:

```
New IPC Namespace
```

Each container receives isolated IPC resources.

---

# Docker IPC Modes

Default:

```
Private IPC Namespace
```

Docker also supports:

```
--ipc=host
```

The container shares the host's IPC Namespace.

Another option:

```
--ipc=container:<container-id>
```

Two containers share the same IPC Namespace.

---

# `--ipc=host`

Example:

```bash
docker run \
--ipc=host \
ubuntu
```

Now the container can access the host's IPC resources.

This reduces isolation and should be used only when appropriate.

---

# Shared IPC Between Containers

Example:

```bash
docker run \
--ipc=container:abc123
```

Both containers share:

✔ Shared Memory

✔ Message Queues

✔ Semaphores

Useful for tightly coupled workloads.

---

# Kubernetes Example

Pods normally isolate IPC resources.

Kubernetes also supports sharing the host IPC namespace by setting:

```yaml
hostIPC: true
```

This should be used carefully because it reduces isolation.

---

# Why IPC Namespaces Matter

Applications using:

- Databases
- Scientific Computing
- High-Performance Computing (HPC)
- Machine Learning

often rely on IPC internally.

Containers keep these IPC resources isolated.

---

# Complete Architecture

```
Container

↓

IPC Namespace

↓

Shared Memory

↓

Message Queue

↓

Semaphore

↓

Processes
```

---

# Hands-on Lab

## Lab 1 — Start a Container

```bash
docker run -it ubuntu bash
```

---

## Lab 2 — View IPC Objects

```bash
ipcs
```

Observe IPC resources inside the container.

---

## Lab 3 — Start Host IPC Container

```bash
docker run \
-it \
--ipc=host \
ubuntu
```

Compare:

```bash
ipcs
```

with the host.

---

## Lab 4 — View Shared Memory

```bash
ipcs -m
```

---

## Lab 5 — View Message Queues

```bash
ipcs -q
```

---

## Lab 6 — View Semaphores

```bash
ipcs -s
```

---

# Interview Questions

## What is an IPC Namespace?

An IPC Namespace isolates kernel IPC resources such as shared memory, message queues, and semaphores.

---

## Why are IPC Namespaces needed?

They prevent containers from accessing each other's IPC resources, improving security and isolation.

---

## What resources are isolated?

- Shared Memory
- Message Queues
- Semaphores

---

## What is Shared Memory?

Shared Memory allows multiple processes to access the same memory region for fast communication.

---

## What is the purpose of a Semaphore?

Semaphores coordinate access to shared resources and help synchronize processes.

---

## What does `--ipc=host` do?

It allows the container to share the host's IPC Namespace.

---

## Can two containers share IPC?

Yes.

Docker supports sharing an IPC Namespace using:

```text
--ipc=container:<container-id>
```

---

# Summary

IPC Namespaces isolate Linux inter-process communication resources.

```
Container

↓

IPC Namespace

↓

Shared Memory

↓

Message Queues

↓

Semaphores

↓

Processes
```

Key concepts:

- IPC enables fast communication between local processes.
- Shared Memory is the fastest IPC mechanism.
- Message Queues provide asynchronous communication.
- Semaphores synchronize access to shared resources.
- IPC Namespaces isolate these kernel objects.
- Docker creates private IPC Namespaces by default.
- Containers can optionally share IPC when required.

At this point, you've learned another essential Linux Namespace used by containers.

Current Namespace knowledge:

- ✅ Network Namespace
- ✅ PID Namespace
- ✅ Mount Namespace
- ✅ UTS Namespace
- ✅ IPC Namespace

Only a few namespaces remain before you've mastered the complete Linux container isolation model.

---

# Next Chapter

## Chapter 54 — User Namespace: Running Root Inside a Container Without Being Root on the Host

We'll explore:

- What is a User Namespace?
- UID & GID Mapping
- Root Inside a Container
- Root on the Host
- User ID Translation
- Capability Isolation
- Docker User Namespace Remapping
- Rootless Containers
- Security Benefits
- Complete User Namespace Architecture

> **This is one of the most important security features in modern container runtimes.**