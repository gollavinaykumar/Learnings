# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 142 — Linux Capabilities — Splitting Root Privileges into Smaller Pieces

---

# Most beginners think:

```
User

↓

No Permission

-----------------------

Root

↓

All Permissions
```

Only two choices.

This was true in early Unix.

But modern Linux is much smarter.

Instead of giving a program **all root privileges**, Linux can grant **only the exact permissions it needs**.

This system is called:

```
Linux Capabilities
```

---

# Learning Objectives

After this chapter you will understand:

- Why root is too powerful
- What capabilities are
- Capability sets
- File capabilities
- Process capabilities
- Ambient capabilities
- Common Linux capabilities
- Security benefits

---

# The Problem

Suppose you write:

```
Network Monitoring Tool
```

It needs:

```
Raw Socket
```

Without capabilities,

the only option was:

```
Run As Root
```

Problem:

Now the program can also:

- Delete files
- Mount disks
- Change users
- Shut down the system

Way too much power.

---

# Linux Solution

Instead of:

```
Root

↓

Everything
```

Linux says:

```
Give Only

↓

Needed Permission
```

Example:

```
Network Capability
```

Nothing else.

---

# Think About an Office

Old system:

```
Master Key

↓

Every Room
```

Modern system:

```
Server Room Key

↓

Only Server Room
```

Capabilities work exactly like specialized keys.

---

# What is a Capability?

Definition:

> **A capability is a specific kernel privilege that can be granted independently of full root access.**

Instead of one huge privilege,

Linux divides it into dozens of smaller privileges.

---

# Visualization

```
Root

↓

Split

↓

Capability A

↓

Capability B

↓

Capability C
```

---

# Why This Matters

Suppose your program is hacked.

Without capabilities:

```
Attacker

↓

Full Root
```

With capabilities:

```
Attacker

↓

Only Limited Privileges
```

Much safer.

---

# Common Capabilities

Examples:

```
CAP_NET_ADMIN

↓

Network Administration

----------------------

CAP_SYS_PTRACE

↓

Debug Other Processes

----------------------

CAP_SYS_TIME

↓

Change System Clock

----------------------

CAP_SYS_BOOT

↓

Reboot System
```

Each grants only one area of authority.

---

# `CAP_NET_ADMIN`

Allows operations such as:

- Configure network interfaces
- Change routing tables
- Manage traffic control

Without giving unrelated privileges.

---

# `CAP_NET_RAW`

Allows:

```
Raw Sockets

↓

Packet Capture

↓

ICMP
```

Tools like packet analyzers often require this capability.

---

# `CAP_SYS_PTRACE`

Required for many advanced debugging operations.

Example:

```
GDB

↓

ptrace()

↓

Other Process
```

Linux checks this capability in some situations.

---

# `CAP_SYS_ADMIN`

One of the broadest capabilities.

Provides many administrative operations.

Because it covers so much functionality,

it's often described as the "new root."

Applications should avoid requesting it unless absolutely necessary.

---

# `CAP_CHOWN`

Allows:

```
Change File Owner
```

Without granting every other administrative privilege.

---

# `CAP_SETUID`

Allows changing process user IDs.

Important for authentication programs.

---

# `CAP_DAC_OVERRIDE`

Normally:

```
Permission Denied
```

With this capability:

```
Permission Checks

↓

Bypassed
```

Useful for specific system programs,

but powerful.

---

# Process Capability Sets

Linux doesn't keep just one capability list.

Each process maintains several capability sets.

Important ones include:

```
Permitted

↓

Effective

↓

Inheritable

↓

Bounding

↓

Ambient
```

---

# Permitted Set

Think of it as:

```
Maximum Allowed
```

Capabilities the process is allowed to enable.

---

# Effective Set

Capabilities currently active.

Kernel checks this set during permission decisions.

---

# Inheritable Set

Capabilities that may be passed to child programs across an `execve()` call, subject to Linux capability rules.

---

# Bounding Set

Upper limit.

A process cannot acquire capabilities outside its bounding set.

---

# Ambient Set

Introduced to make capability inheritance more practical for certain applications.

Useful for long-running services launching helper programs.

---

# File Capabilities

Capabilities can also be attached directly to executables.

Instead of:

```
Run As Root
```

Linux says:

```
Executable

↓

Specific Capability
```

---

# Example

Suppose:

```
packet_capture
```

Needs raw sockets.

Instead of:

```bash
sudo packet_capture
```

Grant:

```
CAP_NET_RAW
```

Only.

---

# Visualization

```
Executable

↓

Capability

↓

Run Normally
```

---

# Traditional `setuid`

Old Unix solution:

```
Executable

↓

setuid root
```

Meaning:

```
Entire Program

↓

Root
```

Large security risk.

Capabilities are a much more fine-grained alternative.

---

