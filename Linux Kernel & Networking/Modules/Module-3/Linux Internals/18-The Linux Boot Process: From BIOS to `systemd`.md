Most Linux users know that the computer boots like this:

```
Power Button

↓

Linux Login Screen
```

But inside Linux, dozens of critical steps happen before you ever see the terminal.

Questions you may have:

- How does GRUB find the Linux kernel?
- What is `vmlinuz`?
- What is `initramfs`?
- How does Linux discover hardware?
- When are device drivers loaded?
- How is the root filesystem mounted?
- Why is `systemd` always PID 1?

Understanding the Linux boot process is essential because **every Linux server, Docker host, Kubernetes node, and cloud VM goes through these steps.**

After this chapter, you'll understand exactly what happens from the moment GRUB starts until Linux is fully operational.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 18 — The Linux Boot Process: From GRUB to `systemd`

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Complete Linux Boot Flow
> - GRUB Internals
> - Linux Kernel Image (`vmlinuz`)
> - `initramfs`
> - Kernel Initialization
> - Device Detection
> - Driver Loading
> - Root Filesystem Mounting
> - `systemd`
> - PID 1
> - Boot Targets
> - Boot Logs
> - Boot Troubleshooting

---

# 📖 Recap

In Module 1 we learned:

```
Power Button

↓

BIOS / UEFI

↓

GRUB

↓

Linux Kernel
```

Now we'll continue **inside Linux**.

Everything from this point onward is controlled by the Linux kernel.

---

# Complete Linux Boot Flow

```
Power Button

↓

BIOS / UEFI

↓

GRUB

↓

vmlinuz

↓

initramfs

↓

Kernel Initialization

↓

Detect Hardware

↓

Load Drivers

↓

Mount Root Filesystem

↓

Start systemd (PID 1)

↓

Start Services

↓

Login Screen
```

Every Linux machine follows this sequence.

---

# Step 1 — GRUB Starts

GRUB's job is very small.

It simply loads:

```
Linux Kernel

+

initramfs
```

into RAM.

Example:

```
/boot

├── vmlinuz

├── initramfs.img

└── grub
```

---

# What is `vmlinuz`?

`vmlinuz` is the compressed Linux kernel image.

```
vmlinuz

↓

Compressed Kernel
```

When GRUB loads it,

the kernel decompresses itself into memory.

---

# Kernel Decompression

```
GRUB

↓

Load vmlinuz

↓

RAM

↓

Decompress Kernel

↓

Kernel Starts
```

Only after decompression does Linux begin executing.

---

# Step 2 — Kernel Initialization

The kernel is now running.

The very first tasks include:

✔ Initialize CPU

✔ Initialize Memory Manager

✔ Initialize Scheduler

✔ Initialize Interrupts

✔ Initialize Kernel Data Structures

At this point,

no user programs exist yet.

---

# Kernel Initialization Flow

```
Kernel Starts

↓

CPU Setup

↓

Memory Manager

↓

Interrupt Handlers

↓

Scheduler

↓

Device Framework

↓

Continue Boot
```

The kernel prepares itself before interacting with hardware.

---

# Step 3 — Detect Hardware

Linux scans available hardware.

Examples:

```
CPU

↓

RAM

↓

SSD

↓

USB

↓

Network Card

↓

Graphics Card
```

This process is often called:

```
Hardware Enumeration
```

---

# Device Drivers

Linux now loads drivers.

Examples:

```
NVMe Driver

↓

USB Driver

↓

Ethernet Driver

↓

Wi-Fi Driver

↓

Filesystem Driver
```

Without drivers,

Linux cannot communicate with hardware.

---

# Kernel Modules

Some drivers are built into the kernel.

Others are loaded dynamically.

```
Kernel

↓

Load Module

↓

Driver Ready
```

Examples:

```bash
lsmod
```

shows loaded kernel modules.

---

# Step 4 — initramfs Starts

The kernel cannot immediately mount the real filesystem.

Why?

Because it may not yet know how to access:

```
SSD

RAID

LVM

Encrypted Disk
```

Instead,

Linux starts a temporary filesystem.

```
initramfs
```

---

# What is `initramfs`?

`initramfs` stands for:

```
Initial RAM Filesystem
```

It is a tiny temporary filesystem stored entirely in RAM.

Contents:

```
BusyBox

↓

Drivers

↓

Mount Scripts

↓

Filesystem Tools
```

---

# Why Do We Need initramfs?

Imagine:

```
Root Filesystem

↓

Stored on NVMe SSD
```

But:

```
NVMe Driver

↓

Not Loaded Yet
```

Linux cannot read the SSD.

`initramfs` solves this problem.

---

# initramfs Boot Flow

```
Kernel

↓

initramfs

↓

Load Required Drivers

↓

Locate Root Filesystem

↓

Mount Root Filesystem
```

After its job is complete,

`initramfs` disappears.

---

# Step 5 — Mount Root Filesystem

Linux now mounts:

```
/
```

Everything becomes available.

```
/

├── bin

├── etc

├── home

├── usr

├── var

└── proc
```

The temporary filesystem is replaced by the real one.

---

# switch_root

Linux performs:

```
initramfs

↓

switch_root

↓

Real Root Filesystem
```

From this point,

the real operating system is running.

---

# Step 6 — Start PID 1

The kernel now starts the first userspace process.

```
systemd
```

This process always receives:

```
PID = 1
```

Every other userspace process ultimately descends from it.

---

# Why PID 1 Is Special

PID 1 has unique responsibilities.

```
systemd

↓

Start Services

↓

Manage Services

↓

Restart Services

↓

Adopt Orphan Processes
```

If PID 1 exits,

the system cannot continue normally.

---

# Step 7 — systemd Starts Services

Examples:

```
Networking

↓

SSH

↓

Docker

↓

cron

↓

Bluetooth

↓

Database

↓

Display Manager
```

Services start according to dependencies.

---

# systemd Units

Everything in `systemd` is managed as a **Unit**.

Examples:

```
nginx.service

docker.service

postgresql.service

ssh.service
```

There are also:

```
.mount

.socket

.target

.timer
```

We'll study these later.

---

# Boot Targets

Instead of traditional runlevels,

modern Linux uses:

```
Targets
```

Examples:

```
multi-user.target

↓

Command Line
```

```
graphical.target

↓

Desktop Environment
```

---

# Service Dependency Graph

systemd does **not** simply start services one by one.

Instead,

it builds a dependency graph.

Example:

```
Network Ready

↓

Docker Starts

↓

Containers Start
```

Independent services may start in parallel.

---

# Boot Logs

Linux records everything during boot.

View:

```bash
dmesg
```

Example messages:

```
CPU Detected

Memory Initialized

USB Loaded

Ethernet Driver Loaded
```

---

# systemd Journal

Modern Linux stores logs using:

```
journalctl
```

View boot logs:

```bash
journalctl -b
```

Previous boot:

```bash
journalctl -b -1
```

---

# Real Example

Suppose an Ubuntu server boots.

```
UEFI

↓

GRUB

↓

vmlinuz

↓

Kernel

↓

initramfs

↓

Detect NVMe

↓

Mount /

↓

systemd

↓

Networking

↓

Docker

↓

SSH

↓

Ready
```

Only now can you SSH into the server.

---

# Why This Matters

Every technology you'll study later depends on this sequence.

Examples:

Docker

```
Docker Service

↓

Started by systemd
```

Kubernetes

```
kubelet

↓

Started by systemd
```

Nginx

```
nginx.service

↓

Started by systemd
```

---

# Hands-on Lab

## Lab 1 — View Kernel Version

```bash
uname -r
```

---

## Lab 2 — View Loaded Kernel Modules

```bash
lsmod
```

---

## Lab 3 — View Boot Messages

```bash
dmesg | less
```

---

## Lab 4 — View Boot Logs

```bash
journalctl -b
```

---

## Lab 5 — View PID 1

```bash
ps -p 1
```

Typical output:

```
systemd
```

---

## Lab 6 — View Running Services

```bash
systemctl list-units --type=service
```

Observe all active services.

---

# Interview Questions

## What is `vmlinuz`?

`vmlinuz` is the compressed Linux kernel image loaded into memory by GRUB.

---

## Why is `initramfs` needed?

`initramfs` provides a temporary root filesystem with the drivers and tools required to locate and mount the real root filesystem.

---

## What is the purpose of `systemd`?

`systemd` is the first userspace process (PID 1) responsible for starting and managing system services.

---

## Why is PID 1 special?

PID 1 initializes userspace, manages services, and adopts orphaned processes.

---

## What is the difference between `dmesg` and `journalctl`?

`dmesg` displays kernel ring buffer messages, while `journalctl` shows logs collected by `systemd-journald`, including boot and service logs.

---

## Why does Linux use Kernel Modules?

Kernel Modules allow drivers and other kernel functionality to be loaded dynamically without rebuilding the entire kernel.

---

# Summary

The Linux boot process transforms a compressed kernel image into a fully operational operating system.

```
GRUB

↓

vmlinuz

↓

Kernel Initialization

↓

initramfs

↓

Mount Root Filesystem

↓

systemd (PID 1)

↓

Start Services

↓

Linux Ready
```

Key concepts:

- GRUB loads the Linux kernel and `initramfs`.
- `vmlinuz` is the compressed kernel image.
- The kernel initializes CPU, memory, scheduling, and interrupts.
- `initramfs` loads the drivers needed to mount the real root filesystem.
- `systemd` (PID 1) starts and manages all userspace services.
- Boot logs can be inspected with `dmesg` and `journalctl`.

Understanding the Linux boot process provides the foundation for debugging boot failures, managing services, and understanding how Linux servers, Docker hosts, and Kubernetes nodes become operational.

---

# Next Chapter

## Chapter 19 — Linux Processes in Depth: `fork()`, `exec()`, `clone()` & the `/proc` Filesystem

We'll explore:

- Linux Process Model
- `fork()`
- `vfork()`
- `exec()`
- `clone()`
- Process Creation Internals
- Process Descriptors (`task_struct`)
- `/proc` Filesystem
- Process States in Linux
- Context Switching in Linux
- How Docker Creates Container Processes