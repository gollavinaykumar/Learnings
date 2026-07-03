# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 29 — containerd & runc

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What containerd is
> - What runc is
> - Why Docker uses containerd
> - Container lifecycle
> - containerd architecture
> - containerd-shim
> - Tasks
> - Snapshots
> - Namespaces
> - OCI Runtime
> - Complete execution flow
> - How containers remain running

---

# 📖 Introduction

Earlier we learned this flow:

```text
docker run nginx

↓

Docker CLI

↓

dockerd

↓

containerd

↓

containerd-shim

↓

runc

↓

Linux Kernel

↓

Container
```

Now let's understand the two most important runtime components:

```
containerd

and

runc
```

Many developers think these are the same.

They are completely different.

---

# Why Do We Need containerd?

Imagine Docker had to manage everything itself.

```
Images

Containers

Snapshots

Networking

Storage

Tasks

Lifecycle

Runtime
```

Docker would become huge.

Instead Docker delegates container management.

```
Docker Engine

↓

containerd
```

containerd is responsible for the lifecycle of containers.

---

# What is containerd?

containerd is a **container runtime manager**.

Think of it as a supervisor.

Responsibilities:

✔ Pull images

✔ Unpack images

✔ Manage snapshots

✔ Create containers

✔ Start tasks

✔ Stop tasks

✔ Delete tasks

✔ Call OCI runtimes

Notice:

containerd does **not** create Linux namespaces itself.

It asks another program.

---

# What is runc?

runc is an **OCI Runtime**.

Its job is much smaller.

```
Create Container

↓

Exit
```

Responsibilities:

- Create namespaces
- Configure cgroups
- Mount filesystem
- Configure capabilities
- Start PID 1

Once the container starts:

```
runc exits
```

Many people incorrectly think runc stays alive.

It doesn't.

---

# containerd vs runc

| containerd | runc |
|------------|------|
| Runtime Manager | OCI Runtime |
| Long-running daemon | Short-lived process |
| Pulls images | Creates container |
| Manages tasks | Starts process |
| Manages snapshots | Configures kernel isolation |
| Talks to Docker/Kubernetes | Talks to Linux Kernel |

---

# containerd Architecture

```text
             Docker / Kubernetes
                     │
                     ▼
              containerd
                     │
      ┌──────────────┴──────────────┐
      ▼                             ▼
Snapshots                   Image Store
      │
      ▼
containerd-shim
      │
      ▼
runc
      │
      ▼
Linux Kernel
```

---

# Image Pulling

Suppose:

```bash
docker pull nginx
```

Flow:

```text
Docker

↓

containerd

↓

Registry

↓

Download Layers

↓

Store Layers

↓

Ready
```

containerd stores image layers locally.

---

# Snapshotters

Images are immutable.

Containers need writable storage.

containerd creates a snapshot.

```text
Image

↓

Snapshot

↓

Writable Layer

↓

Container
```

Common snapshotters:

- OverlayFS
- Btrfs
- ZFS
- Device Mapper

OverlayFS is the default on most Linux systems.

---

# Tasks

A **Container** is metadata.

A **Task** is the running process.

```text
Container

↓

Task

↓

Running Process
```

This distinction is important.

A container may exist without a running task.

---

# Container Lifecycle

```
Image

↓

Container Created

↓

Task Started

↓

Running

↓

Stopped

↓

Deleted
```

containerd manages every stage.

---

# Creating a Container

Step-by-step:

```text
Image

↓

Unpack

↓

Snapshot

↓

Container Metadata

↓

Task

↓

runc

↓

Container Running
```

---

# What Does runc Actually Do?

Suppose containerd says:

```
Start nginx
```

runc performs:

```
Create PID Namespace

↓

Create Mount Namespace

↓

Create Network Namespace

↓

Apply cgroups

↓

Mount Root Filesystem

↓

Start nginx

↓

Exit
```

Everything after that is handled by the kernel and containerd-shim.

---

# Why containerd-shim Exists

Suppose:

```
containerd

↓

Starts Container

↓

Crashes
```

Should the container stop?

No.

Instead:

```text
containerd

↓

containerd-shim

↓

Container
```

The shim becomes the parent process.

Responsibilities:

- Keep container alive
- Forward signals
- Collect exit status
- Reconnect containerd after restart

---

# Process Tree

Example:

```text
systemd
   │
dockerd
   │
containerd
   │
containerd-shim
   │
nginx (PID 1 inside container)
```

Notice:

containerd-shim—not runc—is the long-lived parent process.

---

# Linux Kernel Interaction

Eventually everything reaches the kernel.

```text
containerd

↓

runc

↓

Linux Kernel

↓

Namespaces

↓

cgroups

↓

Filesystem

↓

Networking

↓

Process
```

The kernel performs the real isolation.

---

# Why Kubernetes Uses containerd

Old architecture:

```text
Kubernetes

↓

Docker

↓

containerd

↓

runc
```

Modern architecture:

```text
Kubernetes

↓

containerd

↓

runc
```

Docker Engine is unnecessary because Kubernetes only needs a runtime manager.

---

# Common Misconceptions

❌ containerd creates namespaces.

✔ runc creates namespaces using the Linux kernel.

---

❌ runc stays running forever.

✔ runc exits immediately after starting the container.

---

❌ Docker is required for Kubernetes.

✔ Kubernetes communicates directly with containerd or CRI-O.

---

# Hands-on Exploration

View running processes:

```bash
ps aux | grep containerd
```

Check runc:

```bash
which runc
```

Inspect Docker information:

```bash
docker info
```

Observe how Docker reports the runtime.

---

# Real Execution Flow

Let's follow a complete request.

```text
docker run nginx

↓

Docker CLI

↓

Docker API

↓

dockerd

↓

containerd

↓

Pull Image

↓

Create Snapshot

↓

Create Container Metadata

↓

Create Task

↓

containerd-shim

↓

runc

↓

Linux Kernel

↓

Namespaces

↓

cgroups

↓

Mount Filesystem

↓

PID 1 Starts

↓

Container Running
```

---

# Interview Questions

## What is containerd?

A container runtime manager responsible for image management, snapshots, tasks, and lifecycle management.

---

## What is runc?

An OCI runtime that creates containers by configuring Linux namespaces, cgroups, filesystems, and processes.

---

## Why is containerd-shim needed?

It keeps containers alive independently of containerd and manages the running process lifecycle.

---

## Does runc remain running?

No.

It exits after successfully creating the container.

---

## Why did Kubernetes remove Docker?

Because Kubernetes only requires an OCI-compatible runtime manager like containerd or CRI-O, not the entire Docker Engine.

---

# Summary

containerd and runc have different responsibilities.

```
Docker

↓

containerd

↓

containerd-shim

↓

runc

↓

Linux Kernel

↓

Container
```

- **containerd** manages container lifecycle.
- **runc** creates the container.
- **containerd-shim** keeps the container running.
- **Linux Kernel** provides isolation.

Understanding this architecture explains how Docker and Kubernetes both rely on the same underlying runtime technologies.

---

# Next Chapter

## Chapter 30 — Docker Engine API

We'll learn:

- Docker REST API
- Docker Socket
- Docker SDKs
- Remote Docker API
- Authentication
- Security risks
- Automation
- Building your own Docker client