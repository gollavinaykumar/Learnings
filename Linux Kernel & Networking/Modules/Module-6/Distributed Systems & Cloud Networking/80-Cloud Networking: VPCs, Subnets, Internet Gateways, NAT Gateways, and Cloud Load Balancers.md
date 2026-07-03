Before cloud computing, companies built data centers like this:

```
Office Building

↓

Physical Servers

↓

Switches

↓

Routers

↓

Firewalls
```

Everything was physical.

If the company needed:

```
100 More Servers
```

They had to:

- Buy hardware
- Wait for delivery
- Rack the servers
- Connect cables
- Configure networking

It could take weeks.

Cloud providers changed everything.

Today, you can create:

```
100 Servers

↓

100 Networks

↓

100 Databases
```

in just a few minutes.

But here's the question:

If millions of AWS customers use the same physical infrastructure...

**How does AWS keep everyone's network isolated?**

The answer is:

# Virtual Cloud Networking

Cloud providers build **virtual networks** on top of physical infrastructure.

Understanding cloud networking is essential because almost every modern application runs in:

- AWS
- Microsoft Azure
- Google Cloud Platform (GCP)

After this chapter, you'll understand how cloud networking works internally.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 80 — Cloud Networking: VPCs, Subnets, Internet Gateways, NAT Gateways & Cloud Load Balancers

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Cloud Networking Exists
> - What is a VPC?
> - CIDR Blocks
> - Public vs Private Subnets
> - Route Tables
> - Internet Gateway (IGW)
> - NAT Gateway
> - Security Groups
> - Network ACLs
> - Cloud Load Balancers
> - AWS, Azure & GCP Networking
> - Complete Cloud Network Architecture

---

# 📖 Why Cloud Networking Exists

Imagine AWS owns one giant data center.

Inside:

```
Customer A

Customer B

Customer C

Customer D
```

Question:

How do they prevent:

```
Customer A

↓

Accessing

↓

Customer B
```

They create isolated virtual networks.

---

# What is a VPC?

VPC stands for:

```
Virtual Private Cloud
```

Think of it as:

```
Your Own Private Network

Inside

AWS
```

Even though the hardware is shared,

your network is isolated from everyone else's.

---

# Think of an Apartment Building

Many families live in one building.

Each family has:

- Their own apartment
- Their own keys
- Their own rooms

The building is shared,

but each apartment is isolated.

A VPC works similarly.

---

# CIDR Block

Every VPC receives an IP range.

Example:

```
10.0.0.0/16
```

This defines the addresses available inside the VPC.

---

# Visual Representation

```
AWS

↓

VPC

↓

10.0.0.0/16
```

Everything inside the VPC uses addresses from that range.

---

# Subnets

Instead of one huge network,

divide it into smaller parts.

Example:

```
10.0.1.0/24
```

```
10.0.2.0/24
```

```
10.0.3.0/24
```

Each subnet represents a smaller network.

---

# Public Subnet

A Public Subnet has a route to the Internet.

Typical resources:

- Web Servers
- API Gateways
- Bastion Hosts
- Public Load Balancers

---

# Private Subnet

Private Subnets do **not** accept direct Internet traffic.

Typical resources:

- Databases
- Internal APIs
- Redis
- Kafka
- Backend Services

Safer.

---

# Architecture

```
Internet

↓

Public Subnet

↓

Private Subnet
```

Sensitive services stay private.

---

# Route Table

Every subnet has:

```
Route Table
```

It decides:

```
Destination

↓

Next Hop
```

Just like Linux routing tables.

---

# Example

Destination:

```
0.0.0.0/0
```

↓

Internet Gateway

Destination:

```
10.0.0.0/16
```

↓

Local

---

# Internet Gateway (IGW)

An Internet Gateway connects a VPC to the Internet.

Without it:

```
Public Server

↓

No Internet
```

Even if it has a public IP.

---

# Internet Flow

```
Internet

↓

Internet Gateway

↓

Public Subnet

↓

EC2
```

---

# NAT Gateway

Suppose:

```
Database

↓

Private Subnet
```

Needs to:

- Download updates
- Install packages
- Call external APIs

Question:

How can it access the Internet **without allowing inbound Internet connections?**

Solution:

```
NAT Gateway
```

---

# NAT Flow

```
Private Server

↓

NAT Gateway

↓

Internet
```

Responses come back.

But external users cannot directly initiate connections to the private server.

---

# Why NAT?

Private resources can:

✔ Download software

✔ Access APIs

✔ Install updates

But they remain inaccessible from the public Internet.

---

# Security Groups

Security Groups behave like:

```
Virtual Firewall
```

Applied to individual resources.

Rules define:

- Allowed inbound traffic
- Allowed outbound traffic

---

# Example

Allow:

```
TCP 443
```

Allow:

```
TCP 22
```

Block everything else.

---

# Network ACL (NACL)

Network ACLs also filter traffic,

but at the subnet level.

Difference:

Security Group:

```
Instance Level
```

Network ACL:

```
Subnet Level
```

---

# Compare

| Security Group | Network ACL |
|----------------|-------------|
| Instance-level | Subnet-level |
| Stateful | Stateless |
| Easier to manage | More granular network filtering |

