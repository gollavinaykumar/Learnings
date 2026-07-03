# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 2 — Docker CLI Deep Dive

> **Learning Objective**
>
> Learn the Docker commands you will use every day, what they do internally,
> when to use them, and common mistakes.

---

# 📖 Introduction

The Docker CLI (Command Line Interface) is your primary way to interact with Docker.

Every command you execute follows this flow:

```text
You
 │
 ▼
Docker CLI
 │
 ▼
Docker Daemon
 │
 ▼
containerd
 │
 ▼
runc
 │
 ▼
Container
```

The CLI **does not perform the work itself**. It sends requests to the Docker Daemon.

---

# Docker CLI Command Groups

Docker commands are grouped by resource.

```text
docker image
docker container
docker network
docker volume
docker compose
docker system
docker builder
docker logs
docker exec
```

Older shorthand commands still work:

```bash
docker images
docker ps
```

---

# 1. docker pull

Downloads an image from a registry.

```bash
docker pull nginx
```

Internal flow:

```text
CLI
 │
 ▼
Daemon
 │
 ▼
Docker Hub
 │
 ▼
Download Layers
 │
 ▼
Store Locally
```

Check downloaded images:

```bash
docker images
```

---

# 2. docker run

Creates **and starts** a new container.

```bash
docker run nginx
```

Equivalent to:

```text
docker create
      ↓
docker start
```

Common options:

```bash
docker run -d nginx
docker run --name web nginx
docker run -p 8080:80 nginx
docker run -e NODE_ENV=production myapp
```

---

# 3. docker create

Creates a container but does not start it.

```bash
docker create nginx
```

Use when you want to inspect or configure before starting.

---

# 4. docker start / stop / restart

Start an existing container:

```bash
docker start web
```

Gracefully stop:

```bash
docker stop web
```

Restart:

```bash
docker restart web
```

---

# 5. docker ps

Running containers:

```bash
docker ps
```

All containers:

```bash
docker ps -a
```

---

# 6. docker exec

Execute a command inside a running container.

Open a shell:

```bash
docker exec -it postgres bash
```

For Alpine:

```bash
docker exec -it app sh
```

Example:

```bash
docker exec -it postgres psql -U postgres
```

Use this for debugging and administration.

---

# 7. docker logs

View application output.

```bash
docker logs backend
```

Live logs:

```bash
docker logs -f backend
```

Last 100 lines:

```bash
docker logs --tail 100 backend
```

---

# 8. docker inspect

Shows detailed JSON metadata.

```bash
docker inspect backend
```

Useful information:

- IP Address
- Mounts
- Networks
- Environment Variables
- Restart Policy

---

# 9. docker stats

Monitor resource usage.

```bash
docker stats
```

Displays:

- CPU
- Memory
- Network
- Block I/O

Useful in production troubleshooting.

---

# 10. docker top

Shows processes running inside a container.

```bash
docker top backend
```

---

# 11. docker cp

Copy files.

Host → Container

```bash
docker cp app.js backend:/app
```

Container → Host

```bash
docker cp backend:/app/log.txt .
```

---

# 12. docker rm

Remove containers.

```bash
docker rm backend
```

Force remove:

```bash
docker rm -f backend
```

---

# 13. docker rmi

Remove images.

```bash
docker rmi nginx
```

Cannot remove an image currently used by containers.

---

# 14. docker system

Disk usage:

```bash
docker system df
```

Clean unused resources:

```bash
docker system prune
```

Be careful in production.

---

# Common Workflow

```text
docker pull
      │
docker run
      │
docker ps
      │
docker logs
      │
docker exec
      │
docker stop
      │
docker rm
```

---

# Production Example

Deploy backend:

```bash
docker pull company/backend:v2

docker run -d \
  --name backend \
  --restart=always \
  -p 8080:8080 \
  company/backend:v2
```

Debug:

```bash
docker logs backend

docker exec -it backend sh
```

---

# Common Mistakes

❌ Using `docker run` every time.

This creates a **new** container.

✔ Use:

```bash
docker start existing-container
```

---

❌ Editing files inside containers.

✔ Rebuild the image instead.

---

# Best Practices

- Name important containers.
- Use `docker logs` before `docker exec`.
- Prefer `docker stop` over `docker kill`.
- Remove unused containers and images regularly.
- Use `docker inspect` when debugging networking or mounts.

---

# Hands-on Lab

1.

```bash
docker pull nginx
```

2.

```bash
docker run --name web -d -p 8080:80 nginx
```

3.

```bash
docker ps
```

4.

```bash
docker logs web
```

5.

```bash
docker exec -it web sh
```

6.

```bash
docker inspect web
```

7.

```bash
docker stop web
```

8.

```bash
docker rm web
```

---

# Summary

- Docker CLI communicates with the Docker Daemon.
- `docker run` creates and starts containers.
- `docker exec` is used to work inside running containers.
- `docker logs` is your first debugging tool.
- `docker inspect` provides detailed metadata.
- `docker stats` helps monitor resource usage.

---

# Interview Questions

**Q. Does the Docker CLI create containers?**

No. It sends requests to the Docker Daemon.

**Q. Difference between `docker run` and `docker start`?**

`docker run` creates a new container and starts it.

`docker start` starts an existing stopped container.

---

# What's Next

**Module 2 – Chapter 3: Docker Images Deep Dive**

You'll learn image tags, registries, image history, layers, caching, image optimization and best practices.
