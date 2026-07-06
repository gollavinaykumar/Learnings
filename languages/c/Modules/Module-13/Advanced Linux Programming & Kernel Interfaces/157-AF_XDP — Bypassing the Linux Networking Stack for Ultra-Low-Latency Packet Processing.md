# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 157 — AF_XDP — Bypassing the Linux Networking Stack for Ultra-Low-Latency Packet Processing

---

# Imagine this situation.

Your server receives:

```
40 Million Packets

↓

Every Second
```

Each packet must be analyzed.

Traditional Linux path:

```
NIC

↓

Driver

↓

Kernel Networking Stack

↓

Socket Buffer

↓

Copy Data

↓

Application
```

Even with XDP,

eventually packets may reach user space through normal sockets.

That involves:

- Memory allocations
- Copies
- Socket buffers
- Context switching

For ultra-high-performance networking,

this is still too expensive.

Linux introduced:

```
AF_XDP
```

---

# Learning Objectives

After this chapter you will understand:

- What AF_XDP is
- Why it exists
- Relationship with XDP
- UMEM
- RX Ring
- TX Ring
- Fill Ring
- Completion Ring
- Copy Mode
- Zero-Copy Mode

---

# The Problem

Suppose you're building:

- Software Router
- Firewall
- Load Balancer
- Intrusion Detection System

Packets arrive:

```
10 Million+

↓

Every Second
```

Traditional sockets become the bottleneck.

Need:

```
Minimal Copies

↓

Minimal Context Switches

↓

Maximum Throughput
```

---

# Linux Solution

Instead of:

```
Packet

↓

Kernel Stack

↓

Socket Buffer

↓

Application
```

Linux allows:

```
Packet

↓

XDP

↓

AF_XDP

↓

Application
```

Very little kernel networking overhead.

---

# What is AF_XDP?

Definition:

> **AF_XDP is a Linux socket family that allows user-space applications to receive and transmit packets directly from XDP with extremely low overhead.**

Think of it as:

```
Express Lane

↓

Between NIC

↓

And User Space
```

---

# Relationship with XDP

Remember:

```
Packet

↓

NIC Driver

↓

XDP
```

Instead of:

```
PASS

↓

Kernel Stack
```

XDP can choose:

```
REDIRECT

↓

AF_XDP Socket
```

---

# Visualization

Traditional

```
Packet

↓

Kernel Stack

↓

Socket

↓

Application
```

AF_XDP

```
Packet

↓

XDP

↓

AF_XDP

↓

Application
```

---

# Why Is It Faster?

Traditional networking performs:

- Multiple memory copies
- Socket buffer allocation
- Protocol processing
- Additional bookkeeping

AF_XDP minimizes much of this work.

---

# The Secret

AF_XDP shares memory between:

```
Kernel

↓

User Space
```

Instead of copying packets repeatedly.

---

# What is UMEM?

UMEM means:

```
User Memory
```

Application allocates a large memory region.

Kernel uses the same memory for packet exchange.

---

# Visualization

```
Application

↓

UMEM

↑

Kernel
```

Shared packet storage.

---

# Packet Flow

```
NIC

↓

Driver

↓

XDP

↓

UMEM

↓

Application
```

---

# Rings

Communication happens through ring buffers.

Main rings are:

```
RX Ring

↓

TX Ring

↓

Fill Ring

↓

Completion Ring
```

---

# RX Ring

Incoming packets appear here.

Flow:

```
Packet

↓

RX Ring

↓

Application Reads
```

---

# TX Ring

Outgoing packets are placed here.

Application writes:

```
Packet

↓

TX Ring

↓

NIC Sends
```

---

# Fill Ring

Kernel needs empty packet buffers.

Application supplies them using:

```
Fill Ring
```

Think of it as:

```
Free Packet Buffers
```

---

# Completion Ring

After transmission,

kernel reports:

```
Packet Sent

↓

Buffer Free Again
```

Application can reuse that memory.

---

# Visualization

```
Application

↓

Fill Ring

↓

Kernel

↓

RX Packet

↓

Application

↓

TX Ring

↓

NIC

↓

Completion Ring
```

---

# Copy Mode

Simpler mode.

Packets are copied between kernel memory and user memory.

Works on more hardware.

Performance is still good.

---

# Zero-Copy Mode

Best performance.

Packet remains:

```
NIC

↓

UMEM

↓

Application
```

No additional packet copy between kernel and user space.

Requires hardware and driver support.

---

# Relationship with NIC Drivers

Driver support determines:

```
Native XDP

↓

Zero-Copy AF_XDP

↓

Maximum Performance
```

Not every NIC supports every feature.

---

# Relationship with XDP_REDIRECT

Remember XDP actions:

```
PASS

DROP

TX

REDIRECT
```

AF_XDP relies on:

```
REDIRECT

↓

AF_XDP Socket
```

