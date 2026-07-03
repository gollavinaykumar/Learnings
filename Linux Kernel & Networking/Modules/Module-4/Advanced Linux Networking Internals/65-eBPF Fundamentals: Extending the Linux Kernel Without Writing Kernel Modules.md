A few years ago, if you wanted to extend the Linux kernel, you had only one option:

```
Write

Kernel Module

(.ko)
```

That meant:

❌ Kernel programming

❌ Memory safety risks

❌ Kernel crashes

❌ Difficult debugging

❌ System reboots after mistakes

Very dangerous.

Linux developers wanted something much safer.

They asked:

> "Can we run our own programs **inside the Linux kernel** without risking the entire operating system?"

The answer became one of the biggest innovations in Linux history:

# eBPF

Today, eBPF powers:

- Kubernetes
- Cilium
- Meta (Facebook)
- Google
- Netflix
- Cloudflare
- Datadog
- Grafana
- Security products
- Observability tools

Many people call eBPF:

> **"The JavaScript of the Linux Kernel."**

(That's an analogy, not an official definition.)

After this chapter, you'll understand why eBPF is transforming Linux networking, security, and observability.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 65 — eBPF Fundamentals: Extending the Linux Kernel Without Writing Kernel Modules

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why eBPF exists
> - What is eBPF?
> - eBPF Virtual Machine
> - Kernel Hooks
> - eBPF Programs
> - Maps
> - Verifier
> - JIT Compilation
> - Networking with eBPF
> - Docker & Kubernetes Usage

---

# 📖 Why Was eBPF Introduced?

Suppose you want to:

- Count every HTTP request
- Monitor TCP connections
- Detect attacks
- Load balance packets
- Trace system calls

Traditional approach:

```
Modify Linux Kernel

↓

Recompile

↓

Reboot
```

Very impractical.

Linux needed a safer solution.

---

# What is eBPF?

eBPF stands for:

```
Extended

Berkeley Packet Filter
```

Originally,

BPF only filtered network packets.

Modern eBPF can safely run custom programs inside the Linux kernel.

---

# Think of Plugins

Imagine your browser.

```
Browser

↓

Extensions

↓

New Features
```

eBPF works similarly.

```
Linux Kernel

↓

eBPF Programs

↓

New Features
```

Without modifying kernel source code.

---

# High-Level Architecture

```
Application

↓

eBPF Loader

↓

Verifier

↓

Linux Kernel

↓

eBPF Program
```

---

# eBPF Virtual Machine

Linux contains a tiny virtual machine.

```
Kernel

↓

eBPF VM

↓

Program
```

Programs execute inside this controlled environment.

---

# Why Not Run Native Code?

Suppose someone loads:

```
Infinite Loop

↓

Kernel Freeze
```

That would be catastrophic.

Linux prevents unsafe programs before they execute.

---

# The Verifier

Every eBPF program must pass through:

```
Verifier
```

The verifier checks things like:

✔ Memory safety

✔ Valid pointer usage

✔ Bounded execution

✔ Valid program structure

If the program fails verification,

Linux rejects it.

---

# Visual Representation

```
eBPF Program

↓

Verifier

↓

Approved?

↓

Yes

↓

Kernel

↓

Execute
```

Otherwise:

```
Rejected
```

---

# Hooks

eBPF programs don't run continuously.

They attach to:

```
Hooks
```

Examples:

```
Packet Arrives
```

```
System Call
```

```
TCP Event
```

```
Function Call
```

When the event occurs,

the eBPF program runs.

---

# Networking Example

Packet arrives:

```
NIC

↓

eBPF Program

↓

Allow?

↓

Drop?

↓

Modify?

↓

Continue
```

No traditional firewall rule is required for simple filtering logic.

---

# System Call Example

Application:

```
open()
```

Linux triggers:

```
eBPF Program
```

The program can observe or record the event.

---

# eBPF Maps

Programs need storage.

Linux provides:

```
Maps
```

Think of them as shared data structures.

Examples:

```
Hash Map
```

```
Array
```

```
LRU Cache
```

```
Ring Buffer
```

Both kernel-space eBPF programs and user-space applications can access these maps (subject to permissions).

---

# Example

Packet arrives:

```
Source IP
```

Store count:

```
Hash Map

↓

192.168.1.10

↓

250 Packets
```

Statistics update without changing kernel source code.

---

# User Space Communication

Application:

```
User Space
```

↓

```
Read Map
```

↓

```
Display Statistics
```

Kernel and userspace communicate through eBPF maps.

---

# JIT Compilation

Initially,

eBPF programs are loaded in bytecode form.

Linux can:

```
Bytecode

↓

JIT Compiler

↓

Native Machine Code
```

This improves execution speed significantly.

---

# Why Is eBPF Fast?

Programs run:

```
Inside Kernel
```

No frequent context switches.

No packet copies to userspace.

Very low overhead.

---

# Networking with eBPF

Modern networking systems use eBPF for:

- Firewalls
- Load Balancing
- Packet Filtering
- NAT
- Traffic Accounting

Many networking tasks can execute without relying solely on traditional iptables chains.

---

# Observability

eBPF enables deep visibility into Linux.

Examples:

- CPU usage
- Disk I/O
- Network latency
- TCP retransmissions
- System calls
- Memory allocation

All with minimal overhead.

---

# Security

Security products use eBPF to:

- Detect malware
- Monitor suspicious activity
- Trace process execution
- Observe file access
- Detect privilege escalation attempts

Without modifying applications.

---

# Docker

Docker itself does not require eBPF.

However,

modern container networking and observability tools frequently use eBPF alongside Docker.

---

# Kubernetes

Many Kubernetes networking platforms rely heavily on eBPF.

Most famous example:

```
Cilium
```

Cilium uses eBPF for:

- Networking
- Security Policies
- Load Balancing
- Observability

Often reducing dependence on large iptables rule sets.

---

# Real Companies

Organizations using eBPF include:

- Meta
- Google
- Netflix
- Cloudflare
- Isovalent
- Datadog
- Grafana

It has become a key technology in cloud infrastructure.

---

# Packet Flow

```
Packet

↓

NIC

↓

eBPF Hook

↓

Program

↓

Allow

Drop

Modify

↓

Kernel Networking Stack
```

---

# Complete Architecture

```
Application

↓

eBPF Loader

↓

Verifier

↓

Kernel

↓

Hook

↓

Program

↓

Map

↓

Decision
```

---

# Why eBPF Matters

eBPF enables:

✔ High Performance

✔ Kernel Extensibility

✔ Safe Execution

✔ Observability

✔ Security

✔ Advanced Networking

without modifying the Linux kernel source.

---

# Hands-on Lab

## Lab 1 — Verify eBPF Support

```bash
uname -r
```

Modern Linux kernels provide extensive eBPF support.

---

## Lab 2 — View Loaded Programs (if bpftool is installed)

```bash
sudo bpftool prog show
```

---

## Lab 3 — View Maps

```bash
sudo bpftool map show
```

---

## Lab 4 — Explore Tracepoints

```bash
sudo bpftool feature
```

Observe supported eBPF features on your system.

---

# Interview Questions

## What is eBPF?

eBPF is a Linux kernel technology that allows verified programs to execute safely inside the kernel.

---

## Why was eBPF introduced?

To safely extend kernel functionality without writing traditional kernel modules.

---

## What is the eBPF Verifier?

The verifier checks eBPF programs for safety before allowing them to execute inside the kernel.

---

## What are eBPF Maps?

Maps are shared kernel data structures used for communication and state storage between eBPF programs and userspace.

---

## Why is eBPF fast?

Because programs execute inside the kernel with minimal context switching and can be JIT-compiled into native machine code.

---

## Does Docker require eBPF?

No.

However, many modern networking and observability tools used with Docker rely on eBPF.

---

## Which Kubernetes networking project is most associated with eBPF?

Cilium.

---

# Summary

eBPF allows Linux to safely execute custom programs inside the kernel.

```
Application

↓

eBPF Loader

↓

Verifier

↓

Kernel

↓

Hook

↓

Program

↓

Maps

↓

Result
```

Key concepts:

- eBPF safely extends kernel functionality.
- Programs attach to kernel hooks.
- The verifier prevents unsafe programs.
- Maps store shared state.
- JIT compilation improves performance.
- eBPF powers modern networking, observability, and security tools.
- Cilium is one of the best-known Kubernetes networking solutions built on eBPF.

At this point, you've learned one of the most transformative Linux technologies introduced in the last decade.

Understanding eBPF opens the door to advanced networking, tracing, performance analysis, and cloud-native infrastructure.

---

# Next Chapter

## Chapter 66 — XDP (Express Data Path): Processing Packets Before the Linux Networking Stack

We'll explore:

- What is XDP?
- Why XDP exists
- XDP vs Netfilter
- XDP Hooks
- Early Packet Processing
- DDoS Protection
- High-Speed Packet Filtering
- XDP Actions
- eBPF + XDP
- Complete Packet Processing Architecture

> **This chapter explains how Linux can process millions of packets per second before they even enter the normal networking stack.**