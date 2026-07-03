Most developers know this command:

```bash
ip route
```

Example output:

```text
default via 192.168.1.1 dev eth0

192.168.1.0/24 dev eth0
```

They know it somehow tells Linux where packets should go.

But almost nobody asks:

- How does Linux actually choose a route?
- Why does Linux always pick the "best" route?
- What is a routing table internally?
- What is a FIB?
- What is Longest Prefix Match?
- Why is `default` used only sometimes?
- How does Docker add routes?
- How does Kubernetes add routes?

The answer is:

**The Linux Routing Subsystem.**

Every single packet that leaves your computer goes through the routing subsystem.

Whether you're using:

- Docker
- Kubernetes
- AWS
- Azure
- Google Cloud
- VPN
- Linux Server

the kernel consults its routing tables before forwarding the packet.

After this chapter, you'll understand how Linux decides where every packet should go.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 59 — Linux Routing Table Internals: How Linux Decides Where Every Packet Goes

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Routing Exists
> - Linux Routing Table
> - FIB (Forwarding Information Base)
> - Default Route
> - Longest Prefix Match
> - Route Metrics
> - Gateway
> - Direct Routes
> - Docker Routes
> - Kubernetes Routes
> - Complete Routing Decision Flow

---

# 📖 Why Do We Need Routing?

Suppose your computer has:

```
IP

↓

192.168.1.100
```

You want to visit:

```
google.com

↓

142.250.190.14
```

Question:

```
How does Linux know

where

to send the packet?
```

The answer is:

```
Routing Table
```

---

# Imagine GPS

Routing is like Google Maps.

Suppose you want to travel to:

```
New York
```

Google Maps decides:

```
Road A

↓

Road B

↓

Destination
```

Linux does exactly the same for packets.

---

# Routing Table

Linux stores routes like:

```text
Destination

Gateway

Interface
```

Example:

```text
192.168.1.0/24 dev eth0

default via 192.168.1.1 dev eth0
```

---

# View Routes

Run:

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev wlan0

192.168.1.0/24 dev wlan0
```

---

# Direct Route

Suppose destination:

```
192.168.1.55
```

Host:

```
192.168.1.100
```

Both belong to:

```
192.168.1.0/24
```

Linux decides:

```
Direct Delivery
```

No router required.

---

# Visual Representation

```
Host

↓

Switch

↓

Destination
```

Same network.

No gateway.

---

# Different Network

Destination:

```
8.8.8.8
```

Host:

```
192.168.1.100
```

Different network.

Linux decides:

```
Gateway
```

---

# Default Route

Example:

```text
default via 192.168.1.1
```

Meaning:

```
If No Better Route Exists

↓

Send To

↓

192.168.1.1
```

---

# Gateway

Gateway is simply:

```
Next Router
```

Linux sends the packet there.

The router continues forwarding.

---

# Packet Flow

```
Application

↓

Routing Table

↓

Gateway

↓

Router

↓

Internet
```

---

# Routing Table Internally

Developers often think:

```
Routing Table

↓

Text File
```

Actually,

Linux stores routing information in efficient kernel data structures.

The routing subsystem performs fast lookups for every outgoing packet.

---

# What is the FIB?

Linux uses the:

```
Forwarding Information Base

(FIB)
```

The FIB is the kernel's forwarding database used during packet routing.

User commands such as:

```bash
ip route
```

display information managed by this subsystem.

---

# Longest Prefix Match

This is the most important routing algorithm.

Suppose routes:

```
10.0.0.0/8
```

```
10.10.0.0/16
```

```
10.10.20.0/24
```

Destination:

```
10.10.20.15
```

Linux chooses:

```
10.10.20.0/24
```

Because:

```
/24

>

 /16

>

 /8
```

The most specific route wins.

---

# Example

Routing Table:

```
10.0.0.0/8

↓

eth0
```

```
10.1.0.0/16

↓

eth1
```

Destination:

```
10.1.5.20
```

Linux selects:

```
eth1
```

because `/16` is more specific than `/8`.

---

# Why Not Use Default?

Suppose destination:

```
10.1.5.20
```

Linux already has a specific route.

It never uses:

```
default
```

Specific routes always take priority.

---

# Route Metric

Sometimes multiple routes match equally well.

Linux can compare:

```
Metric
```

Example:

```
Route A

