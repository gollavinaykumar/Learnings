# 🐳 Docker Mastery Roadmap

# Module 2 — Docker CLI & Everyday Docker

# Chapter 23 — Docker Debugging

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - How to debug Docker containers
> - Why containers fail to start
> - How to inspect container internals
> - Debugging networking
> - Debugging volumes
> - Debugging resource issues
> - Debugging Docker builds
> - Production troubleshooting workflow
> - Best practices

---

# 📖 Introduction

Every developer eventually encounters this situation:

```bash
docker run myapp
```

Instead of starting successfully:

```
Container exited immediately.
```

Or:

```
CrashLoop

Restarting...

Restarting...
```

Or:

```
Connection Refused
```

Or:

```
Out Of Memory
```

Docker itself is rarely the problem.

Most issues come from:

- Wrong image
- Wrong configuration
- Missing environment variables
- Network problems
- Volume issues
- Resource limits

Learning to debug systematically is an essential production skill.

---

# The Docker Debugging Mindset

Never guess.

Instead follow a process.

```
Problem

↓

Observe

↓

Collect Information

↓

Identify Root Cause

↓

Fix

↓

Verify
```

Professional engineers avoid random trial-and-error.

---

# Common Docker Problems

Most production issues fall into one of these categories:

```
Container won't start

↓

Application crashes

↓

Networking

↓

Volumes

↓

Permissions

↓

Memory

↓

CPU

↓

Image Issues

↓

Configuration
```

---

# 23.1 Container Won't Start

Example

```bash
docker run myapp
```

Result

```
Exited (1)
```

First check:

```bash
docker ps -a
```

Example

```
CONTAINER ID

STATUS

Exited (1)

backend
```

The exit code tells us the application failed.

---

# 23.2 Check Logs First

Never SSH into the container first.

Always check:

```bash
docker logs backend
```

Example

```
Database connection failed
```

Or

```
Module not found
```

Or

```
Permission denied
```

Logs usually reveal the problem immediately.

---

# Live Logs

Monitor in real time:

```bash
docker logs -f backend
```

Useful when restarting a service.

---

# Last N Lines

```bash
docker logs --tail 100 backend
```

Avoids reading massive logs.

---

# 23.3 Inspect the Container

View complete metadata.

```bash
docker inspect backend
```

Useful information:

```
Image

Mounts

Environment Variables

Network

Restart Policy

Ports

Health Status

IP Address
```

This is one of the most powerful debugging commands.

---

# 23.4 Open a Shell

Sometimes you need to inspect the container manually.

```bash
docker exec -it backend sh
```

or

```bash
docker exec -it backend bash
```

Inside the container:

```
ls

pwd

env

cat

ps

top
```

Useful for verifying:

- Files
- Configuration
- Environment variables

---

# 23.5 Check Running Processes

```bash
docker top backend
```

Example

```
PID

COMMAND

node index.js
```

If no process exists:

Container exits immediately.

---

# 23.6 Monitor Resources

```bash
docker stats
```

Displays:

```
CPU

Memory

Block I/O

Network I/O

PIDs
```

Useful for detecting:

- Memory leaks
- CPU spikes
- Excessive resource usage

---

# 23.7 Debugging Build Problems

Suppose:

```bash
docker build .
```

Fails.

Example

```
COPY failed
```

Check:

```
Dockerfile

↓

Build Context

↓

.dockerignore
```

Most build failures occur because files are not in the build context.

---

# Build Without Cache

Sometimes cached layers hide issues.

```bash
docker build --no-cache .
```

Docker rebuilds every layer.

Useful after dependency changes.

---

# 23.8 Debugging Networking

Container can't reach database.

Example

```
ECONNREFUSED
```

Check:

```bash
docker network ls
```

Inspect:

```bash
docker network inspect app-network
```

Verify both containers belong to the same network.

---

# Test Connectivity

Open a shell:

```bash
docker exec -it backend sh
```

Test:

```bash
ping postgres
```

Or

```bash
curl http://backend:3000
```

If DNS fails:

```
Container not attached

↓

Wrong Network
```

---

# 23.9 Debugging Port Mapping

Application works inside container.

Host can't access it.

Check:

```bash
docker ps
```

Example

```
0.0.0.0:8080->80
```

If missing:

Port wasn't published.

Correct:

```bash
docker run -p 8080:80 nginx
```

---

# 23.10 Debugging Volumes

Suppose:

```
Database Lost
```

Usually:

```
No Volume Mounted
```

Check:

```bash
docker inspect postgres
```

Look for:

```
Mounts
```

Verify:

```
Source

Destination
```

---

# 23.11 Debugging Environment Variables

Application says:

```
DB_HOST missing
```

Verify:

```bash
docker exec backend env
```

Or

```bash
docker inspect backend
```

Look for:

```
Env
```

---

# 23.12 Debugging Health Checks

Check health:

```bash
docker inspect backend
```

Look for:

```
Health

↓

healthy

unhealthy

starting
```

An unhealthy container may be restarted automatically.

---

# 23.13 Debugging Restart Loops

Container repeatedly restarts.

Check:

```bash
docker logs
```

Inspect restart policy:

```bash
docker inspect backend
```

Common causes:

- Missing environment variables
- Database unavailable
- Application crashes immediately
- Wrong command

---

# 23.14 Docker Events

Monitor Docker in real time.

```bash
docker events
```

Example

```
Container Started

Container Stopped

Network Created

Image Pulled
```

Useful for production debugging.

---

# 23.15 Docker System Information

```bash
docker info
```

Shows:

```
Storage Driver

Logging Driver

CPUs

Memory

Docker Root Dir
```

Useful when diagnosing host issues.

---

# 23.16 Production Debugging Workflow

Professional engineers follow a consistent process.

```
Application Down

↓

docker ps

↓

docker logs

↓

docker inspect

↓

docker stats

↓

docker exec

↓

docker network inspect

↓

Fix Root Cause

↓

Restart
```

Avoid random debugging.

---

# Common Debugging Scenarios

## Container exits immediately

Check:

```
docker logs
```

---

## Database connection fails

Check:

```
docker network inspect

↓

Environment Variables

↓

Database Running?
```

---

## Application unreachable

Check:

```
Port Mapping

↓

Firewall

↓

Reverse Proxy
```

---

## Out Of Memory

Check:

```
docker stats
```

Look for:

```
Memory

100%
```

---

## Image Build Fails

Check:

```
Dockerfile

↓

COPY

↓

Build Context
```

---

# Useful Commands

```bash
docker ps

docker logs

docker inspect

docker exec

docker stats

docker top

docker events

docker network inspect

docker volume inspect
```

Master these commands.

They solve most Docker issues.

---

# Best Practices

✔ Read logs first

✔ Inspect before changing

✔ Monitor resources

✔ Use health checks

✔ Name containers

✔ Keep logs on stdout

✔ Use Docker Compose for multi-container debugging

✔ Don't debug by rebuilding immediately

---

# Common Mistakes

❌ Guessing

❌ Ignoring logs

❌ Editing containers manually

❌ Restarting repeatedly without reading errors

❌ Blaming Docker when the application is failing

---

# Hands-on Lab

Run:

```bash
docker run --name nginx-demo -d nginx
```

Inspect:

```bash
docker ps

docker logs nginx-demo

docker inspect nginx-demo

docker exec -it nginx-demo sh

docker top nginx-demo

docker stats
```

Then stop it:

```bash
docker stop nginx-demo
```

Observe:

```bash
docker events
```

---

# Interview Questions

## What is the first command you run when a container fails?

```
docker logs
```

---

## How do you inspect environment variables?

```
docker inspect

or

docker exec env
```

---

## How do you check CPU and memory usage?

```
docker stats
```

---

## How do you debug networking?

```
docker network inspect

ping

curl
```

---

## Which command gives complete container metadata?

```
docker inspect
```

---

# Summary

Docker debugging is about following a **systematic process**, not guessing.

The most important commands are:

- `docker logs`
- `docker inspect`
- `docker exec`
- `docker stats`
- `docker network inspect`
- `docker events`

Mastering these commands allows you to diagnose the majority of real-world Docker issues efficiently.

---

# Next Chapter

## Chapter 24 — Production Docker

We'll learn:

- Production deployment architecture
- Reverse proxies
- High availability
- Zero-downtime deployments
- Resource planning
- Logging & monitoring
- Security hardening
- CI/CD integration
- Production checklists