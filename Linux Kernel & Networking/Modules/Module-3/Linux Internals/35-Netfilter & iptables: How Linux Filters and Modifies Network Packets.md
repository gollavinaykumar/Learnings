Most developers know these commands:

```bash
iptables -L
```

```bash
iptables -t nat -L
```

Or nowadays:

```bash
nft list ruleset
```

Many people memorize firewall rules.

Very few understand what actually happens inside the Linux kernel.

Questions you may have:

- How does Linux inspect every packet?
- At what point does a firewall see a packet?
- How does Docker automatically expose ports?
- What is NAT?
- What is DNAT?
- What is SNAT?
- What is MASQUERADE?
- What is Connection Tracking (`conntrack`)?
- Why does Kubernetes rely heavily on Netfilter?

The answer is:

**Netfilter.**

Netfilter is the packet processing framework built directly into the Linux kernel.

Every incoming and outgoing packet can pass through it.

After this chapter, you'll understand one of the most important networking subsystems in Linux.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 35 — Netfilter & iptables: How Linux Filters and Modifies Network Packets

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is Netfilter?
> - Why Netfilter exists
> - Netfilter Hooks
> - `iptables`
> - Tables
> - Chains
> - Rules
> - Packet Filtering
> - NAT
> - DNAT
> - SNAT
> - MASQUERADE
> - Connection Tracking (`conntrack`)
> - Docker & Kubernetes Packet Processing

---

# 📖 Why Do We Need Netfilter?

Imagine your Linux server receives:

```
1,000,000 Packets
```

Some packets should:

✔ Be accepted

Others should:

❌ Be dropped

Some packets should:

```
Destination Changed
```

Others should:

```
Source Changed
```

Linux needs a framework capable of inspecting every packet.

That framework is:

```
Netfilter
```

---

# What is Netfilter?

Netfilter is a packet processing framework inside the Linux kernel.

It allows Linux to:

✔ Accept Packets

✔ Drop Packets

✔ Modify Packets

✔ Perform NAT

✔ Track Connections

---

# Packet Journey

Suppose a packet arrives.

```
NIC

↓

Driver

↓

IP Layer

↓

Netfilter

↓

Routing

↓

Socket
```

Before Linux delivers the packet,

Netfilter gets a chance to inspect it.

---

# Netfilter Hooks

The Linux kernel exposes several checkpoints called:

```
Hooks
```

Packets pass through these hooks.

```
Packet

↓

Hook

↓

Decision
```

---

# Five Main Hooks

```
PREROUTING
```

Packet just arrived.

---

```
INPUT
```

Packet is for this machine.

---

```
FORWARD
```

Packet should pass through this machine.

---

```
OUTPUT
```

Packet originated on this machine.

---

```
POSTROUTING
```

Packet is leaving the machine.

---

# Hook Flow

Incoming Packet:

```
NIC

↓

PREROUTING

↓

Routing Decision

↓

INPUT

↓

Application
```

Outgoing Packet:

```
Application

↓

OUTPUT

↓

POSTROUTING

↓

NIC
```

Forwarded Packet:

```
NIC

↓

PREROUTING

↓

FORWARD

↓

POSTROUTING

↓

NIC
```

---

# What is iptables?

`iptables` is a userspace tool.

It does **not** process packets itself.

Instead:

```
iptables

↓

Kernel Netfilter

↓

Rules Installed
```

The kernel enforces those rules.

---

# Tables

Netfilter organizes rules into tables.

Main tables:

```
filter
```

```
nat
```

```
mangle
```

```
raw
```

Each has a different purpose.

---

# Filter Table

Most common.

Purpose:

✔ Allow Packets

✔ Block Packets

Example:

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

Allow SSH.

---

# NAT Table

Responsible for:

```
Address Translation
```

Examples:

```
DNAT

SNAT

MASQUERADE
```

---

# Chains

Each table contains Chains.

Example:

```
INPUT

OUTPUT

FORWARD
```

A packet enters a chain,

then Linux evaluates rules one by one.

---

# Rules

Suppose:

```
Packet

↓

Rule 1

↓

Match?

↓

No

↓

Rule 2

↓

Match?

↓

Yes

↓

Action
```

Evaluation stops when a terminal action is taken.

---

# Actions

Common actions:

```
ACCEPT
```

```
DROP
```

```
REJECT
```

```
LOG
```

```
DNAT
```

```
SNAT
```

---

# ACCEPT

Packet continues normally.

```
Packet

↓

Accept

↓

Continue
```

---

# DROP

Packet disappears silently.

```
Packet

↓

Drop

↓

Gone
```

The sender receives no response.

---

# REJECT

Packet is rejected.

Linux returns an error message,

such as an ICMP unreachable message or a TCP reset, depending on the protocol and rule.

---

# NAT

NAT stands for:

```
Network Address Translation
```

Linux changes IP addresses inside packets.

This is extremely common.

---

# Why NAT?

Suppose:

```
Private IP

↓

192.168.1.10
```

Cannot travel directly across the Internet.

Linux replaces it with:

```
Public IP
```

Now routers know where to send replies.

---

# SNAT

Source NAT.

Linux changes:

```
Source IP
```

Example:

```
192.168.1.10

↓

203.0.113.50
```

Used when packets leave a private network.

---

# DNAT

Destination NAT.

Linux changes:

```
Destination IP

or

Port
```

Example:

```
Public Port 80

↓

Container Port 8080
```

---

# MASQUERADE

Special form of SNAT.

