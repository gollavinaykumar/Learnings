# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 31 — Docker Image Internals

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - How Docker images are stored
> - Layered filesystem
> - Image Manifest
> - Image Config
> - Layer Digests
> - SHA256
> - Content Addressing
> - Manifest Lists
> - Multi-Architecture Images
> - OCI Image Format
> - Registry Storage
> - Image Pull Process
> - Image Push Process

---

# 📖 Introduction

Suppose you execute:

```bash
docker pull nginx
```

Docker downloads an image.

Most people imagine:

```
nginx.tar

↓

Docker
```

Reality is very different.

Docker downloads multiple objects.

```
Manifest

↓

Config

↓

Layer 1

↓

Layer 2

↓

Layer 3

↓

Layer 4
```

Everything is identified using SHA256 hashes.

---

# What is a Docker Image?

A Docker image is **not one file**.

It is a collection of immutable objects.

```
Docker Image

├── Manifest

├── Config

├── Layer 1

├── Layer 2

├── Layer 3

└── Layer N
```

Each object has its own cryptographic digest.

---

# Layered Filesystem

Every Dockerfile instruction creates a new layer.

Example:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

CMD ["node","index.js"]
```

Produces something like:

```
Layer 1

Base Image

↓

Layer 2

WORKDIR

↓

Layer 3

package.json

↓

Layer 4

npm install

↓

Layer 5

Application Source

↓

Layer 6

Metadata
```

Layers are stacked together.

---

# Why Layers?

Without layers:

```
Every Image

↓

Complete Copy
```

Huge waste.

With layers:

```
Ubuntu Layer

↓

Node Layer

↓

Application Layer
```

Multiple images reuse identical layers.

Benefits:

✔ Smaller downloads

✔ Faster builds

✔ Less storage

✔ Better caching

---

# Copy-on-Write

Images are read-only.

Containers need writable storage.

Docker creates:

```
Image

(Read Only)

↓

Writable Layer

↓

Container
```

Reads:

```
Writable Layer

↓

Image Layers
```

Writes:

```
Writable Layer Only
```

The original image never changes.

---

# Image Manifest

The manifest describes the image.

Example:

```
Manifest

↓

Config Digest

↓

Layer Digests

↓

Metadata
```

Think of it as:

```
Table of Contents
```

for the image.

---

# Image Config

Stores runtime configuration.

Includes:

- Environment variables
- Entrypoint
- CMD
- Working directory
- Labels
- User
- Exposed ports

Example:

```
Config

↓

ENV

↓

CMD

↓

USER

↓

WORKDIR
```

This information comes from the Dockerfile.

---

# Layer Digests

Every layer has a unique SHA256 hash.

Example:

```
sha256:ab45...

sha256:c71d...

sha256:91fa...
```

Changing **one byte** changes the digest completely.

This guarantees integrity.

---

# SHA256

Docker uses SHA256 because it is:

✔ Fast

✔ Collision resistant

✔ Cryptographically secure

Instead of naming files:

```
layer1.tar
```

Docker uses:

```
sha256:d83d6a2...
```

The content itself determines the identifier.

---

# Content Addressing

Traditional storage:

```
Filename

↓

Content
```

Docker:

```
Content

↓

SHA256 Digest

↓

Identifier
```

This is called **Content Addressing**.

Benefits:

- Detects corruption
- Prevents duplicates
- Enables caching
- Ensures integrity

---

# Image Pull Process

Suppose:

```bash
docker pull nginx
```

Flow:

```
Docker CLI

↓

Registry

↓

Download Manifest

↓

Download Config

↓

Download Missing Layers

↓

Verify SHA256

↓

Store Layers

↓

Image Ready
```

Notice:

Existing layers are skipped.

---

# Image Push Process

Suppose:

```bash
docker push myapp:v1
```

Docker checks:

```
Registry

↓

Already Has Layer?

↓

YES

↓

Skip Upload
```

Only new layers are uploaded.

This makes pushes much faster.

---

# Multi-Architecture Images

Suppose your laptop is:

```
Apple Silicon

