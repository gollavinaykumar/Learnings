Most developers think:

```
root

=

Unlimited Power
```

That was mostly true in older Linux systems.

If a process became:

```
UID 0
```

it could do almost anything.

Examples:

- Mount filesystems
- Configure networking
- Load kernel modules
- Change system time
- Read almost any file
- Kill almost any process

This created a huge security problem.

If an attacker obtained **root**, they obtained **everything**.

Modern Linux solved this by introducing:

# Linux Capabilities

Instead of giving one user **all powers**, Linux broke root privileges into **small independent permissions**.

Docker and Kubernetes rely heavily on this feature.

After this chapter, you'll understand why **root inside a container is usually much less powerful than root on the host.**

---

# 👑 Linux Kernel & Networking Mastery

# Module 8 — Linux Security Internals

# Chapter 55 — Linux Capabilities: Why Root Is No Longer All-Powerful

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Linux Capabilities exist
> - Root Before Linux Capabilities
> - Capability-Based Security
> - Capability Sets
> - Common Linux Capabilities
> - Docker Default Capabilities
> - Adding Capabilities
> - Dropping Capabilities
> - Privileged Containers
> - Security Best Practices

---

# 📖 Why Were Linux Capabilities Introduced?

Imagine a web server.

```
Nginx

↓

Needs

↓

Port 80
```

Should it also be allowed to:

```
Load Kernel Modules

↓

Format Disks

↓

Shutdown Linux
```

Of course not.

It only needs:

```
Bind Port 80
```

Linux Capabilities solve this problem.

---

# Old Linux Security Model

Before capabilities:

```
Normal User

↓

Very Limited
```

```
Root User

↓

Everything
```

There was no middle ground.

---

# Modern Linux Model

Today:

```
Privileges

↓

Split

↓

Small Capabilities
```

A process receives only the capabilities it actually needs.

---

# What is a Linux Capability?

A capability is a **specific kernel permission**.

Examples:

```
Can Mount Filesystems
```

```
Can Configure Network
```

```
Can Change System Time
```

Instead of one huge "root" privilege,

Linux manages many smaller privileges.

---

# Visual Representation

Old Model:

```
root

↓

ALL POWERS
```

Modern Model:

```
Process

↓

CAP_NET_BIND_SERVICE

CAP_NET_ADMIN

CAP_SYS_TIME

CAP_SYS_ADMIN

...

Only Required Ones
```

---

# Capability Sets

Linux maintains several capability sets for each process.

The most commonly discussed are:

- Permitted
- Effective
- Inheritable
- Bounding
- Ambient

These determine which capabilities a process may use.

For now,

remember:

```
Process

↓

Capability Set

↓

Kernel Permission Check
```

---

# Example

Suppose:

```
Application

↓

Bind Port 80
```

Kernel checks:

```
CAP_NET_BIND_SERVICE
```

If present:

```
Allowed
```

Otherwise:

```
Permission Denied
```

---

# Common Linux Capabilities

Some important capabilities:

| Capability | Purpose |
|------------|---------|
| `CAP_NET_BIND_SERVICE` | Bind to ports below 1024 |
| `CAP_NET_ADMIN` | Configure networking |
| `CAP_SYS_ADMIN` | Many powerful administrative operations |
| `CAP_SYS_TIME` | Change system clock |
| `CAP_KILL` | Send signals to other processes (subject to kernel rules) |
| `CAP_CHOWN` | Change file ownership |
| `CAP_SETUID` | Change user IDs |
| `CAP_SETGID` | Change group IDs |

Linux defines many more capabilities.

---

# Most Powerful Capability

One capability deserves special attention:

```
CAP_SYS_ADMIN
```

It controls a very large number of privileged kernel operations.

Many security engineers refer to it as:

```
"The New Root"
```

Avoid granting it unless absolutely necessary.

---

# Docker Default Capability Set

Docker does **not** give containers every Linux capability.

Instead,

it grants a limited default set.

Many powerful capabilities are removed by default.

Examples often **not** granted by default include:

- `CAP_SYS_ADMIN`
- `CAP_SYS_MODULE`
- `CAP_SYS_TIME`

This significantly improves security.

---

# Example

Container:

```
UID 0
```

But:

```
No CAP_SYS_TIME
```

Application tries:

```
Change System Clock
```

Kernel:

```
Denied
```

Even though the process is root inside the container.

---

# Viewing Capabilities

Install capability tools if necessary.

Example:

```bash
capsh --print
```

Observe:

- Effective
- Permitted
- Bounding

Capability sets.

---

# Dropping Capabilities

Docker allows removing capabilities.

Example:

```bash
docker run \
--cap-drop=NET_RAW \
ubuntu
```

