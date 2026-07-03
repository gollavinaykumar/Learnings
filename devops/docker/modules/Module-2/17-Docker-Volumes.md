
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 17 — Docker Volumes

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - Why Docker Volumes exist
> - Container storage
> - Named Volumes
> - Bind Mounts
> - Anonymous Volumes
> - tmpfs mounts
> - Volume Drivers
> - PostgreSQL & Redis persistence
> - Backup & Restore
> - Production best practices

---

# 📖 Introduction

Containers are **ephemeral**.

If a container stores data in its writable layer and is deleted, the data is lost.

Docker Volumes solve this problem by storing data outside the container's writable layer.

```text
Container
     │
Writable Layer
     │
Deleted
     │
❌ Data Lost
```

With a volume:

```text
Container
     │
Volume
     │
Deleted
     │
✅ Data Remains
```

---

# Why Volumes?

Applications like:

- PostgreSQL
- MySQL
- Redis
- MongoDB
- Elasticsearch

must preserve data even when containers are recreated.

---

# Container Storage

Every container has:

```text
Image Layers (Read Only)

+

Writable Layer
```

The writable layer is temporary.

Volumes are persistent.

---

# Named Volumes

Create:

```bash
docker volume create postgres-data
```

Use:

```bash
docker run \
-v postgres-data:/var/lib/postgresql/data \
postgres
```

Docker manages the storage location.

---

# Bind Mounts

Bind a host directory.

```bash
docker run \
-v $(pwd):/app \
node:22
```

Useful for:

- Local development
- Live code changes
- Configuration files

---

# Anonymous Volumes

Create automatically:

```bash
docker run \
-v /data \
nginx
```

Docker assigns a random volume name.

Useful for temporary storage.

---

# tmpfs Mounts

Store data only in memory.

```bash
docker run \
--tmpfs /tmp \
nginx
```

Benefits:

- Fast
- Never written to disk
- Automatically removed

Useful for temporary files and sensitive data.

---

# Volume Drivers

Docker supports different drivers.

Examples:

- local
- NFS
- CIFS
- Cloud storage plugins

Allows persistent storage across hosts.

---

# OverlayFS & Volumes

Without volumes:

```text
Image
   │
Writable Layer
```

With volumes:

```text
Image
   │
Writable Layer
   │
Volume
```

Application writes directly to the mounted volume.

---

# PostgreSQL Persistence

Without volume:

```text
Container Deleted

↓

Database Lost
```

With volume:

```bash
docker run \
-v postgres-data:/var/lib/postgresql/data \
postgres
```

Delete the container.

Create another container using the same volume.

Database is still available.

---

# Redis Persistence

Redis can persist snapshots to a mounted volume.

```bash
docker run \
-v redis-data:/data \
redis
```

---

# Backup & Restore

Backup:

```bash
docker run --rm \
-v postgres-data:/data \
-v $(pwd):/backup \
busybox \
tar czf /backup/db.tar.gz /data
```

Restore by extracting into the volume.

---

# Volume Commands

Create:

```bash
docker volume create app-data
```

List:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect app-data
```

Remove:

```bash
docker volume rm app-data
```

Remove unused:

```bash
docker volume prune
```

---

# Best Practices

- Store database data in volumes.
- Use bind mounts for development.
- Avoid storing persistent data in container writable layers.
- Back up important volumes regularly.
- Name important volumes.

---

# Common Mistakes

❌ Storing database files inside the container.

❌ Deleting containers without persistent storage.

❌ Mounting sensitive host directories unnecessarily.

---

# Hands-on Lab

Create a volume:

```bash
docker volume create demo-data
```

Run PostgreSQL:

```bash
docker run -d \
--name postgres \
-v demo-data:/var/lib/postgresql/data \
postgres
```

Delete the container.

Create a new container using the same volume.

Verify that the database still exists.

---

# Interview Questions

**Why use Docker Volumes?**

To persist data independently of container lifecycle.

**Difference between a Volume and a Bind Mount?**

A Volume is managed by Docker. A Bind Mount maps a specific host directory.

**When should tmpfs be used?**

For temporary or sensitive data that should never be written to disk.

---

# Summary

- Volumes provide persistent storage.
- Named volumes are managed by Docker.
- Bind mounts map host directories.
- Anonymous volumes are auto-created.
- tmpfs stores data only in memory.
- Databases should always use persistent volumes.

---

# Next Chapter

## Chapter 18 — Docker Networking (Very Deep)

We'll learn:

- Linux Bridges
- veth pairs
- Bridge, Host, None, Overlay and Macvlan networks
- Docker DNS
- NAT & iptables
- Port publishing
- Packet flow
- Production networking
