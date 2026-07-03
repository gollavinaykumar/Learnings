Most developers press the Power button and, a few seconds later, see the login screen.

That's it.

But during those few seconds, **thousands of operations happen** before you can even type your password.

Questions you may have:

- How does the computer know where Linux or Windows is installed?
- Who starts the operating system?
- What is BIOS?
- What is UEFI?
- What is GRUB?
- When does the Linux kernel start?
- What is `systemd`?

After this chapter, you'll understand the complete boot process from pressing the Power button to the login screen.

---

# 👑 Linux Kernel & Networking Mastery

# Module 1 — Computer Fundamentals

# Chapter 6 — From Power Button to Login Screen: The Complete Boot Process

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What happens when the Power button is pressed
> - Power Supply (PSU)
> - CPU Reset
> - BIOS
> - UEFI
> - POST (Power-On Self-Test)
> - Boot Device Selection
> - Bootloader
> - GRUB
> - Linux Kernel Loading
> - initramfs
> - systemd / init
> - Login Manager
> - Complete Linux Boot Flow

---

# 📖 Why Do We Need a Boot Process?

Imagine buying a brand-new laptop.

It contains:

- CPU
- RAM
- SSD
- Motherboard

Everything is connected.

But nothing is running.

When electricity arrives, the CPU doesn't magically know:

- Where Linux is
- Where Windows is
- Which SSD to use
- Which program to execute first

Someone must guide the CPU.

That is the job of the boot process.

---

# Complete Boot Process

```
Power Button

↓

Power Supply

↓

CPU Reset

↓

BIOS / UEFI

↓

POST

↓

Find Boot Device

↓

Load Bootloader

↓

Load Linux Kernel

↓

Load initramfs

↓

Start systemd

↓

Start Services

↓

Display Login Screen
```

Every operating system follows a similar sequence.

---

# Step 1 — Power Button

You press:

```
Power
```

Electricity begins flowing through the computer.

Nothing else happens until stable power is available.

---

# Step 2 — Power Supply (PSU)

The Power Supply converts wall electricity into voltages required by the computer.

```
Wall Power

↓

Power Supply

↓

CPU

RAM

SSD

Motherboard
```

If voltage is unstable,

the CPU does not start.

---

# Step 3 — CPU Reset

As soon as power becomes stable,

the CPU resets itself.

```
CPU

↓

Reset Registers

↓

Reset Program Counter

↓

Known Starting Address
```

The CPU begins executing instructions from a predefined memory location.

At this point,

Linux is **not running yet**.

---

# Step 4 — BIOS / UEFI Starts

The CPU executes firmware stored on the motherboard.

Older computers:

```
BIOS
```

Modern computers:

```
UEFI
```

Firmware is permanently stored in flash memory.

---

# What is BIOS?

BIOS stands for:

```
Basic Input Output System
```

Responsibilities:

- Initialize hardware
- Test hardware
- Find bootable devices
- Load the bootloader

BIOS is older technology.

---

# What is UEFI?

UEFI stands for:

```
Unified Extensible Firmware Interface
```

It replaces BIOS.

Advantages:

✔ Faster boot

✔ Better security

✔ Large disk support

✔ Graphical interface

✔ Secure Boot

Almost all modern computers use UEFI.

---

# BIOS vs UEFI

| BIOS | UEFI |
|------|------|
| Older | Modern |
| Limited Features | Advanced Features |
| Slow | Faster |
| MBR | GPT Support |
| Text Mode | GUI Support |
| Limited Disk Size | Very Large Disk Support |

---

# Step 5 — POST

POST means:

```
Power-On Self-Test
```

The firmware checks whether hardware is working.

Checks include:

- CPU
- RAM
- Keyboard
- Graphics
- Storage
- Fans

If something fails,

the computer may beep or display an error.

---

# Step 6 — Detect Hardware

Firmware discovers connected devices.

Example:

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

Graphics
```

Now the firmware knows what hardware exists.

---

# Step 7 — Find Boot Device

The firmware searches for a bootable device.

Possible devices:

```
SSD

Hard Disk

USB

DVD

Network (PXE)
```

The boot order is configurable.

Example:

```
1. SSD

2. USB

3. Network
```

---

# Step 8 — Load the Bootloader

The firmware loads a small program called the bootloader.

Examples:

```
GRUB

Windows Boot Manager
```

The bootloader's only job is to start the operating system.

---

# What is GRUB?

GRUB stands for:

```
GRand Unified Bootloader
```

Responsibilities:

- Display boot menu
- Choose operating system
- Load Linux kernel
- Pass boot parameters

Example:

```
Ubuntu

Fedora

Advanced Options
```

---

# Why Do We Need a Bootloader?

The Linux kernel is a very large program.

Firmware cannot load it directly.

Instead:

```
Firmware

↓

Bootloader

↓

Linux Kernel
```

The bootloader understands where the kernel is stored.

---

# Step 9 — Load Linux Kernel

GRUB loads two important files.

```
Linux Kernel

