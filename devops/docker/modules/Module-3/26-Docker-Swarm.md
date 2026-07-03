# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 26 — Docker Swarm

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Docker Swarm exists
> - Docker Swarm Architecture
> - Cluster concepts
> - Manager Nodes
> - Worker Nodes
> - Services
> - Tasks
> - Desired State
> - Overlay Networking
> - Service Discovery
> - Routing Mesh
> - Scaling
> - Rolling Updates
> - Secrets & Configs
> - High Availability
> - Production Best Practices
> - Hands-on Labs
> - Interview Questions

---

# 📖 Introduction

So far, we've learned how to run containers on **one machine**.

Example:

```
Laptop

├── Backend Container
├── PostgreSQL Container
├── Redis Container
└── Nginx Container
```

This works perfectly.

But what happens when:

- One server is not enough?
- 1 million users visit your application?
- Your server crashes?
- You need multiple backend containers?
- You need zero downtime deployments?

Docker Compose cannot solve these problems.

We need orchestration.

---

# What is Container Orchestration?

Imagine you have 200 containers.

Questions arise:

- Which server should run them?
- What if one crashes?
- How do we scale?
- How do services discover each other?
- How do we update applications without downtime?

Managing all of this manually is nearly impossible.

Container orchestration automates these tasks.

---

# Why Docker Swarm?

Docker Swarm allows multiple Docker hosts to behave as **one cluster**.

Instead of managing individual machines:

```
Server A

Server B

Server C
```

You manage:

```
Docker Cluster
```

This provides:

✔ High Availability

✔ Load Balancing

✔ Scaling

✔ Service Discovery

✔ Rolling Updates

✔ Self-Healing

---

# Before Swarm

Suppose your company has three servers.

```
Server 1

Backend

Database

-----------------

Server 2

Backend

-----------------

Server 3

Redis
```

Problems:

- Manual deployments
- Manual scaling
- Manual recovery
- Manual networking

Every failure requires human intervention.

---

# After Swarm

The same infrastructure becomes:

```
               Docker Swarm Cluster

        ┌──────────────┬──────────────┬──────────────┐
        │              │              │
     Node 1         Node 2         Node 3
        │              │              │
   Backend-1      Backend-2      Backend-3
        │              │              │
        └──────────────┴──────────────┘
                  Shared Network
```

Now Docker manages the cluster.

---

# What is a Swarm?

A Swarm is a group of Docker engines working together.

```
Docker Engine

+

Docker Engine

+

Docker Engine

=

Docker Swarm
```

To the user, the cluster behaves like one logical system.

---

# Swarm Architecture

```
                  Swarm Cluster

        ┌──────────────────────────────┐
        │        Manager Node          │
        └─────────────┬────────────────┘
                      │
      ┌───────────────┼───────────────┐
      ▼               ▼               ▼
 Worker Node     Worker Node     Worker Node
```

There are two types of nodes:

- Manager
- Worker

---

# Manager Node

The Manager controls the cluster.

Responsibilities:

- Cluster management
- Scheduling
- Desired state reconciliation
- Service deployment
- Leader election
- Node management

Think of the Manager as the **brain** of the cluster.

---

# Worker Node

Workers execute tasks assigned by the Manager.

Responsibilities:

- Run containers
- Report health
- Execute workloads

Workers do **not** make scheduling decisions.

---

# Manager vs Worker

| Manager | Worker |
|----------|---------|
| Controls cluster | Runs workloads |
| Schedules services | Executes tasks |
| Stores cluster state | Reports status |
| Leader election | No leadership |

---

# Initializing a Swarm

Create a new Swarm:

```bash
docker swarm init
```

Docker outputs a join token.

Example:

```bash
docker swarm join \
--token <TOKEN> \
192.168.1.10:2377
```

Worker nodes use this command to join the cluster.

---

# Desired State

One of Swarm's most powerful ideas.

You declare:

```
Backend

Replicas = 3
```

Docker continuously ensures that three replicas exist.

If one crashes:

```
Replica Lost

↓

Manager Detects Failure

↓

New Replica Created
```

This is called **desired state reconciliation**.

---

# Services

In Swarm, you don't manage individual containers.

You manage **Services**.

Example:

```bash
docker service create \
--name backend \
nginx
```

A Service describes:

- Image
- Replicas
- Network
- Ports
- Restart policy

Containers become implementation details.

