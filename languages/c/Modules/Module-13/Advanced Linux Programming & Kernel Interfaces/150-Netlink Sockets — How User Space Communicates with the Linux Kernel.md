# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 150 — Netlink Sockets — How User Space Communicates with the Linux Kernel

---

# Most developers think:

```
Application

↓

System Call

↓

Kernel
```

That's how every interaction works.

Usually...

Yes.

But then you run commands like:

```bash
ip addr
```

```bash
ip route
```

```bash
ip link
```

and something strange happens.

These commands can:

- Create network interfaces
- Delete routes
- Add IP addresses
- Receive kernel notifications

How?

Not through:

```
read()

↓

write()

↓

ioctl()
```

Modern Linux uses:

```
Netlink
```

---

# Learning Objectives

After this chapter you will understand:

- What Netlink is
- Why Linux created it
- Netlink sockets
- Kernel ↔ User communication
- Routing subsystem
- Network interfaces
- Kernel notifications
- Multicast groups
- Real-world applications

---

# The Problem

Suppose Linux detects:

```
USB Ethernet Adapter
```

Kernel immediately knows.

But user-space applications like:

```
systemd

↓

NetworkManager

↓

ip

↓

udev
```

must also know.

How does the kernel notify them?

---

# Another Problem

Suppose you type:

```bash
ip addr add 192.168.1.10/24 dev eth0
```

How does the kernel receive this request?

Traditional system calls are too limited.

Linux needed a messaging system.

---

# Linux Solution

Linux introduced:

```
Netlink

↓

Kernel Messaging Bus
```

---

# What is Netlink?

Definition:

> **Netlink is a Linux socket-based IPC mechanism that allows bidirectional communication between the kernel and user-space processes.**

Notice:

```
Bidirectional
```

Both sides can:

```
Send

↓

Receive
```

---

# Think About WhatsApp

Instead of:

```
Ask

↓

Wait

↓

Ask Again
```

Both users can:

```
Send Messages

↓

Receive Messages
```

Netlink works similarly.

---

# Visualization

```
Application

↓

Netlink Socket

↓

Kernel

↓

Network Stack
```

---

# Why Not `ioctl()`?

Older Linux networking used many `ioctl()` calls.

Problems:

- Hard to extend
- Device-specific
- Difficult to maintain

Netlink provides:

```
Structured Messages
```

instead of many custom commands.

---

# Netlink is a Socket

Program creates:

```
socket()

↓

AF_NETLINK
```

Kernel returns:

```
File Descriptor
```

Again,

everything becomes:

```
FD
```

---

# Architecture

```
User Process

↓

Netlink Socket

↓

Kernel

↓

Subsystem
```

---

# Kernel Subsystems

Netlink connects to many kernel components.

Examples:

```
Routing

↓

Networking

↓

Firewall

↓

Audit

↓

SELinux

↓

Generic Netlink
```

---

# Messages

Communication uses:

```
Structured Messages
```

Each message contains:

```
Header

↓

Payload
```

---

# Message Flow

Example:

```
Application

↓

Create Message

↓

Netlink Socket

↓

Kernel

↓

Process Request
```

---

# Reply

Kernel answers:

```
Result

↓

Netlink Socket

↓

Application
```

Two-way communication.

---

# Notifications

Kernel may also send messages without being asked.

Example:

```
Network Cable Unplugged
```

Kernel broadcasts:

```
Interface Down
```

Applications immediately receive the event.

---

# Visualization

```
Kernel

↓

Netlink

↓

NetworkManager

↓

Updates UI
```

---

# Routing Table

Suppose:

```bash
ip route add ...
```

Flow:

```
ip Command

↓

Netlink

↓

Kernel Routing Table

↓

Updated
```

---

# Network Interfaces

Command:

```bash
ip link set eth0 up
```

Flow:

```
ip

↓

Netlink

↓

Kernel

↓

Driver

↓

Device Enabled
```

---

# IP Addresses

Command:

```bash
ip addr add
```

Kernel updates:

```
Interface

↓

Address List
```

using Netlink messages.

---

# Multicast Groups

Multiple applications may subscribe to kernel events.

Example:

```
Kernel

↓

Interface Changed

↓

NetworkManager

↓

systemd

↓

Monitoring Tool
```

One event.

Many receivers.

---

# Generic Netlink

Linux later introduced:

```
Generic Netlink
```

Instead of creating a brand-new protocol for every subsystem,

developers can define families and commands within a common framework.

---

# Relationship with `epoll`

Since Netlink uses sockets,

it integrates naturally with:

```
epoll()
```

Event loop:

```
Socket

↓

timerfd

↓

eventfd

↓

signalfd

↓

Netlink

↓

epoll_wait()
```

---

# Relationship with VFS

Unlike:

```
/proc

↓

/sys
```

Netlink is **not** a filesystem.

