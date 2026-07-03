Before learning seccomp, you need to understand one important fact.

Every application eventually asks the Linux kernel to do something.

Examples:

Open a file:

```c
open("data.txt")
```

Allocate memory:

```c
malloc()
```

Create a process:

```c
fork()
```

Read a file:

```c
read()
```

Send network data:

```c
send()
```

These requests eventually become:

# System Calls (syscalls)

A syscall is the **only legal way** for a userspace program to request services from the Linux kernel.

Now imagine an attacker compromises a container.

If they can execute **every syscall** available in Linux,

they might:

- Load kernel modules
- Mount filesystems
- Debug other processes
- Reboot the machine
- Abuse kernel features

Linux solves this using:

# seccomp

seccomp allows Linux to **filter which system calls a process is allowed to execute**.

After this chapter, you'll understand one of the strongest security features used by Docker, Kubernetes, Chrome, Android, and many other modern systems.

---

# 👑 Linux Kernel & Networking Mastery

# Module 8 — Linux Security Internals

# Chapter 56 — seccomp: How Linux Filters Dangerous System Calls

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a System Call?
> - Why seccomp exists
> - seccomp vs seccomp-BPF
> - Allow Lists
> - Docker Default seccomp Profile
> - Blocking Dangerous System Calls
> - Custom seccomp Profiles
> - seccomp vs Linux Capabilities
> - Performance
> - Real-world Usage

---

# 📖 What is a System Call?

Applications run in:

```
User Space
```

The Linux kernel runs in:

```
Kernel Space
```

Applications cannot directly manipulate kernel resources.

Instead,

they request services through:

```
System Calls
```

---

# User Space vs Kernel Space

```
Application

↓

System Call

↓

Linux Kernel

↓

Hardware
```

Every privileged operation follows this path.

---

# Examples of System Calls

Examples include:

```
open()

read()

write()

close()

fork()

execve()

mmap()

socket()

connect()

mount()

clone()
```

Linux provides hundreds of system calls.

---

# Why Is This a Problem?

Suppose an attacker gains code execution inside a container.

Without restrictions,

the process could invoke any syscall supported by the kernel.

Some syscalls are harmless.

Others are extremely powerful.

---

# Example

Suppose an application tries:

```
mount()
```

Should a web server be allowed to mount filesystems?

Usually:

```
No
```

It doesn't need that capability.

---

# What is seccomp?

seccomp stands for:

```
Secure Computing
```

It allows the kernel to decide:

```
System Call

↓

Allowed

or

Blocked
```

before executing it.

---

# Visual Representation

```
Application

↓

System Call

↓

seccomp Filter

↓

Linux Kernel

↓

Hardware
```

Every syscall passes through the filter.

---

# seccomp-BPF

Modern Linux uses:

```
seccomp + BPF
```

(Berkeley Packet Filter technology adapted for syscall filtering.)

A small filter program evaluates each syscall.

Result:

```
Allow

or

Deny
```

This is commonly called:

```
seccomp-BPF
```

---

# Allow List

The safest approach:

```
Allowed Syscalls

↓

Everything Else

↓

Blocked
```

This is called an **allow list**.

Docker's default profile primarily follows this model.

---

# Deny List

Alternative:

```
Everything Allowed

↓

Except Some
```

Less secure.

Modern container platforms generally prefer allow-list based policies.

---

# Docker Default seccomp Profile

By default,

Docker enables a seccomp profile.

It allows common application syscalls while blocking many high-risk ones.

Most applications run without modification.

---

# Example

Application:

```
open()
```

Filter:

```
Allowed
```

Kernel executes:

```
open()
```

Application continues normally.

---

# Blocked Example

Application:

```
keyctl()
```

Docker's default profile typically blocks this syscall.

Result:

```
Permission Error
```

The kernel refuses the request.

---

# Another Example

Application:

```
reboot()
```

Filter:

```
Blocked
```

Container cannot reboot the host.

---

# Syscall Flow

```
Application

↓

Syscall

↓

seccomp

↓

Allow?

↓

Yes

↓

Kernel

↓

Hardware
```

If denied:

```
Application

↓

Error
```

---

# Docker Command

Disable seccomp:

