Most developers think:

If a process runs as:

```text
root
```

inside a Docker container,

then it has full control over the host machine.

That sounds dangerous.

Imagine running:

```bash
docker run -it ubuntu bash
```

Inside the container:

```bash
whoami
```

Output:

```text
root
```

Question:

**Is this the same root user as the host Linux root?**

The answer is:

**Not necessarily.**

Modern Linux provides:

**User Namespaces.**

A User Namespace allows a process to be:

```
root

Inside

the Container
```

while being:

```
Normal User

On

the Host
```

This is one of the biggest security improvements in Linux containers.

After this chapter, you'll understand how Linux separates user identities between containers and the host.

---

# 👑 Linux Kernel & Networking Mastery

# Module 7 — Linux Process & Filesystem Isolation

# Chapter 54 — User Namespace: Running Root Inside a Container Without Being Root on the Host

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why User Namespaces exist
> - What is a User Namespace?
> - UID & GID
> - User Mapping
> - Root Inside Container
> - Root on Host
> - UID Translation
> - Docker User Namespace Remapping
> - Rootless Containers
> - Security Benefits

---

# 📖 Why Do We Need User Namespaces?

Suppose a container runs:

```bash
whoami
```

Output:

```
root
```

Without User Namespaces,

that process would truly be:

```
Host Root
```

A compromised container could potentially control the host.

That would be a disaster.

---

# Linux Users

Every Linux user has:

```
UID

User ID
```

Examples:

```
root

↓

UID 0
```

```
vinay

↓

UID 1000
```

Linux uses UIDs internally,

not usernames.

---

# Root User

Linux reserves:

```
UID 0
```

for:

```
root
```

UID 0 has broad privileges.

Without isolation,

containers running as root would also be host root.

---

# User Namespace Solution

Linux creates:

```
Container UID

↓

Mapped

↓

Host UID
```

The process sees one identity,

while the kernel enforces another.

---

# What is a User Namespace?

A User Namespace isolates:

✔ User IDs (UID)

✔ Group IDs (GID)

✔ Linux Capabilities

The same process can have different identities inside and outside the namespace.

---

# Visual Representation

```
Container

↓

UID 0

↓

User Namespace

↓

Host UID 100000
```

Inside:

```
root
```

Outside:

```
UID 100000
```

The host treats it as an unprivileged user.

---

# UID Mapping

Example:

| Inside Container | Host |
|------------------|------|
| 0 | 100000 |
| 1 | 100001 |
| 2 | 100002 |
| 1000 | 101000 |

The mapping is configurable.

---

# Same Process

Inside:

```bash
id
```

Output:

```
uid=0(root)
```

Host:

```bash
ps
```

The process belongs to:

```
UID 100000
```

One process.

Two identities.

---

# Why Mapping Matters

Suppose the container tries:

```bash
rm /host/file
```

The kernel checks:

```
Host UID

↓

100000
```

Not:

```
UID 0
```

Permission is denied unless the mapped host user has access.

---

# UID Translation

Application requests:

```
UID 0
```

Linux translates:

```
Container UID

↓

Host UID
```

All permission checks occur using the host UID.

---

# GID Mapping

Groups work exactly the same way.

Inside:

```
GID 0
```

Host:

```
GID 100000
```

Group permissions are translated.

---

# `/proc/self/uid_map`

Linux exposes mappings:

```bash
cat /proc/self/uid_map
```

Example:

```text
0 100000 65536
```

Meaning:

```
Container UID 0

↓

Host UID 100000

↓

Range 65536 IDs
```

---

# `/proc/self/gid_map`

Similarly:

```bash
cat /proc/self/gid_map
```

Shows group ID mappings.

---

# Linux Capabilities

Traditional root:

```
Everything
```

Modern Linux splits privileges into:

```
Capabilities
```

Examples:

- Mount filesystems
- Bind privileged ports
- Configure networking

