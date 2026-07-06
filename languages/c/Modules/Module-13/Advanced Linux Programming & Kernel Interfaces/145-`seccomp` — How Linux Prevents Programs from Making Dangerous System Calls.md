# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 145 — `seccomp` — How Linux Prevents Programs from Making Dangerous System Calls

---

# Most developers think:

```
Program

↓

Can Call Any System Call
```

Actually...

Modern Linux can say:

```
Program

↓

Allowed

↓

read()

↓

write()

↓

exit()

------------------------

Program

↓

Blocked

↓

mount()

↓

ptrace()

↓

reboot()
```

Even if the program tries.

This protection is called:

```
seccomp

(Secure Computing Mode)
```

It is one of the most important Linux sandboxing technologies.

Without it,

containers and browsers would be significantly less secure.

---

# Learning Objectives

After this chapter you will understand:

- What `seccomp` is
- Why it exists
- System call filtering
- Allow lists
- Deny lists
- BPF filters
- Docker seccomp profiles
- Browser sandboxing
- Process security

---

# The Problem

Imagine:

```
PDF Viewer
```

It should only:

- Read a file
- Render text
- Display graphics

Should it also be allowed to:

```
Reboot Linux?

↓

Mount Filesystems?

↓

Debug Other Processes?

↓

Load Kernel Modules?
```

Of course not.

---

# Another Example

Suppose an attacker exploits a bug.

Without restrictions:

```
Malware

↓

Any System Call

↓

Entire Kernel Surface
```

Too dangerous.

---

# Linux Solution

Before executing a system call,

Linux checks:

```
Allowed?

↓

Yes

↓

Run

-----------------------

No

↓

Block
```

---

# What is `seccomp`?

Definition:

> **`seccomp` is a Linux kernel security feature that restricts which system calls a process may execute.**

Think of it as:

```
Firewall

↓

For System Calls
```

---

# Visualization

```
Program

↓

System Call

↓

seccomp Filter

↓

Kernel
```

---

# Normal Execution

Without `seccomp`:

```
Program

↓

Any System Call

↓

Kernel
```

---

# With `seccomp`

```
Program

↓

System Call

↓

Allowed?

↓

Yes

↓

Kernel

------------------------

No

↓

Blocked
```

---

# Think About Airport Security

Passenger:

```
Board Plane
```

Security:

```
Check Ticket

↓

Check Identity

↓

Allow

or

Reject
```

`seccomp` performs a similar check for every system call.

---

# Early `seccomp`

Original mode:

```
Only

↓

read()

↓

write()

↓

exit()

↓

sigreturn()
```

Very restrictive.

Useful for simple sandboxed programs.

---

# Modern `seccomp`

Today Linux supports:

```
seccomp-bpf
```

Instead of a fixed list,

you provide a programmable filter.

Much more flexible.

---

# What is BPF?

BPF stands for:

```
Berkeley Packet Filter
```

Originally developed for network packet filtering.

Linux later extended it for many purposes,

including:

```
seccomp
```

