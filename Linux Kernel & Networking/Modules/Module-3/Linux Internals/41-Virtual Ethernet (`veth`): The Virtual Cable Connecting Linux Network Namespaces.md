Most developers know that Docker creates containers.

Very few know that for **every Docker container**, Linux usually creates something like:

```
veth7c12ab

↓

veth93fd10
```

You can even see them:

```bash
ip link
```

Questions you may have:

- What is a `veth` interface?
- Why are there always two of them?
- How does a packet travel through a `veth`?
- Is it like a physical Ethernet cable?
- Why does Docker create one for every container?
- How does Kubernetes use `veth`?

The answer is:

**Virtual Ethernet (veth).**

A `veth` pair is one of the most elegant networking mechanisms in the Linux kernel.

It acts as a **virtual Ethernet cable** connecting two network namespaces.

Without `veth`, Docker and Kubernetes networking would not exist.

After this chapter, you'll understand exactly how packets move between containers and the host.

---

# 👑 Linux Kernel & Networking Mastery

# Module 5 — Linux Namespaces & Container Foundations

# Chapter 41 — Virtual Ethernet (`veth`): The Virtual Cable Connecting Linux Network Namespaces

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why `veth` exists
> - What is a `veth` Pair?
> - Why `veth` Always Comes in Pairs
> - Packet Flow Through `veth`
> - Creating `veth`
> - Connecting Network Namespaces
> - `veth` + Linux Bridge
> - Docker's `veth` Architecture
> - Kubernetes `veth`
> - Complete Packet Journey

---

# 📖 Why Do We Need `veth`?

Suppose Linux creates:

```
Namespace A
```

and

```
Namespace B
```

Both are completely isolated.

```
Namespace A

❌ Cannot Reach

Namespace B
```

They need a connection.

Think of two houses.

Without a road,

they cannot communicate.

Linux solves this by creating a virtual cable.

---

# What is a `veth` Pair?

A `veth` pair is two virtual Ethernet interfaces permanently connected together.

```
vethA

====================

vethB
```

Whatever enters one end immediately appears on the other end.

---

# Why Does It Come in Pairs?

Unlike a normal NIC,

a `veth` always has:

```
End A

↓

End B
```

One interface cannot exist alone.

If one end is deleted,

the other disappears too.

---

# Think of a Network Cable

Imagine an Ethernet cable.

```
Laptop

========= Cable =========

Switch
```

A `veth` behaves exactly the same,

except both ends are virtual.

---

# Packet Flow

Suppose Namespace A sends:

```
Ping
```

Flow:

```
Namespace A

↓

vethA

====================

vethB

↓

Namespace B
```

The Linux kernel copies the Ethernet frame from one interface to the other.

---

# `veth` Is a Layer 2 Device

A `veth` transports:

```
Ethernet Frames
```

Not raw IP packets.

Everything inside the Ethernet frame is preserved.

---

# Inside the Kernel

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

vethA

====================

vethB

↓

Destination Namespace
```

To the networking stack,

it behaves like a normal Ethernet cable.

---

# Creating a `veth` Pair

Linux command:

```bash
ip link add vethA type veth peer name vethB
```

Result:

```
vethA

↓

Connected

↓

vethB
```

---

# Viewing Interfaces

```bash
ip link
```

Example:

```
vethA

vethB
```

---

# Moving an Interface

Suppose we want:

```
vethA

↓

Namespace red
```

Command:

```bash
ip link set vethA netns red
```

Now:

```
Host

↓

vethB
```

```
Namespace red

↓

vethA
```

---

# Assigning IP Addresses

Namespace:

```
10.0.0.1
```

Host:

```
10.0.0.2
```

Both interfaces behave like ordinary Ethernet adapters.

---

# Bringing Interfaces Up

Example:

```bash
ip link set vethB up
```

Inside namespace:

```bash
ip netns exec red ip link set vethA up
```

Interfaces must be **UP** before they can carry traffic.

---

# Simple Communication

```
Namespace

↓

10.0.0.1

↓

vethA

====================

vethB

↓

10.0.0.2

↓

Host
```

Now:

```bash
ping
```

works.

---

# Why Ethernet Frames?

Remember:

Linux networking expects:

```
Ethernet

↓

IP

↓

TCP

↓

Application
```

The `veth` device fits naturally into the Linux networking stack.

---

# Docker Architecture

Suppose Docker starts:

```
Container
```

Linux creates:

```
Container Namespace

↓

eth0
```

Actually:

```
Container eth0

↓

veth Pair

↓

Host
```

Inside the container,

one end is usually renamed:

```
eth0
```

Outside,

the other end has a random name such as:

```
veth9ab123
```

---

# Docker Packet Flow

```
Application

↓

eth0

↓

veth

↓

docker0 Bridge

↓

Host Routing

↓

NIC

↓

