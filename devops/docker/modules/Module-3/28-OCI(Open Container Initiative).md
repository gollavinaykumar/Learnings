# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 28 — OCI (Open Container Initiative)

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why OCI was created
> - Problems before OCI
> - OCI Image Specification
> - OCI Runtime Specification
> - OCI Distribution Specification
> - OCI-compatible runtimes
> - Why Kubernetes doesn't require Docker
> - OCI ecosystem
> - Production architecture

---

# 📖 Introduction

Let's go back to 2015.

Docker was growing incredibly fast.

Every company wanted to use Docker.

The problem was...

Docker controlled everything.

```
Docker

↓

Image Format

↓

Runtime

↓

Engine

↓

API
```

Everything depended on Docker.

People became worried.

What if Docker changed its format?

What if another company wanted to build a better runtime?

What if Kubernetes wanted to use another container engine?

The ecosystem needed open standards.

---

# The Problem Before OCI

Imagine every phone charger was different.

```
Samsung

↓

Samsung Charger

Apple

↓

Apple Charger

OnePlus

↓

OnePlus Charger
```

Chaos.

The same thing happened with containers.

Every company could have invented its own format.

```
Docker Image

↓

Docker Runtime

↓

Docker Registry

↓

Docker Only
```

No interoperability.

---

# The Birth of OCI

To solve this, the industry created the:

```
OCI

Open Container Initiative
```

Founded in 2015 under the Linux Foundation.

Members include:

- Docker
- Google
- Microsoft
- Red Hat
- IBM
- Intel
- Oracle
- AWS
- VMware

Instead of competing on formats...

They agreed on standards.

---

# What is OCI?

OCI does **not** create containers.

OCI defines rules.

Think of it like:

```
HTML

↓

Browser Standards

↓

Chrome

Firefox

Safari
```

Browsers are different.

But they all understand HTML.

OCI works the same way.

---

# OCI Specifications

OCI has three major specifications.

```
OCI

├── Image Specification

├── Runtime Specification

└── Distribution Specification
```

These specifications ensure compatibility.

---

# OCI Image Specification

Defines:

```
How Images Are Stored
```

Includes:

- Layers
- Metadata
- Configuration
- Manifest
- Digests

Example:

```
Image

↓

Layer 1

↓

Layer 2

↓

Layer 3

↓

Config

↓

Manifest
```

Because of OCI:

An image built by Docker can be run by other OCI-compatible runtimes.

---

# OCI Runtime Specification

Defines:

```
How Containers Run
```

Responsibilities include:

- Filesystem
- Namespaces
- cgroups
- Capabilities
- Mounts
- Process lifecycle

This specification is implemented by runtimes such as:

- runc
- crun
- Kata Containers
- gVisor (partial compatibility)
- youki

---

# OCI Distribution Specification

Defines:

```
How Images Are Shared
```

This is why tools understand registries consistently.

```
Docker Hub

GHCR

Amazon ECR

Google Artifact Registry

Harbor
```

All can expose OCI-compatible images.

---

# OCI Ecosystem

Today the ecosystem looks like this.

```
OCI Standards

↓

Docker

Podman

containerd

CRI-O

Buildah

nerdctl
```

Different tools.

Same standards.

---

# Docker is OCI-Compliant

Modern Docker produces OCI-compatible images.

```
Docker Build

↓

OCI Image

↓

Registry

↓

Any OCI Runtime
```

Docker still exists.

But it is no longer the only option.

---

# Why Kubernetes Doesn't Need Docker

Early Kubernetes worked like this.

```
Kubernetes

↓

Docker

↓

Containers
```

This meant Kubernetes depended on Docker Engine.

Over time:

```
Kubernetes

↓

containerd

↓

runc

↓

Containers
```

Docker Engine became unnecessary.

This change is often called:

```
Dockershim Removal
```

Docker images still work because they follow OCI standards.

---

# OCI-Compatible Tools

Examples:

```
Docker

↓

Build Images

----------------

Podman

↓

Run Containers

----------------

Buildah

↓

Build Images

----------------

containerd

↓

Manage Containers

----------------

CRI-O

↓

Kubernetes Runtime
```

Different responsibilities.

Same standards.

---

# OCI Runtime Flow

```
OCI Runtime Spec

↓

runc

↓

Linux Kernel

↓

Namespaces

↓

cgroups

↓

Container
```

Any OCI runtime follows this specification.

---

# OCI Image Flow

```
Docker Build

↓

OCI Image

↓

Registry

↓

containerd Pull

↓

runc

↓

Container
```

Docker isn't required after the image is built.

---

# Real Production Example

Suppose a company builds images using Docker.

```
Developer

↓

Docker Build

↓

OCI Image

↓

Amazon ECR

↓

Kubernetes

↓

containerd

↓

runc

↓

Linux Kernel
```

Notice:

Production never needs Docker Engine.

---

# Why OCI Matters

Without OCI:

```
Docker Images

↓

Docker Only
```

With OCI:

```
Docker Images

↓

Docker

Podman

containerd

CRI-O

Kubernetes

Buildah
```

Everything works together.

---

# OCI vs Docker

| Docker | OCI |
|---------|-----|
| Product | Standard |
| Implements OCI | Defines OCI |
| Builds images | Defines image format |
| Runs containers | Defines runtime rules |

Docker is an implementation.

OCI is the specification.

---

# Common Misconceptions

❌ OCI is another container runtime.

✔ OCI is a set of specifications.

---

❌ Docker invented the container standard.

✔ OCI standardized container formats across the industry.

---

❌ Kubernetes needs Docker.

✔ Kubernetes needs an OCI-compatible runtime such as containerd or CRI-O.

---

# Hands-on Lab

Inspect an image:

```bash
docker image inspect nginx
```

Observe:

- Layers
- Digests
- Metadata

Pull an image:

```bash
docker pull nginx
```

Remember:

The image follows the OCI Image Specification.

---

# Interview Questions

### What is OCI?

The Open Container Initiative is an open standard defining how container images are built, distributed, and executed.

---

### What are the three OCI specifications?

- OCI Image Specification
- OCI Runtime Specification
- OCI Distribution Specification

---

### Why was OCI created?

To prevent vendor lock-in and ensure interoperability between different container tools.

---

### Does Kubernetes require Docker?

No.

Kubernetes uses OCI-compatible runtimes such as containerd or CRI-O.

---

### Is Docker obsolete?

No.

Docker remains an excellent development platform and image builder.

However, modern production systems often use containerd or CRI-O as the runtime.

---

# Summary

OCI unified the container ecosystem.

```
OCI

↓

Image Standard

↓

Runtime Standard

↓

Distribution Standard
```

Because of OCI:

- Docker images are portable.
- Kubernetes can use multiple runtimes.
- Container tools remain interoperable.

Understanding OCI is essential before learning containerd and Kubernetes internals.

---

# Next Chapter

## Chapter 29 — containerd & runc

We'll dive into:

- containerd architecture
- container lifecycle
- snapshots
- tasks
- shims
- runc internals
- namespaces
- how containers actually stay running