---

# Cloud Load Balancer

Traffic enters:

```
Internet

↓

Load Balancer

↓

Multiple Servers
```

Just like traditional load balancing,

but managed by the cloud provider.

---

# Auto Scaling

Suppose CPU reaches:

```
90%
```

Cloud automatically creates:

```
New Server
```

Load Balancer immediately starts sending traffic to it.

---

# Complete Architecture

```
Internet

↓

Internet Gateway

↓

Public Subnet

↓

Load Balancer

↓

Private Subnet

↓

Application Servers

↓

Database
```

This is one of the most common production architectures.

---

# AWS Networking

Key AWS services:

- VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs
- Elastic Load Balancer (ELB)

---

# Azure Networking

Equivalent concepts:

- Virtual Network (VNet)
- Subnets
- Route Tables
- Network Security Groups (NSGs)
- Azure Load Balancer
- Azure NAT Gateway

---

# Google Cloud Networking

Equivalent concepts:

- VPC
- Subnets
- Firewall Rules
- Cloud NAT
- Cloud Load Balancer

---

# Cloud Networking Flow

```
Client

↓

Internet

↓

Cloud Load Balancer

↓

Public Subnet

↓

Application

↓

Database

↓

Response
```

---

# Why Cloud Networking Matters

Without virtual networking:

```
Millions

Of Customers

↓

One Shared Network
```

Impossible to secure.

With VPCs:

```
Each Customer

↓

Own Private Network
```

Safe.

Scalable.

Isolated.

---

# Hands-on Lab

## Lab 1 — View Local Routing

On a Linux machine:

```bash
ip route
```

Compare Linux routing concepts with cloud route tables.

---

## Lab 2 — Draw a VPC

Sketch:

- One VPC
- Two Public Subnets
- Two Private Subnets
- Internet Gateway
- NAT Gateway
- Load Balancer
- Database

Visualizing the architecture helps reinforce the concepts.

---

## Lab 3 — Explore a Cloud Console

Create a free-tier account (where available) and inspect:

- VPCs
- Subnets
- Route Tables
- Security Groups

Observe how the concepts map to the cloud UI.

---

# Interview Questions

## What is a VPC?

A Virtual Private Cloud is an isolated virtual network within a cloud provider.

---

## Why are Subnets used?

To divide a VPC into smaller logical networks for security, scalability, and organization.

---

## What is the difference between Public and Private Subnets?

Public Subnets have routes to the Internet.

Private Subnets do not allow direct Internet access.

---

## What is an Internet Gateway?

A component that enables communication between a VPC and the public Internet.

---

## Why is a NAT Gateway needed?

It allows private resources to initiate outbound Internet connections while preventing unsolicited inbound connections.

---

## What is the difference between a Security Group and a Network ACL?

Security Groups are stateful and apply to individual instances.

Network ACLs are stateless and apply to entire subnets.

---

## Why do cloud providers use VPCs?

To isolate each customer's network while sharing the same physical infrastructure.

---

# Summary

Cloud providers virtualize networking using isolated software-defined networks.

```
Internet

↓

Internet Gateway

↓

Public Subnet

↓

Load Balancer

↓

Private Subnet

↓

Application

↓

Database
```

Key concepts:

- VPCs provide isolated virtual networks.
- Subnets divide networks into logical segments.
- Internet Gateways connect public resources to the Internet.
- NAT Gateways allow secure outbound Internet access from private resources.
- Security Groups and Network ACLs protect resources.
- Cloud Load Balancers distribute traffic.
- AWS, Azure, and Google Cloud all implement similar networking concepts with different names.

---

# 🎉 Module 12 Complete!

You have now mastered the foundations of **Distributed Systems & Cloud Networking**.

You now understand:

✅ Distributed Systems

✅ CAP Theorem

✅ Consistency Models

✅ Leader Election

✅ Raft Consensus

✅ Service Discovery

✅ API Gateways

✅ Load Balancers

✅ Reverse Proxies

✅ Service Mesh

✅ HTTP/2 & gRPC

✅ Cloud Networking (VPCs, Subnets, Gateways, Security)

---

# 🚀 Next Module

# Module 13 — Cloud Infrastructure & Kubernetes Production Mastery

In this module, we'll move from understanding cloud networking to operating **production-grade Kubernetes and cloud infrastructure**.

Upcoming chapters include:

- **Chapter 81** — Kubernetes Control Plane Internals
- **Chapter 82** — kube-apiserver Deep Dive
- **Chapter 83** — etcd Internals
- **Chapter 84** — kube-scheduler Internals
- **Chapter 85** — kube-controller-manager
- **Chapter 86** — kubelet Internals
- **Chapter 87** — Container Runtime Interface (CRI)
- **Chapter 88** — CSI (Container Storage Interface)
- **Chapter 89** — Kubernetes Storage Deep Dive
- **Chapter 90** — Production Kubernetes Architecture

> **This module focuses on what happens inside a Kubernetes cluster after you run `kubectl apply`, covering the complete lifecycle from the API server to scheduling, storage, controllers, and production deployment.**