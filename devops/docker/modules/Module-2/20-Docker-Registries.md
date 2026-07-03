
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 20 — Docker Registries

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - What a Docker Registry is
> - Docker Hub
> - Private Registries
> - Image Names & Tags
> - Push & Pull
> - Authentication
> - Enterprise Registries
> - Registry Best Practices

---

# 📖 What is a Docker Registry?

A Docker Registry is a service that stores and distributes Docker images.

```text
Developer
    │
docker push
    ▼
Docker Registry
    ▲
docker pull
    │
Production Server
```

Examples:

- Docker Hub
- GitHub Container Registry (GHCR)
- Amazon ECR
- Google Artifact Registry
- Azure Container Registry

---

# Why Registries?

Instead of copying image files manually, teams publish images to a registry.

```text
Build Image
    │
Push
    │
Registry
    │
Pull
    │
Deploy
```

---

# Docker Hub

Default public registry.

```bash
docker pull nginx
```

Internally:

```text
docker.io/library/nginx:latest
```

---

# Image Naming

```text
registry/repository:tag
```

Examples:

```text
nginx:latest
postgres:16
ghcr.io/company/backend:v1.2.0
123456789012.dkr.ecr.us-east-1.amazonaws.com/api:v5
```

---

# Tags

Tags identify image versions.

```text
backend:v1
backend:v2
backend:1.0.3
backend:latest
```

Best Practice:

Use versioned tags in production.

---

# Pull Images

```bash
docker pull nginx:latest
```

Flow:

```text
Registry
   │
Download Layers
   │
Local Image Cache
```

---

# Push Images

Tag:

```bash
docker tag myapp:v1 username/myapp:v1
```

Push:

```bash
docker push username/myapp:v1
```

---

# Authentication

Login:

```bash
docker login
```

Logout:

```bash
docker logout
```

Credentials are used to push and pull private images.

---

# Private Registries

Companies usually use private registries.

Benefits:

- Private source images
- Security
- Compliance
- Internal distribution

---

# Enterprise Registries

Popular choices:

- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- GitHub Container Registry
- Harbor

---

# CI/CD Workflow

```text
Developer
    │
Git Push
    │
CI Pipeline
    │
docker build
    │
docker push
    │
Registry
    │
Production
docker pull
```

Applications are deployed from images—not source code.

---

# Image Versioning

Avoid:

```text
latest
```

Prefer:

```text
v1.0.0
v1.0.1
v2.0.0
```

Makes rollbacks predictable.

---

# Registry Security

- Scan images
- Sign images
- Restrict access
- Remove unused images
- Store secrets outside images

---

# Common Commands

Search:

```bash
docker search nginx
```

Login:

```bash
docker login
```

Push:

```bash
docker push username/app:v1
```

Pull:

```bash
docker pull username/app:v1
```

Logout:

```bash
docker logout
```

---

# Best Practices

- Pin image versions
- Use private registries for internal software
- Automate image publishing in CI/CD
- Enable image scanning
- Clean old image tags

---

# Common Mistakes

❌ Deploying `latest` to production

❌ Storing secrets inside images

❌ Forgetting to authenticate before pushing

---

# Hands-on Lab

```bash
docker login

docker tag myapp:v1 username/myapp:v1

docker push username/myapp:v1

docker pull username/myapp:v1
```

---

# Interview Questions

**What is a Docker Registry?**

A service that stores and distributes Docker images.

**Difference between Docker Hub and a private registry?**

Docker Hub is public by default; private registries are controlled by an organization.

**Why avoid `latest` in production?**

Because it changes over time and makes deployments non-reproducible.

---

# Summary

- Registries store Docker images.
- Docker Hub is the default registry.
- Images should be versioned with tags.
- CI/CD pipelines build, push and deploy images.
- Private registries improve security and governance.

---

# Next Chapter

## Chapter 21 — Docker Security

You'll learn:

- Container isolation
- Root vs non-root containers
- Capabilities
- Seccomp
- AppArmor
- Secrets
- Image scanning
- Security best practices
