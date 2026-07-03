By now you've learned several Linux security mechanisms.

```
Namespaces

↓

Isolation
```

```
cgroups

↓

Resource Limits
```

```
Capabilities

↓

Fine-grained Privileges
```

```
seccomp

↓

System Call Filtering
```

You might think:

> "Containers are now completely secure."

Not yet.

Imagine this situation.

Your application has permission to:

```
read()
```

The syscall is allowed.

The process has the required capability.

The file permissions are correct.

Question:

Should the application be allowed to read:

```
/etc/shadow
```

or

```
/root/.ssh/id_rsa
```

Maybe not.

Linux needs **another layer** that can say:

> "Even if Linux permissions allow it, this application is still NOT allowed."

That layer is:

# Linux Security Modules (LSM)

LSMs provide **Mandatory Access Control (MAC)**.

They define **what an application is allowed to do**, regardless of the user running it.

Docker, Kubernetes, Red Hat, Ubuntu, Android, and many enterprise Linux systems rely heavily on LSMs.

After this chapter, you'll understand the final major security layer protecting Linux containers.

---

# 👑 Linux Kernel & Networking Mastery

# Module 8 — Linux Security Internals

# Chapter 57 — Linux Security Modules (LSM): AppArmor, SELinux, and Mandatory Access Control

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a Linux Security Module?
> - DAC vs MAC
> - Why LSMs exist
> - AppArmor
> - SELinux
> - Security Labels
> - Policy Enforcement
> - Docker Integration
> - Kubernetes Integration
> - Complete Linux Security Stack

---

# 📖 Why Do We Need Another Security Layer?

Suppose:

```
Application

↓

Running as root
```

Capabilities allow the operation.

seccomp allows the syscall.

Linux file permissions allow access.

Can the application now read:

```
/etc/shadow
```

Without another layer:

```
Yes
```

That may still be too much power.

---

# Linux Security Before LSM

Traditional Linux relied mainly on:

```
User

↓

Group

↓

Permission Bits
```

Example:

```
-rwxr-x---
```

This is called:

```
Discretionary Access Control

(DAC)
```

---

# What is DAC?

DAC means:

The owner of a file controls who can access it.

Example:

```
Owner

↓

chmod

↓

Permissions Change
```

Users with sufficient permissions can modify access.

---

# Problem with DAC

Suppose an attacker compromises:

```
Nginx
```

If Nginx runs as a user with permission to read certain files,

DAC alone cannot express:

```
"Nginx must never read this file."
```

---

# Mandatory Access Control (MAC)

MAC introduces another decision.

Instead of asking only:

```
Does the User Have Permission?
```

Linux also asks:

```
Does Security Policy Allow It?
```

Both must approve the operation.

---

# Visual Representation

```
Application

↓

DAC Check

↓

MAC Check

↓

Kernel

↓

File
```

If either check fails,

access is denied.

---

# What is an LSM?

Linux Security Module (LSM) is a framework inside the Linux kernel.

It allows security systems to enforce additional access-control policies.

Examples include:

- AppArmor
- SELinux
- Smack
- TOMOYO
- Landlock (for specific use cases)

---

# AppArmor

Common on:

- Ubuntu
- Debian

AppArmor secures applications using:

```
Profiles
```

Example profile:

```
Nginx

↓

Can Read

↓

/var/www
```

```
Cannot Read

↓

/root
```

---

# AppArmor Concept

Application:

```
Nginx
```

Profile:

```
Allowed Paths

↓

Allowed Operations
```

Anything outside the profile is denied.

---

# SELinux

Common on:

- Red Hat Enterprise Linux
- Fedora
- CentOS
- Rocky Linux
- AlmaLinux

SELinux uses:

```
Security Labels

+

Policies
```

instead of simple path-based rules.

---

# Security Labels

Files receive labels.

Example:

```
/var/www

↓

httpd_sys_content_t
```

Processes also receive labels.

Policies define which labels may interact.

---

# SELinux Decision

Process Label:

```
httpd_t
```

File Label:

```
httpd_sys_content_t
```

Policy:

```
Allowed
```

Access succeeds.

Different label:

```
Denied
```

---

# AppArmor vs SELinux

AppArmor:

```
Path-Based
```

Example:

```
/var/www/index.html
```

SELinux:

```
Label-Based
```

Example:

```
httpd_sys_content_t
```

Both implement Mandatory Access Control.

---

# Docker and AppArmor

On AppArmor-enabled systems,

Docker typically applies a default AppArmor profile to containers.

This limits what containerized processes may do,

even if they are running as root inside the container.

---

