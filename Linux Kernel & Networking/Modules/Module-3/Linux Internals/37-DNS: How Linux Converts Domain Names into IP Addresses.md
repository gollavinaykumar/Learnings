Most developers type URLs like:

```text
https://google.com
```

```text
https://github.com
```

```text
https://openai.com
```

The websites open instantly.

But computers don't understand:

```
google.com
```

They only understand:

```
142.250.190.14
```

or

```
2607:f8b0:4009:80b::200e
```

So the real question is:

**Who converts a domain name into an IP address?**

Questions you may have:

- What is DNS?
- Why was DNS invented?
- Who owns the DNS database?
- What are Root Servers?
- What is a TLD Server?
- What is an Authoritative DNS Server?
- What happens when you type `google.com`?
- Why is DNS so fast?
- How does Linux perform DNS lookups?
- Why does Kubernetes have CoreDNS?

The answer is:

**DNS (Domain Name System).**

DNS is often called:

> **The Phone Book of the Internet.**

After this chapter, you'll understand the complete DNS lookup process from your application to the authoritative server.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 37 — DNS: How Linux Converts Domain Names into IP Addresses

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why DNS exists
> - Domain Names vs IP Addresses
> - DNS Hierarchy
> - Recursive Resolver
> - Root Servers
> - TLD Servers
> - Authoritative Name Servers
> - DNS Records
> - DNS Caching
> - Linux Name Resolution
> - Complete DNS Lookup Flow

---

# 📖 Why Do We Need DNS?

Imagine every website had to be accessed like this:

```
https://142.250.190.14
```

Instead of:

```
https://google.com
```

People are good at remembering names.

Computers are good at using numbers.

DNS connects the two.

---

# What is DNS?

DNS stands for:

```
Domain Name System
```

Its job is simple:

```
Domain Name

↓

IP Address
```

Example:

```
google.com

↓

142.250.x.x
```

---

# DNS Is Like a Phone Book

Suppose you know:

```
Vinay
```

But need:

```
Phone Number
```

You search a contact list.

DNS works the same way.

```
google.com

↓

DNS

↓

IP Address
```

---

# Domain Name vs IP Address

Domain Name:

```
github.com
```

Easy for humans.

---

IP Address:

```
140.82.x.x
```

Easy for computers.

---

# Who Stores DNS?

There is **no single DNS server**.

DNS is distributed across the Internet.

```
Root Servers

↓

TLD Servers

↓

Authoritative Servers
```

This hierarchy makes DNS scalable.

---

# DNS Hierarchy

```
Root

↓

.com

↓

google.com

↓

www.google.com
```

Each level knows about the next level.

---

# Root Servers

At the top are:

```
Root DNS Servers
```

They do **not** know every IP address.

Instead they know:

```
Where .com Servers Are

Where .org Servers Are

Where .net Servers Are
```

Think of them as directory assistants.

---

# TLD Servers

TLD means:

```
Top Level Domain
```

Examples:

```
.com

.org

.net

.dev

.io
```

A `.com` server knows which authoritative servers manage `.com` domains.

---

# Authoritative DNS Server

This server knows the actual DNS records.

Example:

```
google.com

↓

142.250.x.x
```

This is the final source of truth for that domain.

---

# Complete DNS Lookup

Suppose Chrome requests:

```
google.com
```

Flow:

```
Chrome

↓

Linux Resolver

↓

Recursive Resolver

↓

Root Server

↓

TLD Server

↓

Authoritative Server

↓

IP Address

↓

Chrome
```

---

# Recursive Resolver

Usually provided by:

- Your ISP
- Public DNS services
- Your organization

Examples:

```
8.8.8.8
```

```
1.1.1.1
```

```
9.9.9.9
```

The resolver performs the lookup on your behalf.

---

# Why Recursive?

Instead of your laptop contacting:

- Root
- TLD
- Authoritative

The resolver performs those steps for you.

Your system simply asks:

```
What's the IP

for

google.com ?
```

---

# Step 1 — Check Local Cache

Linux first checks:

```
DNS Cache
```

If found:

```
Return Immediately
```

No Internet lookup needed.

---

# Step 2 — Ask Resolver

If not cached:

```
Linux

↓

Recursive Resolver
```

Usually via UDP port 53.

---

# Step 3 — Root Server

Resolver asks:

```
Where is

.com ?
```

Root replies:

```
Ask

.com Servers
```

---

# Step 4 — TLD Server

Resolver asks:

```
Where is

google.com ?
```

TLD replies:

```
Ask Google's

Authoritative Server
```

---

# Step 5 — Authoritative Server

Resolver asks:

```
What is

google.com ?
```

Authoritative server replies:

```
142.250.x.x
```

---

# Step 6 — Cache Result

Resolver stores:

```
google.com

↓

142.250.x.x
```

Linux may also cache the result,

depending on the system configuration.

Future lookups become much faster.

---

# DNS Records

DNS stores different record types.

---

# A Record

Maps:

```
Domain

↓

IPv4 Address
```

Example:

```
example.com

↓

192.0.2.10
```

---

