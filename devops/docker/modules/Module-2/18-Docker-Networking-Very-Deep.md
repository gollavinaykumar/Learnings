
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 18 — Docker Networking (Very Deep)

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - Why Docker networking exists
> - Linux network namespaces
> - veth pairs
> - Linux bridges
> - Docker bridge, host, none, overlay and macvlan networks
> - Docker DNS
> - Port publishing
> - NAT & iptables (high level)
> - Packet flow
> - Container-to-container and host-to-container communication
> - Production networking best practices

---

# 18.1 Why Networking Exists

Containers rarely run alone.

A typical application:

```text
Frontend
    │
Backend
    │
PostgreSQL
    │
Redis
```

These services must communicate securely and efficiently.

Docker networking provides:

- Service discovery
- Isolation
- Port mapping
- Internal communication

---

# 18.2 Linux Network Namespaces

Each container gets its own network namespace.

```text
Host

├── Container A
│     ├── eth0
│     ├── localhost
│     └── Routing Table
│
└── Container B
      ├── eth0
      ├── localhost
      └── Routing Table
```

Each container thinks it owns its network stack.

---

# 18.3 veth Pair

Docker connects containers using a **virtual ethernet pair**.

```text
Container eth0
      │
      │
veth Pair
      │
      │
Host veth
```

Packets entering one end appear at the other end.

---

# 18.4 Linux Bridge

Docker creates a Linux bridge (usually `docker0`).

```text
Container A
      │
      ├── veth
      │
Container B
      │
      ├── veth
      │
   docker0 Bridge
      │
      ▼
Host Network
```

The bridge acts like a virtual switch.

---

# 18.5 Docker Bridge Network

Default Docker network.

Create:

```bash
docker network create my-network
```

Run containers:

```bash
docker run --network my-network nginx
```

Containers on the same bridge communicate directly.

---

# 18.6 Host Network

```bash
docker run --network host nginx
```

Container shares the host network.

Characteristics:

- No network isolation
- No port mapping required
- Maximum performance

Commonly used on Linux.

---

# 18.7 None Network

```bash
docker run --network none nginx
```

Container has no external networking.

Useful for:

- Offline workloads
- Security-sensitive jobs

---

# 18.8 Overlay Network

Used for multi-host communication.

```text
Host A
  │
Overlay
  │
Host B
```

Primarily used by:

- Docker Swarm
- Kubernetes (through CNI plugins)

---

# 18.9 Macvlan

Assigns a container its own MAC address.

```text
Router
   │
Switch
   │
Container
```

Container appears as a physical device on the LAN.

Useful for legacy applications.

---

# 18.10 Docker DNS

Containers on the same user-defined network resolve each other by name.

Example:

```text
backend
   │
postgres
```

Application:

```text
DB_HOST=postgres
```

No hardcoded IPs required.

---

# 18.11 Port Publishing (-p)

Container:

```text
Port 80
```

Host:

```text
Port 8080
```

Command:

```bash
docker run -p 8080:80 nginx
```

Flow:

```text
Browser

↓

Host:8080

↓

Docker

↓

Container:80
```

---

# 18.12 NAT & iptables

Docker uses:

- NAT
- iptables

to translate traffic.

High-level flow:

```text
Internet

↓

Host

↓

iptables

↓

Docker Bridge

↓

Container
```

---

# 18.13 Packet Flow

```text
Client
  │
  ▼
Host Port 8080
  │
iptables
  │
docker0
  │
veth
  │
Container Port 80
```

---

# 18.14 Container ↔ Container Communication

Containers on the same network:

```text
Frontend

↓

Backend

↓

Redis
```

Example:

```bash
curl http://backend:3000
```

Docker DNS resolves `backend`.

---

# 18.15 Host ↔ Container Communication

Host accesses container through published ports.

```bash
docker run -p 5432:5432 postgres
```

Host:

```text
localhost:5432
```

Container:

```text
postgres:5432
```

---

# 18.16 Production Networking

Recommendations:

- Use user-defined bridge networks.
- Avoid exposing unnecessary ports.
- Use reverse proxies.
- Separate frontend and backend networks.
- Use TLS for external traffic.
- Use overlay networking for clustered deployments.

---

# 18.17 Debugging Networks

Useful commands:

```bash
docker network ls
docker network inspect my-network
docker inspect container
docker exec -it container sh
ip addr
ping
curl
```

---

# 18.18 Hands-on Labs

Create a network:

```bash
docker network create demo-net
```

Run two containers:

```bash
docker run -d --name app1 --network demo-net nginx
docker run -d --name app2 --network demo-net alpine sleep 3600
```

Test:

```bash
docker exec -it app2 ping app1
```

Publish a port:

```bash
docker run -d -p 8080:80 nginx
```

Open:

```text
http://localhost:8080
```

---

# 18.19 Interview Questions

**Why does Docker use Linux bridges?**

To connect containers on the same host like a virtual switch.

**What is a veth pair?**

A pair of virtual Ethernet interfaces connecting a container namespace to the host namespace.

**Difference between Bridge and Host network?**

Bridge provides isolation and virtual networking. Host shares the host's network stack.

**Why use Docker DNS?**

To communicate using container names instead of changing IP addresses.

**Why is `-p` needed?**

Containers are isolated. Port publishing exposes container ports to the host.

---

# Summary

- Docker networking is built on Linux networking primitives.
- Network namespaces isolate container network stacks.
- veth pairs connect containers to the host.
- Linux bridges connect multiple containers.
- Docker DNS enables service discovery.
- Port publishing exposes services externally.
- Overlay networks connect containers across multiple hosts.

---

# Next Chapter

## Chapter 19 — Docker Compose

You'll learn:

- compose.yaml
- Services
- Networks
- Volumes
- Environment variables
- Health checks
- Scaling
- Production Compose deployments
