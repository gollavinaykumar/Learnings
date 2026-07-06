# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 152 — eBPF Hook Points — Where Exactly Does an eBPF Program Run?

---

# In the previous chapter you learned:

```
eBPF Program

↓

Runs Inside Kernel
```

But...

A natural question appears:

> **Where exactly inside the kernel does it run?**

The answer is:

```
Hook Points
```

Think of them as places where Linux says:

```
"Something important just happened."

↓

"Do you want to run an eBPF program here?"
```

Modern Linux contains **dozens of hook points**.

Understanding them is the key to mastering eBPF.

---

# Learning Objectives

After this chapter you will understand:

- What hook points are
- XDP
- TC
- Kprobes
- Kretprobes
- Tracepoints
- Uprobes
- USDT
- LSM Hooks
- Cgroup Hooks
- Socket Hooks

---

# The Big Idea

Imagine the Linux kernel as a huge factory.

```
Packet Arrives

↓

Driver

↓

TCP

↓

Application
```

At many places,

Linux allows:

```
Pause

↓

Run eBPF

↓

Continue
```

Those places are:

```
Hook Points
```

---

# Visualization

```
Kernel Event

↓

Hook

↓

eBPF Program

↓

Continue Execution
```

---

# Categories of Hook Points

Most eBPF hook points fall into these groups:

```
Networking

↓

Kernel Functions

↓

User Functions

↓

Tracing

↓

Security

↓

Containers

↓

Sockets
```

---

# 1. XDP (Express Data Path)

Probably the fastest hook.

Packet arrives:

```
NIC

↓

XDP

↓

Kernel Networking Stack
```

Your eBPF program executes **before** most networking processing.

---

# Why XDP?

Imagine:

```
Attack

↓

10 Million Packets
```

Drop them immediately.

Instead of:

```
TCP

↓

Socket

↓

Application
```

Much faster.

---

# Visualization

Without XDP

```
Packet

↓

Driver

↓

Kernel

↓

TCP

↓

Application
```

With XDP

```
Packet

↓

XDP

↓

Drop
```

Huge performance improvement.

---

# Typical XDP Actions

Your program may decide:

```
Pass Packet

↓

Drop Packet

↓

Redirect Packet

↓

Transmit Back
```

depending on the hook's supported actions.

---

# Real-World Example

Cloudflare

↓

Packet arrives

↓

XDP

↓

Drop DDoS Traffic

Millions of packets never reach higher networking layers.

---

# 2. TC (Traffic Control)

Hook occurs later.

Flow:

```
Packet

↓

Kernel Networking

↓

TC

↓

Socket
```

Useful for:

- Traffic shaping
- QoS
- Packet modification
- Packet classification

---

# Difference

```
XDP

↓

Very Early

------------------------

TC

↓

Later
```

XDP is generally faster.

TC provides richer networking context.

---

# 3. Kprobes

Suppose kernel function:

```c
tcp_connect()
```

You want to observe:

```
Every Call
```

Without modifying kernel source.

Linux provides:

```
Kprobe
```

---

# Visualization

```
Kernel Function

↓

Kprobe

↓

eBPF

↓

Original Function
```

---

# Example

Every time:

```
do_sys_open()
```

runs,

your eBPF program executes.

---

# 4. Kretprobes

Normal Kprobe:

```
Before Function
```

Kretprobe:

```
After Function Returns
```

Useful for measuring:

- Execution time
- Return values
- Success/failure

---

# Visualization

```
Function Start

↓

...

↓

Return

↓

Kretprobe
```

---

# 5. Tracepoints

Linux developers create stable instrumentation points.

Unlike Kprobes,

they are designed specifically for tracing.

Example:

```
Process Created

↓

Tracepoint

↓

eBPF
```

---

# Why Tracepoints?

Kernel functions can change.

Tracepoints are intended to provide a more stable tracing interface across kernel versions.

---

# Common Tracepoints

Examples include:

```
Process Fork

↓

Process Exit

↓

Scheduler Switch

↓

Block I/O

↓

System Calls
```

---

# 6. Uprobes

Kernel tracing is useful.

But what about:

```
User Programs?
```

Suppose:

```
Python

↓

malloc()

↓

OpenSSL
```

You can attach to user-space functions using:

```
Uprobe
```

---

# Visualization

```
Application

↓

Function

↓

Uprobe

↓

Continue
```

---

# Example

Every time:

```
malloc()
```

executes,

your eBPF program records statistics.

---

# 7. USDT (User Statically Defined Tracing)

Applications can expose predefined tracing points.

Examples:

- Databases
- Java runtimes
- Node.js
- Other instrumented software

eBPF can attach to these stable user-space probes.

---

# Visualization

```
Node.js

↓

USDT

↓

eBPF
```

---

# 8. LSM Hooks

LSM means:

```
Linux Security Modules
```

Security events:

```
Open File

↓

Create Process

↓

Network Access
```

eBPF can enforce or observe security policies at these hooks.

---

# Example

```
File Open

↓

LSM Hook

↓

eBPF

↓

Allow

or

Deny
```

---

# 9. Cgroup Hooks

Remember:

```
cgroups
```

eBPF programs can attach to cgroup-related operations.

Examples:

```
Socket Creation

↓

Packet Send

↓

Device Access
```

Useful for container-aware policies.

---

# 10. Socket Hooks

Applications create sockets.

eBPF can observe or influence certain socket operations.

Examples include:

```
Socket Create

↓

Bind

↓

Connect

↓

Send

↓

Receive
```

---

# Visualization

```
Socket

↓

Hook

↓

eBPF
```

---

# 11. Raw Tracepoints

Similar to tracepoints,

but provide lower-level access with less abstraction.

Often used when maximum performance is important.

---

# Relationship Between Hooks

```
Packet

↓

XDP

↓

Driver

↓

TC

↓

Socket

↓

Application
```

At every stage,

different hook types may exist.

---

# Relationship with Maps

Every hook can access:

```
eBPF Maps
```

Example:

```
Packet

↓

Hook

↓

Increment Counter

↓

Continue
```

User-space later reads the counter.

---

# Relationship with the Verifier

Regardless of hook type:

```
Program

↓

Verifier

↓

Approved

↓

Attach
```

Unsafe programs never reach the hook.

---

# Real-World Example — Cilium

```
Packet

↓

XDP

↓

Policy

↓

TC

↓

Forward
```

Multiple hook types cooperate.

---

# Real-World Example — Performance Monitoring

```
System Call

↓

Tracepoint

↓

Map

↓

Dashboard
```

---

# Real-World Example — Security

```
Open File

↓

LSM Hook

↓

Check Policy

↓

Allow/Deny
```

---

# Real-World Example — Observability

```
Kernel Function

↓

Kprobe

↓

Latency Counter

↓

Grafana
```

---

# Choosing the Right Hook

| Goal | Common Hook |
|--------|-------------|
| Fast packet filtering | XDP |
| Traffic shaping | TC |
| Trace kernel function | Kprobe |
| Measure function return | Kretprobe |
| Stable kernel tracing | Tracepoint |
| Trace user program | Uprobe |
| Trace application-defined events | USDT |
| Security policy | LSM |
| Container-aware networking | Cgroup Hooks |

---

# Common Mistakes

---

## Thinking Every Hook Is Equal

Different hooks execute at different stages and have different capabilities.

---

## Always Choosing Kprobes

When stable tracepoints exist,

they are often preferred for long-term compatibility.

---

## Using TC When XDP Is Needed

For extremely high-performance packet dropping,

XDP is often the better choice.

---

## Ignoring Verifier Restrictions

Every eBPF program must satisfy verifier rules regardless of hook type.

---

# Hands-on Labs

## Lab 1

Install:

```bash
bpftool
```

List available program types and attachments.

---

## Lab 2

Use:

```bash
bpftrace
```

Attach to a tracepoint and observe events.

---

## Lab 3

Explore Linux tracepoints under:

```text
/sys/kernel/tracing/events
```

(or `/sys/kernel/debug/tracing/events` on some systems).

---

## Lab 4

Research XDP packet processing diagrams.

Compare them with TC.

---

## Lab 5

Study how Cilium combines multiple hook types for networking and security.

---

# Interview Questions

### What is an eBPF hook point?

A location in the kernel or user-space execution path where an eBPF program can be attached and executed.

---

### What is the difference between XDP and TC?

XDP runs very early in packet processing for maximum performance.

TC runs later with richer networking context.

---

### What is a Kprobe?

A mechanism for attaching an eBPF program before a kernel function executes.

---

### What is a Uprobe?

A mechanism for tracing functions in user-space applications.

---

### Why are tracepoints often preferred over Kprobes?

Because tracepoints are designed to provide a more stable tracing interface across kernel changes.

---

# Summary

Hook Hierarchy:

```
Packet

↓

XDP

↓

Driver

↓

TC

↓

Socket

↓

Application
```

Tracing Hooks:

```
Kernel Function

↓

Kprobe

↓

Kretprobe

------------------------

User Function

↓

Uprobe

↓

USDT
```

Security Hooks:

```
File

↓

LSM

↓

Policy

↓

Allow/Deny
```

Major Hook Types:

| Hook | Purpose |
|------|---------|
| XDP | Earliest packet processing |
| TC | Traffic control |
| Kprobe | Before kernel function |
| Kretprobe | After kernel function |
| Tracepoint | Stable kernel tracing |
| Uprobe | User-space tracing |
| USDT | Application-defined tracing |
| LSM | Security enforcement |
| Cgroup | Container-aware policies |
| Socket | Socket operations |

## Key Takeaways

- eBPF programs execute only at specific hook points.
- Different hooks serve different purposes and performance characteristics.
- XDP provides extremely fast packet processing.
- Tracepoints offer stable kernel instrumentation.
- Kprobes and Uprobes enable dynamic tracing.
- Choosing the correct hook is one of the most important design decisions in eBPF development.

---

# Next Chapter

## Chapter 153 — eBPF Maps — The Shared Memory That Connects Kernel Programs and User Space

You'll learn:

- Why eBPF programs need maps
- Hash maps
- Arrays
- Ring buffers
- Per-CPU maps
- LRU maps
- Perf event arrays
- Queue and stack maps
- Pinning maps
- How kernel-space eBPF programs exchange data with user-space applications