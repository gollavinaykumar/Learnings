Most developers know this command:

```bash
ip route
```

Example output:

```text
default via 192.168.1.1 dev eth0

192.168.1.0/24 dev eth0

10.0.0.0/24 dev docker0
```

Most people memorize it.

Very few understand what Linux actually does with it.

Suppose your application sends a packet to:

```
8.8.8.8
```

Questions you may have:

- How does Linux choose `eth0`?
- What if there are multiple network interfaces?
- What is the "default route"?
- What is "Longest Prefix Match"?
- What is the Linux FIB?
- How does Docker add routes?
- How does Kubernetes route Pod traffic?
- Can Linux have multiple routing tables?

After this chapter, you'll understand how Linux makes routing decisions for **every single outgoing packet**.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 31 — Routing: How Linux Chooses the Best Path for Every Packet

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Routing exists
> - Linux Routing Table
> - Longest Prefix Match
> - Default Route
> - Next Hop
> - Forwarding Information Base (FIB)
> - Multiple Network Interfaces
> - Static Routes
> - Dynamic Routing Overview
> - Policy-Based Routing
> - Docker & Kubernetes Routing
> - Complete Linux Routing Flow

---

# 📖 Why Do We Need Routing?

Suppose your laptop has:

```
Wi-Fi

↓

wlan0
```

and

```
Ethernet

↓

eth0
```

Now your browser sends a packet to:

```
8.8.8.8
```

Question:

```
Which Interface?

eth0 ?

wlan0 ?
```

Linux must decide.

That decision is called:

```
Routing
```

---

# What is Routing?

Routing is the process of selecting the best path for an IP packet.

```
Destination IP

↓

Routing Decision

↓

Outgoing Interface

↓

Next Hop
```

This happens for every outgoing packet.

---

# Linux Routing Flow

```
Application

↓

Socket

↓

TCP / UDP

↓

IP Packet

↓

Routing Table

↓

Outgoing Interface

↓

NIC

↓

Network
```

The Routing Table is consulted before the packet reaches Ethernet.

---

# What is a Routing Table?

Linux maintains a table describing available paths.

Example:

```
Destination

↓

Gateway

↓

Interface
```

Each entry tells Linux how to reach a network.

---

# Example Routing Table

```text
Destination        Gateway        Interface

192.168.1.0/24     Direct         eth0

10.0.0.0/24        Direct         docker0

default            192.168.1.1    eth0
```

Linux compares the destination IP against these entries.

---

# Direct Route

Suppose:

```
Destination

↓

192.168.1.25
```

Linux sees:

```
192.168.1.0/24

↓

eth0
```

Result:

```
Direct Delivery

↓

ARP

↓

Send Frame
```

No router required.

---

# Default Route

Suppose:

```
Destination

↓

8.8.8.8
```

No local route exists.

Linux uses:

```
default

↓

192.168.1.1
```

The packet is forwarded to the gateway.

---

# What is a Gateway?

A gateway is usually a router.

```
Laptop

↓

Gateway

↓

Internet

↓

Destination
```

The gateway forwards the packet toward its final destination.

---

# Longest Prefix Match

This is one of the most important routing concepts.

Suppose the Routing Table contains:

```
10.0.0.0/8
```

and

```
10.1.0.0/16
```

Destination:

```
10.1.2.3
```

Both routes match.

Linux chooses:

```
10.1.0.0/16
```

Why?

Because:

```
/16

↓

More Specific
```

Linux always selects the most specific matching route.

---

# Example

Routing Table:

```text
10.0.0.0/8

10.1.0.0/16

10.1.2.0/24
```

Destination:

```
10.1.2.15
```

Chosen route:

```
10.1.2.0/24
```

The longest prefix wins.

---

# Why Longest Prefix Match?

Imagine:

```
Company Network

↓

10.0.0.0/8
```

Engineering Department:

```
10.1.2.0/24
```

Packets for Engineering should follow a special route.

Longest Prefix Match makes this possible.

---

# Forwarding Information Base (FIB)

Internally,

Linux stores routing information inside:

```
FIB

Forwarding Information Base
```

Think of it as the optimized routing database used by the kernel.

---

# Routing Lookup

```
Destination IP

↓

FIB Lookup

↓

Best Route

↓

Next Hop

↓

Interface
```

Every outgoing packet performs this lookup.

---

# Multiple Network Interfaces

Suppose your server has:

```
eth0

↓

Internet
```

```
eth1

↓

Internal Network
```

Routing determines which interface each packet should use.

---

# Static Routes

A Static Route is manually configured.

Example:

```bash
ip route add 10.20.0.0/16 via 192.168.1.254
```

