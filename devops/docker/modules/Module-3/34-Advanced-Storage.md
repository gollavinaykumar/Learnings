# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 34 — Advanced Storage

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Docker storage architecture
> - Storage Drivers
> - OverlayFS
> - AUFS
> - Btrfs
> - ZFS
> - Device Mapper
> - Snapshotters
> - Copy-on-Write Internals
> - Image Storage
> - Writable Layers
> - Storage Performance
> - Production Best Practices

---

# 📖 Introduction

Earlier we learned:

```
Docker Image

↓

Layers

↓

Container

↓

Writable Layer
```

But...

Where are those layers actually stored?

How does Docker combine them into one filesystem?

Why doesn't every container have a full copy of the image?

This chapter answers those questions.

---

# Storage Architecture

A container filesystem is **not a single directory**.

Docker builds it from multiple layers.

```
Application

↓

Writable Layer

↓

Image Layer 5

↓

Image Layer 4

↓

Image Layer 3

↓

Image Layer 2

↓

Base Layer
```

The container sees one filesystem.

Internally, it's many layers merged together.

---

# Storage Driver

A storage driver determines:

- How image layers are stored
- How layers are merged
- How Copy-on-Write works
- How writable layers behave

Think of it as:

```
Docker

↓

Storage Driver

↓

Filesystem
```

Different drivers have different performance characteristics.

---

# Popular Storage Drivers

Docker has supported several storage drivers over time.

| Driver | Status | Notes |
|---------|--------|------|
| OverlayFS (overlay2) | ✅ Default | Fast, simple, widely used |
| AUFS | Legacy | Used before OverlayFS became mainstream |
| Btrfs | Supported | Built on the Btrfs filesystem |
| ZFS | Supported | Enterprise storage features |
| Device Mapper | Legacy | Thin provisioning support |
| VFS | Testing | No Copy-on-Write, mainly for debugging |

Today, **overlay2** is the default on most Linux distributions.

---

# Why OverlayFS?

OverlayFS combines multiple directories into one unified view.

Imagine:

```
Layer 1

/usr

/bin

------------

Layer 2

/app

------------

Layer 3

/config
```

OverlayFS presents them as:

```
/

├── usr

├── bin

├── app

└── config
```

Applications don't know the files come from different layers.

---

# OverlayFS Terminology

OverlayFS uses four important directories:

```
Lowerdir

Upperdir

Workdir

Merged
```

### Lowerdir

Contains the read-only image layers.

```
Image Layer 1

Image Layer 2

Image Layer 3
```

---

### Upperdir

Contains changes made by the running container.

```
New Files

Modified Files

Deleted Files
```

---

### Workdir

A temporary workspace used internally by OverlayFS.

Applications never interact with it directly.

---

### Merged Directory

The final filesystem that the container sees.

```
Container

↓

Merged View
```

The merged view combines the lower and upper layers into one filesystem.

---

# OverlayFS Architecture

```
                Merged
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
    Upperdir             Lowerdir
 (Writable Layer)   (Image Layers)
```

Reads may come from either layer.

Writes always go to the upper layer.

---

# Copy-on-Write (CoW)

Suppose a container reads:

```
/etc/config
```

If the file hasn't changed:

```
Read

↓

Lower Layer
```

No copy is made.

Now modify the file.

```
Write

↓

Copy File

↓

Upper Layer

↓

Modify Copy
```

The original image remains unchanged.

This is the essence of **Copy-on-Write**.

---

# File Deletion

Deleting a file doesn't remove it from the image.

Instead, OverlayFS creates a special marker called a **whiteout**.

```
Lower Layer

config.yml

↓

Delete

↓

Upper Layer

Whiteout Marker
```

The merged view hides the file.

The original image is still intact.

---

# Container Filesystem Lifecycle

```
Docker Image

↓

Create Snapshot

↓

Add Writable Layer

↓

Container Starts

↓

Writes Go to Upperdir

↓

Container Stops

↓

Writable Layer Removed (unless committed)
```

Containers are designed to be ephemeral.

---

# Snapshotters

containerd uses **snapshotters** to prepare writable layers.

```
Image

↓

Snapshotter

↓

Snapshot

↓

Container
```

A snapshot is a writable view of an immutable image.

Common snapshotters:

- OverlayFS
- Btrfs
- ZFS
- Native
- Stargz (optimized image loading)

---

# AUFS

Historically, Docker used AUFS.

Advantages:

- Good layer support
- Mature implementation

Disadvantages:

- Out-of-tree kernel module
- Not included in the main Linux kernel

OverlayFS eventually replaced it.

---

# Btrfs

Btrfs is both:

- A filesystem
- A storage driver

Features:

- Snapshots
- Checksums
- Compression
- RAID support

Good for advanced storage needs.

---

# ZFS

ZFS provides enterprise storage capabilities.

Features:

- Snapshots
- Compression
- Data integrity
- Clones
- Self-healing (with redundant storage)

Popular in storage-heavy environments.

---

# Device Mapper

An older storage driver.

Features:

- Thin provisioning
- Block-level snapshots

Disadvantages:

- More complex
- Lower performance in many workloads

Mostly replaced by OverlayFS.

---

# Image Storage

Images are stored as shared layers.

Suppose:

```
Image A

Ubuntu

↓

Node

↓

Backend
```

Image B:

```
Ubuntu

↓

Node

↓

Frontend
```

Docker stores:

```
Ubuntu

↓

Node
```

only once.

This saves disk space.

---

# Writable Layer

Each container gets its own writable layer.

```
Image

↓

Container A

Upper Layer

------------

Image

↓

Container B

Upper Layer
```

Containers don't interfere with each other's changes.

---

# Performance Considerations

Read operations:

```
Merged

↓

Upper?

↓

Lower?
```

Writes:

```
Upper Layer Only
```

Heavy write workloads may experience more overhead due to Copy-on-Write.

For databases, use volumes instead of relying on the writable layer.

---

# Volumes vs Writable Layer

Bad:

```
PostgreSQL

↓

Writable Layer
```

If the container is removed, the data is lost.

Good:

```
PostgreSQL

↓

Docker Volume

↓

Persistent Storage
```

Volumes bypass the storage driver's writable layer for better durability and often better performance.

---

# Inspect Storage Driver

Check Docker's storage driver:

```bash
docker info
```

Look for:

```
Storage Driver: overlay2
```

---

# Disk Usage

View Docker disk usage:

```bash
docker system df
```

Example:

```
Images

Containers

Volumes

Build Cache
```

Useful for identifying unused resources.

---

# Cleanup

Remove unused objects:

```bash
docker system prune
```

Or include volumes:

```bash
docker system prune --volumes
```

Use with caution in production.

---

# Common Misconceptions

❌ Every container contains a full copy of the image.

✔ Containers share immutable image layers.

---

❌ Modifying a file changes the image.

✔ Changes are written only to the container's writable layer.

---

❌ Deleting a file removes it from the image.

✔ OverlayFS hides it using a whiteout entry.

---

# Hands-on Lab

Inspect the storage driver:

```bash
docker info
```

View disk usage:

```bash
docker system df
```

Run a container:

```bash
docker run -it --name demo alpine sh
```

Create a file:

```sh
echo "Hello" > /tmp/example.txt
```

Exit and remove the container:

```bash
docker rm -f demo
```

Notice that the file disappears because it was stored in the writable layer.

Repeat the experiment using a Docker volume to observe persistent storage.

---

# Interview Questions

## What is a storage driver?

A storage driver determines how Docker stores image layers, creates writable layers, and implements Copy-on-Write.

---

## What is OverlayFS?

A Linux filesystem that merges multiple directories into a single unified filesystem, allowing Docker to combine immutable image layers with a writable layer.

---

## What is Copy-on-Write?

A mechanism where existing data is shared until modified. Modified files are copied into the writable layer, leaving the original image unchanged.

---

## Why should databases use volumes?

Volumes provide persistent storage and avoid the overhead and lifecycle limitations of the container's writable layer.

---

## What is a snapshotter?

A component used by containerd to create writable snapshots from immutable image layers.

---

# Summary

Docker storage is based on layered filesystems.

```
Image Layers

↓

Storage Driver

↓

Snapshot

↓

Writable Layer

↓

Merged Filesystem

↓

Container
```

Key concepts:

- OverlayFS
- Copy-on-Write
- Snapshotters
- Shared image layers
- Writable layers
- Storage drivers
- Persistent volumes

Understanding storage internals explains why Docker images are space-efficient, why containers are ephemeral, and why volumes are essential for persistent data.

---

# Next Chapter

## Chapter 35 — Docker Best Practices

We'll bring everything together by covering:

- Enterprise Docker workflows
- Image versioning strategies
- Immutable infrastructure
- Blue-Green deployments
- Canary deployments
- Security best practices
- Supply-chain security
- SBOMs
- Image signing
- Production checklists