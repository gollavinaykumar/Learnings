For almost 20 years,

Linux networking was controlled using:

```bash
iptables
```

It powered:

- Docker
- Kubernetes
- Firewalls
- NAT
- Routers
- VPNs

But as Linux systems grew larger,

iptables started showing limitations.

Imagine a Kubernetes cluster with:

```
100,000 Pods
```

That could create:

```
Hundreds of Thousands

of Firewall Rules
```

Searching those rules became slower.

Updating them became expensive.

IPv4 and IPv6 required separate rule sets.

Linux developers needed something better.

The answer is:

# nftables

nftables is the modern packet filtering framework that replaces iptables.

Today, most modern Linux distributions use nftables internally.

After this chapter, you'll understand why nftables exists and why it is considered the future of Linux firewalling.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 62 — nftables: The Future of Linux Firewalling

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why nftables was introduced
> - iptables vs nftables
> - nftables Architecture
> - Tables
> - Chains
> - Rules
> - Sets
> - Maps
> - Atomic Updates
> - Docker & Kubernetes Compatibility
> - Performance Improvements

---

# 📖 Why Replace iptables?

Suppose Linux has:

```
500,000 Firewall Rules
```

iptables checks rules:

```
Rule 1

↓

Rule 2

↓

Rule 3

↓

...

↓

Rule 500000
```

Even with optimizations,

very large rule sets become harder to manage efficiently.

Linux developers redesigned the firewall architecture.

---

# What is nftables?

nftables is:

```
Modern Linux Packet Filtering Framework
```

It replaces:

```
iptables

ip6tables

arptables

ebtables
```

with one unified system.

---

# One Framework

Old Linux:

```
IPv4

↓

iptables
```

IPv6:

```
ip6tables
```

ARP:

```
arptables
```

Ethernet Bridge:

```
ebtables
```

Modern Linux:

```
nftables

↓

Everything
```

One framework handles multiple protocol families.

---

# Architecture

```
Application

↓

nft Command

↓

Netfilter

↓

Linux Kernel
```

Like iptables,

nftables programs the Netfilter framework.

---

# Netfilter vs nftables

Remember:

```
Netfilter

=

Kernel Framework
```

```
nftables

=

Userspace Configuration System
```

Netfilter processes packets.

nftables defines the rules.

---

# Tables

Just like iptables,

nftables organizes rules into:

```
Tables
```

Example:

```
inet

↓

filter
```

---

# Chains

Tables contain:

```
Chains
```

Example:

```
INPUT

↓

Rules
```

Very similar conceptually to iptables.

---

# Rules

Rules describe:

```
If

↓

Then
```

Example:

```
TCP Port 22

↓

Accept
```

---

# Sets

This is one of nftables' biggest improvements.

Instead of:

```
Rule

↓

Rule

↓

Rule

↓

Rule
```

Linux stores values inside a:

```
Set
```

Example:

```
Allowed IPs

↓

Set
```

Packet lookup becomes much more efficient.

---

# Example

Instead of:

```
IP A

↓

Rule
```

```
IP B

↓

Rule
```

```
IP C

↓

Rule
```

Use:

```
Set

↓

A

B

C
```

One rule references the entire set.

---

# Maps

Maps extend sets.

Example:

```
Port

↓

Action
```

Instead of many separate rules,

a single lookup can determine what to do.

---

# Why Sets Matter

Suppose:

```
100,000 Allowed IP Addresses
```

iptables:

```
100,000 Rules
```

nftables:

```
One Set
```

Memory usage and lookup efficiency improve significantly.

---

# Atomic Updates

One major iptables problem:

Updating rules could temporarily expose an incomplete firewall configuration.

nftables supports:

```
Atomic Updates
```

Meaning:

```
Old Rules

↓

Replace

↓

New Rules

↓

Instantly
```

Packets never see a partially updated ruleset.

---

# Unified Rule Engine

Old Linux:

```
IPv4 Rules

↓

Separate
```

IPv6:

```
Separate
```

Modern Linux:

```
One Rule

↓

IPv4

+

IPv6
```

Simpler configuration.

---

# Performance