Now the container cannot use operations requiring `CAP_NET_RAW`.

Reducing capabilities lowers risk.

---

# Dropping All

Maximum restriction:

```bash
docker run \
--cap-drop=ALL \
ubuntu
```

Then selectively add only what is required.

This follows the:

```
Least Privilege Principle
```

---

# Adding Capabilities

Suppose an application needs:

```
Network Administration
```

Run:

```bash
docker run \
--cap-add=NET_ADMIN \
ubuntu
```

Now Linux grants:

```
CAP_NET_ADMIN
```

to the container.

---

# Principle of Least Privilege

Never grant:

```
Everything
```

Instead:

```
Only What Is Needed
```

Example:

```
Web Server

↓

CAP_NET_BIND_SERVICE
```

Nothing more.

---

# Privileged Containers

Docker supports:

```bash
docker run \
--privileged
```

This is very different.

Docker grants:

Almost all capabilities,

plus access to many host devices and kernel features.

Effectively,

the container becomes much more powerful.

Use this only when absolutely necessary.

---

# Privileged vs Normal

Normal Container:

```
Limited Capabilities
```

Privileged Container:

```
Nearly Full Host Privileges
```

Security risk increases dramatically.

---

# Kubernetes Example

Kubernetes also controls capabilities.

Example:

```yaml
securityContext:
  capabilities:
    add:
      - NET_ADMIN
```

or

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
```

Fine-grained capability management is a Kubernetes security best practice.

---

# Real Example

Suppose a VPN application needs:

```
Modify Routing Table
```

It requires:

```
CAP_NET_ADMIN
```

Instead of making the entire container privileged,

grant only that capability.

---

# Capability Check Flow

```
Application

↓

Kernel Operation

↓

Capability Check

↓

Allowed

or

Denied
```

The kernel verifies capabilities before performing privileged actions.

---

# Why Capabilities Matter

Every modern container runtime uses them.

Examples:

- Docker
- Kubernetes
- containerd
- Podman
- CRI-O

Capabilities reduce the impact of compromised containers.

---

# Hands-on Lab

## Lab 1 — View Capabilities

```bash
capsh --print
```

---

## Lab 2 — Drop a Capability

```bash
docker run \
--cap-drop=NET_RAW \
-it ubuntu bash
```

Observe application behavior if it requires raw sockets.

---

## Lab 3 — Add a Capability

```bash
docker run \
--cap-add=NET_ADMIN \
-it ubuntu bash
```

---

## Lab 4 — Drop All

```bash
docker run \
--cap-drop=ALL \
-it ubuntu bash
```

Observe how restricted the container becomes.

---

## Lab 5 — Privileged Container

```bash
docker run \
--privileged \
-it ubuntu bash
```

Compare capability sets with a normal container.

---

# Interview Questions

## What are Linux Capabilities?

Linux Capabilities divide traditional root privileges into smaller, independent permissions.

---

## Why were Capabilities introduced?

To avoid granting unrestricted root privileges when only specific permissions are needed.

---

## What is `CAP_NET_ADMIN`?

It allows privileged network administration operations such as configuring network interfaces and routing.

---

## Why is `CAP_SYS_ADMIN` considered dangerous?

Because it controls many powerful administrative kernel operations and should be granted only when necessary.

---

## Does Docker grant every capability by default?

No.

Docker grants a restricted default capability set.

---

## What does `--privileged` do?

It gives the container significantly more privileges, including many capabilities and access to host resources.

---

## What security principle should capability management follow?

The Principle of Least Privilege—grant only the capabilities an application actually requires.

---

# Summary

Linux Capabilities replace the old all-or-nothing root security model.

```
Application

↓

Capability Set

↓

Kernel Check

↓

Allowed

or

Denied
```

Key concepts:

- Root privileges are divided into individual capabilities.
- Docker removes many powerful capabilities by default.
- Containers can add or drop capabilities as needed.
- `CAP_SYS_ADMIN` is especially sensitive.
- `--privileged` grants broad access and should be avoided unless required.
- Following the Principle of Least Privilege greatly improves container security.

At this point, you understand how modern Linux enforces fine-grained privileges instead of relying solely on the root user.

This is one of the core security mechanisms used by Docker, Kubernetes, and Linux itself.

---

# Next Chapter

## Chapter 56 — seccomp: How Linux Filters Dangerous System Calls

We'll explore:

- What is a System Call?
- Why seccomp Exists
- seccomp-bpf
- Allow Lists vs Deny Lists
- Docker Default seccomp Profile
- Blocking Dangerous System Calls
- Custom seccomp Profiles
- seccomp vs Capabilities
- Performance
- Complete System Call Filtering Architecture

> **This chapter introduces one of the strongest kernel-level security features protecting modern containers.**