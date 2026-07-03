By now you've learned almost every Linux technology behind containers:

✅ Namespaces

✅ cgroups

✅ OverlayFS

✅ `veth`

✅ Linux Bridge

✅ Capabilities

✅ seccomp

✅ AppArmor / SELinux

Now it's time to answer the biggest question of all.

When you type:

```bash
docker run nginx
```

What **actually** happens?

Most developers imagine:

```
docker run

↓

Container Starts
```

Reality is much more interesting.

A single `docker run` command triggers dozens of operations across multiple software components and the Linux kernel.

After this chapter, you'll understand the complete journey from your terminal to a running container.

---

# 👑 Linux Kernel & Networking Mastery

# Module 9 — Docker Runtime Internals

# Chapter 58 — Putting It All Together: How Docker Creates a Container Internally

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Complete Docker Architecture
> - Docker CLI
> - Docker Daemon
> - containerd
> - containerd-shim
> - runc
> - OCI Runtime
> - Namespace Creation
> - cgroup Creation
> - OverlayFS Mount
> - Security Configuration
> - Process Startup
> - Complete Container Lifecycle

---

# 📖 One Simple Command

Everything starts with:

```bash
docker run nginx
```

Looks simple.

Behind the scenes,

many different components work together.

---

# High-Level Architecture

```
Docker CLI

↓

Docker Daemon

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

Each component has a specific responsibility.

---

# Step 1 — Docker CLI

You type:

```bash
docker run nginx
```

The Docker CLI:

- Parses the command
- Validates arguments
- Sends a request to the Docker daemon

The CLI does **not** create containers.

It is only a client.

---

# Step 2 — Docker Daemon

The Docker daemon (`dockerd`) receives the request.

Responsibilities include:

- Image management
- Network management
- Volume management
- Container lifecycle
- API handling

Think of it as Docker's control center.

---

# Step 3 — Image Check

Docker asks:

```
Is nginx Image Available?
```

If yes:

```
Use Local Image
```

Otherwise:

```
Docker Hub

↓

Download Image
```

---

# Step 4 — Image Extraction

Docker image:

```
Layer 1

↓

Layer 2

↓

Layer 3
```

Layers already exist on disk.

Docker prepares them for mounting.

---

# Step 5 — OverlayFS

Docker creates:

```
Read-only Layers

+

Writable Layer

↓

Merged Filesystem
```

Applications see:

```
/
```

This becomes the container's root filesystem.

---

# Step 6 — containerd

Docker hands responsibility to:

```
containerd
```

containerd is responsible for:

- Image management
- Snapshot management
- Runtime management
- Container lifecycle

It communicates with OCI runtimes.

---

# Step 7 — containerd-shim

containerd launches:

```
containerd-shim
```

Why?

Without it:

```
containerd Stops

↓

Container Stops
```

Not desirable.

The shim keeps the container alive even if higher-level components restart.

---

# Step 8 — runc

containerd calls:

```
runc
```

`runc` is the OCI runtime responsible for creating the actual container.

It interacts directly with the Linux kernel.

---

# Step 9 — Create Namespaces

`runc` asks the kernel to create:

```
PID Namespace

↓

Network Namespace

↓

Mount Namespace

↓

UTS Namespace

↓

IPC Namespace

↓

User Namespace (when configured)
```

Now the container has isolated views of system resources.

---

# Step 10 — Create cgroups

Linux creates:

```
CPU Limits

Memory Limits

PID Limits

I/O Limits
```

Resource usage is now controlled.

---

# Step 11 — Configure Networking

Linux creates:

```
veth Pair

↓

Linux Bridge

↓

IP Address

↓

Routing
```

Container networking becomes operational.

---

# Step 12 — Mount Filesystem

Linux mounts:

```
OverlayFS

↓

Merged Root Filesystem

↓

/
```

The application now sees what looks like a complete operating system.

---

# Step 13 — Apply Capabilities

Linux assigns:

```
Capability Set
```

Only the required capabilities are granted.

---

# Step 14 — Apply seccomp

Docker loads:

```
Default seccomp Profile
```

Dangerous system calls are filtered.

---

# Step 15 — Apply AppArmor / SELinux

If enabled,

Linux loads:

```
Security Profile
```

Mandatory Access Control becomes active.

---

# Step 16 — Set Hostname

Docker configures:

```
UTS Namespace

↓

Hostname
```

Example:

```
web

or

Container ID
```

---

# Step 17 — Start PID 1

Finally,

Linux executes:

```
execve()

↓

nginx
```

The first process becomes:

```
PID 1
```

inside the container.

---

# Step 18 — Container Running

Application starts.

Example:

```
PID 1

↓

nginx

↓

Worker Processes
```

The container is now operational.

---

# What Happens to `runc`?

Important.

`runc` creates the container,

then exits.

It is **not** a long-running process.

---

# What Keeps the Container Alive?

```
containerd-shim
```

It:

✔ Keeps stdio open

✔ Reports exit status

✔ Manages the running process

---

# Complete Creation Flow

```
docker run nginx

