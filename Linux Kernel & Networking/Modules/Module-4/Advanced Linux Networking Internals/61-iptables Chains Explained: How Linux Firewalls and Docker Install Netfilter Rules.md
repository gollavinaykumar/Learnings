Most developers know this command:

```bash
iptables -L
```

They see something like:

```text
Chain INPUT (policy ACCEPT)

Chain FORWARD (policy DROP)

Chain OUTPUT (policy ACCEPT)
```

or after installing Docker:

```text
DOCKER

DOCKER-USER

DOCKER-FORWARD
```

But very few understand:

- What are tables?
- What are chains?
- What are rules?
- Why are there multiple tables?
- How does `docker run -p 8080:80 nginx` work?
- How does Kubernetes expose Services?
- What actually happens when a packet reaches iptables?

After this chapter, you'll be able to read iptables rules like a Linux kernel engineer.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 61 — iptables Explained: Tables, Chains, Rules, and How Docker Uses Them

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why iptables exists
> - Netfilter vs iptables
> - Tables
> - Chains
> - Rules
> - ACCEPT
> - DROP
> - REJECT
> - DNAT
> - SNAT
> - MASQUERADE
> - Docker-created Chains
> - Complete Packet Processing

---

# 📖 Netfilter vs iptables

Many people think:

```
iptables

=

Linux Firewall
```

Not exactly.

Remember the previous chapter.

```
Netfilter

↓

Kernel Framework
```

```
iptables

↓

User-space Tool
```

iptables simply tells Netfilter:

```
"Here are the rules you should apply."
```

---

# Think of a Security Guard

Imagine a building.

```
Visitor

↓

Security Guard

↓

Decision
```

The security guard asks:

```
Allowed?

↓

Yes

↓

Enter
```

or

```
No

↓

Go Away
```

iptables rules work the same way.

---

# Architecture

```
Application

↓

iptables

↓

Netfilter

↓

Linux Kernel

↓

Packet
```

Applications configure rules.

The kernel enforces them.

---

# What is a Table?

A table is a collection of chains designed for a particular purpose.

Think of it as:

```
Table

↓

Related Rules
```

Linux has several tables.

The most important are:

- filter
- nat
- mangle
- raw

---

# filter Table

Purpose:

```
Firewall
```

Typical decisions:

```
ACCEPT

DROP

REJECT
```

Most firewall rules belong here.

---

# nat Table

Purpose:

```
Network Address Translation
```

Typical actions:

- DNAT
- SNAT
- MASQUERADE

Docker heavily relies on this table.

---

# mangle Table

Purpose:

```
Modify Packet Headers
```

Examples:

- Change packet marks
- Adjust QoS-related fields
- Advanced routing scenarios

Less common for everyday application developers.

---

# raw Table

Purpose:

```
Early Packet Processing
```

Often used for advanced networking scenarios such as bypassing connection tracking for selected traffic.

---

# Visual Representation

```
Packet

↓

raw

↓

mangle

↓

nat

↓

filter

↓

Destination
```

Each table serves a different purpose.

---

# What is a Chain?

A chain is an ordered list of rules.

Example:

```
INPUT

↓

Rule 1

↓

Rule 2

↓

Rule 3
```

Rules are checked one by one.

---

# Built-in Chains

Most commonly used chains are:

```
INPUT

OUTPUT

FORWARD

PREROUTING

POSTROUTING
```

These correspond to Netfilter hook points.

---

# Rule Evaluation

Suppose:

```
Packet Arrives
```

Linux checks:

```
Rule 1

↓

Match?
```

If not,

continue to:

```
Rule 2
```

and so on.

---

# First Match Wins

Example:

```
Rule 1

↓

DROP
```

Packet processing stops.

Later rules are not evaluated.

---

# ACCEPT

Rule:

```
ACCEPT
```

Meaning:

```
Continue

↓

Deliver Packet
```

---

# DROP

Rule:

```
DROP
```

Meaning:

```
Discard Packet

↓

No Reply
```

From the sender's perspective,

the packet simply disappears.

---

# REJECT

Rule:

```
REJECT
```

Meaning:

```
Discard Packet

+

Send Error Response
```

The sender immediately knows the request failed.

---

# DROP vs REJECT

DROP:

```
Silently Ignore
```

REJECT:

```
Reply

↓

Access Denied
```

Both block traffic,

but they behave differently.

---

# Example Firewall Rule

Allow SSH:

```bash
sudo iptables \
-A INPUT \
-p tcp \
--dport 22 \
-j ACCEPT
```

Meaning:

```
TCP Port 22

↓

Allowed
```

---

# Block HTTP

Example:

```bash
sudo iptables \
-A INPUT \
-p tcp \
--dport 80 \
-j DROP
```

Packets to port 80 are silently discarded.

---

# NAT Rules

iptables also performs:

```
DNAT

SNAT

MASQUERADE
```

These belong primarily to the `nat` table.

---

# DNAT Example

Incoming:

```
Host

↓

8080
```

Rule:

```
DNAT

↓

Container

↓

80
```

Destination changes.

---

# SNAT Example

Outgoing:

```
172.17.0.2
```

↓

```
192.168.1.100
```

Source changes.

---

# MASQUERADE

MASQUERADE is a form of Source NAT.

Instead of specifying a fixed source IP,

Linux automatically uses the outgoing interface's IP address.

Docker commonly uses MASQUERADE because host IPs may change.

---

# Docker Port Publishing

Suppose:

```bash
docker run \
-p 8080:80 \
nginx
```

Docker automatically installs NAT rules.

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

No manual firewall configuration required.

---

# Docker Chains

Docker creates custom chains such as:

```
DOCKER
```

```
DOCKER-USER
```

```
DOCKER-FORWARD
```

These help manage container networking without overwriting user firewall rules.

---

# Why DOCKER-USER Exists

Docker updates its own chains automatically.

Administrators can place custom firewall rules in:

```
DOCKER-USER
```

Docker preserves these rules across container lifecycle operations.

---

# Kubernetes

Traditional Kubernetes networking often installs thousands of iptables rules.

Examples include:

- Service load balancing
- Cluster IP translation
- Pod forwarding

Modern clusters may instead use nftables or eBPF depending on the networking stack.

---

# Packet Flow

Incoming:

```
Internet

↓

PREROUTING

↓

DNAT

↓

Routing

↓

INPUT

↓

Application
```

Outgoing:

```
Application

↓

OUTPUT

↓

POSTROUTING

↓

SNAT

↓

Internet
```

---

# Listing Rules

List filter rules:

```bash
sudo iptables -L
```

List NAT rules:

```bash
sudo iptables -t nat -L
```

---

# Why Rule Order Matters

Suppose:

```
Rule 1

↓

DROP Everything
```

```
Rule 2

↓

Allow SSH
```

SSH never reaches Rule 2.

Packets matched by Rule 1 stop processing immediately.

---

# Complete Architecture

```
Application

↓

iptables Rules

↓

Netfilter Hooks

↓

Kernel Decision

↓

Packet
```

iptables provides policy.

Netfilter enforces it.

---

# Hands-on Lab

## Lab 1 — List Filter Rules

```bash
sudo iptables -L
```

---

## Lab 2 — List NAT Rules

```bash
sudo iptables -t nat -L
```

---

## Lab 3 — View Rule Numbers

```bash
sudo iptables -L --line-numbers
```

---

## Lab 4 — Inspect Docker Chains

```bash
sudo iptables -L
```

Look for:

```
DOCKER

DOCKER-USER
```

---

## Lab 5 — View Packet Counters

```bash
sudo iptables -L -v
```

Observe packet and byte counters increasing as traffic flows.

---

# Interview Questions

## What is iptables?

iptables is a userspace tool used to configure Netfilter rules inside the Linux kernel.

---

## What is the difference between Netfilter and iptables?

Netfilter is the kernel framework.

iptables is the userspace utility used to configure it.

---

## What is a table?

A table groups chains that perform a specific function, such as filtering or NAT.

---

## What is a chain?

A chain is an ordered sequence of rules evaluated by the kernel.

---

## What is the difference between ACCEPT, DROP, and REJECT?

- ACCEPT allows the packet.
- DROP silently discards it.
- REJECT discards it and sends an error response.

---

## Why does Docker use the `nat` table?

Docker uses NAT rules to implement port publishing and address translation for containers.

---

## Why does rule order matter?

iptables evaluates rules sequentially.

The first matching rule determines the outcome.

---

# Summary

iptables is the primary userspace interface for configuring Netfilter.

```
Packet

↓

Table

↓

Chain

↓

Rule

↓

ACCEPT

DROP

REJECT

↓

Kernel Action
```

Key concepts:

- Netfilter is the kernel packet-processing framework.
- iptables configures Netfilter rules.
- Tables organize rules by function.
- Chains contain ordered rules.
- Rule evaluation stops at the first match.
- Docker automatically creates iptables rules for container networking.
- Kubernetes has traditionally relied heavily on iptables for Service networking.

Understanding iptables is essential before moving to **nftables**, the modern Linux packet filtering framework that is gradually replacing it.

---

# Next Chapter

## Chapter 62 — nftables: The Future of Linux Firewalling

We'll explore:

- Why nftables was introduced
- iptables vs nftables
- Tables, Chains, and Sets
- Atomic Rule Updates
- Performance Improvements
- Unified IPv4/IPv6 Rules
- Docker Compatibility
- Kubernetes Support
- Migration from iptables
- Complete nftables Architecture

> **This chapter explains why modern Linux distributions are moving from iptables to nftables.**