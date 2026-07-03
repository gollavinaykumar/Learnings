# 🐳 Docker Mastery Roadmap
# Module 1 — Why Docker Exists
## Chapter 7 — Docker Images vs Containers

> **Learning Objective**
>
> By the end of this chapter you'll understand:
>
> - What a Docker Image is
> - What a Docker Container is
> - Image vs Container
> - Image layers
> - Copy-on-Write
> - Why images are immutable
> - Multiple containers from one image

---

# 📖 Introduction

One of the most common beginner mistakes is confusing an **Image** with a **Container**.

People often say:

> "I created a Docker image and started it."

Technically, **you don't start an image**.

You start a **container created from an image**.

Think of it like Java:

```text
Class
   ↓ new
Object
```

Docker works similarly:

```text
Image
   ↓ docker run
Container
```

---

# 🖼️ What is a Docker Image?

A Docker Image is a **read-only blueprint** used to create containers.

It contains:

- Application
- Runtime
- Libraries
- Dependencies
- Configuration
- Metadata

Example:

```text
node-backend:v1

├── Node.js
├── Express
├── Source Code
├── package.json
└── Configuration
```

An image **does not execute code**.

It simply stores everything needed to run an application.

---

# 📦 What is a Docker Container?

A Docker Container is a **running instance of an image**.

Example:

```bash
docker run nginx
```

Docker creates:

```text
Image

↓

Container

↓

Running nginx Process
```

A container has:

- Running processes
- Memory
- Network
- Writable filesystem
- Process IDs

Unlike images, containers have state while they are running.

---

# Java Analogy

```text
Java

Class
   ↓
Object
```

```text
Docker

Image
   ↓
Container
```

One class can create many objects.

One image can create many containers.

---

# Real Example

Image:

```text
backend:v1
```

Run:

```bash
docker run --name api1 backend:v1
docker run --name api2 backend:v1
docker run --name api3 backend:v1
```

Result:

```text
backend:v1

├── api1
├── api2
└── api3
```

One image.

Three independent containers.

---

# Images are Immutable

Immutable means:

> Once an image is built, its contents do not change.

If you modify your application code:

```text
Version 1

↓

Change Code

↓

Version 2
```

You build a **new image**.

Images are never edited in place.

Benefits:

- Predictable deployments
- Versioning
- Easy rollback
- Reproducible builds

---

# Image Layers

Images are built in layers.

Example Dockerfile:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
CMD ["npm","start"]
```

Produces something similar to:

```text
Layer 5 -> Source Code
Layer 4 -> npm install
Layer 3 -> package.json
Layer 2 -> WORKDIR
Layer 1 -> node:22-alpine
```

Each instruction creates a new layer.

---

# Why Layers Matter

Suppose only your source code changes.

Docker reuses:

```text
Layer 1 ✔
Layer 2 ✔
Layer 3 ✔
Layer 4 ✔

Build New

Layer 5
```

This is called **Layer Caching**.

It makes builds much faster.

---

# Copy-on-Write

Images are read-only.

Containers add one writable layer.

```text
Docker Image (Read Only)

↓

Writable Container Layer

↓

Application Writes Files
```

When a container writes data:

- Image stays unchanged.
- Data goes into the writable container layer.

---

# Why Data Disappears

Suppose:

```text
Container

Creates:

/tmp/report.txt
```

You remove the container:

```bash
docker rm mycontainer
```

Writable layer is deleted.

Your file disappears.

This is why Docker Volumes exist (covered later).

---

# Image Lifecycle

```text
Dockerfile
      │
      ▼
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

# Real Industry Example

Your stack:

```text
Frontend Image

Backend Image

PostgreSQL Image

Redis Image
```

Deploy:

```text
Frontend Container

Backend Container

PostgreSQL Container

Redis Container
```

If you need five backend instances:

```text
backend:v1

↓

api1

api2

api3

api4

api5
```

Same image.

Five containers.

---

# Image vs Container

| Docker Image | Docker Container |
|--------------|------------------|
| Blueprint | Running instance |
| Read-only | Writable |
| Immutable | Temporary |
| Built using Dockerfile | Created using docker run |
| Can create many containers | Created from one image |

---

# Common Misconceptions

❌ Images execute code.

✔️ Containers execute code.

---

❌ Editing a running container changes the image.

✔️ Changes stay only in the container's writable layer.

---

❌ One image can create only one container.

✔️ One image can create unlimited containers.

---

# Senior Engineer Notes

Production deployments rarely modify running containers.

Instead:

```text
Change Code

↓

Build New Image

↓

Deploy New Containers

↓

Remove Old Containers
```

This immutable deployment model makes rollbacks safe and predictable.

---

# Summary

- Images are read-only blueprints.
- Containers are running instances of images.
- Images are immutable.
- Containers have a writable layer.
- Multiple containers can be created from one image.
- Docker uses layered images for efficient builds.

---

# Interview Questions

## What is a Docker Image?

A Docker Image is an immutable, read-only package containing an application and everything required to run it.

## What is a Docker Container?

A Docker Container is a running instance of a Docker Image.

## Why are Docker Images immutable?

Immutability ensures reproducible deployments, easier versioning and safe rollbacks.

---

# What's Next

**Chapter 8 — Container Lifecycle**

You'll learn:

- Container states
- Create
- Start
- Stop
- Pause
- Restart
- Remove
- Restart policies
