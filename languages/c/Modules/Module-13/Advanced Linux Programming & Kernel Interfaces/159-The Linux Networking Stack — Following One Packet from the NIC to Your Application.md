# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 159 — The Linux Networking Stack — Following One Packet from the NIC to Your Application

---

# This chapter connects everything you've learned.

Until now you've learned individually about:

- NIC
- DMA
- Interrupts
- NAPI
- XDP
- TC
- Netfilter
- Routing
- TCP
- Sockets
- epoll
- io_uring
- eBPF

Now we'll combine them into **one complete packet journey**.

After this chapter, you'll be able to mentally visualize exactly what happens when someone opens your website.

---

# Learning Objectives

After this chapter you will understand:

- Complete packet flow
- NIC
- DMA
- Interrupts
- NAPI
- XDP
- sk_buff
- TC
- Netfilter
- Routing
- TCP/IP
- Socket layer
- User-space delivery

---

# The Big Question

Suppose someone opens:

```
https://yourwebsite.com
```

Their browser sends:

```
TCP Packet
```

Question:

Where does that packet go?

---

# Step 1 — Packet Arrives at NIC

Physical cable:

```
Ethernet

↓

NIC
```

The network interface card receives the electrical or optical signal and reconstructs the Ethernet frame.

---

# Visualization

```
Internet

↓

Ethernet Cable

↓

NIC
```

---

# Step 2 — DMA

Remember DMA?

Instead of:

```
CPU Copies Packet
```

NIC performs:

```
DMA

↓

RAM
```

Packet is transferred directly into memory.

CPU isn't involved in copying.

---

# Visualization

```
NIC

↓

DMA

↓

RAM
```

---

# Step 3 — Interrupt

Packet arrives.

NIC informs CPU:

```
Interrupt
```

Modern Linux often uses:

```
MSI-X
```

instead of legacy hardware interrupts.

---

# Step 4 — NAPI

Too many interrupts are expensive.

Linux switches to:

```
Polling Mode
```

using:

```
NAPI
```

Driver processes many packets together.

---

# Visualization

```
Packet Burst

↓

One Interrupt

↓

NAPI Poll

↓

100 Packets
```

Huge efficiency gain.

---

# Step 5 — XDP

Packet reaches:

```
Driver

↓

XDP Hook
```

Your eBPF program executes.

Possible results:

```
PASS

DROP

TX

REDIRECT

ABORTED
```

---

# Example

DDoS packet:

```
Packet

↓

XDP

↓

DROP
```

Journey ends immediately.

---

# Step 6 — `sk_buff`

If packet passes XDP,

Linux creates:

```
sk_buff

↓

skb
```

This structure stores packet metadata.

---

# What is `sk_buff`?

Think of it as:

```
Packet Object
```

Contains:

- Packet pointer
- Length
- Protocol
- Device
- Routing info
- Checksum state

and much more.

---

# Visualization

```
Packet Data

+

Metadata

↓

sk_buff
```

---

# Step 7 — Traffic Control (TC)

Packet reaches:

```
Ingress TC
```

An eBPF program may:

- Classify
- Redirect
- Modify
- Drop

the packet.

---

# Visualization

```
skb

↓

TC

↓

Decision
```

---

# Step 8 — Netfilter

Linux firewall.

Packet enters:

```
Netfilter
```

Rules may:

```
Accept

↓

Drop

↓

Reject

↓

NAT
```

---

# Visualization

```
Packet

↓

Firewall Rules

↓

Continue
```

---

# Step 9 — Routing Decision

Kernel asks:

```
Where Should

↓

This Packet Go?
```

Looks up:

```
Routing Table
```

---

# Example

Destination:

```
192.168.1.5
```

Kernel decides:

```
Local Machine
```

or

```
Forward
```

---

# Step 10 — Protocol Processing

Suppose Ethernet payload contains:

```
IPv4

↓

TCP

↓

HTTP
```

Kernel processes protocols layer by layer.

---

# IPv4 Layer

Checks:

- Header
- TTL
- Checksum
- Fragmentation

If valid,

packet continues.

---

# TCP Layer

Kernel verifies:

- Sequence number
- ACK
- Window
- Checksum

Updates connection state.

---

# Visualization

```
Ethernet

↓

IP

↓

TCP

↓

Payload
```

---

# Step 11 — Socket Lookup

Kernel asks:

```
Which Socket

↓

Owns This Packet?
```

Example:

```
Destination Port

↓

443
```

Find listening socket.

---

# Visualization

```
TCP Port

↓

Socket

↓

Application
```

---

# Step 12 — Socket Buffer

Packet enters:

```
Socket Receive Queue
```

Application hasn't read it yet.

Kernel stores it temporarily.

---

# Step 13 — Wake Application

Suppose server uses:

```
epoll()
```

Kernel marks:

```
Socket

↓

Readable
```

Waiting thread wakes.

---

# Visualization

```
Packet

↓

Socket Queue

↓

epoll()

↓

Wake Thread
```

---

# Step 14 — User Space

Application calls:

```c
recv()
```

or

```c
read()
```

Kernel copies or maps data as appropriate,

and the application finally receives the packet.

---

# Complete Journey