```bash
docker run \
--security-opt seccomp=unconfined \
ubuntu
```

Now syscall filtering is removed.

This is **not recommended** unless absolutely necessary.

---

# Custom seccomp Profile

Docker supports:

```bash
docker run \
--security-opt seccomp=profile.json
```

Organizations often create custom profiles tailored to their applications.

---

# seccomp vs Capabilities

Capabilities answer:

```
Should this privileged operation be allowed?
```

seccomp answers:

```
Can this syscall even be executed?
```

Both work together.

---

# Example

Suppose a process has:

```
CAP_SYS_ADMIN
```

but seccomp blocks:

```
mount()
```

Result:

```
Denied
```

Both security mechanisms must permit the action.

---

# Performance

seccomp filters are evaluated inside the kernel.

They are designed to be efficient,

and for most workloads the performance impact is minimal.

---

# Real-World Usage

Many major systems use seccomp:

- Docker
- Kubernetes
- Chrome
- Android
- containerd
- Podman

It is a widely adopted kernel security mechanism.

---

# Why seccomp Matters

Suppose a container is compromised.

Even if the attacker controls the application,

they cannot invoke blocked syscalls.

This significantly reduces the attack surface.

---

# Complete Architecture

```
Application

↓

System Call

↓

seccomp Filter

↓

Capability Check

↓

Linux Kernel

↓

Hardware
```

Multiple kernel security layers work together.

---

# Hands-on Lab

## Lab 1 — View seccomp Status

Run:

```bash
docker info
```

Observe security-related configuration.

---

## Lab 2 — Run Default Container

```bash
docker run -it ubuntu bash
```

Most applications work normally under Docker's default seccomp profile.

---

## Lab 3 — Run Without seccomp

```bash
docker run \
--security-opt seccomp=unconfined \
-it ubuntu bash
```

Compare behavior only if you understand the security implications.

---

## Lab 4 — Inspect Docker Profile

Docker ships with a default seccomp profile.

Reviewing it helps understand which syscalls are permitted or denied.

---

# Interview Questions

## What is a System Call?

A System Call is the interface through which a userspace application requests services from the Linux kernel.

---

## What is seccomp?

seccomp is a Linux kernel feature that filters system calls, allowing or blocking them before execution.

---

## Why does Docker enable seccomp by default?

To reduce the attack surface by preventing containers from using unnecessary or dangerous system calls.

---

## What is seccomp-BPF?

It combines seccomp with Berkeley Packet Filter (BPF) technology to efficiently evaluate syscall filtering rules inside the kernel.

---

## What is the difference between seccomp and Linux Capabilities?

Capabilities control permissions for privileged operations.

seccomp controls whether particular system calls may be executed at all.

---

## Should `seccomp=unconfined` be used in production?

Generally no.

Disabling syscall filtering reduces container security and should be reserved for exceptional cases.

---

# Summary

seccomp protects Linux by filtering system calls before they reach the kernel.

```
Application

↓

System Call

↓

seccomp Filter

↓

Capability Check

↓

Linux Kernel

↓

Hardware
```

Key concepts:

- Applications interact with the kernel through system calls.
- seccomp filters those system calls.
- Docker enables a default seccomp profile automatically.
- seccomp primarily uses an allow-list approach.
- seccomp and Linux Capabilities complement each other.
- Disabling seccomp significantly weakens container security.

At this point, you now understand three major Linux container security layers:

- **User Namespaces** → User Identity Isolation
- **Linux Capabilities** → Fine-Grained Privileges
- **seccomp** → System Call Filtering

These work together to make containers significantly more secure than simply running applications as root.

---

# Next Chapter

## Chapter 57 — Linux Security Modules (LSM): AppArmor, SELinux, and Mandatory Access Control

We'll explore:

- What are Linux Security Modules?
- Discretionary Access Control (DAC) vs Mandatory Access Control (MAC)
- AppArmor
- SELinux
- Security Labels
- Policy Enforcement
- Docker AppArmor Profiles
- Kubernetes Security Contexts
- How LSM Complements Namespaces, cgroups, Capabilities, and seccomp
- Complete Linux Security Stack

> **This chapter completes the core Linux kernel security architecture used by modern container platforms.**