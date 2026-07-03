# 🐳 Docker Mastery Roadmap

# Module 2 — Docker CLI & Everyday Docker

# Chapter 22 — Docker Performance

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Docker performance matters
> - Image optimization
> - Container startup optimization
> - CPU optimization
> - Memory optimization
> - Storage optimization
> - Network optimization
> - Build optimization
> - BuildKit
> - Monitoring performance
> - Benchmarking
> - Production performance tuning
> - Common mistakes
> - Best practices

---

# 📖 Introduction

One of Docker's biggest advantages is performance.

Unlike Virtual Machines, containers don't need an entire operating system.

They simply start another process.

Because of this:

```
Container Startup

↓

Milliseconds
```

instead of

```
Virtual Machine Startup

↓

Minutes
```

However...

A poorly built Docker image can still become:

❌ Slow

❌ Large

❌ Memory hungry

❌ CPU intensive

❌ Expensive to deploy

Understanding Docker performance is essential for production systems.

---

# What Does "Performance" Mean?

Docker performance is not just about speed.

It includes:

- Build Speed
- Image Size
- Startup Time
- Memory Usage
- CPU Usage
- Disk Usage
- Network Throughput

Think of performance as:

```
Performance

├── Build Performance
├── Runtime Performance
├── Storage Performance
├── Network Performance
└── Resource Usage
```

---

# 22.1 Image Size Optimization

One of the easiest performance wins is reducing image size.

Example:

```
Ubuntu Image

≈ 80 MB

↓

Alpine

≈ 7 MB
```

Smaller images mean:

✔ Faster builds

✔ Faster downloads

✔ Faster deployments

✔ Lower storage costs

---

## Bad Example

```dockerfile
FROM ubuntu

RUN apt update

RUN apt install nodejs

RUN apt install npm

COPY . .

RUN npm install
```

Problems

❌ Large base image

❌ Multiple RUN layers

❌ Cache inefficiency

---

## Better Example

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npm","start"]
```

Benefits

✔ Smaller image

✔ Faster build

✔ Better cache

---

# 22.2 Multi-stage Builds

Never deploy build tools.

Instead:

```
Builder Image

↓

Compile

↓

Production Image

↓

Only Compiled Files
```

Example:

```
Builder

Node
TypeScript
npm

↓

Runtime

Node

dist/

Only Production Dependencies
```

Benefits

✔ Smaller images

✔ Better security

✔ Faster startup

---

# 22.3 Layer Optimization

Docker builds images layer by layer.

Bad

```dockerfile
COPY . .

RUN npm install
```

Every source code change forces:

```
npm install
```

again.

Better

```dockerfile
COPY package*.json ./

RUN npm ci

COPY . .
```

Now Docker reuses cached dependencies.

---

# Layer Cache

```
Layer 1 ✔

Layer 2 ✔

Layer 3 ✔

Layer 4 Rebuild
```

Docker rebuilds only what changed.

---

# 22.4 BuildKit Performance

Docker BuildKit is much faster.

Traditional Builder

```
Instruction

↓

Wait

↓

Instruction

↓

Wait
```

BuildKit

```
Instruction A

Instruction B

Instruction C

↓

Parallel Execution
```

Benefits

✔ Parallel builds

✔ Better cache

✔ Secret mounting

✔ Remote cache

Enable

```bash
DOCKER_BUILDKIT=1 docker build .
```

---

# 22.5 CPU Performance

Containers share CPU.

Without limits

```
Container A

↓

100% CPU

↓

Container B

Starving
```

Use:

```bash
docker run \
--cpus=2
```

or

```bash
--cpu-shares
```

Production servers always limit CPU.

---

# CPU Scheduling

Linux uses

```
CFS

(Completely Fair Scheduler)
```

Docker communicates with

```
cgroups

↓

Linux Scheduler

↓

CPU
```

Every container gets fair CPU time.

---

# 22.6 Memory Performance

Containers share RAM.

Without limits

```
Container

↓

Consumes

64 GB RAM

↓

OOM
```

Limit memory

```bash
docker run \
-m 2g
```

Example

```
Backend

2 GB

↓

Redis

512 MB

↓

Postgres

4 GB
```

Every service receives predictable memory.

---

# OOM Killer

When memory finishes

Linux executes

```
OOM Killer
```

The kernel selects one process to terminate.

Docker containers can also be killed if limits are exceeded.

Typical error

```
Exit Code

137
```

Means

```
Killed

↓

Out Of Memory
```

---

# 22.7 Storage Performance

Bad

```
Container

↓

Stores Database

↓

Container Deleted

↓

