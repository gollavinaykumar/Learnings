# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 3 — Docker Images Deep Dive

> **Learning Objective**
>
> By the end of this chapter you'll understand:
>
> - What a Docker image really is
> - How images are stored
> - Image layers
> - Tags
> - Registries
> - OverlayFS & layer reuse
> - Image history
> - Best practices for production

---

# 📖 Introduction

A Docker image is more than just a file.

It is an **immutable, layered filesystem** plus metadata that Docker uses to create containers.

Think of an image as a **template**.

```text
Dockerfile
      │
docker build
      │
      ▼
Docker Image
      │
docker run
      ▼
Container
```

---

# What Does an Image Contain?

A typical image contains:

```text
Application Code
+
Runtime
+
Libraries
+
Dependencies
+
Environment Variables
+
Metadata
```

It does **not** contain a running process.

---

# Image Anatomy

Example:

```text
backend:v1

├── Base OS Layer
├── Node.js Runtime
├── npm Packages
├── Source Code
└── Metadata
```

Each item is stored as one or more layers.

---

# Image Layers

Docker images are layered.

Dockerfile:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
CMD ["npm","start"]
```

Layers:

```text
Layer 6 → CMD
Layer 5 → Source Code
Layer 4 → npm install
Layer 3 → package.json
Layer 2 → WORKDIR
Layer 1 → node:22-alpine
```

Each Dockerfile instruction usually creates a new layer.

---

# Why Layers Matter

Suppose only the source code changes.

Docker reuses:

```text
Layer 1 ✔
Layer 2 ✔
Layer 3 ✔
Layer 4 ✔
```

Builds only:

```text
Layer 5
```

This dramatically speeds up builds.

---

# Image Caching

First build:

```text
10 minutes
```

Second build:

```text
Uses Cached Layers
↓

30 seconds
```

Ordering your Dockerfile correctly improves cache efficiency.

---

# Tags

Images are identified by:

```text
name:tag
```

Examples:

```text
nginx:latest
postgres:16
node:22-alpine
backend:v1
```

If omitted:

```text
latest
```

is used by default.

---

# Registries

Images are stored in registries.

Popular registries:

- Docker Hub
- GitHub Container Registry
- AWS ECR
- Google Artifact Registry

Example:

```bash
docker pull nginx:latest
```

Flow:

```text
Docker Hub
      │
Download Layers
      │
Local Image Cache
```

---

# Image History

View history:

```bash
docker history nginx
```

Shows:

- Layer size
- Commands
- Creation time

Useful for optimization.

---

# Inspect an Image

```bash
docker image inspect nginx
```

Useful information:

- Environment variables
- Entrypoint
- Labels
- Architecture
- OS

---

# Multi-Architecture Images

Modern images can support:

```text
AMD64

ARM64

ARMv7
```

Docker automatically downloads the correct image for your platform.

Example:

- Apple Silicon → ARM64
- Intel → AMD64

---

# OverlayFS (High Level)

Docker stores image layers efficiently.

```text
Layer 1
   │
Layer 2
   │
Layer 3
```

Containers do not copy every layer.

Instead they share the read-only layers and add one writable layer.

This saves:

- Disk space
- Download time
- Build time

---

# Real Production Example

Backend image:

```text
company/backend:v15
```

Deploy:

```text
Server A

Server B

Server C
```

All servers use the same image.

If a bug is found:

```text
Build v16

↓

Deploy v16

↓

Rollback to v15 if needed
```

Immutability makes deployments predictable.

---

# Best Practices

- Use small base images (e.g. alpine when appropriate).
- Pin versions instead of relying on `latest`.
- Order Dockerfile instructions to maximize cache reuse.
- Remove unnecessary build artifacts.
- Rebuild images instead of modifying running containers.

---

# Common Mistakes

❌ Using `latest` in production.

✔ Prefer versioned tags.

---

❌ Installing everything in one huge image.

✔ Keep images minimal.

---

❌ Editing running containers.

✔ Build a new image.

---

# Hands-on Lab

Download an image:

```bash
docker pull nginx
```

List images:

```bash
docker images
```

Inspect:

```bash
docker image inspect nginx
```

History:

```bash
docker history nginx
```

Remove:

```bash
docker rmi nginx
```

---

# Summary

- Images are immutable.
- Images consist of multiple layers.
- Layers improve storage and build performance.
- Registries store and distribute images.
- Containers are created from images.

---

# Interview Questions

**What is a Docker image?**

An immutable, layered package containing an application and everything required to run it.

**Why are image layers useful?**

They enable caching, reduce storage usage and speed up builds and downloads.

**Why should production avoid `latest` tags?**

Because `latest` is mutable and can unexpectedly change deployed software.

---

# What's Next

**Module 2 – Chapter 4: Docker Containers Deep Dive**

You'll explore container internals, writable layers, process model, logging, signals and advanced lifecycle management.
