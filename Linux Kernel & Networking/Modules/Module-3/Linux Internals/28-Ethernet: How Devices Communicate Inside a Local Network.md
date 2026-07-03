Most developers think networking starts with an IP address.

For example:

```
192.168.1.10
```

or

```
8.8.8.8
```

But here's the interesting part:

**IP cannot send anything by itself.**

Before an IP packet can travel,

it must first become an **Ethernet Frame**.

Questions you may have:

- What exactly is Ethernet?
- What is a MAC Address?
- Why do switches care about MAC addresses instead of IP addresses?
- Why does Docker create `veth` devices?
- What is MTU?
- What is Full Duplex?
- What is a Broadcast Domain?

The answer is:

**Ethernet.**

Ethernet is the foundation of almost every Local Area Network (LAN).

Every packet leaving your computer first becomes an Ethernet Frame.

After this chapter, you'll understand how devices communicate inside the same network.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 28 — Ethernet: How Devices Communicate Inside a Local Network

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is Ethernet?
> - Why Ethernet exists
> - Ethernet Frame
> - MAC Address
> - Network Interface Card (NIC)
> - Switch
> - Broadcast
> - Broadcast Domain
> - Collision Domain
> - Half Duplex
> - Full Duplex
> - MTU (Maximum Transmission Unit)
> - VLAN Basics
> - How Docker Uses Virtual Ethernet (`veth`)

---

# 📖 Why Do We Need Ethernet?

Imagine two computers.

```
Computer A

↓

Computer B
```

How do they exchange bits?

The Operating System needs a standard way to:

✔ Package data

✔ Identify devices

✔ Detect errors

✔ Send data across cables

That standard is:

```
Ethernet
```

---

# What is Ethernet?

Ethernet is a Layer 2 protocol.

It defines:

- Frames
- MAC Addresses
- Local communication
- Error detection

It works only inside the local network (LAN).

---

# Ethernet vs IP

Ethernet:

```
Local Network
```

IP:

```
Across Networks
```

Think of it like this:

```
Ethernet

↓

Street
```

```
IP

↓

City-to-City Road
```

---

# Ethernet Frame

Before transmission,

Linux wraps data inside an Ethernet Frame.

```
+--------------------------------------+

Destination MAC

Source MAC

EtherType

Payload

CRC

+--------------------------------------+
```

Everything sent on a LAN is transmitted as a frame.

---

# Ethernet Frame Fields

## Destination MAC

Who should receive this frame?

Example:

```
00:11:22:33:44:55
```

---

## Source MAC

Who sent this frame?

Example:

```
AA:BB:CC:DD:EE:FF
```

---

## EtherType

Indicates what the payload contains.

Examples:

```
0x0800

↓

IPv4
```

```
0x86DD

↓

IPv6
```

```
0x0806

↓

ARP
```

---

## Payload

Contains:

```
IP Packet

↓

TCP

↓

HTTP
```

This is the actual application data.

---

## CRC

CRC stands for:

```
Cyclic Redundancy Check
```

Used to detect transmission errors.

If corrupted,

the frame is discarded.

---

# MAC Address

Every Network Interface Card has a MAC Address.

Example:

```
3C:52:82:91:7A:2F
```

It uniquely identifies the interface on the local network.

---

# MAC vs IP

| MAC Address | IP Address |
|--------------|------------|
| Layer 2 | Layer 3 |
| Local Network | Across Networks |
| Burned into NIC (often) | Assigned by OS/DHCP |
| Used by Switch | Used by Router |

---

# Network Interface Card (NIC)

The NIC is the hardware responsible for sending and receiving Ethernet Frames.

```
Linux

↓

NIC Driver

↓

NIC

↓

Ethernet Cable
```

---

# Switch

Suppose four computers connect together.

```
PC1

↓

Switch

↑

PC2

↑

PC3

↑

PC4
```

The Switch forwards Ethernet Frames.

---

# How Does a Switch Know Where to Send Frames?

The switch builds a:

```
MAC Address Table
```

Example:

| MAC Address | Port |
|--------------|------|
| AA:BB | 1 |
| CC:DD | 2 |
| EE:FF | 3 |

Now it forwards frames only to the correct port.

---

# Switch Learning

Suppose a frame arrives.

```
Source MAC

↓

Store in MAC Table

↓

Remember Port
```

Over time,

the switch learns where every device is connected.

---

# Broadcast

Sometimes the sender doesn't know the destination.

It sends:

```
Broadcast Frame
```

Destination MAC:

```
FF:FF:FF:FF:FF:FF
```

Every device receives it.

---

# Broadcast Domain

A Broadcast Domain is the set of devices receiving a broadcast.

```
Switch

↓

PC1

PC2

PC3

PC4
```

One broadcast reaches everyone.

Routers stop broadcasts.

---

# Collision Domain

Older Ethernet used shared cables.

Two devices transmitting simultaneously caused:

```
Collision
```

Modern switches give each port its own collision domain.

Result:

```
No Shared Medium

↓

No Collisions
```

