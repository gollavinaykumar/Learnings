Up to this point, you've learned:

✅ Network Namespaces

✅ `veth` Pairs

✅ Linux Bridge

✅ Routing

✅ NAT

✅ Connection Tracking

✅ TCP/IP

Now it's time to combine everything.

Most developers think Docker networking is simply:

```bash
docker run nginx
```

and somehow...

```
Container

↓

Internet
```

works automatically.

But what actually happens inside Linux?

Questions you may have:

- How does Docker create a network?
- Where does `docker0` come from?
- How does a container get an IP address?
- How does one container reach another?
- How does a container reach the Internet?
- How does the Internet send packets back?
- Why does `localhost` inside a container not mean the host?
- How does Docker publish ports?

The answer is:

**Docker is mostly orchestrating existing Linux kernel networking features.**

Docker itself does **not** implement a new networking stack.

It combines:

- Network Namespaces
- `veth`
- Linux Bridge
- Routing
- NAT
- Netfilter
- `conntrack`

After this chapter, you'll understand the complete networking architecture behind every Docker container.

---

# 👑 Linux Kernel & Networking Mastery

# Module 5 — Linux Namespaces & Container Foundations

# Chapter 43 — Docker Bridge Network: How Docker Connects Containers to the Internet

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Docker Bridge Network
> - `docker0`
> - Container IP Allocation
> - `veth` Creation
> - Bridge Connection
> - Container-to-Container Communication
> - Container-to-Host Communication
> - Container-to-Internet Communication
> - NAT & `iptables`
> - Complete Docker Packet Flow

---

# 📖 What Happens When Docker Starts?

Suppose you execute:

```bash
docker run nginx
```

Many developers think:

```
Container Started
```

Actually,

Docker performs many networking steps automatically.

---

# Docker Networking Overview

```
Docker CLI

↓

Docker Engine

↓

Linux Kernel

↓

Namespace

↓

veth

↓

Bridge

↓

Routing

↓

NAT

↓

Internet
```

Docker mainly configures Linux.

The Linux kernel does the networking.

---

# Step 1 — Create Network Namespace

Docker first creates:

```
Container

↓

Network Namespace
```

Now the container owns:

- Its own interfaces
- Routing table
- `localhost`
- Socket table

---

# Step 2 — Create `veth` Pair

Linux creates:

```
vethA

====================

vethB
```

One end stays on the host.

One end enters the container.

---

# Step 3 — Rename Inside Container

Inside the container:

```
vethA

↓

eth0
```

Applications only see:

```bash
eth0
```

The `veth` implementation is hidden.

---

# Step 4 — Connect to `docker0`

Host side:

```
vethB

↓

docker0 Bridge
```

Architecture:

```
Container

↓

eth0

↓

veth

↓

docker0
```

---

# Step 5 — Assign IP Address

Docker assigns:

Example:

```
172.17.0.2
```

Gateway:

```
172.17.0.1
```

The gateway is the bridge itself.

---

# Complete Architecture

```
Container

↓

eth0

↓

veth

↓

docker0

↓

Host Network

↓

NIC

↓

Internet
```

Every default Docker container follows this architecture.

---

# Container Routing Table

Inside the container:

```bash
ip route
```

Typical output:

```text
default via 172.17.0.1 dev eth0

172.17.0.0/16 dev eth0
```

Everything outside the local subnet goes to:

```
docker0
```

---

# Container to Container Communication

Suppose:

```
Container A

172.17.0.2
```

```
Container B

172.17.0.3
```

Flow:

```
Container A

↓

eth0

↓

veth

↓

docker0

↓

FDB Lookup

↓

veth

↓

Container B
```

No NAT required.

Same Layer-2 network.

---

# Why No NAT?

Both containers are inside:

```
172.17.0.0/16
```

Linux Bridge switches frames directly.

Routing outside the host is unnecessary.

---

# Container to Host Communication

Suppose:

```
Container

↓

172.17.0.2
```

Host:

```
172.17.0.1
```

Flow:

```
Container

↓

docker0

↓

Host
```

The host acts as the default gateway.

---

# Container to Internet

Suppose:

```
Container

↓

google.com
```

Flow:

```
Application

↓

TCP

↓

IP

↓

eth0

↓

veth

↓

docker0

↓

Host Routing

↓

SNAT

↓

NIC

↓

Internet
```

---

# Why SNAT?

Container IP:

```
172.17.0.2
```

Internet routers cannot route that address.

Linux changes it.

```
172.17.0.2

↓

203.0.113.25
```

Now the packet is routable.

---

# Reply Packet

Google replies:

```
203.0.113.25
```

Linux checks:

```
conntrack
```

Finds:

```
203.0.113.25

↓

172.17.0.2
```

Reverse translation occurs.

Packet reaches the correct container.

---

# Container to Internet Flow

```
Container

↓

eth0

↓

veth

↓

docker0

↓

Routing

↓

SNAT

↓

Internet

↓

Reply

↓

conntrack

↓

Container
```

---

# Why `localhost` Doesn't Work

