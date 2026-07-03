Most developers know Docker creates something called:

```bash
docker0
```

You can see it by running:

```bash
ip addr
```

Example:

```text
docker0: <BROADCAST,MULTICAST,UP>
```

Many people think:

> "docker0 is just another network interface."

That's not true.

It is actually a **virtual Layer-2 Ethernet Switch** built into the Linux kernel.

Questions you may have:

- What is a Linux Bridge?
- Is it the same as a physical switch?
- How do containers communicate without a real switch?
- How does `docker0` work?
- How does Linux know which container should receive a frame?
- What is the Bridge Forwarding Database (FDB)?
- How does MAC learning work?

The answer is:

**Linux Bridge.**

The Linux Bridge is one of the core networking components used by Docker, Kubernetes, KVM, LXC, and many virtualization platforms.

After this chapter, you'll understand how multiple containers communicate through a virtual switch.

---

# 👑 Linux Kernel & Networking Mastery

# Module 5 — Linux Namespaces & Container Foundations

# Chapter 42 — Linux Bridge: The Virtual Switch Behind Docker Networking

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Linux Bridge exists
> - What is a Linux Bridge?
> - Bridge vs Physical Switch
> - Bridge Forwarding Database (FDB)
> - MAC Address Learning
> - Flooding
> - Broadcast Handling
> - Connecting Multiple `veth` Interfaces
> - `docker0`
> - Container-to-Container Packet Flow

---

# 📖 Why Do We Need a Bridge?

Suppose we have:

```
Container A
```

and

```
Container B
```

Each has:

```
Network Namespace

↓

veth
```

But they are not connected.

```
Container A

❌

Container B
```

We need something to connect multiple `veth` interfaces together.

That is the Linux Bridge.

---

# Physical Network

Imagine an office.

```
PC A

↓

Switch

↓

PC B

↓

PC C
```

The switch connects every computer.

---

# Virtual Network

Linux creates a software switch.

```
Container A

↓

veth

↓

Linux Bridge

↓

veth

↓

Container B
```

Exactly the same idea,

but entirely inside the Linux kernel.

---

# What is a Linux Bridge?

A Linux Bridge is a Layer-2 virtual Ethernet switch.

It forwards:

```
Ethernet Frames
```

Not IP packets.

It works using:

```
MAC Addresses
```

just like a physical switch.

---

# Bridge Architecture

```
Container A

↓

vethA

↓

Bridge

↓

vethB

↓

Container B
```

The bridge simply forwards frames between connected interfaces.

---

# Bridge Ports

Every connected interface becomes a:

```
Bridge Port
```

Example:

```
docker0

├── veth1

├── veth2

├── veth3
```

Each `veth` is attached as a port.

---

# Linux Bridge vs Switch

Physical Switch:

```
Hardware
```

Linux Bridge:

```
Software
```

Otherwise,

their forwarding behavior is very similar.

---

# Ethernet Frames

Remember:

A bridge operates using:

```
Destination MAC Address
```

It does **not** inspect:

- TCP
- UDP
- HTTP

Its primary concern is Ethernet forwarding.

---

# MAC Address Learning

Suppose:

```
Container A

↓

AA:AA
```

Sends a frame.

Bridge learns:

```
AA:AA

↓

Port 1
```

Now it remembers where that MAC address lives.

---

# Bridge Forwarding Database (FDB)

The bridge maintains:

```
FDB

Forwarding Database
```

Example:

| MAC Address | Port |
|-------------|------|
| AA:AA | veth1 |
| BB:BB | veth2 |
| CC:CC | veth3 |

This is similar to a physical Ethernet switch.

---

# Learning Process

Frame arrives:

```
Source MAC

↓

AA:AA
```

Bridge records:

```
AA:AA

↓

veth1
```

Future packets use this information.

---

# Packet Forwarding

Suppose:

```
Destination MAC

↓

BB:BB
```

Bridge checks:

```
FDB
```

Finds:

```
BB:BB

↓

veth2
```

Frame is forwarded only to:

```
veth2
```

---

# Unknown Destination

Suppose:

```
Destination MAC

↓

DD:DD
```

Bridge doesn't know where it is.

Result:

```
Flood
```

The frame is copied to every bridge port except the incoming one.

---

# Broadcast Frames

Broadcast destination:

```
FF:FF:FF:FF:FF:FF
```

Bridge forwards the frame to every connected port.

Examples:

- ARP Requests
- DHCP Discovery

---

# Broadcast Example

```
Container A

↓

ARP Request

↓

Bridge

↓

Container B

↓

Container C

↓

Container D
```

Every container receives the broadcast.

---

# Unicast Example

Suppose:

```
Container A

↓

Container C
```

Bridge already knows:

```
Container C

↓

veth3
```

Only:

```
veth3
```

