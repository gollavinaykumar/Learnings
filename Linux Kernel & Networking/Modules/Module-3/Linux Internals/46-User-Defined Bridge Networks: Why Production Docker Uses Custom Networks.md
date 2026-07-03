Most beginners use the default Docker network.

```bash
docker run nginx
```

Everything works.

But in real production environments,

experienced Docker engineers almost never rely on the default bridge network.

Instead, they create custom networks like this:

```bash
docker network create backend
```

or

```yaml
services:
  app:
    networks:
      - backend

  postgres:
    networks:
      - backend
```

Why?

Questions you may have:

- Why isn't the default bridge enough?
- What makes a user-defined bridge better?
- Why can containers find each other by name?
- Where does Docker DNS come from?
- Can we create multiple bridges?
- How do microservices communicate?
- Why does Docker Compose automatically create networks?

The answer is:

**User-Defined Bridge Networks.**

This is the networking mode used by almost every production Docker application.

After this chapter, you'll understand why custom bridge networks are far superior to the default Docker bridge.

---

# 👑 Linux Kernel & Networking Mastery

# Module 5 — Linux Namespaces & Container Foundations

# Chapter 46 — User-Defined Bridge Networks: Why Production Docker Uses Custom Networks

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why User-Defined Networks exist
> - Default Bridge vs Custom Bridge
> - Docker Embedded DNS
> - Automatic Container Discovery
> - Network Isolation
> - Multiple Bridge Networks
> - Multi-Network Containers
> - Docker Compose Networking
> - Production Best Practices

---

# 📖 Why Not Use the Default Bridge?

Suppose you start:

```bash
docker run --name app nginx
```

and

```bash
docker run --name db postgres
```

Both join:

```
docker0
```

The application can communicate using IP addresses.

Example:

```
172.17.0.2

↓

172.17.0.3
```

But IP addresses change.

This is not ideal.

---

# Problem with IP Addresses

Suppose PostgreSQL restarts.

Yesterday:

```
172.17.0.3
```

Today:

```
172.17.0.5
```

Your application breaks.

Hardcoding IP addresses is a bad practice.

---

# User-Defined Bridge

Instead create:

```bash
docker network create backend
```

Architecture:

```
Application

↓

backend Bridge

↓

Database
```

Docker now provides additional features.

---

# Docker Embedded DNS

This is the biggest improvement.

Instead of:

```
172.18.0.2
```

Applications simply use:

```
postgres
```

Docker automatically resolves:

```
postgres

↓

172.18.0.2
```

---

# How It Works

Application:

```
Connect

↓

postgres
```

Docker DNS replies:

```
postgres

↓

172.18.0.2
```

Applications never need to know IP addresses.

---

# Container Discovery

Suppose:

```
Frontend

Backend

Redis

Postgres
```

Every container automatically discovers the others by name.

Example:

```
redis

↓

172.18.0.4
```

---

# Docker Embedded DNS Server

Every user-defined bridge contains an embedded DNS service.

```
Container

↓

DNS Query

↓

Docker DNS

↓

Container IP
```

No external DNS server required.

---

# Default Bridge vs Custom Bridge

Default Bridge:

```
Container A

↓

IP Only
```

Custom Bridge:

```
Container A

↓

DNS Name

↓

Container B
```

Name-based communication is much easier.

---

# Network Isolation

Suppose:

```
frontend
```

Network:

```
frontend-net
```

Database:

```
backend-net
```

Containers on different bridge networks cannot communicate unless explicitly connected.

---

# Multiple Networks

Docker allows multiple bridges.

Example:

```
frontend-net
```

```
backend-net
```

```
monitoring-net
```

Each is independent.

---

# Multi-Network Container

Suppose:

```
API Server
```

Needs access to:

- Frontend
- Database

Docker allows:

```
API

↓

frontend-net
```

and

```
API

↓

backend-net
```

One container can join multiple networks.

---

# Architecture

```
Frontend

↓

frontend-net

↓

API

↓

backend-net

↓

Postgres
```

API acts as the bridge between application layers.

---

# Docker Compose

Compose automatically creates a user-defined bridge.

Example:

```yaml
services:
  app:
  db:
```

Docker Compose creates something similar to:

```
project_default
```

Both services join it automatically.

---

# Service Discovery

Application configuration:

Instead of:

```
172.18.0.3
```

Use:

```
postgres
```

Database migrations,

connection pools,

ORMs,

all work using container names.

---

# Inspecting Networks

View:

```bash
docker network ls
```

Inspect:

```bash
docker network inspect backend
```

Observe:

- Subnet
- Gateway
- Connected Containers

---

# Connecting Existing Container

Suppose:

```
redis
```

Should join:

```
backend
```

Command:

```bash
docker network connect backend redis
```

No container restart required.

---

# Disconnecting

```bash
docker network disconnect backend redis
```

The container leaves the network.

---

# Packet Flow

Application:

```
postgres

↓

Docker DNS

↓

Container IP

↓

Bridge

↓

veth

↓

Database
```

Applications never see the IP lookup.

---

# Why Production Uses Custom Networks

Benefits:

✔ Automatic DNS

✔ Better Isolation

✔ Easier Configuration

✔ No Hardcoded IPs

✔ Cleaner Architecture

✔ Microservice Friendly

---

# Real Microservice Example

```
Frontend

↓

api-service

↓

Redis

↓

Postgres

↓

RabbitMQ
```

Each service connects using names:

```
redis

postgres

rabbitmq
```

Never raw IP addresses.

---

# Docker Doesn't Use External DNS

For containers on the same user-defined network:

```
redis

↓

Docker Embedded DNS

↓

172.18.0.4
```

Resolution happens inside Docker.

---

# Docker Networking Stack

Still uses:

```
Network Namespace

↓

veth

↓

Linux Bridge

↓

Routing
```

User-defined bridges simply add:

```
Embedded DNS

+

Better Isolation
```

---

# Bridge Comparison

| Feature | Default Bridge | User-Defined Bridge |
|----------|----------------|---------------------|
| Automatic DNS | ❌ | ✅ |
| Container Discovery | Limited | ✅ |
| Isolation | Shared default network | Per-network |
| Production Recommended | ❌ | ✅ |
| Docker Compose Default | ❌ | ✅ |

---

# Why User-Defined Networks Matter

Almost every production Docker deployment uses them.

Examples:

- Docker Compose
- Microservices
- API Servers
- Databases
- Message Queues
- Monitoring Stacks

---

# Hands-on Lab

## Lab 1 — Create Network

```bash
docker network create backend
```

---

## Lab 2 — View Networks

```bash
docker network ls
```

---

## Lab 3 — Run PostgreSQL

```bash
docker run -d \
--name postgres \
--network backend \
postgres
```

---

## Lab 4 — Run Application

```bash
docker run -it \
--network backend \
alpine sh
```

Inside:

```sh
ping postgres
```

Observe successful name resolution.

---

## Lab 5 — Inspect Network

```bash
docker network inspect backend
```

Observe:

- Gateway
- Subnet
- Connected Containers

---

## Lab 6 — Connect Existing Container

```bash
docker network connect backend nginx
```

Observe that the container now participates in the custom bridge network.

---

# Interview Questions

## Why are User-Defined Bridge Networks preferred over the default bridge?

Because they provide automatic DNS-based service discovery, better isolation, and are easier to manage in production.

---

## How does Docker resolve container names?

Docker runs an embedded DNS service for user-defined bridge networks.

---

## Can a container join multiple networks?

Yes.

A single container can connect to multiple Docker networks simultaneously.

---

## Does a User-Defined Bridge still use Linux Bridge?

Yes.

Docker creates a Linux Bridge for the network and connects containers using `veth` pairs.

---

## Does Docker Compose create User-Defined Networks?

Yes.

By default, Docker Compose creates a project-specific user-defined bridge network.

---

## Why shouldn't applications use container IP addresses?

Container IPs may change when containers are recreated.

Container names provide stable service discovery.

---

# Summary

User-Defined Bridge Networks are the production-ready way to connect Docker containers.

```
Application

↓

Container Name

↓

Docker DNS

↓

Container IP

↓

Linux Bridge

↓

veth

↓

Destination Container
```

Key concepts:

- Custom bridge networks provide automatic DNS.
- Containers communicate using names instead of IPs.
- Networks provide isolation between groups of containers.
- One container can join multiple networks.
- Docker Compose relies on user-defined bridges by default.
- Linux Bridge, `veth`, and Network Namespaces remain the underlying implementation.

Understanding User-Defined Bridge Networks prepares you for Docker overlay networking, Docker Swarm, Kubernetes CNI plugins, and multi-host container networking.

---

# Next Chapter

## Chapter 47 — Docker Overlay Network: Connecting Containers Across Multiple Hosts

We'll explore:

- Why Overlay Networks exist
- Multi-Host Networking
- VXLAN
- Encapsulation
- Docker Swarm Networking
- Service Discovery
- Routing Mesh
- Encrypted Overlay Traffic
- Cross-Host Packet Flow
- Overlay Network Performance