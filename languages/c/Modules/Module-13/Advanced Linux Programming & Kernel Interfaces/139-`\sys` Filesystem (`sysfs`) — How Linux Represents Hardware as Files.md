# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 139 — `/sys` Filesystem (`sysfs`) — How Linux Represents Hardware as Files

---

# Most Linux users know:

```bash
/sys
```

exists.

Very few know:

- Why does it exist?
- Why isn't `/proc` enough?
- Why do device drivers create files inside `/sys`?
- How does Linux represent hardware as directories?

Understanding `sysfs` is understanding **how Linux sees hardware**.

---

# Learning Objectives

After this chapter you will understand:

- What `sysfs` is
- Linux Device Model
- Kernel Objects (`kobjects`)
- `/sys/class`
- `/sys/block`
- `/sys/devices`
- `/sys/bus`
- Device Drivers
- Hotplug

---

# Before Linux 2.6

Hardware information was scattered.

Some information:

```
/proc
```

Some:

```
Kernel APIs
```

Some:

```
Device Drivers
```

There was no unified device hierarchy.

---

# Linux Solution

Linux introduced:

```
sysfs
```

to represent:

```
Hardware

↓

Kernel Objects

↓

Drivers

↓

Buses
```

using files and directories.

---

# Think About Device Manager

Windows has:

```
Device Manager
```

Linux has:

```
/sys
```

Everything is organized as directories.

---

# What is `sysfs`?

Definition:

> **`sysfs` is a virtual filesystem that exposes the Linux device model through files and directories.**

Filesystem type:

```
sysfs
```

---

# Visualization

```
Application

↓

read()

↓

VFS

↓

sysfs

↓

Kernel Objects
```

Again:

```
No Disk
```

Everything is generated dynamically.

---

# `/proc` vs `/sys`

Many beginners confuse them.

---

# `/proc`

Represents:

```
Processes

↓

Kernel Runtime Information
```

Examples:

```
CPU

↓

Memory

↓

Processes
```

---

# `/sys`

Represents:

```
Hardware

↓

Drivers

↓

Devices

↓

Kernel Objects
```

---

# Comparison

```
/proc

↓

Running System

-----------------------

/sys

↓

Hardware Model
```

---

# Linux Device Model

Linux internally represents every hardware component as an object.

Examples:

```
USB

↓

PCI

↓

SSD

↓

CPU

↓

GPU

↓

Network Card
```

Each becomes a:

```
Kernel Object
```

---

# What is a Kernel Object?

Called:

```
kobject
```

Definition:

> **A kernel object represents an entity inside the Linux kernel and forms the basis of the Linux device model.**

---

# Visualization

```
SSD

↓

kobject

↓

sysfs Directory
```

---

# Device Hierarchy

Example:

```
Computer

↓

PCI Bus

↓

NVMe Controller

↓

SSD
```

Linux preserves this hierarchy inside `/sys`.

---

# `/sys/devices`

Contains:

```
Actual Hardware Hierarchy
```

Example:

```
CPU

↓

PCI

↓

USB

↓

NVMe

↓

GPU
```

---

# Visualization

```
/sys/devices

↓

PCI

↓

USB

↓

Storage
```

---

# `/sys/class`

Groups devices by function,

not by physical location.

Example:

```
Network

↓

Block

↓

TTY

↓

Power
```

---

# Example

Network interface:

```
eth0
```

May appear:

```text
/sys/class/net/eth0
```

---

# `/sys/block`

Contains:

```
Block Devices
```

Examples:

```
sda

↓

nvme0n1

↓

loop0
```

Each device has its own directory.

---

# `/sys/bus`

Linux organizes hardware using buses.

Examples:

```
PCI

USB

I2C

SPI
```

Inside:

```text
/sys/bus
```

---

# Visualization

```
PCI Bus

↓

Device

↓

Driver
```

---

# Device Drivers

Suppose Linux detects:

```
Intel Network Card
```

Kernel loads:

```
Driver
```

Driver creates entries inside:

```
sysfs
```

---

# Device Attributes

Example:

```bash
cat /sys/class/net/eth0/mtu
```

Output:

```
1500
```

Not a real file.

Kernel generates the value.

---

# Another Example

Read:

```bash
cat /sys/class/net/eth0/address
```

Returns:

```
MAC Address
```

Again:

Generated dynamically.

---

# Changing Values

Some sysfs files are writable.

Example:

```bash
echo 9000 > mtu
```

Kernel receives:

```
write()

↓

Driver

↓

Hardware
```

Applications configure devices through files.

---

# Visualization

```
echo

↓

write()

↓

sysfs

↓

Driver

↓

Device
```

---

# CPU Information

Example:

```text
/sys/devices/system/cpu
```

Contains:

```
CPU0

↓

CPU1

↓

CPU2
```