---

# Half Duplex

Only one direction at a time.

```
PC A

↓

PC B
```

Then:

```
PC B

↓

PC A
```

Like a walkie-talkie.

---

# Full Duplex

Both directions simultaneously.

```
PC A

↓

↑

PC B
```

Modern Ethernet operates in Full Duplex.

---

# MTU (Maximum Transmission Unit)

Ethernet has a maximum payload size.

Typical MTU:

```
1500 Bytes
```

If the IP packet exceeds the MTU,

it may need fragmentation or segmentation depending on the protocol and network path.

---

# Jumbo Frames

Some high-speed networks support:

```
9000 Byte MTU
```

Benefits:

✔ Less overhead

✔ Better throughput

Common in:

- Data Centers
- Storage Networks

---

# VLAN Basics

Suppose one switch serves two departments.

```
Engineering

↓

VLAN 10
```

```
Finance

↓

VLAN 20
```

Both share the same switch,

but behave as separate Layer 2 networks.

---

# Ethernet Flow

Suppose Chrome sends a request.

```
Chrome

↓

Socket

↓

TCP

↓

IP

↓

Ethernet Frame

↓

NIC

↓

Switch
```

Only then does it reach the network.

---

# Docker Example

Docker creates:

```
Container

↓

veth

↓

Linux Bridge

↓

Host NIC
```

A `veth` pair behaves like a virtual Ethernet cable.

Each end has its own MAC address.

From Linux's perspective,

containers communicate using normal Ethernet Frames.

---

# Real Example

Suppose:

```
Laptop

↓

Wi-Fi Router

↓

Desktop
```

Flow:

```
Ethernet Frame

↓

Switch

↓

Destination MAC

↓

Desktop
```

The switch never looks at the IP address.

It forwards using the MAC Address.

---

# Why Ethernet Matters

Everything inside a LAN depends on Ethernet.

Examples:

- Office Networks
- Home Wi-Fi
- Docker Bridges
- Kubernetes Nodes
- Cloud Servers
- Virtual Machines

Without Ethernet,

devices couldn't communicate locally.

---

# Hands-on Lab

## Lab 1 — View Network Interfaces

```bash
ip link
```

Observe MAC addresses.

---

## Lab 2 — View Detailed Interface Information

```bash
ip addr
```

---

## Lab 3 — View Link Statistics

```bash
ip -s link
```

---

## Lab 4 — View MTU

```bash
ip link show
```

Observe:

```
mtu 1500
```

---

## Lab 5 — View ARP Cache

```bash
ip neigh
```

Observe IP-to-MAC mappings.

---

## Lab 6 — Capture Ethernet Frames

```bash
sudo tcpdump -e -i any
```

Observe:

- Source MAC
- Destination MAC
- EtherType

---

# Interview Questions

## What is Ethernet?

Ethernet is a Layer 2 networking protocol used for communication between devices on the same local network.

---

## What is an Ethernet Frame?

An Ethernet Frame is the Layer 2 data unit containing source and destination MAC addresses, EtherType, payload, and an error-checking field (CRC).

---

## What is a MAC Address?

A MAC Address is a unique Layer 2 identifier assigned to a network interface.

---

## How does a switch forward frames?

A switch uses its MAC Address Table to determine which port should receive a frame.

---

## What is a Broadcast Frame?

A Broadcast Frame has the destination MAC address `FF:FF:FF:FF:FF:FF` and is delivered to every device within the same broadcast domain.

---

## What is MTU?

MTU (Maximum Transmission Unit) is the largest payload size that can be transmitted in a single Ethernet Frame without further fragmentation or segmentation at higher layers.

---

## Why does Docker create `veth` interfaces?

A `veth` pair acts as a virtual Ethernet cable, allowing containers to communicate using the Linux networking stack.

---

# Summary

Ethernet enables devices on the same local network to exchange frames efficiently.

```
Application

↓

Socket

↓

TCP

↓

IP

↓

Ethernet Frame

↓

NIC

↓

Switch

↓

Destination Device
```

Key concepts:

- Ethernet is the foundation of local networking.
- Ethernet Frames carry Layer 3 packets across a LAN.
- MAC Addresses uniquely identify network interfaces.
- Switches forward frames using MAC Address Tables.
- Broadcast Frames reach every device within a broadcast domain.
- Modern Ethernet uses Full Duplex communication.
- MTU defines the maximum payload size per frame.
- Docker uses virtual Ethernet (`veth`) devices to connect containers.

Understanding Ethernet prepares you to learn ARP, IP routing, Docker bridges, Kubernetes networking, and advanced packet flow through the Linux kernel.

---

# Next Chapter

## Chapter 29 — ARP (Address Resolution Protocol): How Linux Finds MAC Addresses

We'll explore:

- Why ARP exists
- IP vs MAC Address
- ARP Request
- ARP Reply
- ARP Cache
- Gratuitous ARP
- ARP Spoofing
- Proxy ARP
- Linux Neighbor Table
- Complete ARP Packet Flow