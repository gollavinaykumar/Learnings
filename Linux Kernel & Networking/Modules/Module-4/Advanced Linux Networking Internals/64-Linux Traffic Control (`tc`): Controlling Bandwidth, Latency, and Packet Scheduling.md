Most developers think networking works like this:

```
Application

↓

NIC

↓

Internet
```

They assume that as soon as an application sends a packet,

Linux immediately transmits it.

But what happens if:

- 100,000 packets arrive at the same time?
- The network card can only send 10,000 packets per second?
- One application should get higher priority than another?
- You want to limit a container to **10 Mbps**?
- You want to simulate a **300 ms** network delay?

Linux needs a traffic manager.

The answer is:

# Traffic Control (`tc`)

Traffic Control is the Linux kernel subsystem responsible for:

- Packet scheduling
- Bandwidth limiting
- Traffic shaping
- Traffic policing
- Packet prioritization
- Delay simulation
- Packet loss simulation

Cloud providers, Kubernetes, Docker, ISPs, and telecom systems all rely on this subsystem.

After this chapter, you'll understand how Linux controls the speed and behavior of network traffic.

---

# 👑 Linux Kernel & Networking Mastery

# Module 10 — Advanced Linux Networking Internals

# Chapter 64 — Linux Traffic Control (`tc`): Controlling Bandwidth, Latency, and Packet Scheduling

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Traffic Control exists
> - What is `tc`?
> - Queuing Disciplines (qdiscs)
> - Packet Scheduling
> - Traffic Shaping
> - Traffic Policing
> - Bandwidth Limiting
> - Packet Priorities
> - Delay & Packet Loss Simulation
> - Docker & Kubernetes Usage

---

# 📖 Why Do We Need Traffic Control?

Suppose:

```
Application A

↓

500 Mbps
```

```
Application B

↓

800 Mbps
```

Network card:

```
1 Gbps
```

Total demand:

```
1.3 Gbps
```

Question:

```
Who Gets Sent First?
```

Linux must decide.

---

# Without Traffic Control

Packets arrive:

```
Packet

↓

Packet

↓

Packet

↓

Packet
```

NIC becomes overloaded.

Packets may be dropped unpredictably.

---

# With Traffic Control

Linux creates:

```
Queue

↓

Scheduling

↓

Transmission
```

Packets leave the system in a controlled manner.

---

# What is Traffic Control?

Traffic Control is the Linux subsystem responsible for:

✔ Queuing packets

✔ Scheduling packets

✔ Limiting bandwidth

✔ Prioritizing traffic

✔ Simulating network conditions

---

# Airport Analogy

Imagine an airport.

```
Passengers

↓

Security Queue

↓

Priority Boarding

↓

Flight
```

Packets behave the same way.

Linux decides:

```
Who Goes First
```

---

# Where Does `tc` Work?

Packet flow:

```
Application

↓

TCP/IP

↓

Traffic Control

↓

NIC

↓

Internet
```

Traffic Control operates before packets are transmitted by the network interface.

---

# Queue

Packets are not always transmitted immediately.

Linux stores them in a:

```
Queue
```

```
Packet

↓

Queue

↓

NIC

↓

Internet
```

---

# What is a qdisc?

A **Queuing Discipline (qdisc)** defines:

```
How Packets

Wait

and

Leave
```

Think of it as the algorithm controlling the queue.

---

# Default qdisc

Every network interface has a default qdisc.

View it:

```bash
tc qdisc show
```

Example output:

```text
qdisc fq_codel ...
```

Modern Linux distributions often use `fq_codel` by default.

---

# Packet Scheduler

Suppose queue contains:

```
Packet A

Packet B

Packet C
```

Scheduler decides:

```
A

↓

B

↓

C
```

Or another order,

depending on policy.

---

# FIFO Scheduling

Simplest algorithm:

```
First In

↓

First Out
```

Packets leave in the same order they arrive.

---

# Priority Scheduling

Suppose:

```
Voice Traffic
```

```
Video Traffic
```

```
File Download
```

Linux can send:

```
Voice

↓

Video

↓

Download
```

Higher-priority traffic experiences lower delay.

---

# Traffic Shaping

Traffic shaping controls:

```
Transmission Rate
```

Suppose application sends:

```
1 Gbps
```

Linux limits output:

```
100 Mbps
```

Extra packets remain queued until they can be transmitted.

---

# Visual Representation

```
Application

↓

1000 Mbps

↓

Traffic Shaper

↓

100 Mbps

↓

NIC
```

---

# Traffic Policing

Policing is different.

Instead of slowing traffic,

Linux simply discards packets exceeding the configured rate.

```
Too Fast

↓

Drop
```

No queue is maintained for excess traffic.

---

# Shaping vs Policing

Traffic Shaping:

```
Too Fast

↓

Wait

↓

Send Later
```

Traffic Policing:

```
Too Fast

↓

Drop Immediately
```

---

# Token Bucket

Many shaping algorithms use a:

```
Token Bucket
```

Imagine:

```
Bucket

↓

Tokens
```

Each transmitted packet consumes a token.

When no tokens remain,

packets must wait until new tokens are added.

This allows average rate control while permitting short bursts.

---

# Bandwidth Limit

Example:

