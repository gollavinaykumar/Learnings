# 🐳 Docker Mastery Roadmap
# Module 1 — Why Docker Exists
## Chapter 8 — Container Lifecycle

> **Learning Objective**
>
> By the end of this chapter you'll understand:
>
> - Every state of a Docker container
> - Difference between create, run, start, stop, kill and remove
> - Restart policies
> - What happens internally during lifecycle transitions
> - Best practices for production

---

# 📖 Introduction

A Docker container is not permanently "running".

Like any process, it moves through different states during its lifetime.

Understanding these states helps you debug, monitor and manage containers correctly.

---

# Container Lifecycle Overview

```text
          docker create
                │
                ▼
            CREATED
                │
        docker start
                │
                ▼
            RUNNING
        ┌───────┼────────┐
        │       │        │
        ▼       ▼        ▼
     PAUSED   STOPPED   EXITED
        │        │
docker unpause  docker start
        │        │
        └────────┘
                │
                ▼
           docker rm
                │
                ▼
             REMOVED
```

---

# 1. CREATED

Command:

```bash
docker create nginx
```

Docker:

- Creates writable layer
- Configures networking
- Creates metadata

But **does not start** the application.

Think of it as preparing a machine before switching it on.

View created containers:

```bash
docker ps -a
```

---

# 2. RUNNING

Command:

```bash
docker start <container>

# or

docker run nginx
```

`docker run` is actually equivalent to:

```text
docker create
        ↓
docker start
```

The main application process (PID 1 inside the container) is now executing.

---

# 3. PAUSED

Command:

```bash
docker pause myapp
```

The container still exists in memory, but its processes are temporarily suspended.

Resume:

```bash
docker unpause myapp
```

Use cases:

- Temporary maintenance
- Debugging
- Resource management

---

# 4. STOPPED

Command:

```bash
docker stop myapp
```

Docker sends a graceful termination signal.

Typical sequence:

```text
SIGTERM
   │
Wait (default ~10 seconds)
   │
Still running?
   │
SIGKILL
```

Applications get a chance to finish requests and release resources.

---

# 5. KILLED

Command:

```bash
docker kill myapp
```

Docker immediately sends:

```text
SIGKILL
```

The process is terminated without graceful shutdown.

Use only when a container refuses to stop.

---

# 6. EXITED

A container reaches the exited state when its main process finishes.

Example:

```bash
docker run alpine echo "Hello Docker"
```

Output:

```text
Hello Docker
```

The command completes, so the container exits automatically.

---

# 7. REMOVED

Command:

```bash
docker rm myapp
```

Docker removes:

- Writable layer
- Metadata
- Container record

The image is **not** removed.

---

# Image vs Container Removal

```bash
docker rm myapp
```

Removes:

```text
Container
```

Image remains.

---

```bash
docker rmi nginx
```

Removes:

```text
Image
```

Only if no containers depend on it.

---

# Internal Lifecycle

```text
docker run nginx
        │
        ▼
Image
        │
Create Container
        │
Namespaces
        │
cgroups
        │
Writable Layer
        │
Start nginx
        │
RUNNING
```

Stopping:

```text
RUNNING
   │
SIGTERM
   │
STOPPED
```

Removing:

```text
STOPPED
   │
docker rm
   │
REMOVED
```

---

# Restart Policies

Containers can automatically restart after crashes.

## No Restart (Default)

```bash
docker run nginx
```

Crash:

```text
Container Stops
```

---

## Always

```bash
docker run --restart=always nginx
```

If Docker or the server restarts, Docker automatically starts the container again.

---

## Unless Stopped

```bash
docker run --restart=unless-stopped nginx
```

Restart automatically unless you explicitly stopped it.

---

## On Failure

```bash
docker run --restart=on-failure nginx
```

Only restart when the application exits with an error.

---

# Real Production Example

Your stack:

```text
Frontend
Backend
Redis
PostgreSQL
```

Backend crashes.

With:

```text
--restart=always
```

Docker starts it again automatically.

Without restart policy:

```text
Backend Down

↓

API Unavailable
```

---

# Useful Commands

List running containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Start:

```bash
docker start myapp
```

Stop:

```bash
docker stop myapp
```

Restart:

```bash
docker restart myapp
```

Pause:

```bash
docker pause myapp
```

Resume:

```bash
docker unpause myapp
```

Remove:

```bash
docker rm myapp
```

---

# Common Misconceptions

❌ docker run only starts a container.

✔️ It creates and starts a new container.

---

❌ docker stop removes the container.

✔️ It only stops it.

---

❌ docker rm removes the image.

✔️ It removes only the container.

---

# Senior Engineer Notes

Production applications should:

- Handle SIGTERM gracefully.
- Finish active requests before exiting.
- Close database connections.
- Flush logs.
- Avoid using `docker kill` except during emergencies.

---

# Summary

- Containers move through multiple lifecycle states.
- `docker run` = create + start.
- `docker stop` performs graceful shutdown.
- `docker kill` immediately terminates the process.
- Restart policies improve availability.
- Removing a container does not remove its image.

---

# Interview Questions

## What is the difference between docker run and docker start?

`docker run` creates a new container from an image and starts it.

`docker start` starts an existing stopped container.

---

## What is the difference between docker stop and docker kill?

`docker stop` attempts a graceful shutdown using SIGTERM before SIGKILL.

`docker kill` immediately sends SIGKILL.

---

## Does docker rm remove the image?

No. It removes only the container.

---

# What's Next

**Chapter 9 — Module 1 Summary**

We'll review every concept learned so far and connect them into one complete picture before moving to Docker CLI and Dockerfile.
