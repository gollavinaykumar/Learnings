Most developers use TCP every day without realizing it.

Examples:

```javascript
fetch("https://google.com")
```

```java
new Socket(host, port)
```

```go
net.Dial("tcp", "google.com:443")
```

The connection works.

Data arrives correctly.

Nothing seems special.

But here's the question:

**The Internet is unreliable.**

Packets can:

- Be lost
- Arrive out of order
- Be duplicated
- Be delayed
- Take different routes

So how does your browser always receive a correct webpage?

The answer is:

**TCP (Transmission Control Protocol).**

TCP is one of the greatest engineering achievements in networking.

It provides reliable communication on top of an unreliable IP network.

After this chapter, you'll understand exactly how TCP establishes connections, guarantees delivery, and manages communication across the Internet.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 32 — TCP: Reliable Communication Across the Internet

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why TCP exists
> - TCP Segment Structure
> - Three-Way Handshake
> - Four-Way Connection Termination
> - Sequence Numbers
> - Acknowledgments (ACK)
> - Sliding Window
> - Flow Control
> - Congestion Control
> - Retransmissions
> - TCP State Machine
> - How Linux Implements TCP

---

# 📖 Why Do We Need TCP?

IP provides:

```
Packet Delivery
```

But IP does **not** guarantee:

❌ Delivery

❌ Order

❌ No duplicates

Imagine sending:

```
Packet 1

Packet 2

Packet 3
```

The receiver may get:

```
Packet 3

Packet 1

Packet 2
```

Or lose Packet 2 completely.

Applications need something more reliable.

---

# What is TCP?

TCP stands for:

```
Transmission Control Protocol
```

It provides:

✔ Reliable Delivery

✔ Ordered Delivery

✔ Error Detection

✔ Flow Control

✔ Congestion Control

---

# TCP vs IP

```
IP

↓

Moves Packets
```

```
TCP

↓

Makes Communication Reliable
```

Together they form:

```
TCP/IP
```

---

# Connection-Oriented Protocol

TCP establishes a connection before exchanging data.

```
Client

↓

Connect

↓

Server

↓

Connection Established

↓

Data Transfer
```

Unlike UDP,

TCP maintains connection state.

---

# TCP Segment

TCP wraps application data inside a TCP Segment.

```
+------------------------------------+

Source Port

Destination Port

Sequence Number

ACK Number

Flags

Window Size

Checksum

Payload

+------------------------------------+
```

---

# Ports

An IP address identifies:

```
Machine
```

A TCP Port identifies:

```
Application
```

Example:

```
192.168.1.10:80
```

Port:

```
80

↓

HTTP
```

---

# Sequence Number

Every byte transmitted has a sequence number.

Example:

```
Hello
```

```
H

↓

100
```

```
e

↓

101
```

```
l

↓

102
```

This allows TCP to restore the correct order.

---

# ACK Number

Receiver responds:

```
ACK = 105
```

Meaning:

```
Received Everything

↓

Up To 104

↓

Send 105 Next
```

ACKs confirm successful delivery.

---

# Three-Way Handshake

Before data transfer,

TCP establishes a connection.

---

# Step 1

Client sends:

```
SYN
```

Meaning:

```
I'd Like to Connect
```

---

# Step 2

Server responds:

```
SYN

+

ACK
```

Meaning:

```
Connection Accepted
```

---

# Step 3

Client replies:

```
ACK
```

Now both sides agree.

```
Connection Established
```

---

# Three-Way Handshake Flow

```
Client

↓

SYN

↓

Server

↓

SYN + ACK

↓

Client

↓

ACK

↓

Connected
```

Every TCP connection begins this way.

---

# Why Three Steps?

Both sides must:

✔ Confirm reachability

✔ Exchange initial sequence numbers

✔ Synchronize connection state

This prevents many communication problems.

---

# Data Transfer

After connection:

```
Application

↓

TCP Segments

↓

Internet

↓

Receiver

↓

ACK

↓

Continue
```

Data flows in both directions.

---

# Sliding Window

Sending one packet,

then waiting,

would be slow.

Instead TCP sends multiple segments.

```
Packet 1

Packet 2

Packet 3

Packet 4
```

without waiting for every ACK.

This is the:

```
Sliding Window
```

---

# Window Size

Example:

```
Window = 4 Segments
```

Sender transmits:

```
1

2

3

4
```

Receiver acknowledges.

Window moves forward.

---

# Why Sliding Window?

Benefits:

✔ Better Bandwidth Utilization

✔ Higher Throughput

✔ Reduced Waiting

---

# Retransmission

Suppose:

```
Packet 3

↓

Lost
```

Receiver keeps acknowledging:

```
Need Packet 3
```

Sender eventually retransmits the missing packet.

---

# Duplicate ACK

Suppose Packet 3 is lost.

Receiver receives:

```
1

2

4

5
```

Receiver repeatedly sends:

```
ACK 3
```

This tells the sender that Packet 3 is missing.

---

# Fast Retransmit

Instead of waiting for a timeout,

after several duplicate ACKs,

TCP retransmits the missing segment immediately.

This improves performance.

---

# Flow Control

Suppose:

```
Sender

↓

1 Gbps
```

Receiver can process only:

```
100 Mbps
```

