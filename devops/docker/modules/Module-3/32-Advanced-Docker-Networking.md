# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 32 — Advanced Docker Networking

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Linux networking internals
> - Network Namespaces
> - veth pairs
> - Linux Bridge
> - Overlay Networking
> - VXLAN
> - Routing Mesh
> - IPVS
> - Service Discovery
> - Packet Flow
> - Cross-host communication
> - Production networking

---

# 📖 Introduction

Earlier we learned:

```
Bridge Network

Host Network

None Network
```

Those are Docker network **types**.

Now we're going deeper.

We're going to answer:

> **How does a packet travel from one container to another?**

---

# Networking Stack

A container doesn't have its own physical network card.

Instead, Docker builds a virtual network.

```
Application

↓

TCP / UDP

↓

Network Namespace

↓

veth Pair

↓

Linux Bridge

↓

Host NIC

↓

Internet
```

Every packet follows this path.

---

# Network Namespace

Every container receives its own network stack.

```
Container A

eth0

127.0.0.1

Routing Table

ARP Table

Firewall Rules

-------------------

Container B

eth0

127.0.0.1

Routing Table

ARP Table
```

Although both containers have:

```
127.0.0.1
```

they cannot see each other.

Each namespace is isolated.

---

# veth Pair

A container cannot directly connect to the host.

Docker creates a **virtual Ethernet pair**.

Think of it like a network cable with two ends.

```
Container

eth0

──────────────

veth Pair

──────────────

Host
```

One end is placed inside the container.

The other end stays on the host.

Packets entering one end immediately appear on the other.

---

# Linux Bridge

Now imagine 20 containers.

Instead of connecting every container to every other container:

Docker connects every veth interface to a bridge.

```
Container A

     │

Container B

     │

Container C

     │

     ▼

Linux Bridge

(docker0)

     │

Host NIC
```

The bridge behaves like a virtual Ethernet switch.

It forwards packets between connected interfaces.

---

# Packet Flow (Same Host)

Suppose:

```
Container A

↓

HTTP Request

↓

Container B
```

Packet path:

```
Container A

↓

eth0

↓

veth

↓

docker0 Bridge

↓

veth

↓

eth0

↓

Container B
```

No physical network is involved.

Everything stays inside the host.

---

# Docker DNS

Containers should not communicate using IP addresses.

Instead:

```
Backend

↓

postgres
```

Docker runs an internal DNS service.

Example:

```
DB_HOST=postgres
```

Docker resolves:

```
postgres

↓

172.18.0.3
```

automatically.

---

# Why IP Addresses Are Bad

Container IPs change.

```
Restart

↓

New IP
```

DNS names remain stable.

```
postgres

↓

Always Works
```

---

# Overlay Networks

Bridge networks work only on one machine.

Suppose:

```
Node 1

Backend

----------------

Node 2

Redis
```

How do they communicate?

Docker creates an **Overlay Network**.

```
Node 1

──────── VXLAN ────────

Node 2

──────── VXLAN ────────

Node 3
```

Every node behaves as though it is on one virtual LAN.

---

# VXLAN

VXLAN stands for:

```
Virtual Extensible LAN
```

It encapsulates Ethernet frames inside UDP packets.

```
Original Ethernet Packet

↓

VXLAN Header

↓

UDP

↓

IP

↓

Physical Network
```

To the applications:

It appears as one network.

---

# Packet Flow (Cross Host)

Suppose:

```
Container A

Node 1

↓

Container B

Node 2
```

Flow:

```
Container A

↓

eth0

↓

veth

↓

Linux Bridge

↓

VXLAN Tunnel

↓

Physical Network

↓

VXLAN Tunnel

↓

Linux Bridge

↓

veth

↓

Container B
```

Applications don't know the containers are on different machines.

---

# Routing Mesh

Docker Swarm introduces Routing Mesh.

Example:

```
Backend

Replicas = 5
```

User sends request:

```
Internet

↓

Node 1

↓

Routing Mesh

↓

Replica on Node 3
```

Even though the container isn't on Node 1, the request is forwarded.

Every node accepts traffic.

---

# IPVS

Routing Mesh uses IPVS.

IPVS stands for:

```
IP Virtual Server
```

It performs:

- Load balancing
- Connection tracking
- Traffic forwarding

Think of IPVS as a high-performance Layer 4 load balancer inside the Linux kernel.

