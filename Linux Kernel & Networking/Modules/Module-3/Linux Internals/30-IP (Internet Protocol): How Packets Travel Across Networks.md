Most developers think networking is simply:

```
Source IP

↓

Destination IP
```

For example:

```
192.168.1.10

↓

8.8.8.8
```

But here's the real question:

If your laptop is in India and the server is in the United States,

**how does the packet know where to go?**

Questions you may have:

- What exactly is an IP Address?
- Why do we need both IP and MAC?
- What is a Subnet Mask?
- What is CIDR?
- How does Linux know whether a destination is local or remote?
- What is the Default Gateway?
- What is TTL?
- Why do packets sometimes get fragmented?
- How does Linux choose the correct network interface?

The answer is:

**Internet Protocol (IP).**

IP is responsible for delivering packets across different networks.

After this chapter, you'll understand how packets travel from one computer to another anywhere in the world.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 30 — IP (Internet Protocol): How Packets Travel Across Networks

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why IP exists
> - IPv4 Packet Structure
> - IPv6 Overview
> - Public vs Private IP Addresses
> - Subnet Masks
> - CIDR Notation
> - Network ID vs Host ID
> - Default Gateway
> - Routing Decisions
> - TTL (Time To Live)
> - Fragmentation
> - Linux Routing Table
> - Complete IP Packet Journey

---

# 📖 Why Do We Need IP?

Imagine a company with two offices.

```
Office A

↓

Mumbai
```

```
Office B

↓

New York
```

Ethernet works only inside a Local Area Network.

It cannot send packets across cities or countries.

We need a protocol that can route packets between networks.

That protocol is:

```
IP

Internet Protocol
```

---

# Ethernet vs IP

Ethernet:

```
Same Local Network
```

IP:

```
Any Network

↓

Anywhere in the World
```

Ethernet delivers frames.

IP delivers packets.

---

# What is an IP Address?

An IP Address is a logical address assigned to a network interface.

Example:

```
192.168.1.10
```

Think of it like a postal address.

```
House Number

↓

IP Address
```

It tells routers where the packet should go.

---

# IPv4

IPv4 uses:

```
32 Bits
```

Written as:

```
192.168.1.10
```

Four numbers separated by dots.

Each number ranges from:

```
0

↓

255
```

---

# IPv6

IPv6 uses:

```
128 Bits
```

Example:

```
2001:db8::1
```

IPv6 provides a vastly larger address space than IPv4.

---

# IPv4 Packet

Linux wraps TCP or UDP data inside an IP Packet.

```
+--------------------------------------+

Version

Source IP

Destination IP

TTL

Protocol

Payload

+--------------------------------------+
```

---

# Source IP

Who sent the packet?

Example:

```
192.168.1.10
```

---

# Destination IP

Where should the packet go?

Example:

```
8.8.8.8
```

Routers forward packets based on this field.

---

# Protocol Field

Indicates what the payload contains.

Examples:

```
6

↓

TCP
```

```
17

↓

UDP
```

```
1

↓

ICMP
```

---

# Why Both MAC and IP?

Suppose:

```
Laptop

↓

Router

↓

Internet

↓

Server
```

MAC Addresses change at every local network segment.

IP Addresses remain the same from source to destination.

```
MAC

↓

Local Delivery
```

```
IP

↓

End-to-End Delivery
```

---

# Public vs Private IP

Private IP ranges:

```
10.0.0.0/8
```

```
172.16.0.0/12
```

```
192.168.0.0/16
```

Private addresses are used inside local networks.

Public IPs are routable across the Internet.

---

# Network ID vs Host ID

Suppose:

```
192.168.1.25/24
```

Network:

```
192.168.1.0
```

Host:

```
25
```

The subnet determines which part identifies the network and which part identifies the host.

---

# Subnet Mask

Example:

```
255.255.255.0
```

Binary:

```
11111111

11111111

11111111

00000000
```

The 1s represent the network portion.

The 0s represent the host portion.

---

# CIDR Notation

Instead of writing:

```
255.255.255.0
```

Linux uses:

```
/24
```

Examples:

```
/8

↓

255.0.0.0
```

```
/16

↓

255.255.0.0
```

```
/24

↓

255.255.255.0
```

CIDR is simpler and more flexible.

---

# Same Network?

Suppose:

```
Your IP

192.168.1.10/24
```

Destination:

```
192.168.1.80
```

Linux calculates:

```
Same Network?

↓

Yes
```

Then:

```
ARP

↓

Send Directly
```

---

# Different Network?

Destination:

```
8.8.8.8
```

Linux calculates:

```
Same Network?

↓

No
```

Then:

```
Default Gateway
```

---

# Default Gateway

The Default Gateway is usually your router.

Example:

```
192.168.1.1
```

Flow:

```
Laptop

↓

Gateway

↓

Internet
```

Linux forwards unknown networks to the gateway.

---

# Routing Table

Linux maintains a Routing Table.

View:

```bash
ip route
```

Example:

```
Destination

↓

Gateway

↓

Interface
```

Before sending every packet,

