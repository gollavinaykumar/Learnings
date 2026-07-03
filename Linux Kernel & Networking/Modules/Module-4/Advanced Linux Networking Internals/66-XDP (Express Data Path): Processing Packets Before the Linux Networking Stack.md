So far, every packet we've studied followed this path:

```
NIC

↓

Linux Networking Stack

↓

Netfilter

↓

Routing

↓

TCP/IP

↓

Socket

↓

Application
```

This works well.

But imagine a server receiving:

```
50 Million Packets

Per Second
```

Examples:

- Cloudflare
- Google
- AWS
- Meta
- Netflix

Question:

Should Linux fully process every packet,

only to discover later that it should be dropped?

That wastes:

- CPU
- Memory
- Kernel resources

Linux developers wanted something faster.

The solution is:

# XDP (Express Data Path)

XDP processes packets **immediately after they arrive at the network card**, before they travel through the normal Linux networking stack.

Combined with eBPF,

XDP can process millions of packets per second.

After this chapter, you'll understand how modern cloud providers defend against DDoS attacks and build ultra-high-performance networking.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 66 — XDP (Express Data Path): Processing Packets Before the Linux Networking Stack

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why XDP exists
> - What is XDP?
> - XDP Architecture
> - XDP Hook
> - XDP Actions
> - eBPF + XDP
> - High-Speed Packet Filtering
> - DDoS Protection
> - Packet Redirection
> - Complete Packet Flow

---

# 📖 Why Was XDP Introduced?

Imagine an attacker sends:

```
100 Million

Packets

Per Second
```

Traditional Linux flow:

```
NIC

↓

Kernel

↓

Netfilter

↓

Routing

↓

TCP

↓

Drop Packet
```

Linux already spent CPU time processing the packet.

Too late.

---

# The Problem

Every packet consumes resources.

Even packets that will eventually be dropped.

Examples:

- Invalid packets
- Attack traffic
- Unwanted protocols

Linux wanted to reject these packets as early as possible.

---

# What is XDP?

XDP stands for:

```
Express Data Path
```

It is a high-performance packet processing framework integrated into the Linux kernel.

It runs **before** the traditional networking stack.

---

# Think of Airport Security

Traditional:

```
Passenger

↓

Airport

↓

Immigration

↓

Security

↓

Rejected
```

Too much work.

XDP:

```
Passenger

↓

Gate Entrance

↓

Rejected
```

Problem solved immediately.

---

# Traditional Packet Flow

```
NIC

↓

Linux Stack

↓

Netfilter

↓

Routing

↓

Socket

↓

Application
```

---

# XDP Packet Flow

```
NIC

↓

XDP

↓

Linux Stack

↓

Application
```

XDP becomes the first decision point.

---

# Where Does XDP Run?

XDP executes:

```
Immediately

After

NIC Receives Packet
```

Before:

- Routing
- Netfilter
- conntrack
- TCP
- Sockets

---

# XDP Uses eBPF

Remember:

```
eBPF

↓

Kernel Programs
```

XDP programs are written as eBPF programs and attached to the XDP hook.

---

# Architecture

```
Packet

↓

NIC

↓

XDP Hook

↓

eBPF Program

↓

Decision
```

Everything happens extremely early.

---

# XDP Actions

An XDP program returns one of several actions.

The most common are:

```
PASS
```

```
DROP
```

```
TX
```

```
REDIRECT
```

---

# PASS

```
Packet

↓

Continue

↓

Linux Networking Stack
```

Normal packet processing continues.

---

# DROP

```
Packet

↓

Discard
```

Packet is dropped immediately.

The kernel avoids unnecessary processing.

---

# TX

```
Packet

↓

Transmit

↓

Same Interface
```

Often used for fast packet responses.

---

# REDIRECT

Packet can be redirected to:

- Another network interface
- Another CPU
- Another XDP program
- AF_XDP sockets

Without traversing the normal networking stack.

---

# Why Is XDP So Fast?

Traditional path:

```
NIC

↓

Kernel

↓

Routing

↓

Netfilter

↓

TCP

↓

Socket
```

XDP:

```
NIC

↓

XDP

↓

Done
```

Much less work.

---

# DDoS Protection

Suppose attackers send:

```
100 Million SYN Packets
```

Traditional firewall:

```
Kernel

↓

Process

↓

Drop
```

XDP:

```
NIC

↓

Drop Immediately
```