---

# Service Discovery

Every service has a DNS entry.

```
backend

↓

10.0.0.5
```

Swarm automatically updates DNS when replicas change.

Applications continue using the service name.

---

# Network Drivers

Docker supports multiple drivers.

| Driver | Use Case |
|---------|----------|
| Bridge | Single host |
| Host | Maximum performance |
| None | Complete isolation |
| Overlay | Multi-host clusters |
| Macvlan | Containers appear as physical devices |
| IPvlan | Lightweight Layer 2/Layer 3 networking |

Choose the driver based on your architecture.

---

# Macvlan

Macvlan gives each container its own MAC address.

```
Router

│

├── Host

├── Container A

├── Container B
```

Containers appear as independent machines on the physical network.

Useful when legacy applications expect unique MAC/IP addresses.

---

# NAT and iptables

When you publish a port:

```bash
docker run -p 8080:80 nginx
```

Docker configures `iptables` rules.

```
Host:8080

↓

NAT

↓

Container:80
```

The application doesn't know it's behind NAT.

---

# Inspecting Networks

Useful commands:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect my-network
```

View interfaces (Linux):

```bash
ip addr
```

View bridge:

```bash
brctl show
```

Show routing:

```bash
ip route
```

---

# Production Networking

Typical production architecture:

```
Internet

↓

Load Balancer

↓

Reverse Proxy

↓

Overlay Network

↓

Backend Services

↓

Database

↓

Cache
```

Traffic is encrypted between hosts when configured appropriately.

---

# Performance Considerations

Approximate performance:

```
Host Network

↓

Bridge Network

↓

Overlay Network
```

Why?

Overlay networking adds encapsulation (VXLAN), which introduces extra processing.

Choose the simplest network that meets your requirements.

---

# Common Networking Problems

### Connection Refused

Possible causes:

- Wrong port
- Application not listening
- Firewall
- Service down

---

### DNS Failure

Possible causes:

- Wrong network
- Incorrect service name
- DNS misconfiguration

---

### Cross-Host Failure

Possible causes:

- Overlay network issue
- Firewall blocking VXLAN traffic
- Swarm cluster problems

---

# Hands-on Lab

Create a network:

```bash
docker network create app-net
```

Run containers:

```bash
docker run -d --name db --network app-net postgres:16

docker run -d --name api --network app-net nginx
```

Inspect:

```bash
docker network inspect app-net
```

Enter the API container:

```bash
docker exec -it api sh
```

Test DNS:

```bash
ping db
```

Observe that the service name resolves without knowing the IP address.

---

# Interview Questions

## What is a veth pair?

A virtual Ethernet cable with two connected interfaces. One end is placed in the container, the other on the host.

---

## What is the Linux bridge?

A virtual Layer 2 switch that connects multiple container interfaces on the same host.

---

## Why are overlay networks needed?

To allow containers on different Docker hosts to communicate as though they are on the same local network.

---

## What is VXLAN?

A tunneling protocol that encapsulates Layer 2 Ethernet frames inside UDP packets, enabling overlay networks across Layer 3 infrastructure.

---

## Why use service names instead of IP addresses?

Container IP addresses can change. DNS service names remain stable and are automatically updated.

---

## What is Routing Mesh?

A Docker Swarm feature that allows any node in the cluster to accept traffic for a service and forward it to an available replica.

---

# Summary

Docker networking is built on Linux networking primitives.

```
Application

↓

Network Namespace

↓

eth0

↓

veth Pair

↓

Linux Bridge

↓

Overlay Network (Optional)

↓

Physical Network
```

Key concepts:

- Network namespaces isolate networking.
- veth pairs connect containers to the host.
- Linux bridges switch traffic locally.
- Overlay networks extend connectivity across hosts.
- VXLAN carries overlay traffic.
- Routing Mesh and IPVS distribute traffic in Swarm.
- Docker DNS enables service discovery without hardcoded IP addresses.

Mastering these concepts provides the networking foundation needed for Kubernetes networking, CNI plugins, and service meshes.

---

# Next Chapter

## Chapter 33 — BuildKit Internals

We'll explore:

- Why BuildKit was introduced
- Directed Acyclic Graph (DAG) execution
- Parallel builds
- Advanced caching
- Cache export/import
- Secret mounts
- SSH mounts
- Remote cache
- Build optimization
- Production build pipelines