# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 156 — TC (Traffic Control) with eBPF — Building Linux Traffic Shapers, Firewalls, and Packet Manipulators

---

# In the previous chapter you learned:

```
XDP

↓

Runs

↓

Immediately After NIC Driver
```

It is extremely fast.

But there is a limitation.

At the XDP stage,

Linux has not yet built the complete networking context.

Sometimes you need:

- Routing information
- Packet scheduling
- Bandwidth limits
- QoS
- Packet delay
- Packet reordering

XDP is not designed for these tasks.

Linux provides another framework:

```
Traffic Control

↓

TC
```

TC runs later in the networking stack,

where much more information is available.

---

# Learning Objectives

After this chapter you will understand:

- What TC is
- Packet flow
- Ingress
- Egress
- qdisc
- Filters
- Classifiers
- Actions
- eBPF integration
- QoS

---

# The Problem

Suppose your server receives:

```
Video Calls

↓

File Downloads

↓

Game Traffic

↓

Database Replication
```

Should Linux treat every packet equally?

Probably not.

Video traffic should usually receive lower latency than large file downloads.

Linux needs a way to prioritize traffic.

---

# Another Problem

Suppose one client uploads:

```
10 Gbps
```

Everyone else becomes slow.

Need:

```
Bandwidth Limits
```

---

# Linux Solution

Packets enter:

```
Traffic Control

↓

Scheduling

↓

Classification

↓

Actions
```

Before transmission or after reception.

---

# What is Traffic Control?

Definition:

> **Traffic Control (TC) is the Linux subsystem responsible for classifying, scheduling, shaping, prioritizing, and policing network traffic.**

Think of it as:

```
Traffic Police

↓

For Packets
```

---

# Traditional Packet Flow

```
NIC

↓

Driver

↓

Kernel

↓

TCP/IP

↓

Socket
```

---

# Packet Flow with TC

```
NIC

↓

Driver

↓

Kernel

↓

TC

↓

TCP/IP

↓

Application
```

or,

for outgoing traffic:

```
Application

↓

TCP/IP

↓

TC

↓

NIC

↓

Network
```

---

# Two Hook Points

TC provides two major attachment points.

```
Ingress

↓

Incoming Packets

----------------------

Egress

↓

Outgoing Packets
```

---

# Visualization

```
Internet

↓

Ingress

↓

Kernel

↓

Application

↓

Egress

↓

Internet
```

---

# Ingress

Packet arrives.

Flow:

```
Network

↓

NIC

↓

Ingress TC

↓

Kernel
```

Useful for:

- Filtering
- Statistics
- Packet marking
- Policing

---

# Egress

Packet leaves.

Flow:

```
Application

↓

TCP

↓

Egress TC

↓

NIC
```

Useful for:

- Scheduling
- QoS
- Rate limiting
- Shaping

---

# What is a qdisc?

One of the most important TC concepts.

qdisc means:

```
Queueing Discipline
```

Every outgoing interface has one.

---

# Think About a Bank

Customers arrive.

Question:

Who gets served first?

Possible rules:

```
First Come First Served

↓

Priority Queue

↓

Round Robin
```

A qdisc works similarly for packets.

---

# Visualization

```
Packets

↓

Queue

↓

Scheduler

↓

NIC
```

---

# Default qdisc

Linux assigns a default queueing discipline to network interfaces.

The exact default depends on the kernel version and configuration (for example, many modern systems use `fq_codel`).

---

# Packet Classification

Suppose packets arrive.

Need to determine:

```
Video

↓

Gaming

↓

SSH

↓

Database
```

Classification decides where each packet belongs.

---

# Classifier

Classifier examines packet fields.

Example:

```
Destination Port

↓

443

↓

HTTPS
```

Decision:

```
Class A
```

---

# Actions

After classification,

Linux performs actions.

Examples:

```
Allow

↓

Drop

↓

Redirect

↓

Modify

↓

Mirror
```

---

# eBPF Integration

Instead of static rules,

TC can execute:

```
eBPF Program
```

Flow:

```
Packet

↓

TC Hook

↓

eBPF

↓

Decision
```

---

# Packet Modification

An eBPF TC program may:

- Modify headers
- Set packet marks
- Adjust metadata
- Redirect packets

subject to verifier rules.

---

# Difference Between XDP and TC

XDP:

```
Very Early

↓

Maximum Speed
```

TC:

```
Later

↓

More Information
```

---

# Visualization

```
Packet

↓

Driver

↓

XDP

↓

Kernel

↓

TC

↓

Socket
```

---

# Traffic Shaping

Suppose:

```
100 Mbps

↓

Maximum
```

Application sends:

```
1 Gbps
```

TC delays packets to maintain the configured rate.

---

# Visualization

Without shaping:

```
Packets

↓

Immediate Send
```

With shaping:

```
Packets

↓

Queue

↓

Controlled Release
```

---

# Traffic Policing

Different from shaping.

Suppose limit:

```
100 Mbps
```

Traffic arrives:

```
200 Mbps
```

Instead of delaying packets,

policing may:

```
Drop Excess Packets
```

---

# QoS (Quality of Service)

Not every packet has equal importance.

Example:

