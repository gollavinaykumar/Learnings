Most developers know how to send an HTTP request.

```javascript
fetch("https://google.com")
```

```java
HttpClient.send()
```

```go
http.Get()
```

The request works.

But almost nobody asks:

- How does the packet leave my application?
- How does Linux know which network card to use?
- What happens inside the kernel?
- Where does TCP actually live?
- Where is IP implemented?
- What is a socket internally?
- How does Docker intercept packets?
- How does Kubernetes route traffic?

The answer is:

**The Linux Networking Stack.**

Every network packet entering or leaving your computer travels through this stack.

Understanding it is the foundation for:

- TCP/IP
- Docker Networking
- Kubernetes Networking
- Firewalls
- VPNs
- eBPF
- High-performance networking

After this chapter, you'll understand the complete journey of a packet through the Linux kernel.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 27 — Linux Networking Stack: From Ethernet Frame to User Application

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Linux needs a Networking Stack
> - OSI vs TCP/IP
> - Linux Network Stack Layers
> - Network Interface Card (NIC)
> - Ethernet Frames
> - ARP
> - IP
> - Routing
> - TCP
> - UDP
> - Sockets
> - Socket Buffers (`sk_buff`)
> - Packet Journey Inside Linux
> - Docker & Kubernetes Networking Overview

---

# 📖 Why Do We Need a Networking Stack?

Suppose Chrome wants to open:

```text
https://google.com
```

Chrome knows nothing about:

- Ethernet
- ARP
- IP
- Routing
- Network Cards

It simply says:

```
Connect to google.com
```

Linux handles everything else.

---

# The Networking Stack

Think of the networking stack as a delivery company.

```
Application

↓

Linux Networking Stack

↓

Network Card

↓

Internet
```

Applications send data.

The kernel delivers it.

---

# The Complete Packet Journey

When you send data:

```
Application

↓

Socket

↓

TCP / UDP

↓

IP

↓

Ethernet

↓

Network Driver

↓

NIC

↓

Cable / Wi-Fi

↓

Internet
```

Receiving data follows the reverse path.

---

# Why Layers?

Imagine every application implemented TCP itself.

Chrome:

```
Own TCP
```

PostgreSQL:

```
Own TCP
```

Docker:

```
Own TCP
```

Thousands of duplicate implementations.

Instead,

Linux provides one networking stack for everyone.

---

# Linux Networking Layers

```
Application Layer

↓

Socket Layer

↓

Transport Layer

↓

Network Layer

↓

Link Layer

↓

Device Driver

↓

Hardware
```

Each layer has one responsibility.

---

# Application Layer

Applications include:

```
Chrome

↓

Nginx

↓

Redis

↓

Docker

↓

PostgreSQL
```

Applications only use:

```
Sockets
```

They never touch Ethernet or IP directly.

---

# Socket Layer

A socket is the application's connection to the networking stack.

```
Application

↓

Socket

↓

Kernel
```

Think of it as a file descriptor for networking.

---

# Transport Layer

Linux supports two main transport protocols.

```
TCP
```

Reliable.

Ordered.

Connection-oriented.

---

```
UDP
```

Fast.

Connectionless.

No delivery guarantee.

---

# Network Layer

Responsible for:

```
IP Address

↓

Routing

↓

Fragmentation

↓

Forwarding
```

This layer decides where packets should go.

---

# Link Layer

Responsible for communication within the local network.

Uses:

```
Ethernet

↓

MAC Addresses
```

or

```
Wi-Fi
```

---

# Device Driver

Every Network Card has a driver.

```
Kernel

↓

NIC Driver

↓

Network Card
```

The driver understands the hardware.

---

# Network Interface Card (NIC)

The NIC is the physical hardware.

Examples:

```
Ethernet Port
```

```
Wi-Fi Adapter
```

The NIC converts packets into electrical or radio signals.

---

# Outgoing Packet Flow

Suppose:

```javascript
fetch("https://google.com")
```

Internally:

```
Application

↓

Socket

↓

TCP

↓

IP

↓

Ethernet

↓

Driver

↓

NIC

↓

Internet
```

---

# Incoming Packet Flow

A response arrives.

```
Internet

↓

NIC

↓

Driver

↓

Ethernet

↓

IP

↓

TCP

↓

Socket

↓

Application
```

---

# Ethernet Frame

The Link Layer wraps data inside an Ethernet Frame.

```
+----------------------+

Destination MAC

Source MAC

EtherType

Payload

CRC

+----------------------+
```

This frame travels across the local network.

---

# What is a MAC Address?

Every NIC has a unique hardware address.

Example:

```
00:1A:2B:3C:4D:5E
```

MAC addresses identify devices on the same LAN.

---

# IP Packet

Inside the Ethernet Frame is usually an IP Packet.

```
+----------------------+

Source IP

Destination IP

TTL

Protocol

Payload

+----------------------+
```

IP identifies hosts across networks.

---

# ARP

Suppose Linux knows:

```
Destination IP

↓

192.168.1.10
```

But Ethernet requires:

```
Destination MAC
```

Linux asks:

```
Who owns

192.168.1.10 ?
```

This is:

```
ARP

Address Resolution Protocol
```

---

# ARP Flow

```
Need MAC

↓

ARP Broadcast

↓

Target Replies

↓

Store MAC

↓

Send Packet
```

Linux caches ARP results.

---

# Routing

Suppose:

