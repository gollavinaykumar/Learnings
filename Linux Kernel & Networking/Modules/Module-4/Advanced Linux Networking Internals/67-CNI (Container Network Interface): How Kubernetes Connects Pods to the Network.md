Up until now, everything we've learned has been on a **single Linux machine**.

Docker creates:

```
Container

↓

veth Pair

↓

Linux Bridge

↓

Network Namespace
```

Simple.

But Kubernetes introduces a new problem.

Imagine:

```
Node 1

↓

Pod A
```

```
Node 2

↓

Pod B
```

Question:

How does Pod A communicate with Pod B?

Even more importantly...

Kubernetes itself **does not implement networking**.

It delegates networking to another standard.

That standard is:

# CNI (Container Network Interface)

CNI is one of the most important concepts in Kubernetes.

Without CNI:

- Pods cannot receive IP addresses.
- Pods cannot communicate.
- Services cannot work.
- Kubernetes networking would not exist.

After this chapter, you'll understand how Kubernetes connects every Pod to the network.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 67 — CNI (Container Network Interface): How Kubernetes Connects Pods to the Network

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Kubernetes Needs CNI
> - What is CNI?
> - CNI Architecture
> - CNI Plugins
> - IPAM
> - Bridge Plugin
> - Host-Local IPAM
> - Calico
> - Flannel
> - Cilium
> - Pod Networking
> - Complete Networking Flow

---

# 📖 Why Doesn't Kubernetes Handle Networking?

Imagine Kubernetes creates:

```
Pod

↓

Container
```

The container has:

```
No IP Address
```

```
No Route
```

```
No Network Namespace
```

Question:

Who should configure networking?

The Kubernetes developers decided:

```
Let's Define A Standard

↓

Let Others Implement It
```

That standard became:

```
CNI
```

---

# What is CNI?

CNI stands for:

```
Container Network Interface
```

It is **not** a networking solution.

It is:

```
A Specification

+

A Plugin System
```

Think of it as:

```
USB

↓

Standard
```

Many devices work because they follow USB.

Similarly,

many networking plugins work because they follow CNI.

---

# High-Level Architecture

```
Kubernetes

↓

CNI

↓

Plugin

↓

Linux Kernel
```

Kubernetes delegates networking to the CNI plugin.

---

# Think of a Driver

Imagine Windows.

```
Printer

↓

Printer Driver

↓

Windows
```

Windows doesn't know every printer.

It relies on drivers.

Kubernetes behaves the same way.

---

# Kubernetes Workflow

```
Create Pod

↓

Call CNI

↓

Configure Networking

↓

Pod Ready
```

Without successful CNI execution,

the Pod cannot start networking.

---

# What Does a CNI Plugin Do?

When Kubernetes creates a Pod,

the plugin typically performs tasks such as:

✔ Create a Network Namespace

✔ Create a `veth` pair

✔ Connect to a bridge or other networking backend

✔ Assign an IP address

✔ Configure routes

✔ Configure DNS information

---

# Pod Networking

Suppose:

```
Pod A
```

Plugin creates:

```
eth0

↓

veth

↓

Bridge
```

Exactly like Docker,

but managed according to Kubernetes networking requirements.

---

# IP Address Assignment

Every Pod needs:

```
Unique IP
```

Question:

Who assigns it?

Answer:

```
IPAM
```

---

# What is IPAM?

IPAM stands for:

```
IP Address Management
```

Its responsibility:

```
Available IP Pool

↓

Assign One

↓

Pod
```

---

# Host-Local IPAM

One common implementation:

```
Host-Local IPAM
```

Each Kubernetes node manages IP addresses for Pods running on that node.

---

# Bridge Plugin

Simplest CNI plugin:

```
Bridge
```

Architecture:

```
Pod

↓

veth

↓

Linux Bridge
```

Very similar to Docker bridge networking.

---

# Flannel

One of the earliest Kubernetes networking plugins.

Purpose:

```
Connect Pods

Across

Multiple Nodes
```

Flannel commonly uses VXLAN to build an overlay network.

---

# Flannel Architecture

```
Pod

↓

VXLAN

↓

Another Node

↓

Pod
```

Pods communicate as though they are on one network.

---

# Calico

Calico focuses on:

✔ High-performance networking

✔ Network Policies

✔ Layer-3 routing

Depending on configuration,

Calico can operate with or without overlays.

---

# Calico Example