It is:

```
Socket IPC
```

---

# Relationship with `ioctl()`

Older approach:

```
Application

↓

ioctl()

↓

Driver
```

Modern networking often prefers:

```
Application

↓

Netlink

↓

Kernel Networking Stack
```

---

# Relationship with Namespaces

Each network namespace has:

```
Own

↓

Interfaces

↓

Routes

↓

Addresses
```

Netlink communicates with the networking state of the namespace in which the application runs.

---

# Relationship with Containers

Docker creates:

```
veth Pair

↓

Network Namespace
```

Configuration is performed through Netlink.

---

# Real-World Example — `iproute2`

The `ip` command communicates with the kernel almost entirely through Netlink.

---

# Real-World Example — NetworkManager

Detects:

```
Cable Plugged

↓

Wi-Fi Connected

↓

Interface Changed
```

through Netlink notifications.

---

# Real-World Example — `udev`

Hardware changes trigger kernel events.

`udev` receives many of these through Netlink and creates or updates device nodes.

---

# Real-World Example — `systemd`

Monitors system events,

including networking-related changes,

using Netlink where appropriate.

---

# Real-World Example — Kubernetes

Container networking plugins configure interfaces, routes, and namespaces using Linux networking APIs that rely heavily on Netlink.

---

# Common Mistakes

---

## Thinking Netlink Uses TCP/IP

It does not.

Netlink is a local kernel ↔ user-space communication mechanism.

---

## Confusing Netlink with Unix Domain Sockets

Unix domain sockets connect user-space processes.

Netlink connects user space with the Linux kernel.

---

## Thinking Netlink Is Only for Networking

Networking is its biggest use,

but multiple kernel subsystems use Netlink.

---

## Assuming Netlink Replaces System Calls

No.

It complements system calls for structured kernel communication.

---

# Hands-on Labs

## Lab 1

Run:

```bash
ip addr
```

Understand that it communicates with the kernel using Netlink.

---

## Lab 2

Run:

```bash
ip route
```

Observe the routing table.

---

## Lab 3

Read:

```bash
man 7 netlink
```

Study the Netlink protocol families.

---

## Lab 4

Inspect Generic Netlink documentation in the Linux kernel.

---

## Lab 5

Research how `NetworkManager` uses Netlink to monitor interface changes.

---

# Interview Questions

### What is Netlink?

A Linux socket-based communication mechanism between user-space applications and the kernel.

---

### Why was Netlink introduced?

To provide a flexible, extensible messaging interface for kernel subsystems.

---

### Which command-line tool heavily uses Netlink?

`ip` from the `iproute2` suite.

---

### Can the kernel send unsolicited Netlink messages?

Yes.

The kernel can notify subscribed applications about events.

---

### Is Netlink a network protocol like TCP?

No.

It is a local IPC mechanism between user space and the Linux kernel.

---

# Summary

Architecture:

```
Application

↓

Netlink Socket

↓

Kernel

↓

Subsystem
```

Communication:

```
Application

↓

Request

↓

Kernel

↓

Reply
```

Notifications:

```
Kernel

↓

Event

↓

Netlink

↓

Applications
```

Common Users:

| Application | Uses Netlink For |
|-------------|------------------|
| `ip` | Interface and routing configuration |
| NetworkManager | Network events |
| `udev` | Device notifications |
| `systemd` | System event monitoring |
| Container runtimes | Network namespace configuration |

## Key Takeaways

- Netlink provides structured kernel ↔ user-space communication.
- It uses sockets rather than files or traditional `ioctl()` interfaces.
- Both requests and asynchronous notifications are supported.
- Modern Linux networking relies heavily on Netlink.
- It integrates naturally with `epoll` because it is socket-based.
- Understanding Netlink is essential for understanding modern Linux networking tools.

---

# 🎉 Milestone Reached

You have now completed **150 chapters** of the **C Mastery Roadmap**.

At this point, you understand much of the Linux userspace ↔ kernel interface, including:

- Processes
- Memory
- Virtual Memory
- System Calls
- Signals
- Filesystems
- VFS
- `/proc` and `/sys`
- `epoll`
- `eventfd`
- `timerfd`
- `signalfd`
- `inotify`
- `ptrace`
- Capabilities
- Namespaces
- cgroups
- `seccomp`
- Netlink

These are the technologies that power Linux servers, Docker, Kubernetes, databases, browsers, and high-performance applications.

---

# Next Chapter

## Chapter 151 — eBPF — The Technology That Lets You Run Programs Inside the Linux Kernel (Safely)

You'll learn:

- What eBPF is
- Why it was created
- BPF vs eBPF
- The eBPF virtual machine
- Verifier
- Maps
- Helper functions
- Tracing
- Networking
- Security
- Why companies like Meta, Google, Cloudflare, Netflix, and Cilium rely heavily on eBPF