Each directory exposes CPU attributes.

---

# Memory Devices

Example:

```text
/sys/devices/system/memory
```

Represents physical memory blocks.

---

# Power Management

Battery:

```text
/sys/class/power_supply
```

Read:

```
Capacity

↓

Charging

↓

Voltage
```

Many desktop environments use these values.

---

# LEDs

Linux even exposes LEDs.

Example:

```text
/sys/class/leds
```

Writing values may control hardware LEDs.

---

# Hotplug

Suppose:

```
Insert USB
```

Kernel detects:

```
New Device

↓

Create kobject

↓

Create sysfs Directory

↓

Notify Userspace
```

No reboot required.

---

# udev

Userspace daemon:

```
udev
```

Monitors:

```
sysfs

↓

Hotplug Events

↓

Create /dev Nodes
```

This is why device files appear automatically.

---

# Relationship

```
Hardware

↓

Kernel

↓

kobject

↓

sysfs

↓

udev

↓

/dev
```

Everything is connected.

---

# Real-World Example — Network Interface

Read:

```bash
cat /sys/class/net/wlan0/address
```

Kernel asks:

```
Network Driver

↓

MAC Address

↓

Return
```

---

# Real-World Example — NVMe SSD

Read:

```text
/sys/block/nvme0n1
```

Kernel exposes:

- Queue depth
- Size
- Scheduler
- Model

---

# Real-World Example — Docker

Containers rely on:

- cgroups
- namespaces

Both expose information through:

```
sysfs

↓

procfs
```

---

# Real-World Example — Kubernetes

Kubernetes inspects Linux system information through `/sys` and `/proc` when managing nodes and devices.

---

# Common Mistakes

---

## Thinking `/sys` Exists on Disk

It doesn't.

It's generated by the kernel.

---

## Confusing `/proc` and `/sys`

```
/proc

↓

Processes

--------------------

/sys

↓

Hardware
```

---

## Editing Random sysfs Files

Many writable entries immediately affect kernel behavior.

Changes should be made carefully.

---

## Thinking Drivers and `/dev` Are Independent

`sysfs`, `udev`, and device nodes work together.

---

# Hands-on Labs

## Lab 1

Run:

```bash
ls /sys
```

Explore the top-level directories.

---

## Lab 2

Inspect network interfaces:

```bash
ls /sys/class/net
```

---

## Lab 3

Read:

```bash
cat /sys/class/net/<interface>/address
```

Replace `<interface>` with an interface on your system (such as `eth0`, `enp0s3`, or `wlan0`).

---

## Lab 4

Explore:

```bash
ls /sys/block
```

Observe available block devices.

---

## Lab 5

Insert and remove a USB device.

Observe changes under:

```text
/sys/devices
```

---

# Interview Questions

### What is `sysfs`?

A virtual filesystem exposing the Linux device model through files and directories.

---

### What is a `kobject`?

A kernel object representing devices and other kernel entities within the Linux device model.

---

### What is stored in `/sys/class`?

Devices grouped by function, such as network interfaces or block devices.

---

### What is the difference between `/proc` and `/sys`?

`/proc` exposes process and kernel runtime information.

`/sys` exposes hardware, drivers, and kernel objects.

---

### How does Linux detect newly connected USB devices?

The kernel creates new device objects, exposes them through `sysfs`, and userspace tools like `udev` respond by creating device nodes.

---

# Summary

Architecture:

```
Hardware

↓

Kernel

↓

kobject

↓

sysfs

↓

Applications
```

Important Directories:

| Directory | Purpose |
|-----------|----------|
| `/sys/devices` | Physical device hierarchy |
| `/sys/class` | Devices grouped by function |
| `/sys/block` | Block devices |
| `/sys/bus` | Hardware buses |
| `/sys/devices/system` | CPUs, memory, and other system devices |

Relationship:

```
Hardware

↓

Driver

↓

kobject

↓

sysfs

↓

udev

↓

/dev
```

## Key Takeaways

- `sysfs` is a virtual filesystem exposing the Linux device model.
- Hardware is represented through kernel objects (`kobjects`).
- Applications inspect and sometimes configure hardware using ordinary file operations.
- `/sys/class` organizes devices by purpose, while `/sys/devices` reflects physical topology.
- `udev` uses `sysfs` events to create device files dynamically.
- `sysfs` is a central part of modern Linux hardware management.

---

# Next Chapter

## Chapter 140 — `ioctl()` — Talking Directly to Device Drivers

You'll learn:

- What `ioctl()` is
- Why `read()` and `write()` are sometimes not enough
- Device-specific commands
- Driver interfaces
- Network interfaces
- Terminal control
- Block devices
- Custom kernel drivers
- How user-space applications send specialized commands directly to Linux device drivers