receives the frame.

---

# Docker Bridge

Docker creates:

```
docker0
```

Architecture:

```
Container A

↓

veth

↓

docker0

↓

veth

↓

Container B
```

Every container joins the bridge.

---

# Docker Network

Example:

```
docker0

↓

172.17.0.1
```

Containers:

```
172.17.0.2

172.17.0.3

172.17.0.4
```

All communicate through:

```
docker0
```

---

# Packet Journey

Suppose Container A pings Container B.

Flow:

```
Container A

↓

Ethernet Frame

↓

vethA

↓

docker0

↓

FDB Lookup

↓

vethB

↓

Container B
```

No physical hardware involved.

---

# Bridge Does NOT Route

Important.

A bridge does **not** perform routing.

It only switches frames inside the same Layer-2 network.

Routing occurs later,

using routers or the Linux IP stack.

---

# Bridge + Router

Example:

```
Container

↓

Bridge

↓

Host Routing

↓

Internet
```

Bridge switches locally.

Routing sends packets to different networks.

---

# Kubernetes Example

Many Kubernetes CNI plugins create:

```
Pod

↓

veth

↓

Bridge

↓

Host
```

Pods communicate using the bridge before packets are routed elsewhere.

---

# Bridge Performance

Linux Bridge runs inside the kernel.

Benefits:

✔ Low Latency

✔ High Throughput

✔ No Physical Switch Required

It is efficient enough for thousands of containers.

---

# Inspecting the Bridge

View bridges:

```bash
ip link
```

Or:

```bash
bridge link
```

View forwarding database:

```bash
bridge fdb show
```

Observe learned MAC addresses.

---

# Why Linux Bridge Matters

Many systems depend on it.

Examples:

- Docker
- Kubernetes
- KVM
- QEMU
- LXC
- Podman

Without a bridge,

containers connected by `veth` pairs could not easily communicate on the same virtual LAN.

---

# Hands-on Lab

## Lab 1 — View Bridges

```bash
ip link
```

Look for:

```
docker0
```

(if Docker is installed.)

---

## Lab 2 — View Bridge Ports

```bash
bridge link
```

Observe attached `veth` interfaces.

---

## Lab 3 — View FDB

```bash
bridge fdb show
```

Observe MAC-to-port mappings.

---

## Lab 4 — View Docker Network

```bash
docker network inspect bridge
```

Observe:

- Bridge Name
- Subnet
- Connected Containers

---

## Lab 5 — Start Two Containers

```bash
docker run -dit --name c1 alpine sh
```

```bash
docker run -dit --name c2 alpine sh
```

Ping between them:

```bash
docker exec c1 ping <c2-ip>
```

Observe successful communication.

---

# Interview Questions

## What is a Linux Bridge?

A Linux Bridge is a software-based Layer-2 Ethernet switch implemented inside the Linux kernel.

---

## Does a Linux Bridge forward IP packets?

No.

It forwards Ethernet frames using MAC addresses.

IP routing happens separately.

---

## What is the FDB?

The Forwarding Database (FDB) stores MAC address to bridge-port mappings used for forwarding Ethernet frames.

---

## How does the bridge learn MAC addresses?

It observes the source MAC address of incoming frames and records which bridge port they arrived on.

---

## What happens if the destination MAC is unknown?

The bridge floods the frame to all ports except the one on which it was received.

---

## What is `docker0`?

`docker0` is the default Linux bridge created by Docker for the default bridge network.

---

## Why does Docker use a Linux Bridge?

To connect multiple container `veth` interfaces into the same virtual Layer-2 network.

---

# Summary

The Linux Bridge is a virtual Ethernet switch that connects multiple network namespaces together.

```
Container A

↓

veth

↓

Linux Bridge

↓

FDB Lookup

↓

veth

↓

Container B
```

Key concepts:

- A Linux Bridge is a Layer-2 switch.
- It forwards Ethernet frames using MAC addresses.
- The FDB maps MAC addresses to bridge ports.
- Unknown destinations are flooded.
- Broadcast frames are sent to all bridge ports.
- `docker0` is Docker's default Linux bridge.
- Docker and many Kubernetes networking solutions rely on Linux Bridge.

Understanding the Linux Bridge completes the core networking building blocks used by Docker: **Network Namespaces + `veth` + Linux Bridge**.

---

# Next Chapter

## Chapter 43 — Docker Bridge Network: How Docker Connects Containers to the Internet

We'll combine everything you've learned:

- Network Namespaces
- `veth` Pairs
- Linux Bridge (`docker0`)
- Routing
- NAT
- `iptables`
- `conntrack`
- Packet Flow from Container to Internet
- Container-to-Container Communication
- Container-to-Host Communication
- Complete Docker Networking Internals

> **This is where all previous Linux networking concepts come together into Docker networking.**