By now you've learned:

✅ TCP

✅ IP

✅ Routing

✅ DNS

✅ Network Namespaces

✅ `veth`

✅ Linux Bridge

✅ Netfilter

✅ NAT

✅ Connection Tracking

✅ Docker Networking

But many developers still don't know the answer to one question:

> **When my application sends one HTTP request, what exactly happens inside the Linux kernel?**

This is one of the most important topics for senior backend engineers, DevOps engineers, SREs, Kubernetes engineers, and Linux kernel developers.

After this chapter, you'll be able to mentally trace **every packet** from your application to the Internet and back.

---

# 👑 Linux Kernel & Networking Mastery

# Module 5 — Linux Namespaces & Container Foundations

# Chapter 48 — Linux Packet Journey: Following One Packet from a Container to the Internet

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Complete Packet Journey
> - Application → Socket
> - Socket → TCP
> - TCP → IP
> - Routing Decision
> - Network Namespace
> - `veth`
> - Linux Bridge
> - Netfilter
> - NAT
> - NIC Driver
> - Internet Journey
> - Reply Packet
> - Connection Tracking
> - Complete End-to-End Flow

---

# 📖 The Scenario

Suppose we have:

```
Laptop

↓

Docker Container

↓

Nginx
```

Inside the container:

```bash
curl https://google.com
```

One HTTP request is sent.

Let's follow **one single packet**.

---

# Complete Journey Overview

```
Application

↓

Socket

↓

TCP

↓

IP

↓

Routing

↓

eth0

↓

veth

↓

Linux Bridge

↓

Host Routing

↓

Netfilter

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

Container
```

Everything happens in milliseconds.

---

# Step 1 — Application

Suppose:

```bash
curl https://google.com
```

Application creates:

```
HTTPS Request
```

At this moment,

it's only application data.

---

# Step 2 — DNS Lookup

Before connecting,

Linux resolves:

```
google.com
```

↓

```
142.250.x.x
```

Now the application knows the destination IP.

---

# Step 3 — Socket Creation

Application calls:

```
socket()
```

Linux creates:

```
Socket
```

The socket becomes the communication endpoint.

---

# Step 4 — Connect

Application calls:

```
connect()
```

Linux prepares a TCP connection.

---

# Step 5 — TCP Handshake

```
SYN

↓

SYN-ACK

↓

ACK
```

Connection established.

Now data can flow.

---

# Step 6 — TLS Handshake

Because it's HTTPS:

```
TLS Handshake
```

occurs.

Linux exchanges encrypted session information.

After TLS,

HTTP becomes encrypted.

---

# Step 7 — TCP Segment

Application sends:

```
HTTP GET
```

TCP adds:

```
Source Port

Destination Port

Sequence Number

Checksum
```

Now it's a:

```
TCP Segment
```

---

# Step 8 — IP Packet

IP adds:

```
Source IP

Destination IP

TTL

Protocol
```

Now we have:

```
IP Packet
```

---

# Step 9 — Routing Table

Linux checks:

```bash
ip route
```

Question:

```
Where should this packet go?
```

Destination:

```
142.250.x.x
```

Routing table says:

```
Default Gateway
```

---

# Step 10 — Ethernet Frame

Linux knows the next hop.

It creates:

```
Ethernet Header

↓

Destination MAC

↓

Source MAC
```

Now we have an:

```
Ethernet Frame
```

---

# Step 11 — Container Interface

Frame reaches:

```
eth0
```

inside the container.

---

# Step 12 — `veth` Pair

```
eth0

↓

veth

↓

Host
```

The frame exits the container's Network Namespace.

---

# Step 13 — Linux Bridge

Host receives:

```
veth

↓

docker0
```

Bridge checks:

```
FDB
```

Determines the next destination.

Since the packet is leaving the host,

it forwards toward the host networking stack.

---

# Step 14 — Host Routing

Linux host routing table decides:

```
Internet

↓

eth0
```

(or another physical interface).

---

# Step 15 — Netfilter

Packet enters:

```
Netfilter
```

Hooks execute.

Rules are evaluated.

---

# Step 16 — Source NAT

Container source:

```
172.17.0.2
```

becomes:

```
203.0.113.25
```

Linux records the mapping in:

```
conntrack
```

---

# Step 17 — NIC Driver

Packet reaches:

```
Network Driver
```

Driver programs the network card to transmit the frame.

---

# Step 18 — Physical Network

Packet travels:

```
Laptop

↓

Router

↓

ISP

↓

Internet
```

Across many routers.

---

# Step 19 — Google Receives

Google receives:

```
TCP

↓

TLS

↓

HTTP
```

Processes the request.

Generates a reply.

---

# Step 20 — Reply Packet

Google sends:

```
Response
```

to:

```
203.0.113.25
```