(You'll study eBPF in later chapters.)

---

# How Filtering Works

Program calls:

```c
mount()
```

Kernel checks:

```
BPF Filter

↓

Allowed?

↓

No

↓

Reject
```

---

# Allowed Example

```
read()

↓

Filter

↓

Allow

↓

Kernel
```

---

# Blocked Example

```
ptrace()

↓

Filter

↓

Reject

↓

EPERM
```

or another configured action,

depending on the filter.

---

# Allow List

Recommended approach.

```
Allow

↓

read()

↓

write()

↓

close()

↓

exit()
```

Everything else is denied.

---

# Deny List

Alternative:

```
Allow Everything

↓

Except

↓

mount()

↓

reboot()

↓

ptrace()
```

Generally weaker,

because new system calls may appear over time.

---

# Default Action

A `seccomp` filter specifies what happens when a system call doesn't match an allow rule.

Possible actions include:

```
Allow

↓

Return Error

↓

Kill Process

↓

Notify

↓

Trap
```

---

# Relationship with System Calls

Remember Chapter 133.

Flow:

```
Program

↓

syscall

↓

seccomp

↓

Kernel
```

`seccomp` checks occur before the kernel performs the requested operation.

---

# Relationship with Containers

Docker containers normally run with:

```
Namespaces

↓

cgroups

↓

Capabilities

↓

seccomp
```

All four technologies work together.

---

# Docker Default Profile

Docker ships with a default `seccomp` profile.

Many potentially dangerous system calls are blocked by default,

while common application calls remain available.

---

# Browser Example

Modern browsers isolate renderer processes.

Renderer:

```
Render HTML

↓

Render CSS

↓

JavaScript
```

Should **not**:

```
Reboot Linux

↓

Mount Filesystems

↓

Load Kernel Modules
```

`seccomp` helps enforce that restriction.

---

# Sandboxing

```
Application

↓

Sandbox

↓

Limited System Calls
```

Even if compromised,

the attacker has fewer kernel interfaces available.

---

# Performance

Good news.

`seccomp` filtering is designed to be efficient.

For most applications,

the overhead is relatively small compared to the security benefits.

---

# Relationship with Capabilities

Capabilities determine:

```
Privilege
```

`seccomp` determines:

```
Which System Calls

↓

May Be Executed
```

Different security layers.

---

# Relationship with Namespaces

Namespaces isolate resources.

`seccomp` restricts kernel entry points.

Together they provide stronger security.

---

# Relationship with `ptrace`

Security policies may prevent debuggers from attaching,

and `seccomp` filters can also restrict tracing-related system calls.

---

# Real-World Example — Chrome

Chrome renderer:

```
Namespaces

↓

seccomp

↓

Sandbox
```

Designed to reduce the impact of browser vulnerabilities.

---

# Real-World Example — Docker

Default container:

```
Limited System Calls

↓

Safer Container
```

Without application developers needing to configure everything manually.

---

# Real-World Example — Kubernetes

Kubernetes can apply `seccomp` profiles to Pods,

adding another layer of workload protection.

---

# Real-World Example — System Services

Security-focused Linux services often combine:

- Capabilities
- `seccomp`
- Namespaces

to minimize attack surface.

---

# Common Mistakes

---

## Thinking `seccomp` Replaces Permissions

It does not.

Linux permissions and capabilities still apply.

---

## Confusing `seccomp` with Capabilities

Capabilities control privileges.

`seccomp` controls which system calls are permitted.

---

## Using Deny Lists for Everything

Allow lists generally provide stronger long-term security.

---

## Forgetting `seccomp` Only Filters System Calls

It does not isolate filesystems, networking, or processes by itself.

---

# Hands-on Labs

## Lab 1

Inspect Docker's default `seccomp` profile.

Observe which system calls are restricted.

---

## Lab 2

Read:

```bash
man 2 seccomp
```

and

```bash
man 3 seccomp
```

Learn about the programming interface.

---

## Lab 3

Research `libseccomp`,

the library commonly used to build `seccomp` filters.

---

## Lab 4

Compare:

- Namespaces
- cgroups
- Capabilities
- `seccomp`

Understand the different role of each technology.

---

## Lab 5

Study a real browser or container sandbox architecture.

Identify where `seccomp` fits.

---

# Interview Questions

### What is `seccomp`?

A Linux security mechanism that filters and restricts the system calls a process may execute.

---

### Why is `seccomp` important?

It reduces the kernel attack surface available to compromised applications.

---

### What is the difference between capabilities and `seccomp`?

Capabilities determine what privileged operations are allowed.

`seccomp` determines which system calls may be executed.

---

### Why are allow lists generally preferred?

Because only explicitly permitted system calls are available, reducing exposure to future or unexpected system calls.

---

### Where is `seccomp` commonly used?

Containers, browsers, sandboxed applications, and hardened Linux services.

---

# Summary

Architecture:

```
Application

↓

System Call

↓

seccomp Filter

↓

Kernel
```

Container Security Stack:

```
Namespaces

↓

Isolation

+

cgroups

↓

Resource Limits

+

Capabilities

↓

Privileges

+

seccomp

↓

System Call Filtering
```

Typical Allowed Calls:

```
read()

↓

write()

↓

close()

↓

exit()
```

Potentially Restricted Calls:

```
mount()

↓

ptrace()

↓

reboot()

↓

kexec_load()
```

## Key Takeaways

- `seccomp` filters system calls before the kernel executes them.
- It significantly reduces the attack surface of Linux applications.
- Modern implementations use BPF-based filters.
- Docker, Kubernetes, and browsers rely heavily on `seccomp`.
- `seccomp` complements namespaces, cgroups, and capabilities.
- It is a cornerstone of modern Linux sandboxing.

---

# Next Chapter

## Chapter 146 — `inotify` — How Linux Detects File Changes Without Constantly Polling

You'll learn:

- What `inotify` is
- File system event notifications
- Watching files and directories
- Create, modify, delete, move events
- Recursive watching considerations
- How editors, IDEs, Dropbox, VS Code, and webpack instantly detect file changes