User Namespaces interact with these capabilities.

---

# Why Capabilities Matter

Even if a process appears to be:

```
root
```

inside the container,

it may lack important capabilities on the host.

This greatly limits what it can do.

---

# Docker User Namespace Remapping

Docker supports:

```
userns-remap
```

Example configuration:

```
dockremap
```

Docker maps container users to unprivileged host users.

---

# Rootless Docker

Modern Docker supports:

```
Rootless Mode
```

Docker itself runs as a normal user.

Benefits:

✔ Smaller attack surface

✔ No privileged Docker daemon

✔ Improved host security

---

# Rootless Architecture

```
User

↓

Docker

↓

User Namespace

↓

Container
```

No host root privileges required for the daemon.

---

# Kubernetes Example

Kubernetes supports running containers as non-root users.

Combined with User Namespaces (where supported),

this provides even stronger isolation.

---

# Security Example

Without User Namespace:

```
Container Root

↓

Host Root
```

Dangerous.

With User Namespace:

```
Container Root

↓

Host UID 100000
```

Much safer.

---

# Why User Namespaces Matter

They significantly reduce the impact of container escapes and privilege escalation.

Large cloud providers rely heavily on user isolation mechanisms.

---

# Hands-on Lab

## Lab 1 — View Current UID

```bash
id
```

---

## Lab 2 — View UID Mapping

```bash
cat /proc/self/uid_map
```

---

## Lab 3 — View GID Mapping

```bash
cat /proc/self/gid_map
```

---

## Lab 4 — Run a Non-Root Container

```bash
docker run \
-u 1000 \
ubuntu id
```

Observe:

```
UID 1000
```

inside the container.

---

## Lab 5 — Inspect Docker Security Options

```bash
docker info
```

Observe whether user namespace remapping is enabled on your system.

---

# Interview Questions

## What is a User Namespace?

A User Namespace isolates user IDs, group IDs, and capabilities by mapping identities between the container and the host.

---

## Can a container run as root without being host root?

Yes.

User Namespaces allow container UID 0 to map to an unprivileged UID on the host.

---

## What is UID 0?

UID 0 is the Linux root user.

---

## What is the purpose of UID mapping?

UID mapping allows different user identities inside and outside the namespace while preserving isolation.

---

## What is Rootless Docker?

Rootless Docker runs Docker without requiring a privileged root daemon, reducing security risks.

---

## Why are User Namespaces important?

They reduce the impact of privilege escalation by ensuring container root is not automatically host root.

---

# Summary

User Namespaces isolate user identities between containers and the host.

```
Container

↓

UID 0

↓

User Namespace

↓

Host UID 100000

↓

Permission Checks
```

Key concepts:

- Linux identifies users by UID and GID.
- User Namespaces translate IDs between the container and the host.
- Container root does not have to be host root.
- Linux Capabilities further restrict privileged operations.
- Docker supports user namespace remapping and rootless mode.
- User Namespaces are a critical container security feature.

At this point, you've learned almost every major namespace used by Linux containers:

- ✅ Network Namespace
- ✅ PID Namespace
- ✅ Mount Namespace
- ✅ UTS Namespace
- ✅ IPC Namespace
- ✅ User Namespace

Together, these namespaces isolate **what a process can see**, while **cgroups** control **how much it can use**.

---

# Next Chapter

## Chapter 55 — Linux Capabilities: Why Root Is No Longer All-Powerful

We'll explore:

- Why Linux Capabilities were introduced
- Root Before Linux Capabilities
- Capability-Based Security
- Common Capabilities (`CAP_NET_ADMIN`, `CAP_SYS_ADMIN`, etc.)
- Dropping Capabilities
- Adding Capabilities
- Docker Default Capability Set
- Privileged Containers
- Security Best Practices
- Complete Capability Architecture

> **This chapter explains the modern Linux security model used by Docker and Kubernetes.**