# Viewing Capabilities

Useful command:

```bash
getcap file
```

Example:

```
program

↓

cap_net_raw
```

---

# Setting Capabilities

Common tool:

```bash
setcap
```

Example:

```bash
setcap cap_net_raw+ep ./program
```

Meaning:

```
Effective

+

Permitted
```

for that executable.

---

# Relationship with `execve()`

When a program starts:

```
execve()

↓

Kernel

↓

Load Executable

↓

Apply File Capabilities

↓

Process Begins
```

Capabilities become part of the new process according to Linux capability rules.

---

# Capability Check

Suppose:

```
Program

↓

Open Raw Socket
```

Kernel asks:

```
Effective Capability?

↓

Yes

↓

Allow

-----------------------

No

↓

EPERM
```

---

# Relationship with Containers

Containers often run:

```
Root

↓

Inside Container
```

But with many capabilities removed.

This limits what containerized processes can do on the host.

---

# Docker Example

Docker commonly drops many capabilities by default.

Additional capabilities can be added only when required.

This follows the principle of least privilege.

---

# Kubernetes Example

Security contexts can specify:

```
Add Capability

↓

Drop Capability
```

for containers.

---

# Real-World Example — `ping`

Historically:

```
setuid root
```

Modern Linux systems often grant only:

```
CAP_NET_RAW
```

allowing ICMP packets without full root privileges.

---

# Real-World Example — Wireshark

Packet capture utilities may require:

```
CAP_NET_RAW

↓

CAP_NET_ADMIN
```

instead of unrestricted root access.

---

# Security Principle

```
Least Privilege
```

Give a program:

```
Exactly

↓

What It Needs

↓

Nothing More
```

Capabilities are designed around this principle.

---

# Common Mistakes

---

## Thinking Capabilities Replace Permissions

Normal file permissions still apply.

Capabilities provide additional kernel privileges.

---

## Assuming `CAP_SYS_ADMIN` Is Safe

It grants a very broad set of administrative powers.

Avoid it whenever possible.

---

## Giving More Capabilities Than Necessary

Always grant the minimum required.

---

## Confusing File Capabilities with Process Capabilities

File capabilities are attached to executables.

Process capabilities exist while a process is running.

---

# Hands-on Labs

## Lab 1

View capabilities:

```bash
getcap /bin/ping
```

Observe whether your system uses file capabilities.

---

## Lab 2

Inspect current process capabilities:

```bash
cat /proc/self/status
```

Look for:

```
CapEff

CapPrm

CapBnd
```

---

## Lab 3

Read:

```bash
man 7 capabilities
```

Explore the complete capability list.

---

## Lab 4

Compare:

- Traditional `setuid`
- File capabilities

Understand their security differences.

---

## Lab 5

Research Docker's default capability set and why many capabilities are dropped.

---

# Interview Questions

### What is a Linux capability?

A fine-grained kernel privilege that can be granted independently of full root access.

---

### Why were capabilities introduced?

To replace the all-or-nothing root privilege model with the principle of least privilege.

---

### What is `CAP_NET_RAW` used for?

Creating raw sockets and performing low-level network operations.

---

### What is the difference between file capabilities and process capabilities?

File capabilities are stored on executables.

Process capabilities exist in the running process.

---

### Why should `CAP_SYS_ADMIN` be avoided when possible?

Because it grants a very broad range of administrative operations and significantly increases risk.

---

# Summary

Traditional Model:

```
User

↓

Limited

-----------------------

Root

↓

Everything
```

Capability Model:

```
Program

↓

CAP_NET_RAW

↓

Only Raw Sockets
```

Capability Flow:

```
Executable

↓

execve()

↓

Kernel

↓

Process Capabilities
```

Common Capabilities:

| Capability | Purpose |
|------------|---------|
| `CAP_NET_RAW` | Raw sockets |
| `CAP_NET_ADMIN` | Network administration |
| `CAP_SYS_PTRACE` | Debugging/tracing |
| `CAP_CHOWN` | Change ownership |
| `CAP_SETUID` | Change user IDs |
| `CAP_SYS_ADMIN` | Broad administrative operations |

## Key Takeaways

- Linux capabilities divide root privileges into smaller, focused permissions.
- They implement the principle of least privilege.
- Executables can carry file capabilities instead of requiring full root access.
- Running processes maintain multiple capability sets.
- Containers make extensive use of capabilities to improve isolation.
- Capabilities are a fundamental part of modern Linux security.

---

# Next Chapter

## Chapter 143 — Linux Namespaces — How Containers Create Isolated Worlds

You'll learn:

- What namespaces are
- PID namespaces
- Mount namespaces
- Network namespaces
- User namespaces
- IPC namespaces
- UTS namespaces
- Time namespaces
- How Docker creates the illusion that a container has its own operating system