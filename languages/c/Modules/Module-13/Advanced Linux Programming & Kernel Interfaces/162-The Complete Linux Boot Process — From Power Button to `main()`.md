# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 162 — The Complete Linux Boot Process — From Power Button to `main()`

---

# Imagine this situation.

You press:

```
Power Button
```

Five seconds later:

```
Linux Login Screen
```

Or perhaps:

```
SSH Server Ready
```

Or:

```
Docker Containers Running
```

Everything seems magical.

But inside those few seconds, **thousands of operations** occur.

The computer transforms from:

```
Powered Off

↓

Millions of Transistors

↓

Fully Functional Operating System
```

This chapter follows the complete Linux boot process.

---

# Learning Objectives

After this chapter you will understand:

- BIOS vs UEFI
- Firmware initialization
- Bootloader
- GRUB
- Linux kernel loading
- Kernel decompression
- initramfs
- Root filesystem mounting
- `systemd`
- Login services

---

# The Complete Journey

```
Power Button

↓

Firmware

↓

Bootloader

↓

Linux Kernel

↓

initramfs

↓

Root Filesystem

↓

systemd

↓

Services

↓

Login

↓

Shell

↓

Your Program
```

Let's study every stage.

---

# Step 1 — Power Button

You press:

```
ON
```

Electricity reaches:

- CPU
- RAM
- Motherboard
- Storage
- Devices

Initially:

```
RAM

↓

Random Contents

CPU

↓

Reset State
```

The CPU begins execution from a predefined reset vector determined by the hardware architecture.

---

# Step 2 — CPU Reset

CPU registers receive predefined reset values.

Program Counter points to firmware startup code.

CPU has:

```
No Linux

↓

No Processes

↓

No Memory Manager
```

Only firmware code exists.

---

# Step 3 — Firmware Starts

Modern computers use:

```
UEFI
```

Older systems often used:

```
BIOS
```

Firmware initializes the machine.

---

# BIOS vs UEFI

BIOS:

```
Legacy

↓

16-bit Origins

↓

Simple Boot Process
```

UEFI:

```
Modern

↓

64-bit Support

↓

Richer Services

↓

Secure Boot Support
```

Most modern systems use UEFI.

---

# Step 4 — POST

Firmware performs:

```
POST

↓

Power-On Self Test
```

Checks:

- CPU
- RAM
- Keyboard
- Display
- Storage
- Other hardware

If something fails:

```
Boot Stops
```

---

# Step 5 — Hardware Initialization

Firmware initializes devices:

```
CPU

↓

RAM

↓

PCIe

↓

USB

↓

NVMe

↓

GPU
```

The operating system has not started yet.

---

# Step 6 — Boot Device Selection

Firmware determines:

```
Which Device

↓

Should Boot?
```

Example:

```
NVMe SSD

↓

Linux Installed
```

---

# Step 7 — Bootloader

Firmware loads:

```
GRUB

or

Another Bootloader
```

Examples:

- GRUB
- systemd-boot
- U-Boot (embedded)
- Other architecture-specific bootloaders

---

# What is a Bootloader?

Definition:

> **A bootloader loads the operating system kernel into memory and transfers control to it.**

---

# Visualization

```
Firmware

↓

Bootloader

↓

Kernel
```

---

# Step 8 — GRUB Menu

GRUB may display:

```
Ubuntu

Fedora

Rescue Mode
```

User selects one.

Or GRUB boots automatically after a timeout.

---

# Step 9 — Load Linux Kernel

GRUB reads:

```
vmlinuz
```

into RAM.

This is the compressed Linux kernel image.

---

# Step 10 — Load initramfs

Bootloader also loads:

```
initramfs
```

Initial RAM filesystem.

It contains temporary tools needed before the real root filesystem becomes available.

---

# Visualization

```
Kernel

+

initramfs

↓

RAM
```

---

# Step 11 — Transfer Control

Bootloader jumps to:

```
Linux Kernel Entry Point
```

Bootloader's job is finished.

Linux begins executing.

---

# Step 12 — Kernel Decompression

The compressed kernel expands itself into memory.

Result:

```
Compressed Image

↓

Running Kernel
```

---

# Step 13 — Early Kernel Initialization

Kernel initializes:

- Memory management
- CPU structures
- Interrupt handling
- Scheduler foundations
- Early console
- Basic kernel subsystems

Many features are still unavailable.

---

# Step 14 — Detect CPUs

Kernel discovers:

```
CPU0

CPU1

CPU2

CPU3
```

Initializes SMP (Symmetric Multiprocessing) support where available.

---

# Step 15 — Memory Detection

Kernel discovers installed RAM.

Creates:

```
Physical Memory Map
```

Builds:

- Page allocator
- Buddy allocator
- Memory zones

---

# Step 16 — Driver Initialization

Kernel initializes built-in drivers.

Examples:

```
NVMe

↓

USB

↓

PCI

↓

Network
```

Modules may be loaded later.

---

# Step 17 — Mount initramfs

Temporary root filesystem:

```
initramfs

↓

Mounted
```

Kernel executes its initialization program.

---

# Why initramfs?

At this point,

Linux may not yet know how to access the real root filesystem.

initramfs provides:

- Storage drivers
- Filesystem modules
- Device discovery
- Early userspace utilities

---

# Step 18 — Find Root Filesystem

Kernel discovers:

```
Root Partition

↓

Filesystem

↓

Mount
```

Example:

```
/dev/nvme0n1p2

↓

ext4
```

---

# Step 19 — Switch Root

Temporary root:

```
initramfs
```

becomes:

```
Real Root Filesystem
```

The temporary environment is discarded.

---

# Visualization

```
initramfs

↓

switch_root

↓

/
```