```
Internet

↓

NIC

↓

DMA

↓

Interrupt

↓

NAPI

↓

XDP

↓

sk_buff

↓

TC

↓

Netfilter

↓

Routing

↓

IP

↓

TCP

↓

Socket

↓

epoll()

↓

recv()

↓

Application
```

This is one of the most important diagrams in Linux networking.

---

# Outgoing Packet

Reverse direction:

```
Application

↓

send()

↓

Socket

↓

TCP

↓

IP

↓

Routing

↓

Netfilter

↓

TC

↓

Driver

↓

NIC

↓

Internet
```

---

# Relationship with eBPF

eBPF programs may execute at:

```
XDP

↓

TC

↓

Socket

↓

Cgroup

↓

Tracepoints
```

All while packet travels.

---

# Relationship with DMA

Without DMA:

```
CPU Copies Packet
```

With DMA:

```
NIC

↓

RAM
```

Much faster.

---

# Relationship with NAPI

NAPI prevents:

```
Interrupt Storm
```

during heavy traffic.

---

# Relationship with `io_uring`

Eventually application may read packets using modern asynchronous interfaces,

including `io_uring`, depending on application design.

---

# Relationship with AF_XDP

Instead of:

```
XDP

↓

Kernel Stack
```

Packet may go:

```
XDP

↓

AF_XDP

↓

Application
```

Skipping much of the networking stack.

---

# Real-World Example — Nginx

```
Packet

↓

NIC

↓

Kernel

↓

Socket

↓

epoll()

↓

Worker

↓

HTTP Response
```

---

# Real-World Example — Cilium

```
Packet

↓

XDP

↓

Policy

↓

TC

↓

Container
```

---

# Real-World Example — Cloudflare

Attack packet:

```
NIC

↓

XDP

↓

DROP
```

Never reaches TCP.

---

# Real-World Example — Kubernetes

Pod traffic traverses Linux networking components including routing, virtual interfaces, and often eBPF-based networking before reaching the container.

---

# Common Mistakes

---

## Thinking Packets Go Directly to Applications

Packets traverse many kernel layers before user space sees them.

---

## Ignoring `sk_buff`

`sk_buff` is one of the most important networking data structures in Linux.

---

## Thinking XDP Always Runs

Only when an XDP program is attached.

---

## Confusing TC and Netfilter

TC focuses on traffic control.

Netfilter focuses on firewalling, NAT, and packet filtering.

---

# Hands-on Labs

## Lab 1

Capture packets using:

```bash
tcpdump
```

Observe traffic entering your machine.

---

## Lab 2

Read:

```bash
man 7 packet
```

Learn about packet sockets.

---

## Lab 3

Study the Linux networking documentation for `sk_buff`.

---

## Lab 4

Compare packet paths:

- Traditional networking
- XDP
- AF_XDP

---

## Lab 5

Draw the complete packet flow from memory.

If you can reproduce it without looking,

you've mastered the Linux networking pipeline.

---

# Interview Questions

### What is the first Linux component that receives an incoming packet?

The Network Interface Card (NIC).

---

### Why is DMA important?

It transfers packet data directly between the NIC and RAM without requiring the CPU to copy every byte.

---

### What is the purpose of NAPI?

To reduce interrupt overhead by processing multiple packets during polling.

---

### What is `sk_buff`?

The Linux kernel data structure that stores packet metadata and references packet data while it moves through the networking stack.

---

### Where does `epoll()` become involved?

After packets are queued to a socket, the kernel marks the socket as readable and wakes waiting applications.

---

# Summary

Complete Incoming Packet Flow:

```
Internet

↓

NIC

↓

DMA

↓

Interrupt

↓

NAPI

↓

XDP

↓

sk_buff

↓

TC

↓

Netfilter

↓

Routing

↓

IPv4/IPv6

↓

TCP/UDP

↓

Socket Queue

↓

epoll()

↓

recv()

↓

Application
```

Outgoing Packet Flow:

```
Application

↓

send()

↓

Socket

↓

TCP

↓

IP

↓

Routing

↓

Netfilter

↓

TC

↓

Driver

↓

NIC

↓

Internet
```

Major Components:

| Component | Purpose |
|-----------|---------|
| NIC | Receive packets |
| DMA | Copy packets to RAM |
| NAPI | Efficient packet polling |
| XDP | Early packet processing |
| `sk_buff` | Packet metadata |
| TC | Traffic control |
| Netfilter | Firewall/NAT |
| Routing | Packet forwarding decisions |
| TCP/IP | Transport and network protocols |
| Socket | Deliver data to applications |

## Key Takeaways

- Every network packet passes through a well-defined Linux networking pipeline.
- DMA, NAPI, and XDP optimize packet reception before higher networking layers.
- `sk_buff` is the central packet representation inside the Linux kernel.
- TC, Netfilter, and routing each play different roles in packet processing.
- Applications receive packets only after the kernel completes protocol processing and socket delivery.
- Understanding the complete packet journey is essential for debugging, networking, kernel development, and high-performance systems.

---

# Next Chapter

## Chapter 160 — The Complete Linux I/O Stack — Following One `read()` Call from Your C Program to the Physical SSD

You'll learn:

- What really happens when you call `read()`
- VFS
- Page Cache
- Filesystem drivers
- Block layer
- I/O Scheduler
- NVMe/SATA drivers
- DMA
- Interrupt completion
- The complete journey of a file read through the Linux kernel