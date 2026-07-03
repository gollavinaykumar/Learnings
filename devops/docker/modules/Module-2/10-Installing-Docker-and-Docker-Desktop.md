# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 1 — Installing Docker & Understanding Docker Desktop

> **Learning Objective**
>
> By the end of this chapter you'll:
>
> - Understand what actually gets installed with Docker
> - Know the difference between Docker Engine and Docker Desktop
> - Verify your installation
> - Understand where Docker runs on Windows, macOS and Linux
> - Learn the first commands every Docker developer should know

---

# Why Installation Matters

Many beginners think installing Docker simply installs one application.

In reality, Docker consists of multiple components working together.

```
You
 │
 ▼
Docker CLI
 │
 ▼
Docker Daemon (dockerd)
 │
 ▼
containerd
 │
 ▼
runc
 │
 ▼
Linux Kernel
```

On Linux these components run directly.

On macOS and Windows, Docker Desktop runs a lightweight Linux virtual machine because containers require a Linux kernel.

---

# Docker Engine vs Docker Desktop

## Docker Engine

Docker Engine is the actual container platform.

It includes:

- Docker CLI
- Docker Daemon
- Docker API

Linux servers typically install Docker Engine only.

---

## Docker Desktop

Docker Desktop is a developer application for Windows and macOS.

It provides:

- Docker Engine
- Docker CLI
- Docker Compose
- Container management UI
- Integrated Kubernetes (optional)
- Automatic updates

Think of Docker Desktop as a package that includes Docker Engine plus developer tools.

---

# Installation by Operating System

## Linux

Install Docker Engine directly.

Containers use the host Linux kernel.

```
Docker CLI
      │
Docker Engine
      │
Linux Kernel
```

---

## macOS

Containers cannot run directly on macOS because they require Linux.

Docker Desktop starts a lightweight Linux VM.

```
macOS
   │
Docker Desktop
   │
Linux VM
   │
Docker Engine
   │
Containers
```

---

## Windows

Docker Desktop uses either:

- WSL2 (recommended)
- Hyper-V

```
Windows
   │
Docker Desktop
   │
WSL2 Linux
   │
Docker Engine
   │
Containers
```

---

# Verify Installation

Check Docker version:

```bash
docker version
```

Example output:

```text
Client:
 Version: 28.x

Server:
 Engine: 28.x
```

Client = Docker CLI

Server = Docker Daemon

---

Display Docker information:

```bash
docker info
```

Useful information includes:

- Engine version
- Storage driver
- Number of containers
- Number of images
- CPU
- Memory

---

# Your First Commands

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

List downloaded images:

```bash
docker images
```

Display Docker help:

```bash
docker --help
```

---

# First Test

Run the official hello-world image:

```bash
docker run hello-world
```

What happens?

1. Docker checks if the image exists locally.
2. If missing, it downloads the image.
3. Docker creates a container.
4. The container prints a message.
5. The process exits.
6. The container enters the Exited state.

---

# Internal Flow

```text
docker run hello-world
        │
        ▼
Check Local Image
        │
Missing?
        │
        ▼
Download Image
        │
Create Container
        │
Start Process
        │
Print Message
        │
Exit
```

---

# Common Errors

## Cannot connect to the Docker daemon

Reason:

Docker daemon is not running.

Solution:

Start Docker Desktop or the Docker service.

---

## permission denied while trying to connect

Common on Linux.

Usually fixed by adding your user to the docker group.

---

# Best Practices

- Keep Docker updated.
- Use Docker Desktop only for development.
- Use Docker Engine on production Linux servers.
- Verify installation using `docker version` and `docker info`.

---

# Hands-on Lab

1. Run:

```bash
docker version
```

2. Run:

```bash
docker info
```

3. Run:

```bash
docker run hello-world
```

4. View the exited container:

```bash
docker ps -a
```

5. Remove it:

```bash
docker rm <container-id>
```

---

# Summary

- Docker Desktop is a developer application.
- Docker Engine is the actual runtime.
- Linux runs Docker natively.
- macOS and Windows use a lightweight Linux VM.
- `docker run hello-world` demonstrates the complete container lifecycle.

---

# What's Next

**Module 2 – Chapter 2: Docker CLI Deep Dive**

You'll learn every commonly used Docker command, what it does internally, and when to use it in real projects.
