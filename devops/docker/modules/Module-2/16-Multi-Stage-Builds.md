
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 16 — Multi-Stage Builds

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - Why Multi-Stage Builds exist
> - Build Stage vs Runtime Stage
> - Multiple FROM instructions
> - Copying artifacts between stages
> - Image size optimization
> - Security benefits
> - Production best practices

---

# 📖 Introduction

When developers first learn Docker, they often create images like this:

```dockerfile
FROM node:22

WORKDIR /app

COPY . .

RUN npm install

RUN npm run build

CMD ["npm","start"]
```

This works.

But the final image contains:

- Source code
- Build tools
- npm cache
- Development dependencies
- Temporary files

The image becomes unnecessarily large.

---

# Why Multi-Stage Builds?

Production images should contain **only what is needed to run the application**.

Instead of one stage:

```text
Build + Runtime
```

Docker allows:

```text
Stage 1
Build

↓

Stage 2
Runtime
```

Only the compiled application is copied to the final image.

---

# Build Stage

The first stage contains everything needed for compilation.

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build
```

This stage may contain:

- npm
- Build tools
- TypeScript
- Dev dependencies

---

# Runtime Stage

The second stage contains only runtime files.

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist

CMD ["node","dist/index.js"]
```

The build tools remain behind in the builder stage.

---

# Multiple FROM Instructions

A Dockerfile may contain multiple `FROM` instructions.

```dockerfile
FROM node:22 AS builder

...

FROM node:22-alpine

...
```

Each `FROM` starts a new stage.

---

# Copying Between Stages

Docker provides:

```dockerfile
COPY --from=builder /app/dist ./dist
```

Meaning:

```text
Builder Stage
      │
Compiled Files
      │
      ▼
Runtime Stage
```

Only selected files are copied.

---

# Internal Flow

```text
Source Code
      │
      ▼
Builder Stage
      │
Compile
      │
Artifacts
      │
      ▼
Runtime Stage
      │
Small Production Image
```

---

# Node.js Example

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

FROM node:22-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist

COPY package*.json ./

RUN npm ci --omit=dev

CMD ["node","dist/index.js"]
```

---

# Java Example

```dockerfile
FROM maven:3.9 AS builder

COPY . .

RUN mvn package

FROM eclipse-temurin:21-jre

COPY --from=builder target/app.jar app.jar

CMD ["java","-jar","app.jar"]
```

---

# Go Example

```dockerfile
FROM golang:1.24 AS builder

COPY . .

RUN go build -o app

FROM alpine

COPY --from=builder /go/app .

CMD ["./app"]
```

---

# Benefits

- Smaller images
- Faster downloads
- Faster deployments
- Better security
- Reduced attack surface
- Cleaner production containers

---

# Security Benefits

Build tools such as:

- gcc
- make
- npm cache

remain in the builder image and never reach production.

---

# Best Practices

- Separate build and runtime.
- Use lightweight runtime images.
- Copy only required artifacts.
- Install production dependencies only.
- Keep stages clearly named.

---

# Common Mistakes

❌ Running production containers from builder images.

❌ Copying the entire project into the runtime stage.

❌ Leaving development dependencies installed.

---

# Hands-on Lab

Build a Node.js application using two stages.

Compare image sizes:

```bash
docker images
```

Notice the production image is significantly smaller.

---

# Interview Questions

**Why use Multi-Stage Builds?**

To separate compilation from runtime, creating smaller, faster and more secure production images.

**What does `COPY --from=builder` do?**

It copies selected files from a previous build stage into the current stage.

---

# Summary

- Multi-Stage Builds use multiple `FROM` instructions.
- Builder stages compile the application.
- Runtime stages contain only required artifacts.
- Production images become smaller and more secure.

---

# Next Chapter

## Chapter 17 — Docker Volumes

You'll learn:

- Persistent storage
- Named volumes
- Bind mounts
- Anonymous volumes
- tmpfs
- Volume drivers
- Database persistence
- Backup & Restore