nftables generally provides:

✔ Better scalability

✔ Improved rule organization

✔ Efficient set lookups

✔ Lower maintenance complexity

Performance depends on workload,

but nftables was designed for modern large-scale systems.

---

# Example

Old:

```
5000 Rules

↓

Sequential Checks
```

Modern:

```
Lookup

↓

Set

↓

Decision
```

Much cleaner.

---

# Docker Compatibility

Modern Docker works on systems using nftables.

Depending on the Linux distribution,

Docker may interact through compatibility layers or native Netfilter integration.

The networking concepts remain the same.

---

# Kubernetes

Modern Kubernetes networking implementations increasingly support nftables.

Some distributions and CNI plugins are moving from iptables-based implementations to nftables-based ones.

Others use eBPF instead.

---

# nft Command

List rules:

```bash
sudo nft list ruleset
```

Observe:

- Tables
- Chains
- Rules
- Sets

---

# Compare Commands

iptables:

```bash
sudo iptables -L
```

nftables:

```bash
sudo nft list ruleset
```

Different syntax.

Same kernel framework.

---

# Packet Flow

```
Packet

↓

Netfilter Hook

↓

nft Rule

↓

Decision

↓

Kernel
```

Netfilter still processes packets.

Only the configuration interface changes.

---

# Why Linux Moved to nftables

Because modern networking requires:

- Better scalability
- Easier maintenance
- Unified protocol support
- Faster updates

nftables addresses these goals.

---

# Complete Architecture

```
Application

↓

nft

↓

Netfilter

↓

Linux Kernel

↓

Packet
```

---

# Hands-on Lab

## Lab 1 — Check nftables

```bash
sudo nft list ruleset
```

---

## Lab 2 — Compare iptables

```bash
sudo iptables -L
```

Compare outputs.

---

## Lab 3 — View Tables

```bash
sudo nft list tables
```

---

## Lab 4 — View Chains

```bash
sudo nft list ruleset
```

Observe table and chain hierarchy.

---

# Interview Questions

## What is nftables?

nftables is the modern Linux packet filtering framework used to configure Netfilter.

---

## Why was nftables introduced?

To replace multiple legacy tools with a unified, scalable, and more maintainable firewall framework.

---

## What are Sets?

Sets store multiple values that can be matched efficiently using a single rule.

---

## Why are Atomic Updates important?

They replace an entire ruleset safely, preventing inconsistent firewall states during updates.

---

## Can nftables handle both IPv4 and IPv6?

Yes.

It supports unified rule definitions across multiple protocol families.

---

## Does nftables replace Netfilter?

No.

Netfilter remains the kernel packet-processing framework.

nftables is a userspace interface for configuring it.

---

## Does Docker work with nftables?

Yes.

Modern Linux distributions support Docker networking alongside nftables, although implementation details vary.

---

# Summary

nftables is the modern replacement for iptables.

```
Application

↓

nft

↓

Tables

↓

Chains

↓

Rules

↓

Sets

↓

Netfilter

↓

Linux Kernel

↓

Packet
```

Key concepts:

- nftables unifies multiple legacy firewall tools.
- Tables, chains, and rules remain familiar concepts.
- Sets and maps improve scalability and efficiency.
- Atomic updates provide safer rule changes.
- IPv4 and IPv6 can be managed together.
- Netfilter remains the kernel engine underneath.
- Docker and Kubernetes increasingly operate in nftables-based environments.

At this point, you understand both generations of Linux firewall management:

- **iptables** → Traditional rule management
- **nftables** → Modern, scalable rule management

Both ultimately configure the same Netfilter packet-processing framework.

---

# Next Chapter

## Chapter 63 — Connection Tracking (conntrack): How Linux Remembers Every Network Connection

We'll explore:

- What is Connection Tracking?
- Stateful vs Stateless Firewalls
- conntrack Table
- TCP State Tracking
- UDP Tracking
- NAT Integration
- Docker Networking
- Kubernetes Services
- Timeouts
- Complete Connection Lifecycle

> **This chapter explains how Linux remembers millions of active network connections and why NAT would not work without conntrack.**