↓

Docker CLI

↓

dockerd

↓

Image Check

↓

OverlayFS

↓

containerd

↓

containerd-shim

↓

runc

↓

Namespaces

↓

cgroups

↓

Networking

↓

Capabilities

↓

seccomp

↓

AppArmor / SELinux

↓

execve()

↓

PID 1

↓

Running Container
```

---

# Container Architecture

```
Application

↓

PID Namespace

↓

Mount Namespace

↓

Network Namespace

↓

User Namespace

↓

cgroups

↓

Linux Kernel

↓

Hardware
```

Every container shares:

```
One Linux Kernel
```

---

# Why This Architecture?

Each layer has one responsibility.

| Component | Responsibility |
|-----------|----------------|
| Docker CLI | User Interface |
| Docker Daemon | Container Management |
| containerd | Runtime Management |
| containerd-shim | Keep Containers Alive |
| runc | Create Container |
| Linux Kernel | Isolation & Execution |

This separation makes Docker modular and reliable.

---

# Why Docker Doesn't Do Everything

Docker delegates to:

- containerd
- runc
- Linux Kernel

Each project focuses on its own domain.

This modular architecture is one reason the container ecosystem is so flexible.

---

# Real Example

Run:

```bash
docker run nginx
```

Linux performs roughly:

```
Create Namespaces

↓

Create cgroups

↓

Mount OverlayFS

↓

Configure Networking

↓

Apply Security Policies

↓

Start nginx
```

All of this usually happens in less than a second.

---

# Hands-on Lab

## Lab 1 — Run a Container

```bash
docker run -d nginx
```

---

## Lab 2 — View Running Processes

```bash
ps -ef | grep containerd
```

Observe:

```
containerd

containerd-shim
```

---

## Lab 3 — Inspect Container

```bash
docker inspect <container-id>
```

Observe:

- Network
- Mounts
- Hostname
- Process
- Cgroup settings

---

## Lab 4 — View Namespaces

```bash
lsns
```

Observe namespace IDs used by the container.

---

## Lab 5 — View cgroups

```bash
cat /proc/<host-pid>/cgroup
```

Observe the process's cgroup membership.

---

# Interview Questions

## What happens first when `docker run nginx` is executed?

The Docker CLI sends a request to the Docker daemon.

---

## What is the role of `containerd`?

It manages container lifecycle, images, snapshots, and communicates with OCI runtimes.

---

## Why is `containerd-shim` needed?

It keeps containers running independently of higher-level processes and manages the container process after creation.

---

## What does `runc` do?

`runc` creates the container by configuring Linux namespaces, cgroups, mounts, and then starting the initial process.

---

## Does `runc` remain running?

No.

It creates the container and exits.

---

## Who actually isolates containers?

The Linux kernel.

Docker configures the kernel; the kernel enforces isolation and resource limits.

---

## Which Linux features does Docker rely on?

- Namespaces
- cgroups
- OverlayFS
- `veth`
- Linux Bridge
- Capabilities
- seccomp
- AppArmor / SELinux

---

# Summary

A Docker container is created through a sequence of coordinated steps involving Docker components and the Linux kernel.

```
docker run

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

Namespaces

↓

cgroups

↓

OverlayFS

↓

Networking

↓

Security

↓

PID 1

↓

Running Container
```

Key concepts:

- The Docker CLI is only the client.
- `dockerd` orchestrates container creation.
- `containerd` manages container runtime operations.
- `containerd-shim` keeps containers alive.
- `runc` creates the container and exits.
- The Linux kernel provides isolation, networking, filesystems, and resource control.
- A container is fundamentally **a Linux process with namespaces, cgroups, a filesystem, and security policies applied**.

At this point, you now understand what actually happens inside Linux when someone runs:

```bash
docker run nginx
```

This knowledge forms the foundation for mastering Docker, Kubernetes, container runtimes, and Linux internals.

---

# Next Module

# Module 10 — Advanced Linux Networking Internals

We'll dive even deeper into the Linux networking stack:

- Chapter 59 — Linux Routing Table Internals
- Chapter 60 — Netfilter Architecture
- Chapter 61 — iptables Chains Explained
- Chapter 62 — nftables: The Future of Linux Firewalling
- Chapter 63 — conntrack Deep Dive
- Chapter 64 — Linux Traffic Control (`tc`)
- Chapter 65 — eBPF Fundamentals
- Chapter 66 — XDP (Express Data Path)
- Chapter 67 — CNI (Container Network Interface)
- Chapter 68 — Kubernetes Networking Internals

> **These chapters take you from "advanced Docker user" to understanding the Linux networking stack at the level expected of kernel engineers, platform engineers, and senior infrastructure developers.**