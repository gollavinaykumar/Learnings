# 🐳 Docker Mastery Roadmap

# Module 2 — Docker CLI & Everyday Docker

# Chapter 24 — Production Docker

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - How Docker is used in production
> - Production architecture
> - Reverse proxies
> - Zero-downtime deployments
> - High availability
> - Resource planning
> - Logging & monitoring
> - Security hardening
> - CI/CD pipelines
> - Production checklists
> - Disaster recovery
> - Best practices

---

# 📖 Introduction

Running Docker on your laptop is easy.

Running Docker for **millions of users** is different.

A production environment must be:

- Reliable
- Secure
- Scalable
- Observable
- Recoverable

Production Docker is not about running one container.

It is about running an entire platform.

---

# Typical Production Architecture

```
                    Internet
                        │
                        ▼
               Load Balancer
                        │
                        ▼
                 Reverse Proxy
                  (Nginx/Traefik)
                        │
      ┌─────────────────┼─────────────────┐
      ▼                 ▼                 ▼
 Backend-1         Backend-2         Backend-3
      │                 │                 │
      └──────────────┬────────────────────┘
                     ▼
              PostgreSQL Cluster
                     │
                     ▼
                   Redis
```

Notice:

Users never communicate directly with containers.

---

# Why Reverse Proxies?

Instead of exposing every container:

```
Internet

↓

Container
```

We expose only:

```
Internet

↓

Nginx

↓

Docker Containers
```

Benefits:

- SSL termination
- Routing
- Compression
- Load balancing
- Security

Popular reverse proxies:

- Nginx
- Traefik
- HAProxy
- Envoy

---

# High Availability

Never rely on one container.

Bad

```
Backend

↓

Crash

↓

Application Down
```

Good

```
Backend-1

Backend-2

Backend-3
```

If one container crashes:

Traffic automatically goes to another instance.

---

# Load Balancing

Instead of:

```
1000 Users

↓

One Container
```

Use:

```
1000 Users

↓

Load Balancer

↓

Backend-1

Backend-2

Backend-3
```

Benefits:

- Higher throughput
- Better availability
- Easier scaling

---

# Zero-Downtime Deployment

Never stop the old application before the new one is ready.

Bad

```
Stop Old

↓

Deploy New

↓

Users Wait
```

Good

```
Old Version

↓

Start New Version

↓

Health Check

↓

Switch Traffic

↓

Remove Old Version
```

Users experience no downtime.

---

# Health Checks

Production containers should continuously report their health.

Example:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

Possible states:

```
Starting

Healthy

Unhealthy
```

Only healthy containers receive traffic.

---

# Restart Policies

Example:

```yaml
restart: unless-stopped
```

or

```yaml
restart: always
```

If a container crashes:

```
Crash

↓

Restart Automatically
```

---

# Resource Planning

Every production container should have limits.

CPU

```yaml
cpus: "2"
```

Memory

```yaml
memory: 2G
```

Without limits:

One container can consume the entire server.

---

# Storage

Never store important data inside a container.

Bad

```
PostgreSQL

↓

Container Filesystem
```

Good

```
PostgreSQL

↓

Docker Volume

↓

Persistent Storage
```

---

# Image Strategy

Never deploy:

```
latest
```

Instead:

```
backend:v2.4.1
```

Benefits:

- Rollback
- Traceability
- Reproducibility

---

# Logging

Applications should log to:

```
STDOUT

STDERR
```

Production logging pipeline:

```
Application

↓

Docker Logs

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

or

```
Application

↓

Cloud Logging
```

Never rely on log files inside containers.

---

# Monitoring

Production systems monitor:

```
CPU

Memory

Disk

Network

Restart Count

Response Time

Error Rate
```

Common tools:

- Prometheus
- Grafana
- Datadog
- New Relic

---

# Security Hardening

Checklist:

✔ Run as non-root

✔ Minimal images

✔ Read-only filesystem (where possible)

✔ Secrets outside images

✔ Scan images

✔ Use trusted registries

✔ Keep images updated

✔ Restrict exposed ports

---

# CI/CD Pipeline

Typical production workflow:

```
Developer