TCP prevents overwhelming the receiver.

Receiver advertises:

```
Receive Window
```

Sender respects this limit.

---

# Congestion Control

Suppose the network becomes busy.

Too many packets cause:

```
Congestion
```

TCP automatically reduces its sending rate.

Linux implements algorithms such as:

- Reno
- Cubic (default on many Linux systems)
- BBR (available on supported systems)

---

# TCP State Machine

A TCP connection moves through states.

```
CLOSED

↓

SYN_SENT

↓

ESTABLISHED

↓

FIN_WAIT

↓

TIME_WAIT

↓

CLOSED
```

Linux tracks every connection.

---

# Connection Termination

Closing a TCP connection requires four messages.

---

# Step 1

Client:

```
FIN
```

Meaning:

```
Finished Sending
```

---

# Step 2

Server:

```
ACK
```

---

# Step 3

Server:

```
FIN
```

---

# Step 4

Client:

```
ACK
```

Connection closes gracefully.

---

# Four-Way Termination

```
Client

↓

FIN

↓

Server

↓

ACK

↓

FIN

↓

Client

↓

ACK

↓

Closed
```

---

# TIME_WAIT

After closing,

the client usually enters:

```
TIME_WAIT
```

Purpose:

✔ Handle delayed packets

✔ Ensure reliable connection termination

---

# Linux TCP Implementation

Linux maintains a TCP Control Block for each connection.

It tracks:

```
State

Sequence Numbers

ACK Numbers

Timers

Window Sizes
```

The kernel updates this continuously.

---

# Real Example

Suppose Chrome loads:

```
https://google.com
```

Flow:

```
DNS

↓

TCP Handshake

↓

TLS Handshake

↓

HTTP Request

↓

HTTP Response

↓

Connection Close
```

Every HTTPS request begins with TCP.

---

# Docker Example

Suppose a container connects to PostgreSQL.

```
Container

↓

TCP

↓

Bridge

↓

Host

↓

Server

↓

ACK

↓

Response
```

Docker does not replace TCP.

It uses the Linux TCP stack.

---

# Why TCP Matters

Everything depends on TCP.

Examples:

- HTTP
- HTTPS
- SSH
- PostgreSQL
- MySQL
- Redis Replication
- Docker Registry
- Kubernetes API

Without TCP,

reliable Internet communication would not exist.

---

# Hands-on Lab

## Lab 1 — View TCP Connections

```bash
ss -t
```

---

## Lab 2 — View Listening Ports

```bash
ss -tl
```

---

## Lab 3 — View All TCP States

```bash
ss -tan
```

Observe:

```
ESTABLISHED

TIME_WAIT

LISTEN
```

---

## Lab 4 — Capture TCP Handshake

```bash
sudo tcpdump -i any tcp
```

Observe:

```
SYN

SYN-ACK

ACK
```

---

## Lab 5 — Monitor Connection State

Open a browser,

then run:

```bash
ss -tan
```

Observe active TCP connections.

---

## Lab 6 — Inspect TCP Statistics

```bash
cat /proc/net/tcp
```

Observe active TCP sockets managed by the kernel.

---

# Interview Questions

## Why does TCP exist?

TCP provides reliable, ordered communication on top of the unreliable IP protocol.

---

## What is the Three-Way Handshake?

A connection establishment process consisting of:

```
SYN

↓

SYN-ACK

↓

ACK
```

---

## Why does TCP use Sequence Numbers?

To detect lost packets, remove duplicates, and reconstruct data in the correct order.

---

## What is the purpose of ACKs?

ACKs confirm which bytes have been successfully received.

---

## What is a Sliding Window?

A Sliding Window allows multiple segments to be in transit before acknowledgments arrive, improving throughput.

---

## What is Flow Control?

Flow Control prevents a fast sender from overwhelming a slow receiver.

---

## What is Congestion Control?

Congestion Control reduces the sending rate when the network becomes congested.

---

## Why does TCP enter the TIME_WAIT state?

To ensure delayed packets are handled correctly and the connection closes safely.

---

# Summary

TCP transforms unreliable IP packet delivery into reliable end-to-end communication.

```
Application

↓

TCP

↓

Three-Way Handshake

↓

Reliable Data Transfer

↓

Sliding Window

↓

ACKs

↓

Retransmissions

↓

Four-Way Close
```

Key concepts:

- TCP is connection-oriented.
- The Three-Way Handshake establishes connections.
- Sequence Numbers preserve ordering.
- ACKs confirm received data.
- Sliding Windows improve throughput.
- Flow Control protects receivers.
- Congestion Control protects the network.
- Linux maintains connection state for every TCP socket.
- Docker and Kubernetes rely on the Linux TCP implementation.

Understanding TCP prepares you to learn sockets, NAT, firewalls, Docker networking, Kubernetes networking, and high-performance Linux networking.

---

# Next Chapter

## Chapter 33 — UDP: Fast, Lightweight Communication Without Connections

We'll explore:

- Why UDP exists
- UDP Datagram Structure
- Connectionless Communication
- UDP vs TCP
- Checksums
- Broadcast & Multicast
- DNS over UDP
- Streaming & Gaming
- When to Choose UDP
- Linux UDP Implementation
- Complete UDP Packet Flow