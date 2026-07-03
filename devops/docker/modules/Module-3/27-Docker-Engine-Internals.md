# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 27 — Docker Engine Internals

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Docker Engine Architecture
> - Docker CLI
> - Docker REST API
> - dockerd
> - containerd
> - containerd-shim
> - runc
> - OCI Runtime
> - Linux Kernel
> - Complete execution flow of `docker run`
> - How a container is actually created

---

# 📖 Introduction

Throughout this course, we've been using commands like:

```bash
docker run nginx
```

It looks simple.

But internally, Docker performs **dozens of operations** before the container starts.

Many developers think Docker directly creates containers.

It doesn't.

Docker is actually a collection of components working together.

---

# The Big Picture

When you execute:

```bash
docker run nginx
```

The request flows through several layers.

```text
You
 │
 ▼
Docker CLI
 │
 ▼
Docker REST API
 │
 ▼
dockerd
 │
 ▼
containerd
 │
 ▼
containerd-shim
 │
 ▼
runc
 │
 ▼
Linux Kernel
 │
 ▼
Container Starts
```

Every component has a specific responsibility.

---

# Docker Architecture

```text
                Docker Platform

+---------------------------------------+
|           Docker CLI                  |
+---------------------------------------+
                │
                ▼
+---------------------------------------+
|          Docker REST API              |
+---------------------------------------+
                │
                ▼
+---------------------------------------+
|      Docker Daemon (dockerd)          |
+---------------------------------------+
                │
                ▼
+---------------------------------------+
|          containerd                   |
+---------------------------------------+
                │
                ▼
+---------------------------------------+
|       containerd-shim                 |
+---------------------------------------+
                │
                ▼
+---------------------------------------+
|              runc                     |
+---------------------------------------+
                │
                ▼
+---------------------------------------+
|          Linux Kernel                 |
+---------------------------------------+
```

---

# Step 1 — Docker CLI

The Docker CLI is the command-line interface.

Example:

```bash
docker run nginx
```

The CLI itself **does not create containers**.

Its job is to:

- Parse your command
- Validate arguments
- Send an API request to the Docker daemon

Think of it as a client application.

---

# Step 2 — Docker REST API

The Docker CLI communicates with Docker through an API.

```text
Docker CLI

↓

HTTP Request

↓

Docker Daemon
```

Even local Docker commands are converted into API requests.

On Linux, this communication usually happens through a Unix socket:

```text
/var/run/docker.sock
```

On Windows, it may use a named pipe.

This API is also why many programming languages have Docker SDKs.

---

# Step 3 — Docker Daemon (dockerd)

`dockerd` is the main Docker service.

It runs in the background and manages Docker resources.

Responsibilities include:

- Managing images
- Managing containers
- Managing networks
- Managing volumes
- Managing plugins
- Exposing the Docker API
- Delegating container lifecycle tasks

Notice something important:

`dockerd` **does not directly create containers**.

Instead, it delegates that responsibility.

---

# Step 4 — containerd

`containerd` is the container runtime manager.

It is responsible for:

- Pulling images
- Managing image layers
- Managing snapshots
- Managing container lifecycle
- Managing tasks
- Calling OCI runtimes

Flow:

```text
dockerd

↓

containerd
```

Docker Engine relies on containerd for most container management.

Today, Kubernetes also uses containerd directly.

---

# Step 5 — containerd-shim

Why do we need another process?

Suppose:

```
containerd

↓

Creates Container

↓

Exits
```

Who keeps the container alive?

That's the job of `containerd-shim`.

Responsibilities:

- Be the parent process of the container
- Keep STDIN/STDOUT connected
- Report exit status
- Allow containerd to restart without killing running containers

Flow:

```text
containerd

↓

containerd-shim

↓

Container
```

Without the shim, container lifecycle management would be more fragile.

---

# Step 6 — runc

`runc` is the OCI runtime.

Its responsibility is simple:

Create the container.

How?

It asks the Linux kernel to:

