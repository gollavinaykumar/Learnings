Most developers know this command:

```bash
docker run -it ubuntu bash
```

Inside the container:

```bash
ls /
```

Output:

```text
bin
etc
home
lib
proc
tmp
usr
var
```

It looks like a complete Linux operating system.

But here's the question:

**Where is the host filesystem?**

Your host may contain:

```
/Users

/home

/opt

/etc

/var

```

Why can't the container see everything?

Or another question:

If you delete:

```bash
rm -rf /
```

inside a container,

why doesn't your host operating system disappear?

The answer is:

**Mount Namespaces.**

Mount Namespaces give every container its own independent view of the filesystem.

Combined with OverlayFS, they make containers appear to have their own operating system.

After this chapter, you'll understand how Linux isolates filesystems for containers.

---

# 👑 Linux Kernel & Networking Mastery

# Module 7 — Linux Process & Filesystem Isolation

# Chapter 51 — Mount Namespace: How Containers Get Their Own Filesystem View

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Mount Namespaces exist
> - What is a Mount Namespace?
> - Mount Points
> - Root Filesystem
> - Filesystem Isolation
> - Bind Mounts
> - Docker Volumes
> - OverlayFS Integration
> - Read-only Mounts
> - Docker Filesystem Architecture

---

# 📖 Why Do We Need Mount Namespaces?

Suppose the host contains:

```
/

├── home

├── etc

├── var

├── opt

└── users
```

Now Docker starts:

```
Ubuntu Container
```

Without isolation,

the container could see:

```
Entire Host Filesystem
```

This is:

❌ Dangerous

❌ Insecure

Every container should see only its own filesystem.

---

# Normal Linux

One machine.

One filesystem tree.

```
/

├── bin

├── etc

├── home

├── usr

└── var
```

Every process sees the same mounted filesystems.

---

# Mount Namespace Solution

Linux creates:

```
Namespace A

↓

Own Mount Table
```

```
Namespace B

↓

Own Mount Table
```

Now each namespace can have different mounted filesystems.

---

# What is a Mount Namespace?

A Mount Namespace isolates:

✔ Mount Points

✔ Root Filesystem

✔ Mounted Devices

✔ Bind Mounts

✔ Overlay Mounts

Processes only see mounts inside their namespace.

---

# Think of a Window

Imagine the host filesystem as a huge building.

Every Mount Namespace gets its own window.

```
Host Filesystem

↓

Namespace Window

↓

Visible Files
```

Each process sees only its assigned view.

---

# Root Filesystem

Every process begins at:

```
/
```

Inside a container,

`/` is **not** necessarily the host's root filesystem.

It is the container's root filesystem.

---

# Host View

```
/

├── home

├── etc

├── usr

├── var

└── boot
```

---

# Container View

```
/

├── bin

├── etc

├── app

├── usr

└── var
```

Looks similar,

but it's a different filesystem view.

---

# Same Kernel

Important:

Containers **do not** have their own kernel.

```
Host Kernel

↓

Container Root Filesystem
```

Only the filesystem view changes.

---

# Mount Table

Linux maintains:

```
Mount Table
```

Example:

```bash
mount
```

or

```bash
findmnt
```

Each Mount Namespace has its own mount table.

---

# Different Mount Tables

Host:

```
/

↓

Disk
```

Container:

```
/

↓

OverlayFS
```

Different namespaces,

different mount information.

---

# Creating a Mount Namespace

Linux internally creates a new Mount Namespace for a container.

Inside that namespace,

Docker mounts the container's root filesystem.

---

# Changing Mounts

Suppose inside the container:

```bash
mount
```

New mounts appear.

Host mount table remains unchanged.

---

# Bind Mount

Docker example:

```bash
docker run \
-v /host/data:/app/data
```

Host:

```
/host/data
```

appears inside the container as:

```
/app/data
```

No data is copied.

Both paths refer to the same underlying files.

---

# Bind Mount Flow

```
Host Directory

↓

Bind Mount

↓

Container Directory
```

Changes are immediately visible from both sides.

---

# Docker Volume

Volumes are different.

Docker manages them.

Example:

```
/var/lib/docker/volumes
```

Docker mounts the volume into the container.

Applications simply see a normal directory.

---

# Read-only Mount

Example:

```bash
docker run \
-v /config:/config:ro
```

Container:

