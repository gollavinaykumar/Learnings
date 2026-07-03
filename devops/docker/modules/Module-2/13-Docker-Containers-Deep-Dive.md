
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 13 — Docker Containers Deep Dive

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What a running container really is
> - How a container is represented internally
> - Container filesystem and writable layer
> - Copy-on-Write
> - PID 1 and why it matters
> - STDIN, STDOUT and STDERR
> - Interactive vs Detached mode
> - Linux signals
> - Restart policies
> - Environment variables
> - Logging
> - Production best practices

---

# 1. What is a Running Container?

A Docker container is **not a mini virtual machine**.

It is:

- A Linux process (or group of processes)
- Running inside isolated namespaces
- Controlled by cgroups
- Started from a Docker Image

```text
Docker Image
      │
docker run
      │
      ▼
Container
      │
      ▼
Linux Process
```

---

# 2. Anatomy of a Container

```text
Container
├── Metadata
├── Writable Layer
├── Main Process (PID 1)
├── Network Namespace
├── Mount Namespace
├── Logs
└── Resource Limits
```

---

# 3. Container Filesystem

Every container filesystem is composed of:

```text
Read-only Image Layers
          +
Writable Container Layer
          =
Container Filesystem
```

Changes made while the container is running go only into the writable layer.

---

# 4. Copy-on-Write (CoW)

Images are immutable.

When an application modifies a file:

```text
Image Layer (Read Only)
        │
Write Request
        ▼
Writable Layer
```

The original image never changes.

Benefits:

- Faster startup
- Lower disk usage
- Shared image layers

---

# 5. Container Process Model

Every container has a **main process**.

```text
Container
   │
   ▼
PID 1
   │
Application
```

If PID 1 exits, Docker considers the container finished.

---

# 6. Why PID 1 Matters

PID 1 has special responsibilities:

- Receives Linux signals
- Reaps zombie processes
- Controls container lifetime

Production images often use:

- tini
- dumb-init

to correctly manage child processes.

---

# 7. STDIN, STDOUT and STDERR

Docker encourages applications to log to:

- STDOUT
- STDERR

instead of writing log files.

This enables:

```bash
docker logs <container>
```

---

# 8. Interactive Mode

```bash
docker run -it ubuntu bash
```

`-i` keeps STDIN open.

`-t` allocates a pseudo-terminal (TTY).

Useful for debugging.

---

# 9. Detached Mode

```bash
docker run -d nginx
```

The application runs in the background.

Typical for:

- Web servers
- APIs
- Databases

---

# 10. Linux Signals

Stopping a container:

```text
docker stop
     │
     ▼
SIGTERM
     │
Graceful Shutdown
     │
Timeout
     │
SIGKILL (if required)
```

`docker kill` sends SIGKILL immediately.

---

# 11. Restart Policies

Examples:

```bash
--restart=no
--restart=always
--restart=unless-stopped
--restart=on-failure
```

Production services commonly use `always` or `unless-stopped`.

---

# 12. Environment Variables

Example:

```bash
docker run \
-e DB_HOST=postgres \
-e NODE_ENV=production \
myapp
```

Use environment variables for configuration.

Avoid hardcoding secrets.

---

# 13. Logging

Useful commands:

```bash
docker logs backend
docker logs -f backend
docker logs --tail 100 backend
```

Production logging should forward STDOUT/STDERR to centralized logging systems.

---

# 14. Debugging Containers

Useful commands:

```bash
docker ps
docker inspect
docker exec -it app sh
docker top
docker stats
docker logs
```

Typical debugging flow:

1. Check status
2. Read logs
3. Inspect metadata
4. Open a shell if necessary

---

# 15. Production Best Practices

- One primary process per container
- Use health checks
- Handle SIGTERM gracefully
- Don't store persistent data in writable layers
- Rebuild images instead of modifying running containers
- Use restart policies

---

# Common Mistakes

❌ Using `docker kill` for normal shutdowns.

❌ Editing files inside running containers.

❌ Depending on container writable storage for databases.

---

# Hands-on Lab

```bash
docker run --name demo -d nginx

docker ps

docker logs demo

docker exec -it demo sh

docker inspect demo

docker stop demo

docker rm demo
```

---

# Interview Questions

**Q. Why is PID 1 special in Docker?**

Because Docker tracks the lifecycle of the container through PID 1. If PID 1 exits, the container exits.

**Q. Why should applications log to STDOUT?**

Because Docker captures STDOUT and STDERR, making logs available through `docker logs` and external log collectors.

**Q. What is the writable layer?**

The writable layer stores runtime changes made by a container without modifying the underlying immutable image.

---

# Next Chapter

**Chapter 14 — Dockerfile (Complete Guide)**

We'll learn every Dockerfile instruction, best practices, multi-stage builds, optimization, and production-ready Dockerfiles.