```
Destination

↓

8.8.8.8
```

Linux checks:

```
Routing Table
```

Question:

```
Which Interface?

↓

eth0?

↓

wlan0?

↓

docker0?
```

Routing determines the next hop.

---

# Routing Table Example

```
Destination

↓

Gateway

↓

Interface
```

Linux evaluates routes before sending every packet.

---

# TCP

TCP provides:

✔ Reliable Delivery

✔ Ordered Delivery

✔ Retransmissions

✔ Flow Control

Applications like:

- HTTP
- SSH
- PostgreSQL

often use TCP.

---

# UDP

UDP provides:

✔ Fast Delivery

✔ Low Overhead

No guarantee of:

- Delivery
- Ordering
- Retransmission

Used by:

- DNS
- Video Streaming
- Online Games

---

# Socket Buffers (`sk_buff`)

Internally,

Linux stores packets inside:

```
sk_buff

Socket Buffer
```

Think of it as:

```
Packet Object
```

The kernel attaches metadata to every packet.

---

# `sk_buff` Journey

```
NIC

↓

Driver

↓

sk_buff

↓

IP

↓

TCP

↓

Socket

↓

Application
```

The same packet object moves through the networking stack.

---

# Why `sk_buff` Matters

Almost every networking subsystem uses it.

Examples:

- Routing
- Firewall
- NAT
- Traffic Control
- Docker
- Kubernetes

Understanding `sk_buff` is essential for kernel networking.

---

# Packet Processing

Outgoing:

```
Application

↓

Socket

↓

TCP

↓

IP

↓

Ethernet

↓

Driver

↓

NIC
```

Incoming:

```
NIC

↓

Driver

↓

Ethernet

↓

IP

↓

TCP

↓

Socket

↓

Application
```

Every packet follows this path.

---

# Docker Example

Suppose:

```
Container

↓

HTTP Request
```

Packet travels:

```
Container

↓

veth

↓

Bridge

↓

Host Networking Stack

↓

NIC

↓

Internet
```

Docker relies entirely on the Linux networking stack.

---

# Kubernetes Example

Suppose a Pod sends traffic.

```
Pod

↓

veth

↓

Bridge

↓

iptables

↓

Routing

↓

NIC

↓

Internet
```

Again,

the Linux kernel performs the networking.

---

# Why This Matters

Everything depends on the Linux networking stack.

Examples:

- Docker
- Kubernetes
- Nginx
- HAProxy
- Redis
- PostgreSQL
- Chrome
- SSH
- VPNs

Understanding this stack makes advanced networking much easier.

---

# Hands-on Lab

## Lab 1 — View Network Interfaces

```bash
ip link
```

Observe:

```
eth0

lo

docker0

...
```

---

## Lab 2 — View IP Addresses

```bash
ip addr
```

---

## Lab 3 — View Routing Table

```bash
ip route
```

Observe default gateway and interfaces.

---

## Lab 4 — View ARP Cache

```bash
ip neigh
```

Observe IP-to-MAC mappings.

---

## Lab 5 — View Active Sockets

```bash
ss -tuln
```

Observe TCP and UDP sockets.

---

## Lab 6 — Capture Packets

```bash
sudo tcpdump -i any
```

Observe packets entering and leaving the system.

---

# Interview Questions

## What is the Linux Networking Stack?

The Linux Networking Stack is the collection of kernel components responsible for processing network traffic between applications and network hardware.

---

## What is the purpose of a Socket?

A Socket provides the interface through which applications communicate with the Linux networking stack.

---

## What is ARP?

ARP (Address Resolution Protocol) maps an IP address to a MAC address on the local network.

---

## What is the role of the Network Interface Card (NIC)?

The NIC converts packets into electrical or wireless signals for transmission and receives incoming signals from the network.

---

## What is `sk_buff`?

`sk_buff` is the Linux kernel's internal packet data structure used throughout the networking stack.

---

## What is the difference between TCP and UDP?

TCP provides reliable, ordered communication.

UDP provides fast, connectionless communication without delivery guarantees.

---

## Why do Docker and Kubernetes depend on the Linux Networking Stack?

Because containers and Pods ultimately send and receive packets using the host Linux kernel's networking implementation.

---

# Summary

The Linux Networking Stack moves packets between applications and the physical network.

```
Application

↓

Socket

↓

TCP / UDP

↓

IP

↓

Ethernet

↓

Driver

↓

NIC

↓

Internet
```

Key concepts:

- Applications communicate through sockets.
- Linux implements TCP, UDP, IP, and Ethernet inside the kernel.
- ARP resolves IP addresses to MAC addresses.
- Routing determines the outgoing interface and next hop.
- `sk_buff` represents packets inside the kernel.
- Device drivers communicate with the Network Interface Card.
- Docker and Kubernetes rely entirely on the Linux networking stack.

Understanding the Linux networking stack is the foundation for learning Ethernet, ARP, IP, TCP, firewalls, Docker networking, Kubernetes networking, and advanced kernel networking.

---

# Next Chapter

## Chapter 28 — Ethernet: How Devices Communicate Inside a Local Network

We'll explore:

- What is Ethernet?
- Ethernet Frames
- MAC Addresses
- Switches
- Broadcast Domains
- Collision Domains
- Full Duplex vs Half Duplex
- MTU
- VLAN Basics
- Frame Journey Through a Switch
- How Docker Uses Ethernet Internally (`veth`)