- Create namespaces
- Apply cgroups
- Mount the filesystem
- Set capabilities
- Configure networking
- Start PID 1

After this setup, `runc` exits.

It does **not** stay running.

---

# Step 7 — Linux Kernel

The Linux kernel performs the actual isolation.

It creates:

```text
Namespaces
```

Examples:

- PID Namespace
- Mount Namespace
- Network Namespace
- IPC Namespace
- UTS Namespace
- User Namespace

It also applies:

```text
cgroups
```

to limit:

- CPU
- Memory
- Disk I/O
- Network resources

The kernel is where the container truly comes to life.

---

# Complete Execution Flow

Let's trace the entire process.

```text
User

↓

docker run nginx

↓

Docker CLI

↓

Docker REST API

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

Namespaces Created

↓

cgroups Applied

↓

Filesystem Mounted

↓

Network Configured

↓

PID 1 Started

↓

Running Container
```

This is what really happens every time you start a container.

---

# Why Doesn't Docker Create Containers Directly?

Historically, Docker handled everything itself.

Over time, the ecosystem evolved.

Docker delegated container creation to:

- containerd
- runc

Benefits:

- Better modularity
- OCI compatibility
- Reusable runtimes
- Kubernetes integration

This separation is one reason Kubernetes can use containerd without Docker Engine.

---

# Docker Engine vs containerd

| Docker Engine | containerd |
|---------------|------------|
| Full platform | Container runtime |
| Images | Tasks |
| Volumes | Snapshots |
| Networks | Runtime lifecycle |
| Docker API | OCI runtime integration |

Docker Engine provides the user experience.

containerd provides runtime management.

---

# What Happens If dockerd Stops?

A common interview question.

Suppose:

```text
dockerd

↓

Stopped
```

Do running containers stop?

**No.**

Because:

```text
containerd-shim

↓

Still Running

↓

Container Continues
```

You won't be able to create new containers until `dockerd` returns, but existing ones usually continue running.

---

# Hands-on Exploration

Check Docker daemon information:

```bash
docker info
```

View running processes:

```bash
ps aux | grep dockerd
```

Check containerd:

```bash
ps aux | grep containerd
```

On Linux, inspect the Docker socket:

```bash
ls -l /var/run/docker.sock
```

These commands help you see the architecture in action.

---

# Common Misconceptions

❌ Docker Engine creates containers directly.

✔ Docker Engine delegates container creation to containerd and runc.

---

❌ runc stays running forever.

✔ runc exits after creating the container.

---

❌ Docker CLI talks directly to the Linux kernel.

✔ Docker CLI communicates with dockerd through the Docker API.

---

# Interview Questions

### What happens internally when you run `docker run nginx`?

The request flows through:

Docker CLI → Docker API → dockerd → containerd → containerd-shim → runc → Linux Kernel.

---

### What is the role of containerd?

It manages images, snapshots, tasks, and container lifecycle, and invokes OCI runtimes.

---

### Why is containerd-shim needed?

It becomes the parent process for containers, allowing them to continue running even if containerd or dockerd is restarted.

---

### What does runc do?

It creates the container by asking the Linux kernel to configure namespaces, cgroups, mounts, networking, and then starts the container's initial process.

---

### Does Docker Engine require runc?

Yes. Docker Engine relies on an OCI-compatible runtime such as runc to actually create containers.

---

# Summary

Docker is **not a single program**.

It is a layered architecture.

```text
Docker CLI
      │
Docker API
      │
dockerd
      │
containerd
      │
containerd-shim
      │
runc
      │
Linux Kernel
      │
Container
```

Each component has a well-defined responsibility.

Understanding this architecture is essential before learning OCI, containerd internals, and Kubernetes.

---

# Next Chapter

## Chapter 28 — OCI (Open Container Initiative)

We'll learn:

- Why OCI was created
- OCI Image Specification
- OCI Runtime Specification
- OCI Distribution Specification
- Why Docker is no longer required by Kubernetes
- How Podman, CRI-O, containerd, and Docker all work together through OCI standards