# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 33 — BuildKit Internals

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why BuildKit was introduced
> - Legacy Docker Builder vs BuildKit
> - Build Graph (DAG)
> - Parallel Execution
> - Layer Caching
> - Cache Import/Export
> - Secret Mounts
> - SSH Mounts
> - Remote Cache
> - Multi-platform Builds
> - Build Performance
> - Production CI/CD Workflows

---

# 📖 Introduction

Suppose you build an image:

```bash
docker build .
```

The build succeeds.

But have you ever wondered:

- Why is the second build much faster?
- Why are some instructions skipped?
- How does Docker know what changed?
- Why can BuildKit execute instructions in parallel?

The answer is **BuildKit**.

---

# Before BuildKit

Old Docker Builder executed every instruction **one by one**.

```
FROM

↓

COPY

↓

RUN

↓

COPY

↓

RUN

↓

CMD
```

Each instruction waited for the previous one to finish.

This worked, but it wasn't optimal.

---

# Problems with the Legacy Builder

Imagine a Dockerfile:

```dockerfile
FROM node:22

COPY package.json .

RUN npm install

COPY . .

RUN npm run build
```

The old builder always processed instructions sequentially.

```
Instruction 1

↓

Instruction 2

↓

Instruction 3

↓

Instruction 4
```

Even when parts of the build were independent.

---

# Why BuildKit?

Docker introduced BuildKit to solve several problems.

Goals:

✔ Faster builds

✔ Better caching

✔ Parallel execution

✔ Secret handling

✔ Remote cache support

✔ Multi-platform builds

✔ Cleaner architecture

---

# BuildKit Architecture

```
Docker CLI

↓

BuildKit

↓

Build Graph

↓

Workers

↓

Snapshots

↓

Image
```

Instead of treating the Dockerfile as a simple list, BuildKit converts it into a graph.

---

# Directed Acyclic Graph (DAG)

This is the biggest improvement.

Instead of:

```
Instruction 1

↓

Instruction 2

↓

Instruction 3
```

BuildKit creates:

```
        FROM
       /    \
 COPY pkg   COPY docs
      |        |
 npm ci     Compress
       \    /
      Final Image
```

Independent tasks can execute simultaneously.

---

# Parallel Execution

Example:

```
Task A

↓

Download Dependencies

-------------------

Task B

↓

Prepare Metadata
```

Old Builder:

```
A

↓

B
```

BuildKit:

```
A      B

↓

Both Together
```

This reduces build time.

---

# Layer Caching

BuildKit still uses layer caching.

Suppose:

```dockerfile
COPY package.json .

RUN npm ci

COPY . .
```

You modify only:

```
README.md
```

Result:

```
FROM

✔ Cache

COPY package.json

✔ Cache

RUN npm ci

✔ Cache

COPY .

Rebuild

CMD

Reuse
```

Only affected layers rebuild.

---

# Cache Invalidation

Docker determines cache validity using:

- File contents
- Build arguments
- Environment variables
- Command changes

Changing:

```dockerfile
RUN npm ci
```

to

```dockerfile
RUN npm install
```

invalidates that layer and every layer after it.

---

# Cache Export

BuildKit can export its cache.

```
CI Server

↓

Build

↓

Export Cache

↓

Registry
```

Future builds reuse it.

---

# Cache Import

Another machine can download that cache.

```
Developer Laptop

↓

Import Cache

↓

Reuse Layers

↓

Fast Build
```

This is extremely useful for CI/CD.

---

# Inline Cache

BuildKit can embed cache metadata directly into the image.

```
Image

↓

Layers

↓

Cache Metadata
```

Future builds become faster without requiring a separate cache server.

---

# Secret Mounts

Never copy secrets into images.

Bad:

```dockerfile
COPY secret.txt .
```

The secret becomes part of the image forever.

Instead:

```dockerfile
RUN --mount=type=secret,id=npm_token ...
```

The secret is available **only during that build step**.

After the step finishes, the secret disappears.

---

# SSH Mounts

Suppose you need to clone a private Git repository.

Instead of copying your SSH key:

```dockerfile
COPY ~/.ssh/id_rsa
```

Use:

```dockerfile
RUN --mount=type=ssh git clone git@github.com:company/private.git
```

Your SSH agent is temporarily forwarded to the build.

The key is never written into the image.

---

# Multi-Platform Builds

Suppose:

```
Apple Silicon

ARM64
```

Production:

```
AMD64
```

BuildKit supports:

```bash
docker buildx build \
--platform linux/amd64,linux/arm64 .
```

One command builds multiple architectures.

---

# Buildx

Buildx is a BuildKit-based builder.

```
docker buildx

↓

BuildKit

↓

Multi-platform Image
```

Features:

- Multi-platform builds
- Advanced caching
- Remote builders
- Distributed builds

---

# Remote Builders

Large companies often use dedicated build servers.

```
Developer

↓

Remote BuildKit

↓

Build

↓

Registry
```

Your laptop doesn't perform the build.

---

# Build Performance Tips

✔ Use `.dockerignore`

✔ Copy dependency files first

✔ Use `npm ci`

✔ Minimize build context

✔ Multi-stage builds

✔ BuildKit cache

✔ Cache dependencies

✔ Use Alpine images where appropriate

---

# CI/CD Workflow

```
Git Push

↓

CI Pipeline

↓

BuildKit

↓

Import Cache

↓

Build

↓

Export Cache

↓

Push Image

↓

Deploy
```

Notice:

The cache is reused across pipeline runs.

---

# Common Mistakes

❌ Copying the entire project before installing dependencies

❌ Disabling cache unnecessarily

❌ Copying secrets into images

❌ Ignoring `.dockerignore`

❌ Using large build contexts

---

# Hands-on Lab

Enable BuildKit:

```bash
export DOCKER_BUILDKIT=1
```

Build:

```bash
docker build .
```

Compare build times after making small source code changes.

Observe:

- Cache hits
- Cache misses
- Layer reuse

---

# Interview Questions

## Why was BuildKit introduced?

To improve Docker builds with parallel execution, advanced caching, better security, and multi-platform support.

---

## What is a DAG?

A **Directed Acyclic Graph** representing dependencies between build steps. Independent steps can run in parallel.

---

## Why are secret mounts better than COPY?

Secrets are available only during the build step and are never stored in the final image.

---

## What is Buildx?

A Docker CLI plugin built on BuildKit that supports advanced builds such as multi-platform images.

---

## How does BuildKit improve CI/CD?

By importing and exporting caches, reducing repeated work across pipeline runs.

---

# Summary

BuildKit modernized Docker builds.

```
Dockerfile

↓

BuildKit

↓

Build Graph (DAG)

↓

Parallel Workers

↓

Snapshots

↓

Cached Layers

↓

Optimized Image
```

Key concepts:

- Parallel execution
- DAG-based builds
- Layer caching
- Cache import/export
- Secret mounts
- SSH mounts
- Multi-platform builds
- Remote builders

Understanding BuildKit is essential for designing fast, secure, and scalable container build pipelines.

---

# Next Chapter

## Chapter 34 — Advanced Storage

We'll explore:

- OverlayFS internals
- Snapshotters
- Storage drivers
- AUFS
- Btrfs
- ZFS
- Device Mapper
- Copy-on-Write implementation
- Performance characteristics
- Choosing the right storage driver