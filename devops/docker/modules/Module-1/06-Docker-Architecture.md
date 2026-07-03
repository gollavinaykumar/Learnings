# 🐳 Docker Mastery Roadmap
# Module 1 — Why Docker Exists
## Chapter 6 — Docker Architecture

> **Learning Objective**
>
> By the end of this chapter you'll understand:
>
> - Docker's major components
> - Docker CLI
> - Docker Daemon
> - Docker Engine
> - containerd
> - runc
> - OCI
> - What happens internally when you run `docker run`

---

# 📖 Introduction

Most developers know this command:

```bash
docker run nginx
```

But very few know what actually happens after pressing **Enter**.

Docker performs many steps before your application starts.

---

# High-Level Architecture

```text
                 You
                  │
                  ▼
          docker run nginx
                  │
                  ▼
           Docker CLI (Client)
                  │
        REST API / Unix Socket
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
 Linux Kernel (Namespaces + cgroups)
                  │
                  ▼
          Running Container
```

---

# 1. Docker CLI

The Docker CLI is the command-line tool.

Examples:

```bash
docker run nginx
docker ps
docker images
docker logs
```

The CLI **does not create containers**.

Its job is simply to send requests to the Docker Daemon.

Think of it as a remote control.

---

# 2. Docker Daemon (dockerd)

The Docker Daemon is the brain of Docker.

Responsibilities:

- Build images
- Pull images
- Create containers
- Manage networks
- Manage volumes
- Talk to containerd

It runs continuously in the background.

Verify:

```bash
docker info
```

If the daemon is stopped:

```text
Cannot connect to the Docker daemon
```

---

# CLI → Daemon Communication

```text
Docker CLI
      │
      ▼
Unix Socket
/var/run/docker.sock
      │
      ▼
Docker Daemon
```

On Linux, communication normally happens through a Unix socket.

---

# 3. Docker Engine

Docker Engine is the complete runtime platform.

It includes:

```text
Docker CLI
+
Docker Daemon
+
Docker API
```

When people say "Install Docker", they usually mean installing Docker Engine.

---

# 4. containerd

Docker does not directly start containers anymore.

Instead it delegates to:

```text
containerd
```

Responsibilities:

- Manage container lifecycle
- Pull images
- Store images
- Snapshot management
- Supervise containers

Docker asks containerd to create containers.

---

# 5. runc

containerd uses:

```text
runc
```

runc is a low-level runtime.

Its job:

- Create namespaces
- Configure cgroups
- Mount filesystem
- Start the application process

Once the process starts, runc exits.

---

# 6. OCI

OCI stands for:

```text
Open Container Initiative
```

OCI defines standards for:

- Container images
- Container runtimes

Because Docker follows OCI standards, other tools like Podman and Kubernetes can use OCI-compatible images.

---

# What Happens During `docker run nginx`

Step 1

```bash
docker run nginx
```

Step 2

```text
Docker CLI
        │
        ▼
Docker Daemon
```

Step 3

Daemon checks whether the image exists.

```text
Image Found?
```

If not:

```text
Pull Image
```

from Docker Hub.

---

Step 4

Docker asks containerd:

```text
Create Container
```

---

Step 5

containerd calls:

```text
runc
```

---

Step 6

runc performs:

```text
Create Namespaces
        ↓
Create cgroups
        ↓
Mount Filesystem
        ↓
Start nginx Process
```

---

Step 7

```text
nginx Running
```

Container is ready.

---

# Visual Flow

```text
You
 │
 ▼
docker run nginx
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

---

# Real Example

Your stack:

```text
Frontend
Backend
PostgreSQL
Redis
```

Running:

```bash
docker compose up
```

Docker repeats the same architecture flow for every service.

---

# Why This Design?

Instead of one huge program doing everything:

```text
Docker
```

Docker delegates responsibilities.

| Component | Responsibility |
|-----------|----------------|
| CLI | User commands |
| Daemon | Management |
| containerd | Container lifecycle |
| runc | Create container |
| Linux Kernel | Isolation & resource control |

This modular design improves maintainability.

---

# Common Misconceptions

❌ Docker CLI starts containers.

✔️ Docker CLI only sends requests.

---

❌ Docker directly creates namespaces.

✔️ runc creates namespaces and cgroups using Linux kernel features.

---

# Senior Engineer Notes

Modern Kubernetes uses container runtimes such as:

- containerd
- CRI-O

Understanding containerd makes Kubernetes internals much easier to understand.

---

# Summary

- Docker CLI is the client.
- Docker Daemon is the brain.
- Docker Engine includes CLI + Daemon + API.
- containerd manages container lifecycle.
- runc creates the container.
- Linux kernel provides namespaces and cgroups.

---

# Interview Questions

## What happens internally after `docker run`?

CLI → Daemon → containerd → runc → Linux kernel → Container.

## What is containerd?

A container runtime responsible for managing the lifecycle of containers.

## What is runc?

A low-level OCI runtime that creates containers using Linux namespaces and cgroups.

---

# What's Next

**Chapter 7 — Docker Images vs Containers**

You'll learn:

- What is a Docker Image?
- Layers
- Copy-on-write
- Why images are immutable
- Multiple containers from one image
