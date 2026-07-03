# 🐳 Docker Mastery Roadmap
# Module 1 — Why Docker Exists
## Chapter 4 — Linux Namespaces

> **Learning Objective**
>
> By the end of this chapter you will understand:
>
> - What Linux Namespaces are
> - Why Docker did not invent containers
> - How containers become isolated
> - Different namespace types
> - Real-world examples of process, network and filesystem isolation

---

# 📖 Introduction

In the previous chapter, we learned that:

> **A container is an isolated process.**

The next question is:

**How does Linux isolate one process from another?**

The answer is:

```
Linux Namespaces
```

Docker uses Linux namespaces to make one process believe it has its own machine.

---

# 🤔 What is a Namespace?

A namespace is a Linux kernel feature that gives a process its **own isolated view** of a system resource.

Think of it as creating a private world for a process.

Without namespaces:

```text
Host OS
│
├── Process A
├── Process B
└── Process C

All processes can see the same system resources.
```

With namespaces:

```text
Host OS
│
├── Container A
│   └── Process A
│
├── Container B
│   └── Process B
│
└── Container C
    └── Process C

Each process has its own isolated view.
```

---

# 🏠 Real-Life Analogy

Imagine an apartment building.

Without namespaces:

```
Everyone shares one room.
```

With namespaces:

```
Apartment 101

Apartment 102

Apartment 103
```

Each family believes it owns its apartment.

The building is shared, but each apartment is isolated.

Linux namespaces work in a similar way.

---

# 🧠 What Resources Can Be Isolated?

Linux supports multiple namespace types.

| Namespace | Isolates |
|-----------|----------|
| PID | Processes |
| NET | Network interfaces, IPs, ports |
| MNT | Filesystem mounts |
| UTS | Hostname and domain name |
| IPC | Shared memory and message queues |
| USER | User IDs and group IDs |
| CGROUP | cgroup hierarchy visibility |

Docker combines these namespaces to build containers.

---

# 1️⃣ PID Namespace (Process Isolation)

Normally:

```text
Host

PID 1 -> systemd
PID 500 -> Chrome
PID 800 -> Node
PID 1000 -> Redis
```

Every process can see many other processes.

With a PID namespace:

```text
Container

PID 1 -> Node App
PID 15 -> Worker
PID 22 -> Logger
```

Inside the container:

```
Node believes it is PID 1.
```

It cannot see host processes.

---

# 2️⃣ Network Namespace

Every container gets its own:

- Network interface
- IP address
- Routing table
- Port space

Example:

```text
Container A

IP: 172.18.0.2

localhost -> Container A
```

```text
Container B

IP: 172.18.0.3

localhost -> Container B
```

Important:

```
localhost
```

inside a container always refers to **that container**, not the host.

That's why containers communicate using service names such as:

```text
postgres
redis
backend
```

instead of localhost.

---

# 3️⃣ Mount Namespace

Containers get their own filesystem view.

Host:

```text
/home
/etc
/var
```

Container:

```text
/app
/usr
/etc
```

A container cannot freely browse your host filesystem.

It only sees what Docker mounts into it.

---

# 4️⃣ UTS Namespace

UTS isolates:

- Hostname
- Domain name

Container A:

```text
Hostname

backend
```

Container B:

```text
Hostname

postgres
```

Both believe they have different machine names.

---

# 5️⃣ IPC Namespace

IPC stands for:

```
Inter Process Communication
```

It isolates:

- Shared memory
- Message queues
- Semaphores

Processes inside one container cannot accidentally communicate with another container using IPC resources.

---

# 6️⃣ User Namespace

User namespaces map users inside the container to different users on the host.

Example:

Inside container:

```
root
```

Host:

```
UID 1001
```

This improves security because container root doesn't always equal host root.

---

# How Docker Builds a Container

When you execute:

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

The result feels like a separate machine, but it is only an isolated process.

---

# 🌍 Real Example

Your stack:

```text
Backend Container
PostgreSQL Container
Redis Container
```

Each container has:

```
Own PID namespace

Own Network namespace

Own Mount namespace

Own Hostname

Own IPC namespace
```

Yet they all share:

```
Host Linux Kernel
```

---

# 🧩 Why localhost Confuses Beginners

Backend:

```text
localhost:3000
```

means:

```
Backend Container
```

PostgreSQL:

```text
localhost:5432
```

means:

```
PostgreSQL Container
```

They are different network namespaces.

To connect:

```text
backend
    │
    ▼
postgres:5432
```

Docker DNS resolves the container name.

---

# 💼 Industry Benefits

Namespaces provide:

```
✅ Process isolation

✅ Network isolation

✅ Filesystem isolation

✅ Security

✅ Multi-tenant environments

✅ Lightweight containers
```

---

# ⚠️ Common Misconceptions

❌ Docker invented namespaces.

✔️ Linux introduced namespaces years before Docker.

---

❌ Containers have separate operating systems.

✔️ Containers share the host kernel but have isolated namespaces.

---

# 📌 Summary

- Namespaces are Linux kernel features.
- They isolate processes from each other.
- Docker combines multiple namespaces to create containers.
- Each container has its own view of processes, networking, filesystem and hostname.
- Containers share the same Linux kernel.

---

# 🎤 Interview Questions

## What are Linux namespaces?

Linux namespaces are kernel features that isolate system resources such as processes, networking, filesystems and hostnames, giving processes their own private view of the system.

---

## Why does localhost inside a container not refer to the host?

Because every container has its own network namespace. Its localhost refers only to itself.

---

## Did Docker invent containers?

No. Docker uses existing Linux kernel technologies like namespaces and cgroups to build containers.

---

# 📚 What's Next

**Chapter 5 — cgroups**

You'll learn:

- CPU limits
- Memory limits
- Disk I/O limits
- Why one container cannot consume all server resources
- How Docker prevents resource starvation
