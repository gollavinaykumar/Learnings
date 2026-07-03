Most developers know this command exists:

```bash
docker run --network none nginx
```

The container starts successfully.

But something strange happens.

Inside the container:

```bash
ping google.com
```

fails.

Even:

```bash
ping 8.8.8.8
```

fails.

Why?

Did Docker break networking?

No.

Docker intentionally created a container with **almost no networking at all.**

Questions you may have:

- What is the None Network?
- Why does Docker create it?
- Does the container have an IP address?
- Why can't it access the Internet?
- Why is `localhost` still working?
- When should we use None Networking?
- How is it different from Host and Bridge?

The answer is:

**Docker creates an isolated Network Namespace with only a loopback interface.**

After this chapter, you'll understand the simplest Docker networking mode and why it is useful for highly secure workloads.

---

# 👑 Linux Kernel & Networking Mastery

# Module 5 — Linux Namespaces & Container Foundations

# Chapter 45 — Docker None Network: Completely Isolated Networking

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is Docker None Network?
> - Why None Network exists
> - Empty Network Namespace
> - Loopback Interface
> - No `veth`
> - No Linux Bridge
> - No Routing
> - No Internet Access
> - Security Advantages
> - Real-world Use Cases

---

# 📖 Why Do We Need None Networking?

Not every container needs network access.

Suppose you have:

```
PDF Generator
```

It only:

```
Read File

↓

Generate PDF

↓

Write File
```

No network required.

Giving Internet access only increases risk.

Docker solves this by providing:

```
None Network
```

---

# Default Bridge Mode

Normally:

```
Container

↓

Network Namespace

↓

eth0

↓

veth

↓

docker0

↓

Internet
```

---

# None Network

Instead:

```
Container

↓

Network Namespace

↓

lo
```

That's it.

No external connectivity.

---

# What Does Docker Create?

Docker still creates:

```
Network Namespace
```

But it does **not** create:

❌ `veth`

❌ Bridge Connection

❌ IP Address on `eth0`

❌ Gateway

❌ NAT

---

# Interfaces

Run:

```bash
ip addr
```

Output:

```text
lo
```

Only:

```
Loopback
```

exists.

There is no `eth0`.

---

# Why Loopback Exists?

Linux expects every Network Namespace to have a loopback interface.

```
127.0.0.1
```

Applications can still communicate with services running inside the same container.

---

# Localhost Still Works

Suppose your application starts:

```
Web Server

↓

127.0.0.1:8080
```

Inside the same container:

```
curl localhost:8080
```

Works perfectly.

Because:

```
localhost

↓

Same Namespace
```

---

# No IP Address

Bridge Mode:

```
172.17.0.2
```

Host Mode:

```
Host IP
```

None Mode:

```
No External IP
```

The container cannot send packets to another machine.

---

# No Routing Table

Bridge Mode:

```bash
ip route
```

Output:

```
Default Gateway

↓

172.17.0.1
```

None Mode:

```bash
ip route
```

Usually shows no default route.

Without a route,

Linux has nowhere to send external packets.

---

# No Gateway

Normally:

```
Container

↓

Gateway

↓

Internet
```

None Mode:

```
Container

↓

No Gateway
```

No path exists beyond the namespace.

---

# No `veth`

Bridge Mode:

```
eth0

↓

veth

↓

docker0
```

None Mode:

```
No eth0

No veth
```

Nothing connects the namespace to the host.

---

# No Linux Bridge

Since there is no `veth`,

the container is not attached to:

```
docker0
```

The bridge has no knowledge of this container.

---

# No NAT

Without an external interface,

there is nothing to translate.

No SNAT.

No DNAT.

No `iptables` rules for connectivity.

---

# Packet Journey

Suppose the application sends:

```
HTTP Request
```

Linux tries:

```
Socket

↓

TCP

↓

IP

↓

Routing Table
```

Routing fails.

```
No Route

↓

Packet Dropped
```

The packet never leaves the namespace.

---

# Communication Inside Container