Suppose:

```
Container

↓

localhost
```

That means:

```
127.0.0.1

↓

Inside Container
```

Not:

```
Host
```

Every Network Namespace has its own loopback interface.

---

# Publishing Ports

Suppose:

```bash
docker run -p 8080:80 nginx
```

Docker creates:

```
DNAT Rule
```

Flow:

```
Client

↓

Host:8080

↓

DNAT

↓

Container:80
```

The container never knows the client connected to port 8080.

It simply receives traffic on port 80.

---

# Docker Network Driver

Default driver:

```
bridge
```

View:

```bash
docker network ls
```

Output:

```
bridge

host

none
```

The default bridge uses:

```
docker0
```

---

# Docker DNS

Containers on the same user-defined bridge network can resolve each other by container name.

Example:

```
web

↓

172.18.0.2
```

```
redis

↓

172.18.0.3
```

Docker runs an embedded DNS service for these networks.

---

# Docker Inspect

View networking:

```bash
docker inspect nginx
```

Observe:

- IP Address
- Gateway
- MAC Address
- Network Name

---

# Complete Packet Journey

Suppose Chrome inside a container loads:

```
https://google.com
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

eth0

↓

veth

↓

docker0

↓

Routing

↓

SNAT

↓

NIC

↓

Internet

↓

Google

↓

Reply

↓

conntrack

↓

DNAT (reverse translation)

↓

docker0

↓

veth

↓

Container
```

Every packet follows roughly this path.

---

# Docker Doesn't Replace Linux

Docker uses:

✔ Network Namespaces

✔ `veth`

✔ Linux Bridge

✔ Routing

✔ Netfilter

✔ NAT

✔ Connection Tracking

The Linux kernel performs the actual packet forwarding.

Docker simply configures it.

---

# Why Docker Networking Matters

Every containerized application depends on it.

Examples:

- Nginx
- PostgreSQL
- Redis
- Node.js
- Java
- Go
- Python
- Kubernetes Nodes

Understanding Docker networking makes debugging dramatically easier.

---

# Hands-on Lab

## Lab 1 — View Docker Networks

```bash
docker network ls
```

---

## Lab 2 — Inspect Default Bridge

```bash
docker network inspect bridge
```

Observe:

- Subnet
- Gateway
- Connected Containers

---

## Lab 3 — View `docker0`

```bash
ip addr show docker0
```

---

## Lab 4 — Start Two Containers

```bash
docker run -dit --name c1 alpine sh
```

```bash
docker run -dit --name c2 alpine sh
```

---

## Lab 5 — Find Container IP

```bash
docker inspect c1
```

Locate:

```
IPAddress
```

---

## Lab 6 — Ping Between Containers

```bash
docker exec c1 ping <c2-ip>
```

Observe direct container-to-container communication.

---

## Lab 7 — Observe NAT Rules

```bash
sudo iptables -t nat -L
```

Observe Docker-created NAT rules.

---

# Interview Questions

## Does Docker implement its own networking stack?

No.

Docker configures existing Linux networking features such as Network Namespaces, `veth`, Linux Bridge, Routing, NAT, and Netfilter.

---

## What is `docker0`?

`docker0` is the default Linux bridge used by Docker's default bridge network.

---

## Why does every container have its own IP address?

Each container runs in its own Network Namespace with its own virtual Ethernet interface.

---

## How do containers communicate with each other on the default bridge?

Packets travel through the Linux Bridge (`docker0`) using MAC-based switching.

---

## Why is NAT required for Internet access?

Container IP addresses are private and not routable on the Internet, so the host performs Source NAT before packets leave.

---

## Why doesn't `localhost` inside a container refer to the host?

Because every container has its own loopback interface within its own Network Namespace.

---

## How does `docker run -p 8080:80` work?

Docker installs DNAT rules so traffic arriving at host port 8080 is redirected to port 80 inside the container.

---

# Summary

Docker networking is built by combining existing Linux kernel networking features.

```
Container

↓

Network Namespace

↓

eth0

↓

veth

↓

docker0 Bridge

↓

Routing

↓

NAT

↓

Internet

↓

Reply

↓

Connection Tracking

↓

Container
```

Key concepts:

- Docker creates a Network Namespace for every container.
- A `veth` pair connects the container to the host.
- `docker0` acts as a virtual Layer-2 switch.
- Containers on the same bridge communicate directly.
- The host performs routing and NAT for external traffic.
- Docker uses Netfilter and `conntrack` for port publishing and return traffic.
- The Linux kernel—not Docker—performs the actual networking.

At this point, you understand nearly everything required to explain how a Docker container reaches the Internet from first principles.

---

# Next Chapter

## Chapter 44 — Docker Host Network: Sharing the Host's Network Namespace

We'll explore:

- What is Host Network Mode?
- Host Network vs Bridge Network
- Namespace Sharing
- Performance Characteristics
- Port Conflicts
- Why Host Networking Is Faster
- Security Trade-offs
- Real-world Use Cases
- Packet Flow Without `veth`
- When to Use Host Networking