ARM64
```

Server:

```
Intel

AMD64
```

Same command:

```bash
docker pull nginx
```

Different image downloaded.

How?

---

# Manifest List

Instead of pointing directly to layers:

```
Manifest List

├── AMD64 Manifest

├── ARM64 Manifest

├── ARM Manifest

└── S390x Manifest
```

Docker chooses the correct manifest automatically.

---

# Multi-Architecture Flow

```
docker pull nginx

↓

Registry

↓

Manifest List

↓

Detect CPU

↓

Download Correct Manifest

↓

Download Layers
```

You never choose manually.

---

# OCI Image Format

Modern Docker images follow the OCI Image Specification.

```
OCI Image

↓

Manifest

↓

Config

↓

Layers
```

This is why:

Docker

Podman

containerd

CRI-O

can all use the same images.

---

# Registry Storage

A registry stores:

```
Repository

↓

Tags

↓

Manifest

↓

Config

↓

Layers
```

Example:

```
backend

↓

v1.0.0

↓

Manifest

↓

Layer Digests
```

Registries don't store images as ZIP files.

They store objects indexed by digests.

---

# Local Image Storage

Docker stores image metadata locally.

```
Image

↓

Manifest

↓

Config

↓

Layers

↓

Snapshot
```

The exact storage location depends on the storage driver (OverlayFS, Btrfs, ZFS, etc.).

---

# Image Caching

Suppose:

```
Image A

↓

Ubuntu

↓

Node

↓

React
```

Image B:

```
Ubuntu

↓

Node

↓

Backend
```

Docker stores:

```
Ubuntu

↓

Node
```

only once.

Huge storage savings.

---

# Image IDs

Display images:

```bash
docker images
```

Example:

```
IMAGE ID

3fd9065eaf02
```

Internally this is derived from the image configuration digest.

---

# Inspect an Image

```bash
docker image inspect nginx
```

You'll see:

- RepoTags
- RepoDigests
- Layers
- Config
- Environment variables
- Entrypoint
- Architecture

Very useful for debugging.

---

# Common Misconceptions

❌ Docker images are compressed virtual machines.

✔ Docker images are layered filesystems.

---

❌ Images are mutable.

✔ Images are immutable.

---

❌ Images are stored as one file.

✔ Images consist of manifests, configs, and multiple immutable layers.

---

# Hands-on Lab

Pull an image:

```bash
docker pull nginx
```

Inspect it:

```bash
docker image inspect nginx
```

View local images:

```bash
docker images
```

Observe:

- Image ID
- Repository
- Tag
- Size

---

# Interview Questions

## Why are Docker images layered?

To maximize layer reuse, reduce storage, improve caching, and speed up downloads.

---

## What is a Docker Manifest?

A JSON document describing an image, including its configuration and layer digests.

---

## Why does Docker use SHA256?

To uniquely identify content, verify integrity, and support content-addressable storage.

---

## What is Copy-on-Write?

Containers add a writable layer above immutable image layers so that writes don't modify the original image.

---

## How does Docker support ARM and AMD64 using the same image name?

By using a **Manifest List**, which points to architecture-specific manifests.

---

# Summary

A Docker image is **not a single file**.

It is a structured collection of immutable objects:

```
Docker Image

↓

Manifest

↓

Config

↓

Layer Digests

↓

Immutable Layers
```

Key concepts:

- Layered filesystem
- Copy-on-Write
- SHA256 digests
- Content addressing
- Manifest lists
- OCI image format

Understanding image internals explains why Docker builds are fast, why registries are efficient, and how Kubernetes reliably pulls container images.

---

# Next Chapter

## Chapter 32 — Advanced Docker Networking

We'll go far beyond bridge networks and explore:

- Linux bridges
- veth internals
- VXLAN
- Overlay networking
- Routing Mesh
- IPVS
- Packet encapsulation
- Service discovery
- Network performance
- Production networking