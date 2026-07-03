# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 35 — Docker Best Practices

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Enterprise Docker workflows
> - Image versioning
> - Immutable Infrastructure
> - Blue-Green Deployment
> - Canary Deployment
> - Security Best Practices
> - Supply Chain Security
> - SBOM (Software Bill of Materials)
> - Image Signing
> - Production Checklists
> - Common Mistakes

---

# 📖 Introduction

Anyone can run:

```bash
docker run nginx
```

But running Docker successfully in production requires discipline.

Production Docker emphasizes:

- Consistency
- Security
- Reproducibility
- Automation
- Observability
- Reliability

The goal is to make deployments predictable and recoverable.

---

# Docker Development Lifecycle

A typical workflow:

```text
Developer
      │
      ▼
Write Dockerfile
      │
      ▼
Build Image
      │
      ▼
Run Tests
      │
      ▼
Security Scan
      │
      ▼
Push Registry
      │
      ▼
Deploy
      │
      ▼
Monitor
```

Every step should be automated.

---

# Best Practice 1 — Use Small Base Images

Avoid unnecessarily large images.

❌

```dockerfile
FROM ubuntu
```

Better:

```dockerfile
FROM node:22-alpine
```

Benefits:

✔ Smaller downloads

✔ Faster startup

✔ Lower storage

✔ Reduced attack surface

---

# Best Practice 2 — Multi-Stage Builds

Development tools don't belong in production.

Instead of:

```dockerfile
Node

TypeScript

npm

Source Code
```

Deploy:

```dockerfile
Compiled App

Runtime

Production Dependencies
```

Benefits:

- Smaller images
- Better security
- Faster deployments

---

# Best Practice 3 — Pin Image Versions

Avoid:

```dockerfile
FROM node:latest
```

Prefer:

```dockerfile
FROM node:22.11-alpine
```

Why?

`latest` changes over time.

Version pinning makes builds reproducible.

---

# Best Practice 4 — Run as Non-Root

Avoid:

```dockerfile
USER root
```

Better:

```dockerfile
RUN addgroup -S app && adduser -S app -G app
USER app
```

Benefits:

- Reduced privileges
- Better isolation
- Lower security risk

---

# Best Practice 5 — Keep Images Immutable

Never modify a running container.

Bad:

```bash
docker exec

apt install vim
```

Good:

```
Update Dockerfile

↓

Build New Image

↓

Deploy New Container
```

Containers should be replaceable, not manually maintained.

---

# Immutable Infrastructure

Instead of repairing servers:

```
Old Container

↓

Delete

↓

New Image

↓

New Container
```

Infrastructure changes happen through image builds, not manual edits.

---

# Best Practice 6 — Store Data Outside Containers

Never rely on the writable layer for persistent data.

Bad:

```
Database

↓

Container Filesystem
```

Good:

```
Database

↓

Named Volume

↓

Persistent Storage
```

Containers are ephemeral; volumes preserve data.

---

# Best Practice 7 — Manage Secrets Securely

Never embed secrets in images.

❌

```dockerfile
ENV DB_PASSWORD=secret123
```

Better:

- Docker Secrets
- Environment variables from secure sources
- External secret managers

Examples:

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

---

# Best Practice 8 — Add Health Checks

Example:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

Benefits:

- Automatic health monitoring
- Better orchestration decisions
- Faster failure detection

---

# Best Practice 9 — Use Restart Policies

Example:

```yaml
restart: unless-stopped
```

or

```yaml
restart: always
```

Containers recover automatically after unexpected failures.

---

# Best Practice 10 — Scan Images

Before deployment:

```
Build Image

↓

Security Scan

↓

Deploy
```

Popular tools:

- Docker Scout
- Trivy
- Grype

Scanning detects known vulnerabilities before production.

---

# Best Practice 11 — Use CI/CD

Avoid manual deployments.

Preferred flow:

```text
Git Push

↓

CI Pipeline

↓

Tests

↓

docker build

↓

Security Scan

↓

Push Registry

↓

Deploy
```

This reduces human error and improves consistency.

---

# Best Practice 12 — Tag Images Properly

Avoid:

```
backend:latest
```

Prefer:

```
backend:v1.0.0

backend:v1.0.1

backend:v2.0.0
```

Versioned tags simplify rollbacks and auditing.

---

# Deployment Strategies

## Blue-Green Deployment

Maintain two environments.

