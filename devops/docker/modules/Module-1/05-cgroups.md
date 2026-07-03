# 🐳 Docker Mastery Roadmap
# Module 1 — Why Docker Exists
## Chapter 5 — Linux cgroups (Control Groups)

> **Learning Objective**
>
> By the end of this chapter you will understand:
>
> - What cgroups are
> - Why namespaces alone are not enough
> - How Docker limits CPU, memory and other resources
> - Why one container cannot consume the entire server
> - How cgroups protect production systems

---

# 📖 Introduction

In the previous chapter we learned that **Namespaces isolate resources**.

However, namespaces do **not** limit resource usage.

Imagine two containers:

```text
Container A → Node.js API
Container B → PostgreSQL
```

If Container A starts using **100% CPU** and **all available RAM**, what happens?

Without resource limits:

```text
Server
│
├── Node Container → Uses 100% CPU
│
└── PostgreSQL → Starved of resources
```

The database becomes slow or even crashes.

Docker needs another Linux feature.

That feature is called:

```text
cgroups (Control Groups)
```

---

# 🤔 What is a cgroup?

A **Control Group (cgroup)** is a Linux kernel feature that controls **how much** of a resource a process or group of processes may use.

Think of it as a resource manager.

Namespaces answer:

> "What can this process see?"

cgroups answer:

> "How much CPU, memory and I/O can this process use?"

---

# 🏠 Real-Life Analogy

Imagine an apartment building.

Namespaces give every family its own apartment.

cgroups decide:

- Electricity quota
- Water quota
- Parking spaces

Everyone is isolated **and** prevented from consuming everything.

---

# Why Are cgroups Needed?

Suppose a server has:

```text
CPU : 8 Cores
RAM : 16 GB
```

Three containers are running:

```text
Backend

PostgreSQL

Redis
```

Without cgroups:

```text
Backend
CPU 100%
RAM 16 GB
```

Everything else slows down.

Production becomes unstable.

---

# With cgroups

Docker can enforce limits.

```text
Backend

CPU → 2 cores

Memory → 2 GB
```

Even if the backend has a bug, PostgreSQL and Redis continue running.

---

# Resource Types Managed by cgroups

cgroups can limit:

- CPU
- Memory
- Disk I/O
- Network priority (indirectly)
- Number of processes (PIDs)

---

# 1️⃣ CPU Limits

Example:

```bash
docker run --cpus=2 myapp
```

Meaning:

```text
Server: 8 CPUs

Container

Maximum Usage

2 CPUs
```

The container cannot consume more CPU than allowed.

---

# 2️⃣ Memory Limits

Example:

```bash
docker run -m 512m myapp
```

Meaning:

```text
Maximum Memory

512 MB
```

If the application exceeds the limit:

```text
Out Of Memory (OOM)

↓

Linux OOM Killer

↓

Container Terminated
```

---

# 3️⃣ PID Limits

Every application creates processes.

Without limits:

```text
Application Bug

↓

Creates Millions of Processes

↓

Server Crash
```

Docker can limit process count.

Example:

```bash
docker run --pids-limit=200 myapp
```

---

# 4️⃣ Block I/O Limits

Some applications constantly read/write files.

Heavy disk usage affects everyone.

cgroups can reduce the impact by limiting disk throughput.

---

# 🌍 Real Production Example

Imagine an e-commerce platform.

```text
Nginx

↓

Backend

↓

PostgreSQL

↓

Redis
```

Backend suddenly enters an infinite loop.

Without cgroups:

```text
Backend

CPU 100%

RAM 100%
```

Result:

```
❌ Website slow

❌ Database slow

❌ Redis slow

❌ Customers affected
```

With cgroups:

```text
Backend

CPU 2 Cores

RAM 2 GB
```

The backend may fail, but other services continue operating.

---

# Docker Commands

Limit memory:

```bash
docker run -m 1g myapp
```

Limit CPU:

```bash
docker run --cpus=2 myapp
```

Limit processes:

```bash
docker run --pids-limit=200 myapp
```

---

# Namespaces vs cgroups

| Feature | Namespaces | cgroups |
|---------|------------|----------|
| Purpose | Isolation | Resource Control |
| Processes | ✔ | ✖ |
| Network | ✔ | ✖ |
| Filesystem | ✔ | ✖ |
| CPU Limit | ✖ | ✔ |
| Memory Limit | ✖ | ✔ |
| Disk I/O Limit | ✖ | ✔ |

They work together.

```text
Namespaces
        +
cgroups
        =
Container
```

---

# Docker Internals

When you run:

```bash
docker run nginx
```

Docker roughly performs:

```text
Create Namespaces
        ↓
Create cgroups
        ↓
Prepare Filesystem
        ↓
Start nginx Process
```

Namespaces isolate.

cgroups limit.

---

# 💼 Why Companies Care

Resource limits help with:

```
✅ Multi-tenant systems

✅ Predictable performance

✅ Stable production

✅ Preventing noisy neighbors

✅ Better server utilization
```

---

# ⚠️ Common Misconceptions

❌ Namespaces limit memory.

✔️ cgroups limit memory.

---

❌ Docker itself limits CPU.

✔️ Docker configures Linux cgroups, which enforce the limits.

---

# 🧠 Senior Engineer Notes

In Kubernetes, every Pod should define:

- CPU requests
- CPU limits
- Memory requests
- Memory limits

These values are enforced using Linux cgroups.

Understanding cgroups makes Kubernetes resource management much easier.

---

# 📌 Summary

- cgroups are Linux kernel features.
- They control resource usage.
- Docker uses cgroups for CPU, memory, PID and I/O limits.
- Namespaces isolate containers.
- cgroups prevent containers from exhausting server resources.

---

# 🎤 Interview Questions

## What is a cgroup?

A cgroup (Control Group) is a Linux kernel feature that limits, prioritizes and monitors resource usage for a process or group of processes.

---

## Why are cgroups important in Docker?

They prevent one container from consuming excessive CPU, memory or other resources, improving stability and fairness.

---

## What's the difference between Namespaces and cgroups?

Namespaces provide isolation.

cgroups provide resource control.

---

# 📚 What's Next

**Chapter 6 — Docker Architecture**

You'll learn:

- Docker CLI
- Docker Engine
- Docker Daemon
- containerd
- runc
- OCI
- What happens internally after `docker run`