```
Read

✔

Write

❌
```

Useful for configuration files.

---

# OverlayFS

This is where Mount Namespaces become powerful.

Suppose the image contains:

```
Ubuntu Files
```

Docker creates:

```
Read-only Layers

+

Writable Layer
```

Mount Namespace presents them as one filesystem.

Applications believe they're working with a single disk.

---

# OverlayFS Architecture

```
Read-only Layer

↓

Read-only Layer

↓

Read-only Layer

↓

Writable Layer

↓

Merged View

↓

Container
```

The merged view becomes:

```
/
```

inside the container.

---

# Deleting Files

Suppose:

```bash
rm file.txt
```

Docker does **not** modify the lower read-only image layers.

Instead,

OverlayFS records the deletion in the writable layer.

The original image remains unchanged.

---

# Container Isolation

Suppose:

```
Container A

↓

Deletes File
```

Container B:

```
File Still Exists
```

Each container has its own writable layer.

---

# Host Files

Unless explicitly mounted,

the container cannot access host directories.

This protects the host filesystem.

---

# Docker Example

Run:

```bash
docker run ubuntu
```

Docker creates:

```
Mount Namespace

↓

OverlayFS

↓

Container Root
```

Applications see:

```
/
```

without knowing OverlayFS is underneath.

---

# Kubernetes Example

Each Pod gets:

```
Mount Namespace
```

Volumes such as:

- `emptyDir`
- Persistent Volumes
- ConfigMaps
- Secrets

are mounted inside that namespace.

---

# Why Mount Namespaces Matter

Everything involving container storage depends on them.

Examples:

- Docker
- Kubernetes
- Podman
- LXC
- containerd

Without Mount Namespaces,

containers would directly access the host filesystem.

---

# Hands-on Lab

## Lab 1 — Start a Container

```bash
docker run -it ubuntu bash
```

---

## Lab 2 — View Mounts

```bash
mount
```

or

```bash
findmnt
```

Observe the container's mount table.

---

## Lab 3 — Bind Mount

```bash
docker run \
-v $(pwd):/workspace \
-it ubuntu bash
```

Observe the shared files.

---

## Lab 4 — Read-only Mount

```bash
docker run \
-v $(pwd):/workspace:ro \
-it ubuntu bash
```

Attempt to create a file.

Observe the permission failure.

---

## Lab 5 — Inspect Overlay Storage

On the host:

```bash
docker inspect <container-id>
```

Observe storage-related information.

---

# Interview Questions

## What is a Mount Namespace?

A Mount Namespace isolates a process's view of mounted filesystems.

---

## Why does every container appear to have its own root filesystem?

Because Docker creates a separate Mount Namespace and mounts a container-specific root filesystem within it.

---

## Does a Mount Namespace create another Linux kernel?

No.

All containers share the host kernel.

Only the filesystem view is isolated.

---

## What is the difference between a Bind Mount and a Docker Volume?

A Bind Mount maps an existing host directory into the container.

A Docker Volume is managed by Docker and stored in Docker's volume storage.

---

## Why is OverlayFS used together with Mount Namespaces?

OverlayFS merges multiple image layers and a writable layer into a single filesystem view, which the Mount Namespace presents as the container's root filesystem.

---

## Can a container normally see the host filesystem?

No.

Only explicitly mounted directories become visible.

---

# Summary

Mount Namespaces isolate filesystem views for Linux processes.

```
Container

↓

Mount Namespace

↓

OverlayFS

↓

Merged Root Filesystem

↓

Application
```

Key concepts:

- Mount Namespaces isolate mounted filesystems.
- Every container gets its own root filesystem.
- Bind Mounts expose selected host directories.
- Docker Volumes provide managed persistent storage.
- OverlayFS combines image layers into a unified filesystem.
- Containers share the host kernel while seeing independent filesystem layouts.

Together with PID and Network Namespaces, Mount Namespaces make containers behave like independent operating systems.

---

# Next Chapter

## Chapter 52 — UTS Namespace: How Containers Get Their Own Hostname

We'll explore:

- What is the UTS Namespace?
- Hostname Isolation
- Domain Name Isolation
- `hostname` Command
- `/etc/hostname`
- `/etc/hosts`
- Docker Hostname Configuration
- Kubernetes Pod Hostnames
- Why Hostname Isolation Matters
- Complete UTS Namespace Architecture