Internet
```

Every container follows this pattern.

---

# Kubernetes Architecture

Each Pod gets:

```
Pod Namespace

↓

eth0

↓

veth

↓

CNI Plugin

↓

Bridge

↓

Host
```

Different CNI plugins use the `veth` pair differently,

but almost all rely on it.

---

# One `veth` Per Container

Suppose:

```
Container A
```

```
Container B
```

```
Container C
```

Linux creates:

```
veth1

↓

Container A
```

```
veth2

↓

Container B
```

```
veth3

↓

Container C
```

Each container has its own dedicated connection.

---

# Performance

Because `veth` stays entirely inside the Linux kernel:

✔ No physical cable

✔ No hardware switch

✔ Very low latency

✔ High throughput

It is much faster than sending packets to a physical network and back.

---

# MTU

Like physical Ethernet interfaces,

`veth` interfaces have an MTU.

Example:

```bash
ip link show
```

Output:

```
mtu 1500
```

Docker and Kubernetes often adjust MTU values depending on overlays and encapsulation.

---

# Troubleshooting

Useful commands:

View interfaces:

```bash
ip link
```

View namespace interfaces:

```bash
ip netns exec red ip addr
```

Check peer information:

```bash
ethtool -S vethXXXXX
```

(On supported systems.)

---

# Real Example

Suppose Chrome inside a container sends:

```
HTTP Request
```

Flow:

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

eth0

↓

veth

↓

docker0

↓

Host Routing

↓

NIC

↓

Internet
```

Every packet follows this path.

---

# Why `veth` Matters

Everything in container networking depends on it.

Examples:

- Docker
- Kubernetes
- Podman
- containerd
- CRI-O
- Linux Containers (LXC)

Without `veth`,

Network Namespaces would remain isolated forever.

---

# Hands-on Lab

## Lab 1 — Create a `veth` Pair

```bash
sudo ip link add vethA type veth peer name vethB
```

---

## Lab 2 — View Interfaces

```bash
ip link
```

Observe:

```
vethA

vethB
```

---

## Lab 3 — Create a Namespace

```bash
sudo ip netns add red
```

---

## Lab 4 — Move One End

```bash
sudo ip link set vethA netns red
```

---

## Lab 5 — Assign IP Addresses

Host:

```bash
sudo ip addr add 10.0.0.2/24 dev vethB
```

Namespace:

```bash
sudo ip netns exec red ip addr add 10.0.0.1/24 dev vethA
```

---

## Lab 6 — Bring Interfaces Up

Host:

```bash
sudo ip link set vethB up
```

Namespace:

```bash
sudo ip netns exec red ip link set vethA up
sudo ip netns exec red ip link set lo up
```

Now test:

```bash
sudo ip netns exec red ping 10.0.0.2
```

---

# Interview Questions

## What is a `veth` pair?

A `veth` pair is a pair of virtual Ethernet interfaces permanently connected together. Packets entering one interface immediately exit through the other.

---

## Why does a `veth` always have two interfaces?

It represents two ends of a virtual Ethernet cable. Removing one end removes the other.

---

## What layer does `veth` operate at?

`veth` behaves like a Layer 2 Ethernet interface and carries Ethernet frames.

---

## Why does Docker create `veth` interfaces?

Docker uses a `veth` pair to connect a container's network namespace to the host networking infrastructure.

---

## Can two namespaces communicate without a `veth` or another connection mechanism?

No. Separate network namespaces are isolated until connected by devices such as a `veth` pair.

---

## Why is `veth` fast?

Because packets remain inside the Linux kernel without traversing physical networking hardware.

---

## How does Kubernetes use `veth`?

Most Kubernetes CNI plugins create a `veth` pair that connects the Pod's network namespace to the host networking stack.

---

# Summary

A `veth` pair acts as a virtual Ethernet cable connecting isolated network namespaces.

```
Namespace A

↓

eth0

↓

vethA

====================

vethB

↓

Bridge

↓

Host

↓

Internet
```

Key concepts:

- `veth` always exists as a pair.
- It transports Ethernet frames.
- One end is commonly placed inside a network namespace.
- The other remains in the host namespace or is attached to a bridge.
- Docker renames the namespace side to `eth0`.
- Kubernetes relies heavily on `veth` for Pod networking.
- `veth` provides fast, kernel-based packet forwarding.

Understanding `veth` prepares you for the next fundamental building block of Docker networking: the **Linux Bridge**, which connects many `veth` interfaces into a virtual switch.

---

# Next Chapter

## Chapter 42 — Linux Bridge: The Virtual Switch Behind Docker Networking

We'll explore:

- What is a Linux Bridge?
- Bridge vs Switch
- Bridge Forwarding Database (FDB)
- MAC Learning
- Connecting Multiple `veth` Interfaces
- Broadcast Handling
- Packet Forwarding
- `docker0` Bridge
- Kubernetes Bridge Networking
- Complete Container-to-Container Packet Flow