CPU usage drops dramatically.

---

# Packet Filtering

Suppose only:

```
TCP Port 443
```

is allowed.

XDP can inspect:

```
Ethernet

↓

IP

↓

TCP

↓

Port
```

Everything else is dropped immediately.

---

# Load Balancing

Suppose:

```
Incoming Packets
```

XDP can decide:

```
Server A
```

or

```
Server B
```

before packets enter the normal networking stack.

---

# Packet Statistics

XDP programs often update:

```
eBPF Maps
```

Example:

```
Packets

↓

Count

↓

Per Source IP
```

Used for monitoring and analytics.

---

# AF_XDP

Sometimes applications need to receive packets directly.

Linux provides:

```
AF_XDP
```

This allows user-space applications to exchange packets efficiently with XDP, bypassing much of the traditional networking stack.

---

# Cloud Providers

Large companies use XDP for:

- DDoS mitigation
- Load balancing
- Packet filtering
- Telemetry
- High-performance networking

Examples:

- Cloudflare
- Meta
- Google
- AWS

---

# Kubernetes

Projects like:

```
Cilium
```

can use XDP to accelerate networking and security.

This reduces dependence on iptables in some scenarios.

---

# XDP vs Netfilter

Netfilter:

```
Packet

↓

Kernel Networking Stack

↓

Hook
```

XDP:

```
Packet

↓

Immediately

↓

Decision
```

XDP happens first.

---

# Complete Flow

Without XDP:

```
NIC

↓

Netfilter

↓

Routing

↓

Socket

↓

Application
```

With XDP:

```
NIC

↓

XDP

↓

PASS

↓

Linux Stack

↓

Application
```

or

```
NIC

↓

XDP

↓

DROP
```

Done.

---

# Why XDP Matters

High-performance systems require:

✔ Early filtering

✔ Low latency

✔ High throughput

✔ Minimal CPU overhead

XDP provides all of these.

---

# Hands-on Lab

## Lab 1 — Check XDP Support

```bash
ip link show
```

Observe available network interfaces.

---

## Lab 2 — View eBPF Programs

```bash
sudo bpftool prog show
```

If XDP programs are attached,

they will appear here.

---

## Lab 3 — View Interface Details

```bash
ip -details link show
```

Some interfaces display XDP attachment information.

---

## Lab 4 — Explore Kernel Features

```bash
sudo bpftool feature
```

Check whether XDP is supported.

---

# Interview Questions

## What is XDP?

XDP (Express Data Path) is a high-performance packet processing framework that runs before the traditional Linux networking stack.

---

## Where does XDP execute?

Immediately after the NIC receives a packet, before routing, Netfilter, conntrack, or TCP processing.

---

## Why is XDP faster than Netfilter?

Because it processes or drops packets before they travel through the normal networking stack.

---

## What technology powers XDP programs?

eBPF.

---

## What are common XDP actions?

- PASS
- DROP
- TX
- REDIRECT

---

## Why is XDP useful for DDoS protection?

It can discard malicious packets almost immediately, greatly reducing CPU and kernel overhead.

---

## Which Kubernetes project makes extensive use of XDP?

Cilium.

---

# Summary

XDP is the earliest packet processing framework in Linux.

```
Packet

↓

NIC

↓

XDP Hook

↓

eBPF Program

↓

PASS

DROP

TX

REDIRECT

↓

Linux Networking Stack
```

Key concepts:

- XDP runs before the traditional networking stack.
- It is implemented using eBPF programs.
- Packets can be passed, dropped, transmitted, or redirected.
- XDP is widely used for DDoS protection and high-performance networking.
- It minimizes CPU usage by making decisions as early as possible.
- Cloud providers and modern Kubernetes networking solutions increasingly rely on XDP.

At this point, you've progressed from understanding traditional Linux networking to one of the fastest packet-processing technologies available in modern kernels.

---

# Next Chapter

## Chapter 67 — CNI (Container Network Interface): How Kubernetes Connects Pods to the Network

We'll explore:

- Why Kubernetes Needs CNI
- What is CNI?
- CNI Architecture
- CNI Plugins
- Bridge Plugin
- Host-Local IPAM
- Calico
- Flannel
- Cilium
- Pod Networking
- Complete Kubernetes Networking Architecture

> **This chapter bridges everything you've learned about Linux networking into Kubernetes networking, showing how Pods are connected using standard CNI plugins.**