(the host's public IP).

---

# Step 21 — NIC Receives

Host NIC receives:

```
Ethernet Frame
```

Driver passes it to the Linux kernel.

---

# Step 22 — Netfilter Again

Incoming packet reaches:

```
PREROUTING
```

Linux consults:

```
conntrack
```

Finds:

```
203.0.113.25

↓

172.17.0.2
```

Reverse translation is applied.

---

# Step 23 — Host Routing

Linux determines:

```
172.17.0.2

↓

docker0
```

The packet should be delivered to the Docker bridge.

---

# Step 24 — Linux Bridge

Bridge checks:

```
FDB
```

Finds:

```
Container veth
```

Frame forwarded.

---

# Step 25 — `veth`

```
Host

↓

veth

↓

Container
```

Packet re-enters the container's Network Namespace.

---

# Step 26 — Container Network Stack

Packet reaches:

```
eth0

↓

IP

↓

TCP
```

TCP validates:

- Sequence Number
- Checksum

Then reassembles the byte stream.

---

# Step 27 — TLS Decryption

TLS decrypts the received application data using the session keys established earlier.

---

# Step 28 — Socket Buffer

Decrypted bytes are placed into the socket's receive buffer.

The application is notified that data is available.

---

# Step 29 — Application Reads

Application calls:

```
recv()

or

read()
```

Linux copies data from the socket buffer.

---

# Step 30 — Response Displayed

```
curl

↓

Print HTML
```

Request completed.

---

# Complete Flow Diagram

```
Application

↓

DNS

↓

Socket

↓

TCP

↓

TLS

↓

IP

↓

Routing

↓

eth0

↓

veth

↓

docker0

↓

Host Routing

↓

Netfilter

↓

SNAT

↓

NIC

↓

Internet

↓

Server

↓

Reply

↓

NIC

↓

conntrack

↓

docker0

↓

veth

↓

TCP

↓

TLS

↓

Socket

↓

Application
```

---

# Where Linux Kernel Works

The kernel is responsible for:

✔ Socket Management

✔ TCP

✔ IP

✔ Routing

✔ Netfilter

✔ NAT

✔ Connection Tracking

✔ Drivers

Applications simply request networking operations using system calls.

---

# Why Understanding Packet Flow Matters

When debugging:

- Packet Loss
- Connection Timeouts
- Docker Networking
- Kubernetes Services
- Firewall Issues
- NAT Problems
- DNS Failures
- TLS Errors

Knowing the packet journey helps identify exactly where the problem occurs.

---

# Hands-on Lab

## Lab 1 — Observe Routes

```bash
ip route
```

---

## Lab 2 — Observe Interfaces

```bash
ip addr
```

---

## Lab 3 — Observe Connections

```bash
ss -t
```

---

## Lab 4 — Observe NAT Rules

```bash
sudo iptables -t nat -L
```

---

## Lab 5 — Observe Connection Tracking

```bash
sudo conntrack -L
```

---

## Lab 6 — Capture Packets

Host:

```bash
sudo tcpdump -i any port 443
```

Then inside a container:

```bash
curl https://google.com
```

Observe packets moving through the system.

---

# Interview Questions

## What happens first when an application connects to `https://google.com`?

A DNS lookup resolves the domain name to an IP address, followed by socket creation and connection establishment.

---

## What happens before HTTP data is transmitted?

A TCP handshake and, for HTTPS, a TLS handshake establish a secure connection.

---

## Why is the routing table consulted?

To determine the next hop and the outgoing network interface for the packet.

---

## Why does Docker use `veth`?

A `veth` pair connects the container's Network Namespace to the host networking infrastructure.

---

## Why is SNAT required?

Because container IP addresses are private and must be translated before packets can be routed on the public Internet.

---

## How does Linux know where to send the reply packet?

`conntrack` remembers the NAT mapping and performs the reverse translation when the reply arrives.

---

## At what point does the application receive data?

After TCP validation, TLS decryption (if used), and delivery into the socket receive buffer.

---

# Summary

Every network request follows a well-defined path through the Linux kernel.

```
Application

↓

Socket

↓

TCP

↓

TLS

↓

IP

↓

Routing

↓

veth

↓

Linux Bridge

↓

Netfilter

↓

NAT

↓

NIC

↓

Internet

↓

Server

↓

Reply

↓

Connection Tracking

↓

Linux Bridge

↓

veth

↓

TCP

↓

TLS

↓

Socket

↓

Application
```

Key concepts:

- Applications communicate through sockets.
- DNS resolves names before connections are made.
- TCP establishes reliable communication.
- TLS secures HTTPS traffic.
- IP and routing determine where packets travel.
- `veth` and Linux Bridge connect containers to the host.
- Netfilter and NAT prepare packets for external networks.
- `conntrack` enables correct handling of reply traffic.
- The Linux kernel performs the entire networking pipeline.

Understanding this packet journey gives you the mental model needed to debug real-world Linux, Docker, and Kubernetes networking issues.

---

# Next Chapter

## Chapter 49 — cgroups: How Linux Controls CPU, Memory, Disk, and Network Resources

We'll explore:

- Why cgroups exist
- What is a Control Group?
- CPU Limits
- Memory Limits
- Disk I/O Limits
- Network Resource Control
- OOM Killer
- cgroup v1 vs cgroup v2
- How Docker Uses cgroups
- Complete Resource Isolation Architecture

> **This chapter begins the second major pillar of containers: Resource Isolation, alongside Namespaces.**