Metric 100
```

```
Route B

Metric 200
```

Lower metric generally has higher priority.

---

# Multiple Interfaces

Suppose your computer has:

```
eth0

↓

Office LAN
```

```
wlan0

↓

Wi-Fi
```

Linux chooses routes based on:

- Destination
- Prefix Length
- Metric
- Policy Routing (when configured)

---

# Docker Routes

Suppose Docker creates:

```
172.17.0.0/16
```

Linux automatically adds:

```text
172.17.0.0/16 dev docker0
```

Now packets destined for containers use:

```
docker0
```

instead of the default gateway.

---

# Kubernetes Routes

Kubernetes commonly installs routes for Pod CIDRs.

Example:

```
10.244.0.0/24

↓

Node A
```

```
10.244.1.0/24

↓

Node B
```

The routing subsystem forwards packets toward the appropriate node.

---

# Policy Routing

Advanced Linux supports:

```
Multiple Routing Tables
```

Example:

```
Table 100

↓

VPN
```

```
Table 200

↓

Internet
```

Rules determine which table is consulted.

Large organizations often use this feature.

---

# Complete Routing Decision

```
Packet

↓

Destination IP

↓

Routing Lookup

↓

Longest Prefix Match

↓

Metric

↓

Gateway

↓

Interface

↓

NIC
```

---

# Why Routing Matters

Every packet uses routing.

Examples:

- Web Browsing
- Docker
- Kubernetes
- VPN
- AWS EC2
- Azure VM
- Linux Servers

Without routing,

packets never leave the machine.

---

# Hands-on Lab

## Lab 1 — View Routes

```bash
ip route
```

---

## Lab 2 — View Route for One Destination

```bash
ip route get 8.8.8.8
```

Observe the selected interface and gateway.

---

## Lab 3 — View Docker Routes

```bash
ip route
```

Look for:

```
docker0
```

---

## Lab 4 — Add Temporary Route

```bash
sudo ip route add 10.10.0.0/16 via 192.168.1.1
```

Then verify:

```bash
ip route
```

---

## Lab 5 — Remove Route

```bash
sudo ip route del 10.10.0.0/16
```

---

# Interview Questions

## What is a routing table?

A routing table contains rules that determine how packets are forwarded toward their destinations.

---

## What is the default route?

The default route is used only when no more specific route matches the destination.

---

## What is Longest Prefix Match?

Linux selects the route with the most specific matching network prefix.

---

## What is the FIB?

The Forwarding Information Base is the kernel's routing database used for forwarding packets.

---

## Why are route metrics used?

Metrics help choose between routes of equal specificity by assigning preference values.

---

## How does Docker modify routing?

Docker adds routes pointing container subnets toward interfaces such as `docker0`.

---

## Does every outgoing packet use the routing table?

Yes.

The Linux kernel performs a routing lookup before forwarding outgoing packets.

---

# Summary

The Linux routing subsystem determines where every outgoing packet should travel.

```
Application

↓

Destination IP

↓

Routing Table

↓

Longest Prefix Match

↓

Gateway

↓

Network Interface

↓

Internet
```

Key concepts:

- Linux consults its routing subsystem for every outgoing packet.
- The FIB stores forwarding information.
- Longest Prefix Match selects the most specific route.
- Default routes are fallbacks.
- Metrics help break ties.
- Docker and Kubernetes automatically install routes for container networks.
- Routing is one of the most fundamental operations performed by the Linux kernel.

Understanding Linux routing is essential before learning **Netfilter**, because routing and packet filtering work together during packet processing.

---

# Next Chapter

## Chapter 60 — Netfilter Architecture: The Packet Processing Engine Inside the Linux Kernel

We'll explore:

- What is Netfilter?
- Netfilter Hooks
- PREROUTING
- INPUT
- FORWARD
- OUTPUT
- POSTROUTING
- Packet Traversal
- Where NAT Happens
- Where Firewalls Work
- Complete Packet Processing Pipeline

> **This chapter explains the internal packet-processing engine that powers iptables, nftables, Docker networking, Kubernetes networking, NAT, and Linux firewalls.**