Linux immediately updates its routing table.

---

# Dynamic Routing

Large networks use routing protocols.

Examples:

```
OSPF

BGP

RIP

IS-IS
```

Instead of manually adding routes,

routers learn routes automatically.

Linux can participate using routing software such as FRRouting or BIRD.

---

# Policy-Based Routing

Sometimes destination IP alone is not enough.

Example:

```
Traffic From User A

↓

eth0
```

```
Traffic From User B

↓

VPN
```

Linux supports:

```
Policy-Based Routing
```

using multiple routing tables and rules.

---

# Multiple Routing Tables

Linux can maintain several routing tables.

```
Table 100

↓

Office Network
```

```
Table 200

↓

VPN
```

Rules decide which table to consult.

---

# Route Metrics

Suppose two routes exist.

```
Route A

Metric 100
```

```
Route B

Metric 50
```

Lower metric usually means:

```
Preferred Route
```

Metrics help Linux choose between otherwise equivalent routes.

---

# Packet Routing Example

Suppose Chrome sends:

```
https://google.com
```

Flow:

```
Chrome

↓

Socket

↓

TCP

↓

IP Packet

↓

FIB Lookup

↓

Default Route

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

Suppose Docker creates:

```
172.17.0.0/16

↓

docker0
```

Linux automatically adds:

```text
172.17.0.0/16 dev docker0
```

Packets destined for containers are routed through the Docker bridge.

---

# Kubernetes Example

Suppose a Pod network uses:

```
10.244.0.0/16
```

Linux adds routes pointing to the correct interfaces or tunnel devices.

Packets are routed to the appropriate Pod network.

---

# Why Routing Matters

Every network application depends on routing.

Examples:

- Docker
- Kubernetes
- VPNs
- Cloud Servers
- Databases
- Load Balancers
- Routers

Without routing,

packets would never leave the local network.

---

# Hands-on Lab

## Lab 1 — View Routing Table

```bash
ip route
```

---

## Lab 2 — View Detailed Routes

```bash
ip route show table main
```

---

## Lab 3 — Find Route to a Destination

```bash
ip route get 8.8.8.8
```

Observe:

- Interface
- Gateway
- Source IP

---

## Lab 4 — Add a Static Route

```bash
sudo ip route add 10.20.0.0/16 via 192.168.1.254
```

(Remove it afterward if appropriate.)

---

## Lab 5 — Delete the Route

```bash
sudo ip route del 10.20.0.0/16
```

---

## Lab 6 — View Routing Rules

```bash
ip rule
```

Observe policy routing rules.

---

# Interview Questions

## What is Routing?

Routing is the process of selecting the best path for an IP packet to reach its destination.

---

## What is the purpose of the Routing Table?

The Routing Table tells Linux how to reach different destination networks.

---

## What is the Default Route?

The Default Route is used when no more specific route matches the destination.

---

## What is Longest Prefix Match?

When multiple routes match a destination, Linux selects the route with the most specific network prefix.

---

## What is the FIB?

The Forwarding Information Base (FIB) is the kernel's optimized routing database used during packet forwarding.

---

## What is Policy-Based Routing?

Policy-Based Routing allows Linux to select routing tables based on policies such as source address, incoming interface, or packet marks—not just the destination IP.

---

## How does Docker use Linux routing?

Docker creates routes for container networks (such as `docker0`) so packets destined for containers are forwarded through the appropriate virtual interface.

---

# Summary

Linux determines the best path for every outgoing packet before transmission.

```
Application

↓

Socket

↓

TCP / UDP

↓

IP Packet

↓

FIB Lookup

↓

Longest Prefix Match

↓

Gateway

↓

Ethernet

↓

NIC

↓

Network
```

Key concepts:

- Routing selects the path for IP packets.
- Linux stores routes in the Routing Table and FIB.
- Direct routes are used for local networks.
- The Default Route handles unknown destinations.
- Longest Prefix Match chooses the most specific route.
- Metrics help select between similar routes.
- Policy-Based Routing enables advanced routing decisions.
- Docker and Kubernetes rely heavily on Linux routing.

Understanding routing prepares you for the next layer of networking—**TCP**, where reliable end-to-end communication is established over the routes selected by IP.

---

# Next Chapter

## Chapter 32 — TCP: Reliable Communication Across the Internet

We'll explore:

- Why TCP exists
- TCP Segment Structure
- Three-Way Handshake
- Four-Way Connection Termination
- Sequence Numbers
- Acknowledgments (ACKs)
- Sliding Window
- Flow Control
- Congestion Control
- Retransmissions
- TCP State Machine
- How Linux Implements TCP