↓

Git Push

↓

GitHub Actions

↓

Run Tests

↓

docker build

↓

Image Scan

↓

Push Registry

↓

Deploy

↓

Health Check

↓

Production
```

Notice:

Production servers deploy **images**, not source code.

---

# Backup Strategy

Production data must be backed up.

Example:

```
PostgreSQL Volume

↓

Nightly Backup

↓

Cloud Storage

↓

Disaster Recovery
```

Containers are replaceable.

Data is not.

---

# Disaster Recovery

Suppose an entire server fails.

Recovery steps:

```
New Server

↓

Install Docker

↓

Pull Images

↓

Restore Volumes

↓

Start Containers
```

Infrastructure becomes reproducible.

---

# Scaling

When traffic increases:

```
Backend

↓

Scale to

↓

5 Containers

↓

10 Containers

↓

20 Containers
```

Scaling containers is much easier than provisioning new virtual machines.

---

# Production Folder Structure

```
project/

├── Dockerfile
├── compose.yaml
├── .dockerignore
├── .env
├── nginx/
├── monitoring/
├── logs/
└── backups/
```

Keep infrastructure organized.

---

# Production Checklist

Before deployment:

✔ Small image

✔ Multi-stage build

✔ Non-root user

✔ Health check

✔ Restart policy

✔ CPU limit

✔ Memory limit

✔ Named volumes

✔ Environment variables

✔ Secrets management

✔ Versioned image tag

✔ Logging

✔ Monitoring

✔ Backups

✔ Image scanning

✔ Tested rollback

---

# Common Production Mistakes

❌ Using `latest`

❌ Running as root

❌ No health checks

❌ No restart policy

❌ No backups

❌ No monitoring

❌ Exposing database ports publicly

❌ Storing secrets in Dockerfiles

❌ Editing running containers manually

---

# Hands-on Lab

Build a production-ready Node.js container:

1. Create a multi-stage Dockerfile.
2. Run as a non-root user.
3. Add a health check.
4. Set memory and CPU limits.
5. Use a named volume for uploads.
6. Add restart policy.
7. Deploy using Docker Compose.

---

# Interview Questions

## Why should production avoid `latest`?

Because `latest` changes over time and makes deployments non-reproducible.

---

## Why use a reverse proxy?

To provide SSL, routing, load balancing, and a single entry point.

---

## Why run containers as non-root?

To reduce the impact of security vulnerabilities.

---

## Why use health checks?

To ensure traffic is sent only to healthy containers.

---

## Why are Docker volumes critical?

Because containers are ephemeral, while production data must persist.

---

## Why deploy images instead of source code?

Images are immutable, versioned, and tested. Every environment runs the exact same artifact.

---

# Production Readiness Checklist

```
Dockerfile
      ✔

Multi-stage Build
      ✔

Small Image
      ✔

Non-root User
      ✔

Health Check
      ✔

Restart Policy
      ✔

Resource Limits
      ✔

Volumes
      ✔

Monitoring
      ✔

Logging
      ✔

Backups
      ✔

CI/CD
      ✔

Versioned Images
      ✔

Image Scanning
      ✔

Ready for Production ✅
```

---

# Summary

Production Docker is about much more than running containers.

A production-ready deployment should be:

- Secure
- Observable
- Scalable
- Recoverable
- Reproducible

By combining multi-stage builds, proper networking, persistent storage, health checks, monitoring, and CI/CD, Docker becomes a reliable platform for running modern applications at scale.

---

# Next Chapter

## Chapter 25 — Module 2 Summary

We'll review everything learned in Module 2, connect all concepts together, and prepare for the next stage of the course:

**Module 3 — Advanced Docker & Orchestration**, where we'll cover Docker Swarm, advanced networking, image internals, and then transition into Kubernetes.