Linux checks this table.

---

# Example Routing Table

```
192.168.1.0/24

↓

eth0
```

```
default

↓

192.168.1.1

↓

eth0
```

Packets are matched against the most specific route.

---

# Routing Decision

Suppose:

```
Destination

↓

8.8.8.8
```

Linux:

```
Check Routing Table

↓

Find Best Route

↓

Choose Interface

↓

Send Packet
```

---

# TTL (Time To Live)

Every IP packet contains:

```
TTL
```

Example:

```
64
```

Each router decreases TTL by one.

```
64

↓

63

↓

62

↓

...
```

If TTL reaches:

```
0
```

The packet is discarded.

---

# Why TTL Exists

Imagine a routing loop.

```
Router A

↓

Router B

↓

Router C

↓

Router A
```

Without TTL,

the packet would circulate forever.

TTL prevents infinite loops.

---

# Fragmentation

Suppose:

```
IP Packet

↓

3000 Bytes
```

Network MTU:

```
1500 Bytes
```

The packet may need to be fragmented or otherwise handled according to network capabilities and protocol behavior.

Smaller packets are transmitted separately.

---

# IPv4 Fragmentation

```
Large Packet

↓

Fragment 1

↓

Fragment 2

↓

Fragment 3
```

The destination reassembles the fragments.

(Modern networks often try to avoid fragmentation using techniques such as Path MTU Discovery.)

---

# Packet Journey

Suppose Chrome requests:

```
https://example.com
```

Flow:

```
Chrome

↓

TCP

↓

IP Packet

↓

Routing Table

↓

Gateway

↓

Ethernet

↓

NIC

↓

Internet
```

---

# Docker Example

Suppose a container sends traffic.

```
Container

↓

Bridge

↓

Host Routing Table

↓

Gateway

↓

Internet
```

The host Linux kernel performs the routing decision.

---

# Kubernetes Example

A Pod sends traffic.

```
Pod

↓

veth

↓

Bridge

↓

Routing Table

↓

Gateway

↓

Internet
```

Again,

Linux IP routing handles the packet.

---

# Why IP Matters

Every Internet application depends on IP.

Examples:

- Web Browsers
- Docker
- Kubernetes
- Databases
- VPNs
- SSH
- Cloud Platforms

Without IP,

packets could never leave the local network.

---

# Hands-on Lab

## Lab 1 — View IP Addresses

```bash
ip addr
```

---

## Lab 2 — View Routing Table

```bash
ip route
```

Observe:

- Local routes
- Default route
- Gateway

---

## Lab 3 — View Default Gateway

```bash
ip route | grep default
```

---

## Lab 4 — Test Connectivity

```bash
ping 8.8.8.8
```

Observe successful IP communication.

---

## Lab 5 — Trace Packet Path

```bash
traceroute google.com
```

Observe routers decrementing TTL along the path.

---

## Lab 6 — View Interface Statistics

```bash
ip -s addr
```

---

# Interview Questions

## What is the purpose of IP?

IP provides logical addressing and routing so packets can travel between different networks.

---

## What is the difference between MAC and IP?

MAC addresses are used for local Layer 2 communication.

IP addresses are used for Layer 3 routing across networks.

---

## What is a Subnet Mask?

A Subnet Mask determines which portion of an IP address represents the network and which portion represents the host.

---

## What is CIDR?

CIDR (Classless Inter-Domain Routing) is a compact notation for representing network prefixes, such as `/24`.

---

## What is the Default Gateway?

The Default Gateway is the router that forwards packets destined for networks outside the local subnet.

---

## What is TTL?

TTL (Time To Live) limits how many routers a packet can traverse before it is discarded, preventing routing loops.

---

## Why does Linux consult the Routing Table before sending a packet?

The Routing Table determines the best outgoing interface and next hop for the destination IP address.

---

# Summary

The Internet Protocol enables communication between different networks.

```
Application

↓

TCP / UDP

↓

IP Packet

↓

Routing Table

↓

Gateway

↓

Ethernet

↓

NIC

↓

Internet
```

Key concepts:

- IP provides logical addressing and routing.
- IPv4 uses 32-bit addresses; IPv6 uses 128-bit addresses.
- Public and Private IP addresses serve different purposes.
- Subnet Masks and CIDR define network boundaries.
- The Routing Table determines the outgoing path.
- The Default Gateway forwards packets outside the local network.
- TTL prevents packets from looping indefinitely.
- Docker and Kubernetes rely on the Linux IP stack for routing.

Understanding IP prepares you to learn routing algorithms, TCP, UDP, NAT, firewalls, Docker networking, Kubernetes networking, and advanced Linux networking.

---

# Next Chapter

## Chapter 31 — Routing: How Linux Chooses the Best Path for Every Packet

We'll explore:

- What is Routing?
- Routing Tables
- Longest Prefix Match
- Default Routes
- Static vs Dynamic Routing
- Next Hop
- Multiple Network Interfaces
- Policy-Based Routing
- Linux FIB (Forwarding Information Base)
- Route Cache
- Complete Packet Routing Decision