Processes inside the same container still communicate.

Example:

```
Application A

↓

localhost

↓

Application B
```

Because they share:

```
Loopback
```

---

# Security Advantage

Suppose an attacker compromises the container.

Bridge Mode:

```
Internet

↓

Reachable
```

None Mode:

```
Internet

↓

Not Reachable
```

The attack surface is greatly reduced.

---

# Common Use Cases

Examples:

- Offline Data Processing
- PDF Generation
- Image Processing
- Video Encoding
- Batch Jobs
- Cryptographic Operations
- Scientific Computation

No network is required.

---

# Temporary Network Isolation

Sometimes developers start a container with:

```
None Network
```

Then manually connect it later.

Example:

```bash
docker network connect bridge my-container
```

Now the container gains network connectivity.

---

# Comparison

Bridge:

```
Internet

✔
```

Host:

```
Internet

✔
```

None:

```
Internet

❌
```

---

# Bridge vs Host vs None

| Feature | Bridge | Host | None |
|---------|--------|------|------|
| Network Namespace | Separate | Shared | Separate |
| `eth0` | Yes | Host Interface | No |
| Loopback | Yes | Shared Host Loopback | Yes |
| `veth` Pair | Yes | No | No |
| Linux Bridge | Yes | No | No |
| Internet Access | Yes | Yes | No |
| NAT | Yes | No | No |
| Port Publishing | Required | Not Required | Not Possible |

---

# Real Example

Bridge:

```
Application

↓

eth0

↓

Internet
```

Host:

```
Application

↓

Host NIC

↓

Internet
```

None:

```
Application

↓

localhost

↓

Same Container
```

Nothing beyond that.

---

# Why None Networking Matters

Some workloads should never communicate externally.

The None Network provides a simple way to enforce that isolation at the networking level.

---

# Hands-on Lab

## Lab 1 — Start a None Network Container

```bash
docker run -it --network none alpine sh
```

---

## Lab 2 — View Interfaces

```bash
ip addr
```

Observe:

```
lo
```

Only the loopback interface exists.

---

## Lab 3 — View Routes

```bash
ip route
```

Observe the absence of a default route.

---

## Lab 4 — Test Internet

```bash
ping 8.8.8.8
```

Fails.

---

## Lab 5 — Test Localhost

Run a service bound to `127.0.0.1` inside the container and connect to it using `localhost`.

Observe successful local communication.

---

# Interview Questions

## What is Docker None Network?

The None Network creates a container with an isolated Network Namespace containing only a loopback interface.

---

## Does a None Network container have Internet access?

No.

There is no external interface, gateway, or routing path.

---

## Why does `localhost` still work?

Because the container still has its own loopback interface (`lo`).

---

## Does Docker create a `veth` pair in None Mode?

No.

The container is not connected to the host networking infrastructure.

---

## Can a None Network container communicate with other containers?

Not by default.

It has no network connection to them.

---

## When should the None Network be used?

For workloads that do not require network access or where minimizing the attack surface is important.

---

# Summary

The None Network creates a completely isolated networking environment.

```
Container

↓

Network Namespace

↓

Loopback (lo)

↓

Application
```

Key concepts:

- The container has its own Network Namespace.
- Only the loopback interface is available.
- No `eth0` is created.
- No `veth` pair exists.
- No Linux Bridge is used.
- No routing or Internet connectivity is available.
- `localhost` continues to function inside the container.
- The None Network is ideal for secure, offline workloads.

Understanding the None Network completes the three primary Docker networking modes:

- Bridge
- Host
- None

These form the foundation for all Docker networking configurations.

---

# Next Chapter

## Chapter 46 — User-Defined Bridge Networks: Why Production Docker Uses Custom Networks

We'll explore:

- Why User-Defined Networks exist
- Default Bridge vs Custom Bridge
- Automatic Container DNS
- Better Isolation
- Multiple Bridges
- Network Scoping
- Container Discovery
- Creating Custom Networks
- Docker Embedded DNS
- Production Best Practices