+

initramfs
```

Both are copied into RAM.

Control is transferred to the kernel.

At this moment:

```
Linux Starts Running
```

---

# What is the Linux Kernel?

The Linux kernel is the heart of the operating system.

Responsibilities:

- Process Management
- Memory Management
- Device Drivers
- Networking
- File Systems
- Security
- Scheduling

Without the kernel,

Linux cannot function.

---

# Step 10 — initramfs

Before the real file system is available,

Linux uses a temporary file system called:

```
initramfs
```

It contains:

- Essential drivers
- Storage drivers
- Filesystem drivers

Purpose:

Help the kernel locate the real root filesystem.

---

# Step 11 — Mount Root File System

After loading the required drivers,

the kernel mounts:

```
/
```

This is the root filesystem.

Now Linux can access:

```
/bin

/etc

/usr

/home

/var
```

The operating system becomes fully accessible.

---

# Step 12 — Start init / systemd

The kernel starts the first userspace process.

Modern Linux:

```
systemd
```

Older Linux:

```
init
```

This is the first process.

It always receives:

```
PID = 1
```

We'll study PID in the Operating Systems module.

---

# What Does systemd Do?

systemd starts the entire operating system.

Examples:

```
Network

↓

SSH

↓

Docker

↓

Database

↓

Display Manager
```

Almost every service starts because of systemd.

---

# Step 13 — Start Services

systemd starts required services.

Example:

```
Networking

↓

Bluetooth

↓

Audio

↓

Printing

↓

Docker

↓

Database
```

Services start in dependency order.

---

# Step 14 — Login Screen

Finally,

the display manager starts.

Examples:

```
GNOME

KDE

LightDM
```

Now you see:

```
Username

Password
```

The computer is ready.

---

# Complete Linux Boot Flow

```
Power Button

↓

Power Supply

↓

CPU Reset

↓

UEFI

↓

POST

↓

Find SSD

↓

GRUB

↓

Linux Kernel

↓

initramfs

↓

Mount Root Filesystem

↓

systemd (PID 1)

↓

Start Services

↓

Login Screen
```

This sequence happens every time your computer starts.

---

# Real Example — Booting Ubuntu

```
Power On

↓

UEFI

↓

POST

↓

GRUB

↓

Linux Kernel

↓

Detect CPU

↓

Detect RAM

↓

Load Drivers

↓

Mount Root Filesystem

↓

systemd

↓

Network Starts

↓

Docker Starts

↓

Desktop Starts

↓

Login Screen
```

---

# Why This Matters

Every Linux server follows this process.

Whether it's:

- AWS EC2
- Docker Host
- Kubernetes Node
- Raspberry Pi
- Personal Laptop

The boot process is fundamentally the same.

---

# Hands-on Lab

## Lab 1 — Check Firmware Type

Linux:

```bash
ls /sys/firmware
```

If you see:

```
efi
```

Your system uses UEFI.

---

## Lab 2 — View Kernel Version

Linux/macOS:

```bash
uname -r
```

---

## Lab 3 — View Running Kernel

```bash
uname -a
```

---

## Lab 4 — View Boot Logs

Linux:

```bash
dmesg
```

Observe messages printed by the kernel during boot.

---

## Lab 5 — View systemd Process

Linux:

```bash
ps -p 1
```

Output typically shows:

```
systemd
```

This confirms that PID 1 is systemd.

---

# Interview Questions

## What is the difference between BIOS and UEFI?

BIOS is the older firmware standard.

UEFI is the modern replacement with better performance, security, and support for larger disks.

---

## What is POST?

POST (Power-On Self-Test) verifies that essential hardware is functioning correctly before booting the operating system.

---

## What is GRUB?

GRUB is a bootloader that loads the Linux kernel into memory and transfers control to it.

---

## Why is initramfs needed?

It provides temporary drivers and tools required for the kernel to locate and mount the real root filesystem.

---

## What is systemd?

systemd is the first userspace process (PID 1) that starts and manages services after the Linux kernel finishes initialization.

---

# Summary

The boot process transforms a powered-off machine into a fully operational operating system.

```
Power

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

Login Screen
```

Key concepts:

- The CPU starts by executing firmware instructions.
- UEFI/BIOS initializes hardware.
- POST verifies hardware health.
- The bootloader loads the Linux kernel.
- initramfs prepares the system before the real filesystem is mounted.
- systemd (PID 1) starts all essential services.
- The login screen appears only after the operating system is fully initialized.

Understanding the boot process is the foundation for learning Linux internals, kernel architecture, system initialization, and troubleshooting startup problems.

---

# Next Chapter

## Module 2 — Operating Systems

### Chapter 7 — What is an Operating System? Kernel, User Space & System Calls

We'll explore:

- Why operating systems exist
- Kernel vs User Space
- Privileged mode vs User mode
- System Calls
- Process abstraction
- Resource management
- Why every application depends on the operating system