```
Pod

↓

Routing

↓

Pod
```

Instead of relying solely on overlays,

Calico can distribute routing information between nodes.

---

# Cilium

Cilium is one of the most advanced CNI plugins.

Built using:

```
eBPF

+

XDP
```

Benefits include:

✔ Fast networking

✔ Security policies

✔ Observability

✔ Reduced dependence on large iptables rule sets

---

# Compare Plugins

| Plugin | Technology |
|---------|------------|
| Bridge | Linux Bridge |
| Flannel | VXLAN Overlay |
| Calico | Layer-3 Routing / Optional Overlay |
| Cilium | eBPF |

Each plugin uses different Linux networking technologies under the hood.

---

# Kubernetes Doesn't Care

From Kubernetes' perspective:

```
CNI

↓

Works?
```

If yes:

```
Pod Ready
```

The cluster can use any compliant CNI plugin.

---

# Pod-to-Pod Communication

Same Node:

```
Pod

↓

veth

↓

Bridge

↓

veth

↓

Pod
```

---

# Different Nodes

```
Pod

↓

CNI

↓

Physical Network

↓

CNI

↓

Pod
```

The CNI implementation determines exactly how packets travel.

---

# Kubernetes Networking Model

Kubernetes defines a simple networking model:

✔ Every Pod gets its own IP.

✔ Pods can communicate with other Pods without NAT.

✔ Nodes can communicate with Pods.

✔ Applications don't need to know whether Pods are on the same node.

The CNI plugin makes this possible.

---

# Packet Flow

```
Application

↓

eth0

↓

veth

↓

CNI Network

↓

Remote Node

↓

veth

↓

eth0

↓

Application
```

---

# Why CNI Matters

Without CNI:

```
Pods

↓

No IP

↓

No Network

↓

No Kubernetes
```

Networking is one of the first things a Kubernetes node configures.

---

# Hands-on Lab

## Lab 1 — View CNI Configuration

```bash
ls /etc/cni/net.d
```

Observe installed CNI configuration files.

---

## Lab 2 — View CNI Plugins

```bash
ls /opt/cni/bin
```

Observe available plugin binaries.

---

## Lab 3 — View Pod Interfaces

```bash
ip addr
```

On a Kubernetes node,

observe Pod-related interfaces.

---

## Lab 4 — View Routing

```bash
ip route
```

Observe Pod network routes.

---

# Interview Questions

## What is CNI?

CNI (Container Network Interface) is a specification that defines how container networking is configured.

---

## Does Kubernetes implement networking itself?

No.

Kubernetes delegates networking to CNI plugins.

---

## What is IPAM?

IP Address Management.

It assigns IP addresses to Pods.

---

## Name some popular CNI plugins.

- Bridge
- Flannel
- Calico
- Cilium

---

## What technology does Flannel commonly use?

VXLAN Overlay Networking.

---

## What technology is Cilium built around?

eBPF.

---

## What networking model does Kubernetes expect?

- Every Pod has its own IP.
- Pods can communicate directly with other Pods.
- Nodes can communicate with Pods.
- Applications do not need to know Pod locations.

---

# Summary

CNI provides the standard interface between Kubernetes and container networking.

```
Kubernetes

↓

CNI

↓

Plugin

↓

Linux Networking

↓

Pod
```

Key concepts:

- CNI is a specification, not a networking implementation.
- Kubernetes delegates networking to CNI plugins.
- CNI plugins create interfaces, assign IPs, and configure routes.
- IPAM assigns Pod IP addresses.
- Flannel, Calico, and Cilium implement networking in different ways.
- Kubernetes relies on CNI to satisfy its Pod networking model.

At this point, you've connected everything you've learned about Linux networking to Kubernetes.

You now understand how Pods obtain network connectivity using the same Linux primitives you've studied throughout this roadmap.

---

# Next Chapter

## Chapter 68 — Kubernetes Networking Internals: Following One Packet Across an Entire Cluster

We'll explore the complete journey of a packet:

- Application
- Pod Network Namespace
- `eth0`
- `veth`
- CNI Plugin
- Linux Bridge / eBPF
- Node Routing
- Overlay Network
- Remote Node
- kube-proxy
- Service
- Pod
- Reply Packet

> **This final chapter combines everything from TCP, routing, Netfilter, conntrack, CNI, eBPF, XDP, and Kubernetes into one complete end-to-end packet journey across a real Kubernetes cluster.**