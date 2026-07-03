# 🐳 Docker Mastery Roadmap
# Module 1 — Summary
## Why Docker Exists & Container Fundamentals

> **Module Goal**
>
> In this module you learned **why Docker exists** and the Linux technologies that make containers possible.
>
> Before learning Docker commands, it is essential to understand these core concepts.

---

# 🗺️ Module Roadmap

```text
Chapter 1  → Why Docker Exists
Chapter 2  → Virtual Machines vs Containers
Chapter 3  → What is a Container?
Chapter 4  → Linux Namespaces
Chapter 5  → Linux cgroups
Chapter 6  → Docker Architecture
Chapter 7  → Images vs Containers
Chapter 8  → Container Lifecycle
```

---

# 1. Why Docker Exists

## The Problem

Applications depend on much more than source code.

```text
Application
+
Runtime
+
Libraries
+
Operating System
+
Configuration
+
Environment Variables
+
Database
```

Different developer environments caused:

- "It works on my machine"
- Version conflicts
- Missing dependencies
- Deployment failures

## Docker's Solution

Package the application together with everything it needs.

```text
Docker Image

├── Application
├── Runtime
├── Libraries
├── Dependencies
└── Configuration
```

Core philosophy:

> **Build Once, Run Anywhere**

---

# 2. Virtual Machines vs Containers

## Virtual Machine

```text
Application
↓
Runtime
↓
Guest Operating System
↓
Hypervisor
↓
Hardware
```

## Container

```text
Application
↓
Runtime
↓
Docker Engine
↓
Host Operating System
↓
Hardware
```

### Key Difference

VMs include a guest operating system.

Containers share the host operating system kernel.

---

# 3. What is a Container?

A container is **not** a virtual machine.

A container is:

> **An isolated process running on the host operating system.**

Think:

```text
Java

Class
  ↓
Object
```

Docker:

```text
Image
  ↓
Container
```

---

# 4. Linux Namespaces

Namespaces provide isolation.

Docker uses them to isolate:

- Processes
- Network
- Filesystem
- Hostname
- IPC
- Users

```text
Container A

localhost
PID 1
Filesystem A
```

```text
Container B

localhost
PID 1
Filesystem B
```

Each container has its own view of the system.

---

# 5. Linux cgroups

Namespaces isolate.

cgroups control resources.

cgroups limit:

- CPU
- Memory
- Disk I/O
- Process count

Without cgroups:

```text
One container

↓

Consumes Entire Server
```

With cgroups:

```text
Every container

↓

Gets Fair Resources
```

---

# 6. Docker Architecture

```text
You
 │
 ▼
Docker CLI
 │
 ▼
Docker Daemon
 │
 ▼
containerd
 │
 ▼
runc
 │
 ▼
Linux Kernel
 │
 ▼
Container
```

Responsibilities:

| Component | Responsibility |
|-----------|----------------|
| Docker CLI | Accept user commands |
| Docker Daemon | Manage Docker |
| containerd | Manage container lifecycle |
| runc | Create containers |
| Linux Kernel | Isolation & resource control |

---

# 7. Images vs Containers

## Image

- Blueprint
- Read-only
- Immutable

## Container

- Running instance
- Writable layer
- Running processes

Relationship:

```text
Dockerfile
      │
docker build
      │
Image
      │
docker run
      │
Container
```

---

# 8. Container Lifecycle

```text
CREATE
   │
START
   │
RUNNING
 │  │
 │  ├── STOPPED
 │  ├── PAUSED
 │  └── EXITED
 │
REMOVE
```

Common commands:

```bash
docker create
docker run
docker start
docker stop
docker restart
docker pause
docker unpause
docker rm
```

---

# Complete Picture

```text
Developer
     │
Writes Dockerfile
     │
     ▼
docker build
     │
     ▼
Docker Image
     │
docker run
     │
     ▼
Docker CLI
     │
     ▼
Docker Daemon
     │
     ▼
containerd
     │
     ▼
runc
     │
     ▼
Namespaces + cgroups
     │
     ▼
Running Container
```

---

# Common Misconceptions

❌ Docker invented containers.

✔ Linux kernel already had namespaces and cgroups.

---

❌ Containers are virtual machines.

✔ Containers are isolated processes.

---

❌ Images execute applications.

✔ Containers execute applications.

---

❌ docker run only starts a container.

✔ docker run creates and starts a new container.

---

# Senior Engineer Notes

A senior engineer understands that Docker is built on Linux kernel features.

Docker itself does not isolate processes.

Instead it orchestrates:

- Linux namespaces
- Linux cgroups
- Overlay filesystem
- OCI runtimes

Understanding these concepts makes Kubernetes and production debugging much easier.

---

# Interview Revision

## Why was Docker created?

To solve environment inconsistency by packaging applications with their dependencies.

---

## What is a container?

A lightweight isolated process.

---

## Why are containers lightweight?

They share the host operating system kernel.

---

## What provides isolation?

Linux namespaces.

---

## What limits CPU and memory?

Linux cgroups.

---

## What happens after docker run?

CLI → Daemon → containerd → runc → Linux Kernel → Container.

---

## Image vs Container?

Image = Blueprint

Container = Running instance

---

# Module 1 Checklist

- [x] Why Docker Exists
- [x] Virtual Machines
- [x] Containers
- [x] Namespaces
- [x] cgroups
- [x] Docker Architecture
- [x] Images
- [x] Containers
- [x] Lifecycle

---

# Next Module

## Module 2 — Docker CLI & Everyday Docker

You'll learn:

- Installing Docker
- Docker CLI
- docker pull
- docker run
- docker exec
- docker logs
- docker inspect
- docker cp
- docker stats
- docker top
- docker image
- docker container
- Real-world hands-on labs

By the end of Module 2 you'll be comfortable working with Docker daily as a developer.
