# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 155 — XDP (Express Data Path) — Processing Network Packets Before the Linux Network Stack

---

# Imagine this situation.

A server receives:

```
10 Million Packets

↓

Every Second
```

Most of them are:

```
DDoS Packets

↓

Garbage Traffic
```

Normally Linux processes every packet through:

- Network Driver
- Kernel Network Stack
- Firewall
- Routing
- Socket Layer

Only then does it decide:

```
Drop Packet
```

Too late.

Millions of CPU cycles have already been wasted.

Linux needed a way to stop packets **before** they entered the networking stack.

That solution is:

```
XDP

Express Data Path
```

---

# Learning Objectives

After this chapter you will understand:

- What XDP is
- Packet processing pipeline
- NIC drivers
- XDP hook
- XDP actions
- Native Mode
- Generic Mode
- Offload Mode
- Real-world use cases

---

# The Traditional Packet Flow

Suppose a packet arrives.

Linux processes:

```
NIC

↓

Driver

↓

Kernel Network Stack

↓

Netfilter

↓

Routing

↓

Socket

↓

Application
```

If the packet is malicious,

Linux still performed all that work.

---

# The Problem

Imagine:

```
100 Million

↓

Bad Packets
```

Linux wastes CPU processing packets that will eventually be discarded.

---

# Linux Solution

Place an eBPF hook:

```
Immediately

↓

After NIC Driver
```

Packet arrives.

Before anything else:

```
Run eBPF

↓

Decision
```

---

# What is XDP?

Definition:

> **XDP (Express Data Path) is an eBPF-based packet processing framework that runs at the earliest possible point in the Linux networking path.**

Think of it as:

```
Security Guard

↓

Standing At The Door
```

Instead of:

```
Inside The Building
```

---

# Visualization

Traditional

```
Packet

↓

Kernel

↓

TCP

↓

Socket

↓

Application
```

With XDP

```
Packet

↓

XDP

↓

Drop

↓

Done
```

Huge performance improvement.

---

# Why So Fast?

Because packets haven't yet reached:

- TCP
- UDP
- Routing
- Firewall
- Socket Buffers

Most kernel work hasn't happened yet.

---

# Packet Flow with XDP

```
NIC

↓

Driver

↓

XDP

↓

PASS?

↓

Kernel Stack

↓

Application
```

---

# The XDP Hook

Every packet reaches:

```
Driver

↓

XDP Hook
```

Your eBPF program executes here.

This is the earliest programmable point for packet processing on many Linux systems.

---

# XDP Actions

Your program must return one of several actions.

Let's study them.

---

# 1. XDP_PASS

Meaning:

```
Packet Is Good

↓

Continue
```

Flow:

```
Packet

↓

XDP

↓

PASS

↓

Kernel Networking
```

---

# 2. XDP_DROP

Meaning:

```
Packet

↓

Discard
```

Never reaches:

- TCP
- Firewall
- Application

Very efficient.

---

# Visualization

```
Packet

↓

DROP

↓

Finished
```

---

# Real-World Example

DDoS packet:

```
Packet

↓

XDP

↓

DROP
```

No further CPU time wasted.

---

# 3. XDP_TX

Meaning:

```
Send Packet

↓

Back

↓

Out Same Interface
```

Useful for:

- Fast replies
- Packet reflection
- Certain network appliances

---

# Visualization

```
Packet

↓

XDP

↓

TX

↓

NIC
```

---

# 4. XDP_REDIRECT

Instead of:

```
Current Interface
```

Send packet to:

```
Another Interface

↓

Another CPU

↓

Another XDP Program

↓

AF_XDP Socket
```

Very powerful.

---

# Example

Incoming:

```
eth0
```

Redirect:

```
eth1
```

Without entering the traditional networking stack.

---

# 5. XDP_ABORTED

Used when the program encounters an unexpected internal error.

Primarily useful for debugging and diagnostics.

---

# Native Mode

Fastest common deployment.

Flow:

```
NIC Driver

↓

Native XDP

↓

Packet
```

Runs directly in the driver's receive path.

---

# Generic Mode

Suppose your driver doesn't support native XDP.

Linux provides:

```
Generic XDP
```

Runs later in the networking path.

Slower,

but widely compatible.

---

# Offload Mode

Some network interface cards support executing XDP programs on the NIC itself.

Flow:

```
Packet

↓

NIC Hardware

↓

XDP

↓

CPU
```

The CPU may never see packets that are dropped.

Hardware support varies by vendor and device.

---

# Packet Parsing

Suppose packet:

```
Ethernet

↓

IPv4

↓

TCP

↓

HTTP
```

XDP program examines headers directly.

Example:

```
Source IP

↓

Destination IP

↓

Port
```

Decision made immediately.

---

# Packet Modification

An XDP program may:

- Update headers
- Adjust metadata
- Redirect traffic

subject to verifier rules and hook capabilities.

