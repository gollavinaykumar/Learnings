# 🐳 Docker Mastery Roadmap

# Module 3 — Advanced Docker & Container Internals

# Chapter 30 — Docker Engine API

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What the Docker Engine API is
> - Docker Client vs Docker Server
> - Docker REST API
> - Docker Socket
> - Docker SDKs
> - Remote Docker API
> - Authentication
> - Security Risks
> - API Versioning
> - Automation
> - Building your own Docker client

---

# 📖 Introduction

Suppose you execute:

```bash
docker run nginx
```

Many developers imagine:

```
Docker CLI

↓

Container Starts
```

This is **incorrect**.

The real flow is:

```
Docker CLI

↓

Docker REST API

↓

Docker Daemon

↓

containerd

↓

runc

↓

Linux Kernel
```

The CLI never creates containers.

It only sends HTTP requests.

---

# Client-Server Architecture

Docker follows a Client-Server model.

```
+----------------+

Docker CLI

(Client)

+----------------+
        │
HTTP REST API
        │
+----------------+

Docker Engine

(Server)

+----------------+
```

The CLI is only one possible client.

Many other tools communicate with Docker using the same API.

---

# Docker CLI

Examples:

```bash
docker run nginx

docker ps

docker images

docker logs
```

The CLI:

✔ Parses commands

✔ Validates arguments

✔ Sends API requests

It performs very little work itself.

---

# Docker Engine (dockerd)

The Docker Engine is the server.

Responsibilities:

- Manage containers
- Manage images
- Manage networks
- Manage volumes
- Expose the REST API

Every Docker client communicates with this service.

---

# Docker REST API

Internally:

```
docker ps
```

becomes something like:

```
GET /containers/json
```

Similarly:

```
docker images
```

becomes:

```
GET /images/json
```

And:

```
docker run nginx
```

is translated into multiple API calls:

```
POST /containers/create

↓

POST /containers/{id}/start
```

The CLI is essentially an HTTP client.

---

# Docker Socket

How does the CLI communicate with the Engine?

Usually through:

```
/var/run/docker.sock
```

This is a Unix domain socket.

```
Docker CLI

↓

docker.sock

↓

dockerd
```

Unlike a TCP socket, communication happens locally through the operating system.

---

# Why Use a Socket?

Benefits:

- Faster than TCP
- No network overhead
- Local-only by default
- Better security

On Linux:

```bash
ls -l /var/run/docker.sock
```

Example:

```
srw-rw----

docker.sock
```

---

# Docker SDKs

Because Docker exposes a REST API, many programming languages provide SDKs.

Examples:

- Go
- Python
- Java
- JavaScript
- C#
- Rust

Python example:

```python
import docker

client = docker.from_env()

client.containers.list()
```

The SDK converts method calls into Docker API requests.

---

# Remote Docker API

Docker can expose its API over TCP.

```
Developer Laptop

↓

HTTPS

↓

Docker Server
```

Useful for:

- CI/CD
- Remote management
- Automation
- Cloud platforms

Example endpoint:

```
tcp://192.168.1.10:2375
```

Production systems should use TLS.

---

# API Versioning

Docker evolves over time.

Instead of breaking clients:

```
API v1.41

API v1.42

API v1.43
```

Clients negotiate a compatible version.

This allows older clients to work with newer Docker Engines.

---

# Example API Flow

Suppose:

```bash
docker ps
```

Internally:

```
Docker CLI

↓

GET /containers/json

↓

dockerd

↓

containerd

↓

Container List

↓

JSON Response

↓

CLI Displays Output
```

---

# Docker Desktop

Docker Desktop is also a client.

```
Docker Desktop UI

↓

Docker REST API

↓

dockerd
```

The graphical interface performs the same API requests as the CLI.

---

# VS Code Integration

The Docker extension in VS Code works similarly.

```
VS Code

↓

Docker API

↓

dockerd
```

This is why you can:

- Start containers
- Stop containers
- View logs
- Inspect images

without opening a terminal.

---

# Portainer

Portainer is a web interface for Docker.

```
Browser

↓

Portainer

↓

Docker API

↓

Docker Engine
```

Again, everything uses the same API.

---

# Security Risks

One of the most important topics.

Suppose a user can access:

```
/var/run/docker.sock
```

That user can:

- Create containers
- Stop containers
- Delete images
- Mount host directories
- Access secrets

Effectively:

```
Docker Socket

≈

Root Access
```

Never expose:

```
docker.sock
```

to untrusted applications.

---

# Bad Practice

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

inside random containers.

If compromised:

The attacker controls Docker.

---

# Secure Alternatives

Instead of exposing the socket:

✔ Docker API with TLS

✔ Reverse proxy

✔ Authentication

✔ Authorization

✔ Least privilege

---

# Automation

The API enables automation.

Example pipeline:

```
Git Push

↓

CI Server

↓

Docker API

↓

Build Image

↓

Run Tests

↓

Push Registry

↓

Deploy
```

Humans don't need to type Docker commands.

Everything can be automated.

---

# Build Your Own Docker Client

Because Docker uses HTTP,

you can build your own client.

Example:

```
Mobile App

↓

HTTP

↓

Docker Engine
```

or

```
Internal Dashboard

↓

Docker API

↓

Containers
```

Many enterprise platforms work exactly this way.

---

# Common Misconceptions

❌ Docker CLI creates containers.

✔ Docker CLI sends HTTP requests.

---

❌ Docker Desktop has its own runtime.

✔ Docker Desktop communicates with Docker Engine.

---

❌ Docker API is only for Docker CLI.

✔ Any application can use the API.

---

# Hands-on Lab

Inspect the Docker socket:

```bash
ls -l /var/run/docker.sock
```

Display Docker information:

```bash
docker info
```

Inspect API version:

```bash
docker version
```

Observe:

```
Client

↓

Server
```

Notice that both have separate version information.

---

# Interview Questions

## What is Docker Engine API?

A REST API exposed by Docker Engine that allows clients to manage Docker resources.

---

## Does Docker CLI create containers?

No.

The CLI sends API requests to Docker Engine, which manages container creation.

---

## What is docker.sock?

A Unix domain socket used for local communication between Docker clients and the Docker daemon.

---

## Why is docker.sock considered sensitive?

Because anyone with access to it can control Docker, effectively gaining root-level capabilities on the host.

---

## Can you build your own Docker client?

Yes.

Any language capable of making HTTP requests can interact with the Docker Engine API.

---

# Summary

Docker follows a client-server architecture.

```
Docker CLI

↓

Docker REST API

↓

docker.sock

↓

dockerd

↓

containerd

↓

runc

↓

Linux Kernel
```

Understanding the Docker Engine API explains how Docker Desktop, VS Code, Portainer, CI/CD systems, and SDKs all manage containers using the same underlying interface.

---

# Next Chapter

## Chapter 31 — Image Internals

We'll explore:

- Image manifests
- Layer digests
- SHA256
- Manifest lists
- Multi-architecture images
- Registry storage
- Content addressing
- How Docker stores images internally