---

# Relationship with `epoll`

AF_XDP sockets behave like Linux file descriptors.

Applications can integrate them with:

```
epoll()

↓

timerfd()

↓

eventfd()

↓

signalfd()
```

One event loop handles everything.

---

# Relationship with UMEM Buffers

Packets stay inside preallocated memory.

Benefits:

- Fewer allocations
- Better cache locality
- Lower latency

---

# Packet Lifecycle

```
Free Buffer

↓

Fill Ring

↓

NIC Receives Packet

↓

RX Ring

↓

Application Processes

↓

TX Ring (Optional)

↓

Completion Ring

↓

Reuse Buffer
```

---

# Relationship with the Verifier

Any XDP program redirecting packets to AF_XDP must still pass the eBPF verifier.

Safety guarantees remain unchanged.

---

# Real-World Example — DDoS Appliance

```
Packet

↓

XDP

↓

AF_XDP

↓

Application

↓

Decision

↓

Drop
```

Millions of packets per second.

---

# Real-World Example — Software Load Balancer

Incoming:

```
HTTP Packet
```

Application selects backend.

Packet transmitted immediately.

---

# Real-World Example — Network Monitoring

Packet:

```
AF_XDP

↓

Application

↓

Statistics

↓

Dashboard
```

Minimal overhead.

---

# Real-World Example — Financial Trading

Ultra-low-latency systems often use technologies that minimize packet copies and kernel overhead.

AF_XDP is one available Linux approach.

---

# Performance Comparison

Traditional Socket

```
Packet

↓

Multiple Copies

↓

Higher Latency
```

AF_XDP

```
Packet

↓

Shared Memory

↓

Minimal Copies

↓

Lower Latency
```

---

# Common Mistakes

---

## Thinking AF_XDP Replaces XDP

AF_XDP complements XDP.

XDP decides whether packets should be redirected.

---

## Assuming Zero-Copy Works Everywhere

It depends on NIC and driver capabilities.

---

## Forgetting UMEM

UMEM is the foundation of AF_XDP performance.

---

## Using AF_XDP for Ordinary Web Servers

Traditional sockets are usually sufficient.

AF_XDP is intended for specialized high-performance networking workloads.

---

# Hands-on Labs

## Lab 1

Read:

```bash
man 7 xdp
```

and Linux AF_XDP documentation.

---

## Lab 2

Explore:

```bash
bpftool
```

and XDP redirect examples.

---

## Lab 3

Study AF_XDP sample programs included with the Linux kernel source.

---

## Lab 4

Compare:

- Traditional sockets
- AF_XDP

Understand the packet path differences.

---

## Lab 5

Research which NICs support zero-copy AF_XDP.

---

# Interview Questions

### What is AF_XDP?

A Linux socket family that enables high-performance packet processing using XDP and shared memory.

---

### Why is AF_XDP faster than traditional sockets?

It minimizes packet copies, socket-buffer overhead, and unnecessary kernel processing.

---

### What is UMEM?

A shared user-space memory region used to exchange packets efficiently between the kernel and the application.

---

### What are the four major AF_XDP rings?

RX Ring, TX Ring, Fill Ring, and Completion Ring.

---

### What is the difference between Copy Mode and Zero-Copy Mode?

Copy Mode copies packet data between kernel and user memory.

Zero-Copy Mode allows supported hardware to exchange packets without additional copies.

---

# Summary

Architecture:

```
NIC

↓

Driver

↓

XDP

↓

AF_XDP

↓

Application
```

Memory Model:

```
Application

↓

UMEM

↑

Kernel
```

Ring Buffers:

| Ring | Purpose |
|------|---------|
| RX Ring | Receive packets |
| TX Ring | Transmit packets |
| Fill Ring | Supply empty buffers |
| Completion Ring | Recycle transmitted buffers |

Packet Lifecycle:

```
Fill Buffer

↓

Receive Packet

↓

Application

↓

Transmit

↓

Recycle Buffer
```

## Key Takeaways

- AF_XDP provides an ultra-low-overhead path between XDP and user-space applications.
- UMEM enables efficient shared-memory packet exchange.
- Ring buffers coordinate packet movement without excessive synchronization.
- Zero-Copy Mode offers the highest performance when supported.
- AF_XDP integrates naturally with XDP and Linux event-driven programming.
- It is designed for specialized networking systems requiring extremely high packet throughput and low latency.

---

# Next Chapter

## Chapter 158 — io_uring — The Future of High-Performance Asynchronous I/O in Linux

You'll learn:

- Why `read()` and `write()` are becoming bottlenecks
- What `io_uring` is
- Submission Queue (SQ)
- Completion Queue (CQ)
- Shared ring buffers
- SQEs and CQEs
- Zero-copy concepts
- How modern databases, web servers, and storage engines achieve massive I/O throughput with `io_uring`