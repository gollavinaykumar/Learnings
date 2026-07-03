
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 21 — Docker Security

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - Why Docker security matters
> - Container isolation
> - Root vs non-root containers
> - Linux capabilities
> - Seccomp
> - AppArmor & SELinux (overview)
> - Secrets management
> - Image scanning
> - Supply-chain security
> - Runtime security
> - Production best practices

---

# 📖 Why Docker Security?

Containers improve isolation, but they are **not security boundaries by themselves**.

Security requires:

- Secure images
- Least privilege
- Runtime protection
- Proper configuration

---

# Container Isolation

Docker relies on Linux:

- Namespaces
- cgroups
- Capabilities
- Seccomp

```text
Application
     │
Container
     │
Linux Kernel
```

Isolation reduces the impact of compromise.

---

# Root vs Non-Root Containers

Avoid:

```dockerfile
USER root
```

Prefer:

```dockerfile
USER node
```

Benefits:

- Reduced privileges
- Smaller attack surface
- Better compliance

---

# Linux Capabilities

Instead of full root access, Linux splits privileges into capabilities.

Examples:

- NET_ADMIN
- SYS_ADMIN
- CHOWN

Drop unnecessary capabilities whenever possible.

---

# Seccomp

Seccomp filters Linux system calls.

```text
Application
     │
System Call
     │
Seccomp Profile
     │
Kernel
```

Dangerous syscalls can be blocked.

---

# AppArmor & SELinux

Mandatory Access Control systems.

They restrict:

- File access
- Device access
- Process interactions

Common on enterprise Linux distributions.

---

# Secrets Management

Do **not** bake secrets into images.

Bad:

```dockerfile
ENV DB_PASSWORD=secret123
```

Better:

- Docker Secrets
- Environment variables from secure stores
- External secret managers

---

# Image Scanning

Scan images for vulnerabilities.

Popular tools:

- Docker Scout
- Trivy
- Grype

Scan regularly in CI/CD pipelines.

---

# Supply-Chain Security

Protect the entire build pipeline.

Recommendations:

- Use trusted base images
- Pin image versions
- Verify image signatures
- Review dependencies
- Rebuild images frequently

---

# Runtime Security

Monitor running containers.

Useful practices:

- Read-only root filesystem
- Drop unused capabilities
- Limit CPU & memory
- Restrict network access
- Enable logging and auditing

---

# Best Practices

- Run as non-root.
- Keep images minimal.
- Scan images.
- Keep dependencies updated.
- Never store secrets in images.
- Apply least privilege.
- Use health checks and monitoring.

---

# Common Mistakes

❌ Running everything as root

❌ Using outdated base images

❌ Publishing unnecessary ports

❌ Hardcoding passwords

❌ Ignoring vulnerability reports

---

# Hands-on Lab

Create a non-root Dockerfile:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY . .

RUN addgroup -S app && adduser -S app -G app

USER app

CMD ["node","index.js"]
```

Build and verify the container starts successfully.

---

# Interview Questions

**Why shouldn't containers run as root?**

Because compromising the application would grant elevated privileges inside the container.

**What is Seccomp?**

A Linux feature that filters system calls to reduce the attack surface.

**Why scan Docker images?**

To detect known vulnerabilities before deployment.

---

# Summary

- Docker security is layered.
- Run containers as non-root.
- Minimize capabilities.
- Use Seccomp and MAC frameworks.
- Store secrets securely.
- Scan images continuously.

---

# Next Chapter

## Chapter 22 — Docker Performance

You'll learn:

- Image optimization
- Resource limits
- Build optimization
- Startup performance
- Monitoring
- Production tuning
