Most developers think:

> "The Linux kernel is one huge program."

That's only partially true.

Imagine buying a new Wi-Fi adapter.

Do you need to:

- Download Linux source code?
- Recompile the kernel?
- Reboot into a new kernel?

Usually, **no**.

Instead, Linux simply loads a **Kernel Module**.

Questions you may have:

- What is a Kernel Module?
- Why doesn't Linux put every driver inside the kernel?
- How does `modprobe` work?
- Why does `lsmod` show hundreds of modules?
- How does Docker depend on kernel modules?
- Can we write our own kernel module?

The answer is:

**Loadable Kernel Modules (LKMs).**

Kernel Modules allow Linux to extend the running kernel without rebuilding or rebooting.

After this chapter, you'll understand how Linux dynamically extends its kernel and why this feature is one of Linux's greatest strengths.

---

# 👑 Linux Kernel & Networking Mastery

# Module 3 — Linux Internals

# Chapter 23 — Linux Kernel Modules: Extending the Kernel Without Rebooting

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What are Kernel Modules?
> - Why Kernel Modules exist
> - Modular Monolithic Kernel
> - Loadable Kernel Modules (LKM)
> - Built-in vs Loadable Modules
> - Module Lifecycle
> - `insmod`
> - `rmmod`
> - `modprobe`
> - Module Dependencies
> - Kernel Symbols
> - Device Drivers
> - Docker-related Kernel Modules
> - Writing Your First Kernel Module

---

# 📖 Why Do We Need Kernel Modules?

Imagine Linux supported every device directly inside the kernel.

The kernel would permanently contain:

- Every Wi-Fi Driver
- Every USB Driver
- Every Graphics Driver
- Every Filesystem Driver
- Every Network Driver

Even if your computer never used them.

```
Huge Kernel

↓

Large Memory Usage

↓

Long Boot Time
```

Not efficient.

---

# The Better Solution

Linux loads only what is needed.

```
Minimal Kernel

↓

Load Module

↓

Use Module
```

If you connect new hardware,

Linux loads the corresponding module.

---

# What is a Kernel Module?

A Kernel Module is a piece of kernel code that can be loaded and unloaded while the system is running.

```
Kernel

+

Module

↓

Extended Kernel
```

Modules execute in:

```
Kernel Space
```

They have the same privileges as the kernel.

---

# Built-in Kernel Code

Some functionality is compiled directly into the kernel.

Examples:

```
Scheduler

Memory Manager

Virtual Memory

Interrupt Handling
```

These are always present.

---

# Loadable Modules

Other functionality is optional.

Examples:

```
Wi-Fi Driver

↓

Module
```

```
USB Driver

↓

Module
```

```
Filesystem Driver

↓

Module
```

Linux loads them only when required.

---

# Modular Monolithic Kernel

Linux is called a:

```
Modular Monolithic Kernel
```

Meaning:

```
Monolithic Kernel

+

Loadable Modules
```

Everything still executes in Kernel Space,

but modules can be added dynamically.

---

# Kernel Module Lifecycle

```
Module File (.ko)

↓

Load

↓

Initialize

↓

Running

↓

Unload

↓

Cleanup
```

Every module follows this lifecycle.

---

# Module Files

Kernel Modules usually have the extension:

```
.ko

Kernel Object
```

Example:

```
overlay.ko

bridge.ko

veth.ko
```

---

# Module Initialization

When a module loads,

Linux calls its initialization function.

```
Load Module

↓

module_init()

↓

Ready
```

The module registers itself with the kernel.

---

# Module Cleanup

When unloading,

Linux calls:

```
module_exit()

↓

Release Resources

↓

Unload
```

Everything allocated during initialization must be cleaned up.

---

# Viewing Loaded Modules

Linux provides:

```bash
lsmod
```

Example output:

```
overlay

bridge

veth

xfs

ext4
```

These modules are currently active.

---

# `insmod`

Loads a module manually.

Example:

```bash
sudo insmod hello.ko
```

Flow:

```
Module File

↓

Kernel

↓

Loaded
```

---

# `rmmod`

Removes a module.

Example:

```bash
sudo rmmod hello
```

Flow:

```
Kernel

↓

Cleanup

↓

Module Removed
```

---

# Problem with `insmod`

Suppose:

```
Module A

↓

Requires

↓

Module B
```

`insmod` loads only Module A.

Dependencies are **not** resolved automatically.

---

# `modprobe`

`modprobe` solves this problem.

Example:

```bash
sudo modprobe overlay
```

Flow:

```
Required Module

↓

Check Dependencies

↓

Load Dependencies

↓

Load Module
```

This is the recommended way to load modules.

---

# Module Dependencies

Example:

```
overlay

↓

Depends On

↓

other_module
```

`modprobe` loads everything required.

---

# Where Are Modules Stored?

Typically:

```
/lib/modules/<kernel-version>/
```

Example:

```bash
ls /lib/modules/$(uname -r)
```

You'll find:

- Drivers
- Filesystems
- Networking
- Security Modules

---

# Kernel Symbols

Modules often call kernel functions.

Example:

```
printk()

kmalloc()

kfree()
```

The kernel exports these functions as:

```
Kernel Symbols
```

Modules use them after loading.

---

# Viewing Kernel Symbols

Linux provides:

```bash
cat /proc/kallsyms
```

This lists exported kernel symbols.

---

# Device Drivers

Most hardware drivers are kernel modules.

Examples:

```
Ethernet Driver

↓

Module
```

```
USB Driver

↓

Module
```

```
Bluetooth Driver

↓

Module
```

Without the driver,

Linux cannot use the hardware.

---

# Filesystem Modules

Many filesystems are also modules.

Examples:

```
ext4

xfs

btrfs

overlay
```

Linux loads only the filesystems you actually use.

---

# Docker and Kernel Modules

Docker depends heavily on kernel modules.

Examples:

```
overlay

↓

OverlayFS
```

```
bridge

↓

Bridge Networking
```

```
veth

↓

Virtual Ethernet Pairs
```

```
br_netfilter

↓

Container Networking
```

Without these modules,

Docker cannot function correctly.

---

# Example

Suppose Docker starts.

Linux loads:

```
overlay

↓

bridge

↓

veth

↓

nf_tables

↓

Ready
```

Docker now has storage and networking support.

---

# Writing a Simple Kernel Module

Example:

```c
#include <linux/module.h>
#include <linux/kernel.h>

static int __init hello_init(void)
{
    printk(KERN_INFO "Hello Kernel\n");
    return 0;
}

static void __exit hello_exit(void)
{
    printk(KERN_INFO "Goodbye Kernel\n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
```

When loaded:

```
Hello Kernel
```

appears in the kernel log.

---

# Why `printk()`?

User programs use:

```c
printf()
```

Kernel code cannot.

Instead it uses:

```c
printk()
```

Messages appear in:

```bash
dmesg
```

---

# Module Load Flow

```
hello.ko

↓

insmod

↓

Kernel

↓

module_init()

↓

Running
```

---

# Module Unload Flow

```
rmmod

↓

module_exit()

↓

Cleanup

↓

Removed
```

---

# Why Kernel Modules Matter

Kernel Modules power almost everything.

Examples:

- Device Drivers
- Filesystems
- Networking
- Security Features
- Virtualization
- Docker

Without modules,

Linux would either become enormous or require recompiling for every hardware change.

---

# Hands-on Lab

## Lab 1 — View Loaded Modules

```bash
lsmod
```

---

## Lab 2 — View Module Information

```bash
modinfo overlay
```

Observe:

- Version
- Description
- Dependencies

---

## Lab 3 — Load a Module

```bash
sudo modprobe loop
```

---

## Lab 4 — Remove a Module

```bash
sudo modprobe -r loop
```

---

## Lab 5 — View Kernel Messages

```bash
dmesg | tail
```

Observe messages when modules load or unload.

---

## Lab 6 — View Module Directory

```bash
ls /lib/modules/$(uname -r)
```

---

# Interview Questions

## What is a Kernel Module?

A Kernel Module is a piece of kernel code that can be loaded and unloaded dynamically while the system is running.

---

## Why does Linux use Kernel Modules?

To keep the kernel smaller, reduce memory usage, support new hardware dynamically, and avoid rebuilding the kernel for optional functionality.

---

## What is the difference between `insmod` and `modprobe`?

`insmod` loads a module directly.

`modprobe` automatically resolves and loads module dependencies.

---

## What is a `.ko` file?

A `.ko` file is a compiled Linux Kernel Module (Kernel Object).

---

## Why do Kernel Modules run in Kernel Space?

Because they need direct access to kernel APIs and hardware resources.

---

## Which Docker features depend on Kernel Modules?

Docker relies on modules such as `overlay`, `bridge`, `veth`, and networking-related modules for storage and container networking.

---

## What is `printk()`?

`printk()` is the kernel's logging function, similar to `printf()` in user-space programs.

---

# Summary

Kernel Modules allow Linux to extend the running kernel dynamically.

```
Module (.ko)

↓

modprobe

↓

Kernel

↓

module_init()

↓

Running

↓

module_exit()

↓

Unload
```

Key concepts:

- Linux uses a Modular Monolithic Kernel.
- Kernel Modules extend functionality without rebooting.
- `.ko` files contain compiled kernel code.
- `insmod` loads modules directly.
- `modprobe` resolves dependencies automatically.
- Most device drivers and filesystems are implemented as modules.
- Docker relies on several kernel modules for storage and networking.
- Kernel Modules execute with full kernel privileges.

Understanding Kernel Modules is essential before studying namespaces, cgroups, networking internals, Docker, Kubernetes, and Linux kernel development.

---

# Next Chapter

## Chapter 24 — Signals & Process Control: How Linux Communicates with Running Processes

We'll explore:

- What are Signals?
- Signal Delivery
- Signal Handlers
- Default Actions
- `SIGINT`
- `SIGTERM`
- `SIGKILL`
- `SIGHUP`
- `SIGSTOP`
- `SIGCONT`
- Signal Masks
- Real-Time Signals
- How Linux Interrupts Running Processes