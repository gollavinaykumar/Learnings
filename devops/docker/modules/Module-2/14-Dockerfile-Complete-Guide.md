
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 14 — Dockerfile (Complete Guide)

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - Why Dockerfiles exist
> - Every major Dockerfile instruction
> - How Docker builds images
> - Layer caching
> - Production Dockerfile patterns
> - Common mistakes
> - Best practices

---

# 📖 What is a Dockerfile?

A Dockerfile is a text file containing instructions that Docker follows to build an image.

```text
Dockerfile
      │
docker build
      ▼
Docker Image
      │
docker run
      ▼
Container
```

---

# Why Dockerfile?

Without Dockerfile:

- Manual setup
- Different environments
- Difficult deployments

With Dockerfile:

- Automated builds
- Repeatable environments
- Version-controlled infrastructure

---

# Build Process

```text
Dockerfile
     │
docker build
     │
Read Instruction
     │
Create Layer
     │
Repeat
     │
Docker Image
```

Every major instruction creates a layer.

---

# FROM

Defines the base image.

```dockerfile
FROM node:22-alpine
```

Best Practices

- Use trusted base images
- Pin versions
- Prefer smaller images when appropriate

---

# WORKDIR

Sets the working directory.

```dockerfile
WORKDIR /app
```

Docker creates it if it doesn't exist.

Avoid repeatedly using `RUN cd ...`.

---

# COPY

Copies files from build context.

```dockerfile
COPY package*.json ./
COPY . .
```

Most commonly used instruction.

---

# ADD

Can copy files and extract local archives.

```dockerfile
ADD app.tar.gz /app
```

Prefer `COPY` unless ADD's extra features are needed.

---

# RUN

Executes commands during image build.

```dockerfile
RUN npm install
```

Multiple RUN instructions create multiple layers.

Better:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl
```

---

# CMD

Default command executed when the container starts.

```dockerfile
CMD ["npm","start"]
```

Only the last CMD is used.

Can be overridden by `docker run`.

---

# ENTRYPOINT

Defines the main executable.

```dockerfile
ENTRYPOINT ["node","server.js"]
```

Often combined with CMD.

```dockerfile
ENTRYPOINT ["node"]
CMD ["server.js"]
```

---

# CMD vs ENTRYPOINT

CMD

- Default arguments
- Easily overridden

ENTRYPOINT

- Main executable
- Usually fixed

---

# ARG

Build-time variable.

```dockerfile
ARG VERSION=1.0
```

Available only during build.

---

# ENV

Runtime environment variable.

```dockerfile
ENV NODE_ENV=production
```

Accessible inside the running container.

---

# LABEL

Metadata.

```dockerfile
LABEL maintainer="Vinay"
LABEL version="1.0"
```

Useful for documentation and automation.

---

# USER

Run the application as a non-root user.

```dockerfile
USER node
```

Production recommendation:

Avoid running containers as root.

---

# EXPOSE

Documents which ports the application listens on.

```dockerfile
EXPOSE 3000
```

Does not publish ports.

Publishing happens with:

```bash
docker run -p 3000:3000
```

---

# VOLUME

Marks directories for persistent storage.

```dockerfile
VOLUME ["/data"]
```

Useful for databases and uploads.

---

# HEALTHCHECK

Allows Docker to verify application health.

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

Possible states:

- starting
- healthy
- unhealthy

---

# SHELL

Changes the shell used for RUN instructions.

```dockerfile
SHELL ["/bin/bash","-c"]
```

Rarely needed.

---

# Layer Caching

Docker caches layers.

Example:

```dockerfile
COPY package*.json .
RUN npm install
COPY . .
```

If only source code changes:

```text
Reuse

FROM ✔
WORKDIR ✔
COPY package*.json ✔
RUN npm install ✔

Rebuild

COPY . .
```

This speeds up builds.

---

# Production Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

USER node

CMD ["npm","start"]
```

---

# Best Practices

- Pin image versions
- Use `.dockerignore`
- Keep images small
- Combine related RUN commands
- Use non-root users
- Prefer COPY over ADD
- Put rarely changing instructions first
- Rebuild instead of modifying containers

---

# Common Mistakes

❌ Using `latest` in production

❌ Copying unnecessary files

❌ Installing build tools in runtime images

❌ Running everything as root

---

# Hands-on Lab

Create a Dockerfile:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm","start"]
```

Build:

```bash
docker build -t myapp:v1 .
```

Run:

```bash
docker run -p 3000:3000 myapp:v1
```

---

# Interview Questions

**Why should COPY package.json happen before COPY . . ?**

To maximize Docker layer cache reuse and avoid reinstalling dependencies on every code change.

**Difference between ARG and ENV?**

ARG exists only during image build.

ENV exists inside the running container.

**Difference between CMD and ENTRYPOINT?**

ENTRYPOINT defines the executable.

CMD supplies default arguments or a default command.

---

# Summary

- Dockerfiles automate image creation.
- Each instruction typically creates a layer.
- Layer ordering affects build performance.
- Use COPY, not ADD, unless extra features are needed.
- Prefer non-root users.
- Use HEALTHCHECK and EXPOSE appropriately.
- Build immutable, production-ready images.

---

# Next Chapter

## Chapter 15 — Docker Build Process

We'll explore:

- docker build internals
- Build Context
- .dockerignore
- BuildKit
- Cache invalidation
- Multi-stage builds
- Build optimization
- CI/CD integration