```bash
tc qdisc add \
dev eth0 \
root tbf \
rate 10mbit
```

Linux limits transmission to:

```
10 Mbps
```

---

# Delay Simulation

Need to test slow networks?

Example:

```bash
tc qdisc add \
dev eth0 \
root netem \
delay 100ms
```

Every packet waits:

```
100 ms
```

before transmission.

---

# Packet Loss Simulation

Example:

```bash
tc qdisc add \
dev eth0 \
root netem \
loss 5%
```

Linux randomly drops approximately:

```
5%

of Packets
```

Useful for testing unreliable networks.

---

# Packet Reordering

Linux can even simulate:

```
Out-of-Order Delivery
```

Applications can be tested under real-world network conditions.

---

# Jitter

Real networks rarely have constant delay.

Example:

```text
100 ms

±20 ms
```

Traffic Control can simulate varying latency (jitter).

---

# Docker Example

Containers may need:

```
Maximum

↓

20 Mbps
```

Traffic Control can be used (often by higher-level orchestration or networking tools) to enforce bandwidth limits.

---

# Kubernetes Example

Some Kubernetes networking plugins use Linux Traffic Control to implement:

- Bandwidth limits
- QoS policies
- Network shaping

The exact implementation depends on the CNI plugin.

---

# Cloud Providers

Traffic Control is used extensively by:

- AWS
- Azure
- Google Cloud
- ISPs
- Telecom Networks

Examples include:

- Bandwidth guarantees
- Rate limiting
- Traffic prioritization

---

# View qdiscs

Run:

```bash
tc qdisc show
```

Observe active queuing disciplines for each interface.

---

# Packet Flow

```
Application

↓

Socket

↓

TCP/IP

↓

Queue

↓

Scheduler

↓

NIC

↓

Internet
```

---

# Complete Architecture

```
Packet

↓

Traffic Control

↓

Queue

↓

Scheduler

↓

Bandwidth Check

↓

NIC

↓

Internet
```

---

# Why Traffic Control Matters

Everything depends on it.

Examples:

- Video Calls
- Cloud Networking
- Kubernetes
- Docker
- ISPs
- QoS
- Network Simulation

Without Traffic Control,

Linux cannot intelligently manage outgoing traffic.

---

# Hands-on Lab

## Lab 1 — View qdiscs

```bash
tc qdisc show
```

---

## Lab 2 — Add Delay

```bash
sudo tc qdisc add \
dev eth0 \
root netem \
delay 100ms
```

---

## Lab 3 — Remove Delay

```bash
sudo tc qdisc del \
dev eth0 \
root
```

---

## Lab 4 — Simulate Packet Loss

```bash
sudo tc qdisc add \
dev eth0 \
root netem \
loss 10%
```

---

## Lab 5 — Limit Bandwidth

```bash
sudo tc qdisc add \
dev eth0 \
root tbf \
rate 10mbit \
burst 32kbit \
latency 400ms
```

Observe the effect using a network speed test or file transfer.

---

# Interview Questions

## What is Linux Traffic Control?

Traffic Control is the Linux kernel subsystem responsible for packet scheduling, shaping, policing, and bandwidth management.

---

## What is a qdisc?

A qdisc (Queuing Discipline) defines how packets are queued and transmitted on a network interface.

---

## What is the difference between Traffic Shaping and Traffic Policing?

Traffic Shaping delays excess traffic to meet a configured rate.

Traffic Policing immediately drops excess traffic.

---

## What is the purpose of the Token Bucket algorithm?

It controls average transmission rates while allowing limited bursts of traffic.

---

## Can Linux simulate network latency?

Yes.

The `netem` qdisc can simulate delay, packet loss, jitter, duplication, and reordering.

---

## Where does Traffic Control operate?

It operates in the Linux networking stack before packets are transmitted by the network interface.

---

## Does Kubernetes use Traffic Control?

Some CNI plugins use Linux Traffic Control to implement bandwidth management and QoS features.

---

# Summary

Linux Traffic Control manages how packets leave the system.

```
Application

↓

TCP/IP

↓

Traffic Control

↓

Queue

↓

Scheduler

↓

Bandwidth Policy

↓

NIC

↓

Internet
```

Key concepts:

- Traffic Control manages outgoing packet transmission.
- qdiscs define queue behavior.
- Traffic Shaping delays packets to enforce bandwidth limits.
- Traffic Policing drops packets that exceed configured rates.
- The Token Bucket algorithm enables controlled bursts.
- `netem` simulates delay, loss, jitter, duplication, and reordering.
- Traffic Control is widely used in cloud networking, Kubernetes, Docker, and telecommunications.

At this point, you've mastered how Linux schedules and controls network traffic before it leaves the machine.

---

# Next Chapter

## Chapter 65 — eBPF Fundamentals: Extending the Linux Kernel Without Writing Kernel Modules

We'll explore:

- What is eBPF?
- Why eBPF was introduced
- eBPF Virtual Machine
- Safe Kernel Execution
- Hooks
- Maps
- Verifier
- JIT Compilation
- Networking with eBPF
- Docker, Kubernetes, and Cilium

> **This chapter introduces one of the biggest innovations in the Linux kernel over the last decade and the technology powering many next-generation networking and observability tools.**