# AAAA Record

Maps:

```
Domain

↓

IPv6 Address
```

---

# CNAME Record

Alias.

Example:

```
www.example.com

↓

example.com
```

---

# MX Record

Mail server.

```
example.com

↓

mail.example.com
```

Email depends on MX records.

---

# TXT Record

Stores text.

Common uses:

- Domain Verification
- SPF
- DKIM
- DMARC

---

# NS Record

Specifies:

```
Authoritative Name Servers
```

for a domain.

---

# TTL (Time To Live)

DNS records include:

```
TTL
```

Example:

```
300 Seconds
```

Resolvers cache records until the TTL expires.

---

# Why Caching Matters

Without caching:

Every website visit would contact:

```
Root

↓

TLD

↓

Authoritative
```

That would be far too slow.

Caching dramatically reduces DNS traffic.

---

# Linux Name Resolution

Applications usually call functions such as:

```
getaddrinfo()
```

The system resolver handles:

```
DNS Query

↓

IP Address

↓

Return to Application
```

Applications don't implement DNS themselves.

---

# `/etc/resolv.conf`

Linux stores DNS resolver configuration here.

Example:

```text
nameserver 8.8.8.8
```

This tells Linux which resolver to query.

---

# `/etc/hosts`

Linux checks this file before querying DNS.

Example:

```text
127.0.0.1 localhost

192.168.1.20 myserver
```

Useful for local overrides and testing.

---

# Real Example

Suppose Chrome opens:

```
https://openai.com
```

Flow:

```
Browser

↓

DNS Lookup

↓

IP Address

↓

TCP Handshake

↓

TLS

↓

HTTP
```

DNS always happens before the TCP connection.

---

# Docker Example

Suppose one container accesses:

```
redis
```

Docker's embedded DNS resolves:

```
redis

↓

Container IP
```

Containers on the same Docker network can reach each other by name.

---

# Kubernetes Example

Suppose a Pod connects to:

```
database.default.svc.cluster.local
```

CoreDNS resolves:

```
Service Name

↓

Cluster IP
```

Pods use DNS instead of hardcoded IP addresses.

---

# Why DNS Matters

Every Internet application depends on DNS.

Examples:

- Web Browsers
- Docker
- Kubernetes
- Email
- Databases
- APIs
- Cloud Services

Without DNS,

people would need to remember IP addresses.

---

# Hands-on Lab

## Lab 1 — View Resolver Configuration

```bash
cat /etc/resolv.conf
```

---

## Lab 2 — Query an A Record

```bash
dig google.com
```

---

## Lab 3 — Query an MX Record

```bash
dig google.com MX
```

---

## Lab 4 — Query an NS Record

```bash
dig google.com NS
```

---

## Lab 5 — Use `nslookup`

```bash
nslookup openai.com
```

---

## Lab 6 — Trace the Full DNS Lookup

```bash
dig +trace google.com
```

Observe:

- Root Servers
- TLD Servers
- Authoritative Servers

---

# Interview Questions

## What is DNS?

DNS (Domain Name System) translates human-readable domain names into IP addresses.

---

## Why does DNS exist?

Because humans remember names more easily than IP addresses.

---

## What is a Recursive Resolver?

A Recursive Resolver performs DNS lookups on behalf of clients by querying other DNS servers as needed.

---

## What is the role of a Root Server?

Root Servers direct resolvers to the correct Top Level Domain (TLD) servers.

---

## What is an Authoritative DNS Server?

An Authoritative DNS Server stores the official DNS records for a domain.

---

## What is an A Record?

An A Record maps a domain name to an IPv4 address.

---

## What is the purpose of TTL in DNS?

TTL defines how long a DNS record may be cached before it should be refreshed.

---

## Why does Kubernetes use CoreDNS?

CoreDNS provides DNS-based service discovery so Pods can communicate using service names instead of IP addresses.

---

# Summary

DNS translates domain names into IP addresses using a distributed hierarchy.

```
Application

↓

Linux Resolver

↓

Recursive Resolver

↓

Root Server

↓

TLD Server

↓

Authoritative Server

↓

IP Address

↓

Application
```

Key concepts:

- DNS maps names to IP addresses.
- DNS is distributed, not centralized.
- Root, TLD, and Authoritative servers work together.
- Recursive Resolvers perform lookups for clients.
- DNS caching improves performance.
- Linux uses resolver libraries along with `/etc/resolv.conf` and `/etc/hosts`.
- Docker and Kubernetes provide built-in DNS for service discovery.

Understanding DNS prepares you to learn TLS, HTTPS, load balancing, service meshes, cloud networking, and advanced distributed systems.

---

# Next Chapter

## Chapter 38 — TLS & HTTPS: How Secure Communication Works on the Internet

We'll explore:

- Why HTTPS exists
- HTTP vs HTTPS
- TLS Handshake
- Certificates
- Certificate Authorities (CA)
- Public & Private Keys
- Symmetric vs Asymmetric Encryption
- Digital Signatures
- Session Keys
- Perfect Forward Secrecy
- Complete HTTPS Request Flow