---

# Tasks

A Service is broken into Tasks.

```
Service

↓

Task 1

Task 2

Task 3
```

Each Task usually corresponds to one running container.

---

# Scaling

Increase replicas:

```bash
docker service scale backend=5
```

Result:

```
Backend

↓

5 Containers
```

Scaling is automatic.

---

# Self-Healing

Suppose one worker crashes.

```
Worker Down

↓

Container Lost

↓

Manager Detects Failure

↓

Starts New Task

↓

Cluster Healthy
```

No manual intervention is required.

---

# Overlay Network

Containers running on different servers still communicate.

```
Node A

Backend

──────────── Overlay Network ────────────

Node B

Redis

──────────── Overlay Network ────────────

Node C

Database
```

The network spans multiple machines.

---

# Service Discovery

Every service gets an internal DNS name.

Example:

```
Backend

↓

postgres
```

Instead of using IP addresses:

```
DB_HOST=postgres
```

Swarm automatically resolves the service.

---

# Routing Mesh

Suppose:

```
Backend

Replicas = 5
```

User request:

```
Internet

↓

Node 1

↓

Routing Mesh

↓

Backend Replica
```

Even if the container is running on another node, the request is forwarded automatically.

This gives a single entry point for the service.

---

# Rolling Updates

Without Swarm:

```
Stop Old Version

↓

Deploy New Version

↓

Downtime
```

With Swarm:

```
Replica 1 Updated

↓

Healthy

↓

Replica 2 Updated

↓

Healthy

↓

Replica 3 Updated
```

Users continue using the application during deployment.

---

# Secrets

Sensitive data should not be stored in images.

Example:

```
Database Password

↓

Swarm Secret

↓

Injected at Runtime
```

Secrets are encrypted and distributed securely.

---

# Configs

Configuration files can also be managed centrally.

Examples:

- nginx.conf
- application.properties
- config.yml

Instead of rebuilding images, update configs separately.

---

# High Availability

Production clusters should have multiple Managers.

```
Manager 1

Manager 2

Manager 3
```

If one Manager fails:

Another Manager becomes leader.

The cluster continues operating.

---

# Production Best Practices

✔ Use at least 3 Manager nodes.

✔ Separate Managers and Workers.

✔ Use Overlay Networks.

✔ Store secrets securely.

✔ Use Health Checks.

✔ Configure Restart Policies.

✔ Monitor the cluster.

✔ Use versioned images.

---

# Common Mistakes

❌ Running only one Manager.

❌ Hardcoding secrets.

❌ Using `latest` image tags.

❌ Exposing every service publicly.

❌ Ignoring health checks.

---

# Hands-on Lab

Initialize a Swarm:

```bash
docker swarm init
```

List nodes:

```bash
docker node ls
```

Create a service:

```bash
docker service create \
--name web \
--replicas 3 \
nginx
```

List services:

```bash
docker service ls
```

Scale:

```bash
docker service scale web=5
```

Inspect tasks:

```bash
docker service ps web
```

---

# Interview Questions

## Why was Docker Swarm created?

To manage and orchestrate containers across multiple Docker hosts.

---

## Difference between a Container and a Service?

A Container is a running instance.

A Service defines the desired state and manages one or more containers.

---

## What is Desired State?

The configuration you declare (for example, 5 replicas). Swarm continuously works to match the actual state to that desired state.

---

## What is a Manager Node?

The node responsible for scheduling, maintaining cluster state, and coordinating the Swarm.

---

## What is Routing Mesh?

A networking feature that allows any node in the cluster to receive traffic for a service and forward it to an available replica.

---

# Summary

Docker Swarm introduces the concepts of **clustering, scheduling, desired state, services, scaling, self-healing, and rolling updates**.

These concepts are fundamental to modern container orchestration and will reappear in Kubernetes.

Swarm provides a simpler orchestration platform than Kubernetes while teaching many of the same core ideas.

---

# Next Chapter

## Chapter 27 — Docker Engine Internals

We'll answer one of the most important questions:

> **What actually happens inside Docker when you run:**

```bash
docker run nginx
```

We'll trace the request through:

- Docker CLI
- Docker API
- Docker Daemon (`dockerd`)
- `containerd`
- `containerd-shim`
- `runc`
- Linux Kernel
- Namespaces
- cgroups

By the end of that chapter, you'll understand the complete container creation process from command to running process.