```
Blue

(Current)

Green

(New)
```

Deployment flow:

```
Deploy Green

↓

Run Health Checks

↓

Switch Traffic

↓

Remove Blue (optional)
```

Advantages:

- Easy rollback
- Minimal downtime

---

## Canary Deployment

Release gradually.

```
Users

↓

10%

↓

50%

↓

100%
```

If issues appear:

```
Rollback
```

Only a small percentage of users are affected initially.

---

# Supply Chain Security

Your application depends on many external components.

```
Base Image

↓

OS Packages

↓

Language Runtime

↓

Libraries

↓

Application
```

Every dependency should be trusted and regularly updated.

---

# SBOM (Software Bill of Materials)

An SBOM lists everything included in an image.

Example:

```
Image

↓

OpenSSL

↓

Node.js

↓

Express

↓

Prisma

↓

Redis Client
```

Benefits:

- Security audits
- License compliance
- Vulnerability tracking

---

# Image Signing

How do you know an image wasn't tampered with?

Image signing provides integrity verification.

```
Build Image

↓

Sign Image

↓

Push Registry

↓

Verify Before Deployment
```

Common technologies:

- Cosign
- Notary v2

---

# Logging Best Practices

Applications should log to:

```
STDOUT

STDERR
```

Typical production pipeline:

```
Application

↓

Docker Logs

↓

Log Collector

↓

Elasticsearch

↓

Kibana
```

Avoid writing logs to files inside containers.

---

# Monitoring Best Practices

Track:

- CPU
- Memory
- Disk
- Network
- Response Time
- Error Rate
- Restart Count

Popular tools:

- Prometheus
- Grafana
- Datadog
- New Relic

---

# Resource Limits

Every production container should define limits.

Example:

```yaml
cpus: "2"

memory: 2G
```

Benefits:

- Prevent resource starvation
- Improve stability
- Enable predictable scheduling

---

# Enterprise Production Checklist

Before deployment:

✔ Multi-stage build

✔ Small base image

✔ Version-pinned image

✔ Non-root user

✔ Health check

✔ Restart policy

✔ Named volumes

✔ Secure secrets

✔ Resource limits

✔ Security scan

✔ Logging

✔ Monitoring

✔ Backup strategy

✔ Versioned tags

✔ Tested rollback

---

# Common Mistakes

❌ Using `latest`

❌ Running as root

❌ Hardcoding secrets

❌ No health checks

❌ Editing running containers

❌ Ignoring image scanning

❌ Logging inside containers

❌ No monitoring

❌ Large images

❌ Missing backups

---

# Hands-on Lab

Create a production-ready Dockerfile that:

- Uses `node:22-alpine`
- Uses a multi-stage build
- Runs as a non-root user
- Defines a health check
- Pins dependency versions
- Exposes only required ports

Then:

1. Build the image.
2. Scan it for vulnerabilities.
3. Push it to a registry.
4. Deploy using Docker Compose.

---

# Interview Questions

## Why avoid `latest`?

Because it changes over time, making deployments non-reproducible.

---

## What is Immutable Infrastructure?

Infrastructure where changes are made by replacing components with new versions instead of modifying existing ones.

---

## Why use Blue-Green deployments?

To achieve near zero-downtime deployments with easy rollback.

---

## What is a Canary deployment?

A deployment strategy that releases a new version to a small percentage of users before wider rollout.

---

## What is an SBOM?

A Software Bill of Materials that lists the software components included in an image.

---

## Why sign container images?

To verify authenticity and detect tampering before deployment.

---

# Summary

Production-ready Docker is about building **secure, reproducible, observable, and maintainable systems**.

```
Developer

↓

Dockerfile

↓

Build

↓

Scan

↓

Sign

↓

Push

↓

Deploy

↓

Monitor

↓

Improve
```

Key principles:

- Small images
- Immutable infrastructure
- Secure secrets
- Versioned images
- Automated pipelines
- Health checks
- Resource limits
- Monitoring
- Continuous security

Following these practices leads to reliable and scalable containerized applications.

---

# Next Chapter

## Chapter 36 — Module 3 Summary

We'll review everything learned across:

- Docker Swarm
- Docker Engine Internals
- OCI
- containerd & runc
- Docker API
- Image Internals
- Advanced Networking
- BuildKit
- Advanced Storage
- Production Best Practices

After that, we'll begin **Module 4 — Kubernetes Mastery**, starting from first principles.