Everything Lost
```

Good

```
Container

↓

Volume

↓

Persistent Data
```

Volumes are faster than repeatedly copying files into writable layers.

---

# 22.8 Network Performance

Docker networking is fast.

But:

```
Host Network

>

Bridge Network

>

Overlay Network
```

Approximate performance order.

Bridge networking is usually sufficient.

Host networking removes NAT overhead but reduces isolation.

Overlay networking is designed for multi-host communication and introduces additional encapsulation.

---

# 22.9 Logging Performance

Bad

```
Application

↓

Writes

100 GB Logs

↓

Container Filesystem
```

Filesystem becomes large.

Better

```
stdout

↓

Docker Logs

↓

ELK

↓

Grafana

↓

Cloud Logging
```

Never write huge logs inside containers.

---

# 22.10 Startup Performance

Reduce startup time.

Avoid

```
Container Starts

↓

Downloads Dependencies

↓

Runs Migrations

↓

Starts Application
```

Instead

```
Everything

Already Built

↓

Container Starts

↓

Application Runs
```

Containers should start within seconds.

---

# 22.11 Monitoring Performance

Useful commands

```bash
docker stats
```

Example

```
CPU

Memory

Network

Block I/O

PIDs
```

Inspect

```bash
docker inspect
```

Processes

```bash
docker top
```

Real-time monitoring

```
docker stats
```

is usually the first step when debugging performance.

---

# 22.12 Build Performance

Improve builds by

✔ Using `.dockerignore`

✔ Ordering Dockerfile instructions

✔ Multi-stage builds

✔ BuildKit

✔ Small contexts

Bad

```
Project

↓

3 GB Build Context
```

Good

```
Project

↓

120 MB Build Context
```

Huge contexts waste time.

---

# 22.13 Production Performance

Large companies optimize for

```
Smaller Images

↓

Less Network Traffic

↓

Faster Scaling

↓

Lower Cloud Cost
```

Netflix

Uber

Spotify

Amazon

Google

All optimize container images aggressively.

Sometimes saving

```
300 MB

↓

50 MB
```

across thousands of deployments saves terabytes of network transfer every day.

---

# 22.14 Performance Checklist

Before deploying

✔ Multi-stage build

✔ Small base image

✔ BuildKit

✔ Non-root user

✔ Memory limits

✔ CPU limits

✔ Volumes

✔ Health Checks

✔ Restart Policies

✔ Layer caching

---

# Common Mistakes

❌ Using Ubuntu when Alpine is sufficient

❌ Large build contexts

❌ Ignoring `.dockerignore`

❌ No CPU limits

❌ No memory limits

❌ Writing logs into the container filesystem

❌ Installing unnecessary packages

❌ Running build tools in production

---

# Hands-on Lab

## Compare Image Sizes

```bash
docker images
```

Build one image using:

```
ubuntu
```

Build another using:

```
node:22-alpine
```

Compare sizes.

---

## Observe Resource Usage

Run

```bash
docker stats
```

Open your application.

Watch:

CPU

Memory

Network

---

## Test Memory Limit

```bash
docker run \
-m 256m \
your-image
```

Observe what happens when the application exceeds the limit.

---

# Interview Questions

## Why are Alpine images preferred?

Because they are much smaller, reducing image size, download time, and attack surface.

---

## What causes slow Docker builds?

- Large build contexts
- Poor Dockerfile ordering
- Cache invalidation
- Multiple unnecessary layers

---

## Why use BuildKit?

To improve build speed through parallel execution, advanced caching, and secure secret handling.

---

## What does Exit Code 137 mean?

The container was terminated, typically because it exceeded its memory limit and was killed by the Linux OOM Killer.

---

## Which is faster: Bridge or Host networking?

Host networking generally has less overhead because it bypasses Docker's bridge and NAT, but it provides less isolation.

---

# Summary

Docker performance depends on **efficient image construction, optimized runtime configuration, and proper resource management**.

Key takeaways:

- Build smaller images.
- Use multi-stage builds.
- Leverage BuildKit.
- Optimize layer caching.
- Set CPU and memory limits.
- Use volumes for persistent data.
- Monitor containers with `docker stats`.
- Prefer production-ready Dockerfiles over development images.

A well-optimized Docker image is **smaller, faster, more secure, and cheaper to operate at scale.**

---

# Next Chapter

## Chapter 23 — Docker Debugging

We'll learn:

- Debugging container startup failures
- CrashLoop debugging
- Logs
- docker exec
- docker inspect
- docker events
- docker top
- docker stats
- Network debugging
- Volume debugging
- Production troubleshooting workflow