# Docker and SELinux

On SELinux-enabled systems,

Docker labels container processes and files appropriately.

SELinux policies help prevent containers from accessing resources they shouldn't.

---

# Kubernetes

Kubernetes supports LSM integration.

Examples:

- AppArmor profiles (on supported systems)
- SELinux security contexts

Administrators can define policies for Pods and containers.

---

# Example

Suppose an application tries:

```
Read

↓

/root/.ssh/id_rsa
```

Linux checks:

```
DAC

↓

Pass
```

Then:

```
AppArmor

↓

Denied
```

Final Result:

```
Permission Denied
```

Even though traditional permissions allowed it.

---

# Complete Security Stack

```
Application

↓

User Namespace

↓

Capabilities

↓

seccomp

↓

DAC

↓

AppArmor / SELinux

↓

Linux Kernel

↓

Hardware
```

Multiple layers protect the system.

---

# Defense in Depth

Each mechanism solves a different problem.

| Mechanism | Purpose |
|-----------|---------|
| Namespaces | Isolation |
| cgroups | Resource Control |
| Capabilities | Fine-grained Privileges |
| seccomp | Syscall Filtering |
| LSM | Mandatory Access Control |

Together,

they create a strong security model.

---

# Why LSMs Matter

Large organizations depend on them.

Examples:

- Banks
- Cloud Providers
- Government Systems
- Healthcare Platforms
- Enterprise Kubernetes Clusters

Mandatory Access Control helps contain damage even after a compromise.

---

# Hands-on Lab

## Lab 1 — Check AppArmor Status

Ubuntu:

```bash
aa-status
```

Observe loaded AppArmor profiles.

---

## Lab 2 — Check SELinux Status

RHEL/Fedora:

```bash
getenforce
```

Possible outputs:

```
Enforcing

Permissive

Disabled
```

---

## Lab 3 — Inspect Docker Security Options

```bash
docker info
```

Observe whether AppArmor or SELinux integration is enabled.

---

## Lab 4 — View Process Labels (SELinux)

```bash
ps -eZ
```

On SELinux-enabled systems,

observe security contexts.

---

# Interview Questions

## What is a Linux Security Module (LSM)?

LSM is a Linux kernel framework that allows security modules such as AppArmor and SELinux to enforce Mandatory Access Control policies.

---

## What is the difference between DAC and MAC?

DAC relies on file ownership and permission bits.

MAC enforces additional kernel-defined security policies that users cannot override.

---

## What is the difference between AppArmor and SELinux?

AppArmor primarily uses path-based policies.

SELinux primarily uses security labels and policy rules.

---

## Why are LSMs important for containers?

They provide another layer of protection even if traditional permissions, capabilities, or namespaces are bypassed or misused.

---

## Does Docker integrate with AppArmor and SELinux?

Yes.

Docker can automatically integrate with AppArmor or SELinux depending on the Linux distribution and configuration.

---

## Can an operation fail even if Linux file permissions allow it?

Yes.

An LSM policy can still deny the operation.

---

# Summary

Linux Security Modules provide Mandatory Access Control for applications.

```
Application

↓

User Namespace

↓

Capabilities

↓

seccomp

↓

DAC

↓

LSM (AppArmor / SELinux)

↓

Linux Kernel

↓

Hardware
```

Key concepts:

- LSM is a kernel framework for security modules.
- DAC and MAC complement each other.
- AppArmor uses path-based policies.
- SELinux uses security labels and policy rules.
- Docker and Kubernetes integrate with LSMs.
- LSMs provide another critical layer in Linux's defense-in-depth strategy.

At this point, you understand the **complete core Linux container security stack**:

- ✅ Namespaces → Isolation
- ✅ cgroups → Resource Limits
- ✅ User Namespaces → User Identity Mapping
- ✅ Capabilities → Fine-Grained Privileges
- ✅ seccomp → System Call Filtering
- ✅ AppArmor / SELinux → Mandatory Access Control

This is the foundation used by Docker, Kubernetes, containerd, Podman, OpenShift, and most modern cloud-native platforms.

---

# Next Chapter

## Chapter 58 — Putting It All Together: How Docker Creates a Container Internally

We'll explore the complete container creation process from the moment you run:

```bash
docker run nginx
```

Step by step:

- Docker CLI
- Docker Daemon
- containerd
- runc
- OCI Runtime
- Namespaces
- cgroups
- OverlayFS
- Capabilities
- seccomp
- AppArmor / SELinux
- Process Creation
- Container Startup

> **This chapter ties together everything you've learned into one complete end-to-end container creation walkthrough.**