```
Voice

↓

Highest Priority

----------------------

SSH

↓

High

----------------------

Video

↓

Medium

----------------------

Backup

↓

Low
```

Scheduler sends important packets first.

---

# Packet Scheduling

Suppose queue contains:

```
Voice

↓

SSH

↓

Backup
```

Scheduler may transmit:

```
Voice

↓

SSH

↓

Backup
```

instead of arrival order.

---

# Common qdisc Types

Examples include:

```
pfifo_fast

↓

fq

↓

fq_codel

↓

htb

↓

tbf
```

Each implements different scheduling or shaping strategies.

---

# HTB

HTB means:

```
Hierarchical Token Bucket
```

Used for:

- Bandwidth guarantees
- Hierarchical traffic classes

Widely used in Linux traffic shaping.

---

# TBF

Token Bucket Filter.

Useful for:

```
Rate Limiting
```

Simple and efficient.

---

# Relationship with eBPF Maps

Packet:

```
Source IP

↓

Map Lookup

↓

Priority

↓

Apply Rule
```

Maps allow dynamic policies.

---

# Relationship with XDP

Remember:

```
XDP

↓

Earliest

↓

Drop Obvious Garbage
```

Later:

```
TC

↓

Advanced Decisions
```

Many systems use both.

---

# Relationship with Netfilter

Netfilter primarily focuses on firewalling and NAT.

TC focuses on scheduling, shaping, QoS, and packet manipulation.

Some deployments combine both.

---

# Relationship with the Verifier

Every TC eBPF program must pass:

```
Verifier

↓

Approved

↓

Attach
```

before execution.

---

# Real-World Example — ISP

Customer:

```
1 Gbps Link
```

Plan purchased:

```
100 Mbps
```

TC shapes outgoing traffic.

---

# Real-World Example — Kubernetes

Container traffic:

```
TC

↓

QoS

↓

Bandwidth Control
```

depending on the networking solution.

---

# Real-World Example — Video Conferencing

Voice packets:

```
Highest Priority
```

File downloads:

```
Lower Priority
```

Better user experience.

---

# Real-World Example — Cloud Platforms

Virtual machine traffic may be rate-limited and prioritized using Linux Traffic Control.

---

# Common Mistakes

---

## Thinking TC Replaces XDP

They solve different problems and often complement each other.

---

## Confusing Shaping and Policing

Shaping delays packets.

Policing typically drops excess traffic.

---

## Ignoring qdiscs

Every egress interface uses a queueing discipline.

It is central to TC.

---

## Assuming QoS Means Faster Internet

QoS prioritizes important traffic.

It does not create additional bandwidth.

---

# Hands-on Labs

## Lab 1

Inspect current qdiscs:

```bash
tc qdisc show
```

---

## Lab 2

Read:

```bash
man tc
```

Explore Traffic Control concepts.

---

## Lab 3

Study common qdisc types:

- `fq_codel`
- `htb`
- `tbf`

---

## Lab 4

Compare:

- XDP
- TC

Understand where each hook executes.

---

## Lab 5

Research how Cilium attaches eBPF programs at TC hooks.

---

# Interview Questions

### What is Linux Traffic Control?

The Linux subsystem responsible for scheduling, shaping, classifying, and controlling network traffic.

---

### What is the difference between ingress and egress?

Ingress handles incoming packets.

Egress handles outgoing packets.

---

### What is a qdisc?

A queueing discipline that determines how outgoing packets are queued and transmitted.

---

### What is the difference between shaping and policing?

Shaping delays traffic to enforce limits.

Policing usually drops traffic that exceeds configured limits.

---

### Why would you choose TC instead of XDP?

When you need richer networking context, packet scheduling, QoS, or traffic shaping.

---

# Summary

Packet Flow:

```
NIC

↓

Driver

↓

XDP

↓

Kernel

↓

TC

↓

Socket
```

Hook Points:

| Hook | Purpose |
|------|---------|
| Ingress | Incoming packet processing |
| Egress | Outgoing packet processing |

Traffic Pipeline:

```
Packet

↓

Classifier

↓

qdisc

↓

Scheduler

↓

NIC
```

Comparison:

| Feature | XDP | TC |
|---------|-----|----|
| Execution Point | Earliest | Later |
| Performance | Highest | High |
| Traffic Shaping | No | Yes |
| QoS | Limited | Yes |
| Scheduling | No | Yes |
| Rich Network Context | Limited | Yes |

## Key Takeaways

- Traffic Control manages packet scheduling, shaping, and prioritization.
- TC provides ingress and egress hook points.
- qdiscs determine how packets are queued and transmitted.
- eBPF programs can extend TC with programmable packet processing.
- XDP and TC complement each other rather than compete.
- TC is essential for QoS, bandwidth control, and advanced Linux networking.

---

# Next Chapter

## Chapter 157 — AF_XDP — Bypassing the Linux Networking Stack for Ultra-Low-Latency Packet Processing

You'll learn:

- What AF_XDP is
- Zero-copy networking
- UMEM
- RX/TX rings
- Fill and Completion rings
- Copy mode vs Zero-copy mode
- Relationship with XDP
- How modern packet-processing systems achieve tens of millions of packets per second