---

# Relationship with Maps

Suppose:

```
Blocked IP

↓

Hash Map
```

Packet arrives:

```
Lookup

↓

Found?

↓

DROP
```

Very common design.

---

# Relationship with the Verifier

Before loading:

```
Program

↓

Verifier

↓

Approved

↓

Attach To XDP
```

Unsafe programs are rejected.

---

# Relationship with TC

Remember:

```
XDP

↓

Very Early
```

Later:

```
Traffic Control (TC)
```

XDP prioritizes speed.

TC provides richer packet context.

---

# Relationship with Netfilter

Traditional firewall:

```
Packet

↓

Netfilter

↓

Decision
```

With XDP:

```
Packet

↓

Drop

↓

Never Reaches Netfilter
```

---

# Relationship with AF_XDP

Linux provides:

```
AF_XDP
```

allowing high-performance packet delivery between XDP and user-space applications.

This minimizes copying and improves throughput.

---

# Real-World Example — Cloudflare

Millions of packets:

```
NIC

↓

XDP

↓

DROP Attack Traffic

↓

PASS Legitimate Traffic
```

---

# Real-World Example — Cilium

Container packet:

```
XDP

↓

Policy

↓

Redirect

↓

Container
```

---

# Real-World Example — Meta

Large-scale infrastructure uses XDP for efficient packet filtering and networking optimizations.

---

# Real-World Example — Load Balancer

Packet:

```
Client

↓

XDP

↓

Choose Backend

↓

REDIRECT
```

Very high performance.

---

# Performance

Traditional path:

```
Packet

↓

Many Kernel Layers
```

XDP:

```
Packet

↓

Decision

↓

Done
```

Much less work per packet.

---

# Common Mistakes

---

## Thinking XDP Replaces TCP

It processes packets before the normal networking stack.

It does not replace TCP or UDP.

---

## Assuming Every NIC Supports Native XDP

Some drivers support only Generic Mode,

and hardware offload is available only on certain devices.

---

## Using XDP for Complex Application Logic

XDP is optimized for fast packet decisions,

not heavy computation.

---

## Forgetting the Verifier

Every XDP program must pass verifier checks before loading.

---

# Hands-on Labs

## Lab 1

Install:

```bash
bpftool
```

Inspect available XDP programs.

---

## Lab 2

Read kernel documentation on XDP architecture.

---

## Lab 3

Research supported XDP network drivers for your hardware.

---

## Lab 4

Study Cilium's XDP-based packet processing pipeline.

---

## Lab 5

Compare packet processing paths:

- Traditional networking
- TC
- XDP

Understand where each hook executes.

---

# Interview Questions

### What is XDP?

An eBPF-based packet processing framework that executes at the earliest stage of Linux packet reception.

---

### Why is XDP faster than traditional packet processing?

Because it makes decisions before packets traverse most of the Linux networking stack.

---

### What does `XDP_DROP` do?

It immediately discards the packet.

---

### What is the difference between Native XDP and Generic XDP?

Native XDP runs in supported NIC drivers.

Generic XDP runs later in the networking path for broader compatibility.

---

### What is `XDP_REDIRECT` used for?

Redirecting packets to another interface, CPU, or supported destination without following the normal networking path.

---

# Summary

Traditional Flow:

```
Packet

↓

Driver

↓

TCP

↓

Socket

↓

Application
```

XDP Flow:

```
Packet

↓

Driver

↓

XDP

↓

PASS

or

DROP

or

TX

or

REDIRECT
```

Execution Modes:

| Mode | Description |
|------|-------------|
| Native | Runs in the NIC driver |
| Generic | Runs in the networking stack |
| Offload | Runs on supported NIC hardware |

XDP Actions:

| Action | Result |
|--------|--------|
| `XDP_PASS` | Continue normal processing |
| `XDP_DROP` | Discard packet |
| `XDP_TX` | Transmit back through the interface |
| `XDP_REDIRECT` | Redirect packet elsewhere |
| `XDP_ABORTED` | Abort due to program error |

## Key Takeaways

- XDP is the fastest programmable packet-processing framework in Linux.
- It executes before most of the networking stack.
- eBPF programs attached to XDP can efficiently filter, redirect, or modify traffic.
- Native XDP offers the highest performance, while Generic XDP provides broader compatibility.
- XDP is widely used for DDoS mitigation, load balancing, and high-performance networking.
- Understanding XDP is fundamental to modern Linux networking and eBPF development.

---

# Next Chapter

## Chapter 156 — TC (Traffic Control) with eBPF — Building Linux Traffic Shapers, Firewalls, and Packet Manipulators

You'll learn:

- What Traffic Control (TC) is
- Ingress vs Egress
- Queuing disciplines (qdiscs)
- Packet classification
- Packet scheduling
- eBPF classifiers and actions
- QoS
- Traffic shaping
- How Linux controls network traffic after packets enter the networking stack