---

# Step 20 — Start PID 1

Kernel launches:

```
PID 1
```

On most modern Linux systems:

```
systemd
```

If PID 1 exits unexpectedly,

the system cannot continue operating normally.

---

# Step 21 — `systemd`

`systemd` becomes the first user-space process.

Responsibilities include:

- Starting services
- Mounting filesystems
- Managing devices
- Logging
- Networking

---

# Step 22 — Service Startup

Examples:

```
udev

↓

NetworkManager

↓

sshd

↓

cron

↓

Docker
```

Each starts according to dependency rules.

---

# Step 23 — Login Services

Depending on system configuration:

```
Console Login

or

SSH Server

or

Display Manager
```

becomes available.

---

# Step 24 — User Login

User enters:

```
Username

↓

Password
```

Authentication succeeds.

---

# Step 25 — Shell Starts

Linux launches:

```
bash

or

zsh

or

fish
```

Now:

```
User Space

↓

Interactive Session
```

---

# Step 26 — Your Program

Finally:

```bash
./hello
```

Shell performs:

```
fork()

↓

execve()

↓

Your Program
```

Exactly as you learned in the previous chapter.

---

# Complete Boot Timeline

```
Power

↓

CPU Reset

↓

Firmware

↓

POST

↓

Bootloader

↓

Kernel

↓

Decompress

↓

Memory

↓

Drivers

↓

initramfs

↓

Root Filesystem

↓

systemd

↓

Services

↓

Login

↓

Shell

↓

fork()

↓

execve()

↓

main()
```

---

# Relationship with the Kernel

Bootloader loads:

```
Kernel
```

Kernel starts:

```
systemd
```

Kernel never directly starts your applications.

User-space initialization does.

---

# Relationship with Virtual Memory

Early boot:

```
Physical Memory
```

Later:

```
Virtual Memory

↓

Page Tables

↓

Processes
```

Virtual memory becomes fully operational during kernel initialization.

---

# Relationship with Device Drivers

Drivers become available gradually.

Without storage drivers,

the kernel cannot mount the root filesystem.

---

# Relationship with `fork()` and `execve()`

Once boot completes,

every new application follows:

```
fork()

↓

execve()
```

The boot process only happens once per system startup.

---

# Real-World Example — Ubuntu Server

```
Power

↓

UEFI

↓

GRUB

↓

Kernel

↓

systemd

↓

sshd

↓

SSH Login
```

---

# Real-World Example — Docker Host

After boot:

```
systemd

↓

Docker Daemon

↓

Containers
```

---

# Real-World Example — Kubernetes Node

```
Kernel

↓

systemd

↓

containerd

↓

kubelet

↓

Pods
```

---

# Real-World Example — Embedded Linux

```
Power

↓

U-Boot

↓

Kernel

↓

BusyBox

↓

Application
```

---

# Common Mistakes

---

## Thinking GRUB Is Linux

GRUB is a bootloader.

Linux starts only after GRUB transfers control.

---

## Thinking `systemd` Is the Kernel

`systemd` is the first user-space process (PID 1 on most modern systems).

---

## Ignoring initramfs

Many systems cannot mount the real root filesystem without it.

---

## Thinking Your Program Starts Directly After Boot

Many initialization steps occur before any user application executes.

---

# Hands-on Labs

## Lab 1

View kernel boot messages:

```bash
dmesg
```

---

## Lab 2

Inspect current bootloader configuration:

```bash
ls /boot
```

---

## Lab 3

Check PID 1:

```bash
ps -p 1
```

---

## Lab 4

Inspect mounted filesystems:

```bash
mount
```

---

## Lab 5

Draw the complete Linux boot sequence from memory.

---

# Interview Questions

### What executes immediately after the power button is pressed?

The CPU begins executing firmware startup code after reset.

---

### What is the role of the bootloader?

It loads the Linux kernel into memory and transfers control to it.

---

### Why is initramfs needed?

It provides a temporary userspace with the tools and drivers needed to locate and mount the real root filesystem.

---

### What is PID 1?

The first user-space process started by the kernel, typically `systemd` on modern Linux systems.

---

### When does your application start?

Only after the system has completed boot, started user-space services, and a shell (or another launcher) executes it using `fork()` and `execve()`.

---

# Summary

Complete Boot Process:

```
Power

↓

Firmware

↓

Bootloader

↓

Linux Kernel

↓

Kernel Initialization

↓

initramfs

↓

Root Filesystem

↓

systemd

↓

Services

↓

Login

↓

Shell

↓

fork()

↓

execve()

↓

main()
```

Major Components:

| Component | Purpose |
|-----------|---------|
| Firmware (BIOS/UEFI) | Initialize hardware |
| Bootloader | Load kernel |
| Linux Kernel | Initialize operating system |
| initramfs | Temporary root environment |
| Root Filesystem | Permanent system files |
| `systemd` | Start user-space services |
| Shell | Launch user programs |

## Key Takeaways

- Boot begins with firmware, not the Linux kernel.
- The bootloader loads the compressed kernel and initramfs into memory.
- The kernel initializes hardware, memory management, and core subsystems.
- initramfs prepares the system before the real root filesystem is mounted.
- `systemd` (or another init system) becomes PID 1 and starts user-space services.
- Every application you run after boot ultimately starts through `fork()` and `execve()`.

---

# Next Chapter

## Chapter 163 — Linux Kernel Architecture — Understanding the Entire Kernel as One System

You'll learn:

- The complete architecture of the Linux kernel
- Core kernel subsystems
- Scheduler
- Memory Manager
- VFS
- Networking Stack
- Block Layer
- Device Drivers
- Security Modules
- IPC
- How all kernel components work together as one integrated operating system