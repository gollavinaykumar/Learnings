
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 15 — Docker Build Process

> **Learning Objectives**
>
> Understand what happens internally when you execute:
>
> ```bash
> docker build -t myapp .
> ```
>
> Learn:
> - Build Context
> - .dockerignore
> - BuildKit
> - Layer creation
> - Cache & cache invalidation
> - Image optimization
> - Build arguments
> - Build secrets
> - CI/CD best practices

---

# 📖 Introduction

Building an image is more than simply reading a Dockerfile.

Docker performs multiple steps before an image is created.

```text
Dockerfile
      │
docker build
      │
Read Build Context
      │
Process Instructions
      │
Create Layers
      │
Store Image
```

---

# Step 1 — Build Context

Command:

```bash
docker build -t myapp .
```

The final `.` is called the **build context**.

Docker sends every file inside this directory (except ignored files) to the Docker daemon.

```text
Project
│
├── Dockerfile
├── package.json
├── src/
└── public/

        │
        ▼
Build Context
```

Large build contexts slow down builds.

---

# Step 2 — .dockerignore

Prevent unnecessary files from entering the build context.

Example:

```text
node_modules
.git
.env
coverage
dist
```

Benefits:

- Faster builds
- Smaller context
- Better security

---

# Step 3 — Read Dockerfile

Docker reads instructions from top to bottom.

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm","start"]
```

Each instruction is processed sequentially.

---

# Step 4 — Layer Creation

Most Dockerfile instructions create a new image layer.

```text
Layer 6  CMD
Layer 5  COPY .
Layer 4  RUN npm ci
Layer 3  COPY package.json
Layer 2  WORKDIR
Layer 1  FROM
```

Layers are immutable.

---

# Step 5 — Layer Cache

Docker checks whether an instruction has already been executed.

If nothing changed:

```text
Reuse Cached Layer ✔
```

Otherwise:

```text
Rebuild Layer
```

---

# Cache Invalidation

Changing:

```dockerfile
COPY . .
```

invalidates all layers after it.

Good ordering:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

Dependencies remain cached when only application code changes.

---

# BuildKit

BuildKit is Docker's modern build engine.

Advantages:

- Faster builds
- Better caching
- Parallel execution
- Secret mounting
- Remote cache support

Enable:

```bash
DOCKER_BUILDKIT=1 docker build .
```

---

# Build Arguments

Build-time variables.

Dockerfile:

```dockerfile
ARG VERSION=1.0
```

Build:

```bash
docker build --build-arg VERSION=2.0 .
```

Only available while building.

---

# Build Secrets

Avoid copying secrets into images.

With BuildKit:

```text
Secret
    │
Temporary Mount
    │
Build
    │
Removed
```

Secrets never become image layers.

---

# Image Optimization

Prefer:

- Small base images
- Multi-stage builds
- Remove temporary files
- Use npm ci instead of npm install (CI)
- Combine related RUN instructions

---

# Internal Flow

```text
docker build
      │
Read Context
      │
Apply .dockerignore
      │
Read Dockerfile
      │
Create/Re-use Layers
      │
Store Image
```

---

# CI/CD Example

Pipeline:

```text
Git Push
   │
CI Server
   │
docker build
   │
Run Tests
   │
Push Image
   │
Deploy
```

Images—not source code—are deployed.

---

# Common Mistakes

❌ Huge build contexts

❌ Copying .git

❌ Copying node_modules

❌ Poor Dockerfile instruction order

❌ Storing secrets in image layers

---

# Hands-on Lab

Create `.dockerignore`:

```text
node_modules
.git
.env
```

Build:

```bash
docker build -t demo:v1 .
```

Build again after changing only source code.

Observe cached layers.

---

# Interview Questions

**Why is `.dockerignore` important?**

It reduces build context size, improves performance and prevents sensitive files from entering images.

**What is BuildKit?**

Docker's modern build engine with improved caching, performance and secret handling.

**Why should COPY package.json come before COPY . ?**

To maximize cache reuse and avoid reinstalling dependencies unnecessarily.

---

# Summary

- Docker sends the build context to the daemon.
- `.dockerignore` reduces context size.
- Docker builds images layer by layer.
- Cached layers speed up builds.
- BuildKit provides faster and more secure builds.
- Proper Dockerfile ordering greatly improves performance.

---

# Next Chapter

## Chapter 16 — Multi-Stage Builds

Learn how professional teams create tiny, secure production images by separating build and runtime stages.