Useful when the public IP changes dynamically.

Example:

```
Home Internet

↓

Dynamic IP

↓

MASQUERADE
```

Linux automatically uses the current outgoing interface address.

---

# Docker Example

Suppose:

```bash
docker run -p 8080:80 nginx
```

Docker automatically creates:

```
DNAT Rule
```

Flow:

```
Client

↓

Host Port 8080

↓

DNAT

↓

Container Port 80
```

The application inside the container never sees port 8080.

---

# Connection Tracking (`conntrack`)

Linux remembers active connections.

Example:

```
TCP Connection

↓

Tracked
```

This allows:

```
Outgoing Packet

↓

Return Packet

↓

Automatically Accepted
```

Without creating separate firewall rules for every reply.

---

# Connection States

Common states:

```
NEW
```

First packet.

---

```
ESTABLISHED
```

Existing connection.

---

```
RELATED
```

Related to an existing connection.

---

```
INVALID
```

Malformed or unexpected packet.

---

# Example Rule

Allow established traffic:

```bash
iptables -A INPUT \
-m conntrack \
--ctstate ESTABLISHED,RELATED \
-j ACCEPT
```

Very common firewall rule.

---

# Packet Flow Through Netfilter

Incoming:

```
NIC

↓

PREROUTING

↓

Routing

↓

INPUT

↓

Socket
```

Outgoing:

```
Application

↓

OUTPUT

↓

POSTROUTING

↓

NIC
```

Forwarded:

```
NIC

↓

PREROUTING

↓

FORWARD

↓

POSTROUTING

↓

NIC
```

---

# Kubernetes Example

Kubernetes Services often use Netfilter rules.

Example:

```
Client

↓

ClusterIP

↓

DNAT

↓

Pod IP
```

The Linux kernel transparently redirects traffic.

---

# iptables vs nftables

Modern Linux systems increasingly use:

```
nftables
```

Internally,

many distributions provide compatibility so `iptables` commands still work.

The underlying packet processing framework remains:

```
Netfilter
```

---

# Why Netfilter Matters

Everything depends on it.

Examples:

- Docker
- Kubernetes
- VPNs
- Firewalls
- NAT Gateways
- Cloud Platforms
- Routers
- Load Balancers

Without Netfilter,

Linux would not be able to secure or translate network traffic effectively.

---

# Hands-on Lab

## Lab 1 — View Filter Rules

```bash
sudo iptables -L
```

---

## Lab 2 — View NAT Rules

```bash
sudo iptables -t nat -L
```

---

## Lab 3 — View Packet Counters

```bash
sudo iptables -L -v
```

Observe packet and byte counts.

---

## Lab 4 — View Connection Tracking

```bash
cat /proc/net/nf_conntrack
```

(On some systems, the file may instead be available under `/proc/net/ip_conntrack` or via the `conntrack` tool.)

---

## Lab 5 — View nftables Rules

```bash
sudo nft list ruleset
```

---

## Lab 6 — Observe Docker Rules

Start a container with published ports:

```bash
docker run -d -p 8080:80 nginx
```

Then:

```bash
sudo iptables -t nat -L
```

Observe the automatically created DNAT rules.

---

# Interview Questions

## What is Netfilter?

Netfilter is the Linux kernel framework responsible for packet filtering, NAT, and connection tracking.

---

## What is `iptables`?

`iptables` is a userspace utility used to configure Netfilter rules inside the Linux kernel.

---

## What is the difference between Netfilter and `iptables`?

Netfilter performs packet processing in the kernel.

`iptables` is one of the tools used to configure Netfilter.

---

## What are the five main Netfilter hooks?

- PREROUTING
- INPUT
- FORWARD
- OUTPUT
- POSTROUTING

---

## What is the difference between SNAT and DNAT?

SNAT modifies the source address.

DNAT modifies the destination address or destination port.

---

## What is MASQUERADE?

MASQUERADE is a form of Source NAT commonly used when the external IP address is assigned dynamically.

---

## What is Connection Tracking?

Connection Tracking (`conntrack`) allows Linux to recognize packets that belong to existing connections, enabling stateful firewall behavior.

---

## How does Docker publish container ports?

Docker installs Netfilter DNAT rules so traffic arriving at a host port is redirected to the container's IP address and port.

---

# Summary

Netfilter provides the Linux kernel with a flexible framework for inspecting, filtering, modifying, and tracking network packets.

```
Packet

↓

Netfilter Hook

↓

Table

↓

Chain

↓

Rules

↓

Action

↓

Continue
```

Key concepts:

- Netfilter is implemented inside the Linux kernel.
- `iptables` configures Netfilter rules.
- Packets pass through well-defined hooks.
- Tables organize rules by purpose.
- Chains evaluate packets sequentially.
- NAT modifies source or destination addresses.
- `conntrack` enables stateful packet filtering.
- Docker and Kubernetes rely heavily on Netfilter for networking.

Understanding Netfilter is essential before learning Docker bridge networking, Kubernetes Services, eBPF networking, and advanced Linux packet processing.

---

# Next Chapter

## Chapter 36 — Connection Tracking (`conntrack`): How Linux Remembers Every Network Connection

We'll explore:

- What is `conntrack`?
- Why Stateless Firewalls Aren't Enough
- Connection States
- The Connection Tracking Table
- TCP & UDP Tracking
- Timeouts
- NAT Integration
- Stateful Firewalls
- Docker & Kubernetes Connection Tracking
- Debugging with `conntrack`