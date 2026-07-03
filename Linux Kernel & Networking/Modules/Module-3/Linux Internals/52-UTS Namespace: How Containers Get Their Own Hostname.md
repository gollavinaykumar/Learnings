Most developers have run:

```bash
docker run -it ubuntu bash
```

Then:

```bash
hostname
```

Output:

```text
4d2f8a3c9f17
```

Every container prints a different hostname.

But here's the interesting question.

Your Linux machine already has a hostname.

Example:

```text
vinay-laptop
```

How can every container have its **own hostname** while using the **same Linux kernel**?

If the container changes its hostname:

```bash
hostname my-container
```

Why doesn't your host machine become:

```text
my-container
```

The answer is:

**UTS Namespace.**

The UTS Namespace isolates the hostname and domain name for each container.

Although it's one of the smallest Linux namespaces,

it's essential for containers because applications often identify themselves using the hostname.

After this chapter, you'll understand how Linux gives every container its own identity.

---

# 👑 Linux Kernel & Networking Mastery

# Module 7 — Linux Process & Filesystem Isolation

# Chapter 52 — UTS Namespace: How Containers Get Their Own Hostname

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why UTS Namespace exists
> - What is a UTS Namespace?
> - Hostname Isolation
> - Domain Name Isolation
> - `hostname`
> - `/etc/hostname`
> - `/etc/hosts`
> - Docker Hostnames
> - Kubernetes Pod Hostnames
> - UTS Namespace Architecture

---

# 📖 Why Do We Need a UTS Namespace?

Suppose your Linux server hostname is:

```
prod-server-01
```

Now Docker starts:

```
Container A
```

```
Container B
```

```
Container C
```

Without isolation,

every container would think:

```
Hostname

↓

prod-server-01
```

Applications could not distinguish containers.

---

# What Does UTS Mean?

UTS stands for:

```
UNIX Time-sharing System
```

Historically,

the UTS Namespace isolates:

✔ Hostname

✔ NIS Domain Name (rarely used today)

In practice,

developers mostly care about the hostname.

---

# What is a UTS Namespace?

A UTS Namespace gives processes their own:

```
Hostname

+

Domain Name
```

Changing them affects only that namespace.

---

# Host Without UTS Namespace

Host:

```bash
hostname
```

Output:

```
prod-server-01
```

Every process sees:

```
prod-server-01
```

---

# With UTS Namespace

Host:

```
prod-server-01
```

Container A:

```
web
```

Container B:

```
redis
```

Container C:

```
postgres
```

Each namespace has its own hostname.

---

# Visual Representation

```
Linux Kernel

├── Host

│      Hostname

│      prod-server-01

│

├── Container A

│      Hostname

│      web

│

├── Container B

│      Hostname

│      redis

│

└── Container C

       Hostname

       postgres
```

---

# Checking Hostname

Run:

```bash
hostname
```

or

```bash
uname -n
```

Both display the hostname from the current UTS Namespace.

---

# Changing Hostname

Inside a namespace:

```bash
hostname backend
```

Result:

```
backend
```

Only processes inside that namespace observe the change.

The host hostname remains unchanged.

---

# `/etc/hostname`

Linux stores the configured hostname in:

```text
/etc/hostname
```

Docker commonly updates this file inside the container to match the container's hostname.

---

# `/etc/hosts`

Docker also generates:

```text
/etc/hosts
```

Example:

```text
127.0.0.1 localhost

172.17.0.2 web
```

This helps local hostname resolution inside the container.

---

# Why Does Hostname Matter?

Many applications use the hostname for:

- Logging
- Monitoring
- Distributed Systems
- Metrics
- Debugging

Example log:

```text
[web-01]

Application Started
```

Without hostname isolation,

every container would produce identical hostnames.

---

# Docker Default Hostname

If you don't specify one:

```bash
docker run ubuntu
```

Docker usually assigns:

```
Container ID
```

Example:

```
4d2f8a3c9f17
```

---

# Custom Hostname

Example:

```bash
docker run \
--hostname web \
ubuntu
```

Inside:

```bash
hostname
```

Output:

```
web
```

---

# UTS Namespace and Network Namespace

They are independent.

```
UTS Namespace

↓

Hostname
```

```
Network Namespace

↓

IP Address
```

Changing the hostname does **not** change the IP address.

Changing the IP address does **not** change the hostname.

---

# Kubernetes Example

Suppose:

```
Pod

↓

api-server
```

Inside:

```bash
hostname
```

Output:

```
api-server
```

Each Pod receives its own hostname.

Kubernetes can also configure DNS names based on Pod and Service settings.

---

# Why Separate Hostnames?

Imagine:

```
100 Containers
```

Without UTS Namespaces:

```
All

↓

Same Hostname
```

Troubleshooting would be extremely difficult.

---

# Hostname vs Container Name

These are different.

Container Name:

```text
my-nginx
```

Used by Docker for management.

Hostname:

```text
web
```

Visible inside the container.

Often they match,

but they don't have to.

---

# Container Identity

Applications often identify themselves using:

```
hostname()
```

Linux returns the hostname stored in the current UTS Namespace.

---

# Why UTS Namespace Matters

Although simple,

it provides clean isolation for:

- Logs
- Monitoring
- Distributed Applications
- Cluster Management

Every container gets its own identity.

---

# Hands-on Lab

## Lab 1 — Start a Container

```bash
docker run -it ubuntu bash
```

---

## Lab 2 — View Hostname

```bash
hostname
```

Observe the default hostname.

---

## Lab 3 — Set Custom Hostname

```bash
docker run \
-it \
--hostname web \
ubuntu
```

Then:

```bash
hostname
```

Observe:

```
web
```

---

## Lab 4 — View `/etc/hostname`

```bash
cat /etc/hostname
```

---

## Lab 5 — View `/etc/hosts`

```bash
cat /etc/hosts
```

Observe the hostname entry.

---

# Interview Questions

## What is a UTS Namespace?

A UTS Namespace isolates the hostname and domain name seen by processes.

---

## What resources does the UTS Namespace isolate?

- Hostname
- NIS Domain Name

---

## Why does every Docker container have a different hostname?

Because Docker creates a separate UTS Namespace for each container and assigns it a hostname.

---

## Does changing a container's hostname affect the host?

No.

The change is limited to the container's UTS Namespace.

---

## What is the default Docker hostname?

By default,

Docker commonly uses the container ID as the hostname.

---

## Is a Docker container name the same as its hostname?

No.

They are different concepts, although they may be configured to use the same value.

---

# Summary

The UTS Namespace gives every container its own hostname and domain name.

```
Container

↓

UTS Namespace

↓

Hostname

↓

Application
```

Key concepts:

- UTS Namespaces isolate hostnames.
- Every container can have a different hostname.
- Changing the hostname affects only the current namespace.
- Docker updates `/etc/hostname` and `/etc/hosts` inside the container.
- Kubernetes also uses UTS Namespaces for Pod identity.
- UTS Namespaces improve logging, monitoring, and debugging.

UTS Namespaces may be small, but they complete another important piece of container isolation.

---

# Next Chapter

## Chapter 53 — IPC Namespace: How Containers Isolate Shared Memory, Semaphores, and Message Queues

We'll explore:

- What is an IPC Namespace?
- Shared Memory
- Message Queues
- Semaphores
- POSIX IPC
- System V IPC
- Why IPC Isolation Matters
- Docker IPC Modes
- Kubernetes IPC Sharing
- Complete IPC Architecture

> **This chapter introduces process-to-process communication inside the Linux kernel and how containers isolate it.**