Imagine your application becomes successful.

Instead of:

```
1 Web Server
```

You now have:

```
Web Server 1

Web Server 2

Web Server 3

Web Server 4
```

Great!

More servers mean you can handle more users.

But a new question appears.

A user opens:

```
https://amazon.com
```

Question:

Which server should handle the request?

```
Server 1?
```

```
Server 2?
```

```
Server 3?
```

```
Server 4?
```

If everyone connects to Server 1...

```
Server 1

↓

100% CPU
```

while

```
Server 2

↓

Idle
```

your system wastes resources.

The solution is:

# Load Balancer

A Load Balancer intelligently distributes incoming requests across multiple servers.

Every large-scale platform uses load balancers.

Examples:

- Google
- Amazon
- Netflix
- YouTube
- Uber
- Kubernetes
- ChatGPT

After this chapter, you'll understand how billions of requests are distributed every day.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 76 — Load Balancers: Distributing Millions of Requests Across Thousands of Servers

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Load Balancers Exist
> - Layer 4 vs Layer 7
> - Load Balancing Algorithms
> - Round Robin
> - Least Connections
> - Weighted Algorithms
> - Health Checks
> - Sticky Sessions
> - Active-Active vs Active-Passive
> - Popular Load Balancers
> - Complete Request Flow

---

# 📖 Why Do We Need Load Balancers?

Suppose one server can handle:

```
10,000 Requests

Per Second
```

Your website now receives:

```
100,000 Requests

Per Second
```

One server isn't enough.

Solution:

```
10 Servers
```

But now...

Who decides where requests go?

---

# Without a Load Balancer

```
Client

↓

Server 1
```

Server 1 becomes overloaded.

Other servers sit idle.

---

# With a Load Balancer

```
Clients

↓

Load Balancer

↓

Server 1

Server 2

Server 3

Server 4
```

Traffic is distributed.

---

# Think of a Receptionist

Imagine a hospital.

Patients arrive.

Reception decides:

```
Doctor A

Doctor B

Doctor C
```

Patients don't choose.

The receptionist balances the workload.

Load Balancer works exactly like this.

---

# High-Level Architecture

```
Internet

↓

Load Balancer

↓

Backend Servers
```

Clients only know the load balancer.

---

# Layer 4 Load Balancer

Layer 4 works at:

```
TCP

UDP
```

It forwards traffic using:

- IP Address
- Port

It does **not** inspect HTTP requests.

---

# Layer 4 Flow

```
Client

↓

TCP Connection

↓

Server
```

Fast.

Very efficient.

---

# Layer 7 Load Balancer

Layer 7 understands:

- HTTP
- HTTPS
- URLs
- Headers
- Cookies
- Methods

It can make routing decisions based on application data.

---

# Example

Request:

```
/images
```

↓

Image Server

Request:

```
/payments
```

↓

Payment Service

Same domain.

Different destinations.

---

# L4 vs L7

| Layer 4 | Layer 7 |
|----------|----------|
| TCP/UDP | HTTP/HTTPS |
| IP + Port | URL, Headers, Cookies |
| Faster | Smarter |
| Less Flexible | More Flexible |

---

# Round Robin

Simplest algorithm.

Requests:

```
1

↓

Server A
```

```
2

↓

Server B
```

```
3

↓

Server C
```

Then repeat.

---

# Visual Representation

```
A

↓

B

↓

C

↓

A

↓

B

↓

C
```

Every server receives approximately equal traffic.

---

# Least Connections

Suppose:

```
Server A

↓

100 Active Users
```

```
Server B

↓

20 Active Users
```

Next request goes to:

```
Server B
```

Less busy.

---

# Weighted Round Robin

Not all servers are equal.

Example:

```
Server A

16 CPU
```

```
Server B

8 CPU
```

Weights:

```
A = 2

B = 1
```

Server A receives approximately twice as many requests.

---

# Least Response Time

Load balancer measures:

```
Latency
```

Next request goes to:

```
Fastest Server
```

Useful when server load varies.

---

# Health Checks

Suppose:

```
Server 2

↓

Crash
```

Load balancer periodically checks:

```
Healthy?
```

If unhealthy,

traffic stops immediately.

---

# Health Check Flow

```
Load Balancer

↓

Ping

↓

Healthy?

↓

Yes

↓

Receive Traffic
```

Otherwise:

```
Remove From Pool
```

---

# Sticky Sessions

Suppose user logs in.

Session stored on:

```
Server A
```

Future requests should return to:

```
Server A
```

This behavior is called:

```
Sticky Session
```

---

# Problem with Sticky Sessions

Suppose:

```
Server A

↓

Crash
```

User session disappears unless:

- Sessions are replicated
- Sessions are stored in Redis or another shared store

Modern systems often avoid relying on sticky sessions.

---

# Active-Active

```
Server A

↓

Serving
```

```
Server B

↓

Serving
```

```
Server C

↓

Serving
```

Everyone handles traffic.

---

# Active-Passive

```
Primary

↓

Serving
```

Backup:

```
Idle

↓

Waiting
```

If primary fails,

backup takes over.

---

# SSL Termination

Many load balancers terminate:

```
HTTPS

↓

HTTP
```

Backend servers avoid handling TLS directly.

---

# Global Load Balancing

Imagine:

```
India

USA

Europe
```

Users connect to:

```
Nearest Region
```

Latency decreases significantly.

---

# Cloud Example

AWS:

```
Client

↓

Elastic Load Balancer

↓

EC2 Instances
```

Same concept.

Different implementation.

---

# Kubernetes Example

Typical flow:

```
Internet

↓

Cloud Load Balancer

↓

Ingress

↓

Service

↓

Pod
```

Several load balancing layers may exist.

---

# Popular Load Balancers

Examples include:

- NGINX
- HAProxy
- Envoy
- AWS ELB
- Azure Load Balancer
- Google Cloud Load Balancer
- F5 BIG-IP

---

# Complete Request Flow

```
Client

↓

Load Balancer

↓

Health Check

↓

Choose Server

↓

Forward Request

↓

Response
```

---

# Why Load Balancers Matter

Without one:

```
Traffic

↓

One Server
```

With one:

```
Traffic

↓

Many Servers
```

Higher availability.

Higher scalability.

---

# Hands-on Lab

## Lab 1 — Run Multiple NGINX Containers

```bash
docker run -d --name web1 nginx
docker run -d --name web2 nginx
docker run -d --name web3 nginx
```

Imagine placing a load balancer in front of them.

---

## Lab 2 — Stop One Server

```bash
docker stop web2
```

Observe that a healthy load balancer would stop routing requests to that instance after failed health checks.

---

## Lab 3 — Kubernetes Services

```bash
kubectl get svc
```

Notice how Kubernetes Services distribute traffic across Pods.

---

# Interview Questions

## What is a Load Balancer?

A Load Balancer distributes incoming traffic across multiple backend servers.

---

## Why do applications need Load Balancers?

To improve scalability, availability, fault tolerance, and resource utilization.

---

## What is the difference between Layer 4 and Layer 7 Load Balancers?

Layer 4 makes decisions using transport-layer information such as IP addresses and ports.

Layer 7 understands application-layer protocols such as HTTP and can route based on URLs, headers, or cookies.

---

## What is Round Robin?

A scheduling algorithm that distributes requests sequentially across available servers.

---

## What is Least Connections?

An algorithm that sends new requests to the server with the fewest active connections.

---

## What are health checks?

Periodic checks that determine whether a backend server is healthy enough to receive traffic.

---

## What is a Sticky Session?

A technique that keeps a user's requests directed to the same backend server.

---

# Summary

Load Balancers intelligently distribute traffic across backend servers.

```
Clients

↓

Load Balancer

↓

Health Check

↓

Algorithm

↓

Backend Server

↓

Response
```

Key concepts:

- Load Balancers distribute incoming requests.
- Layer 4 balances TCP/UDP traffic.
- Layer 7 understands HTTP and application data.
- Common algorithms include Round Robin, Least Connections, and Weighted Round Robin.
- Health checks prevent traffic from reaching failed servers.
- Sticky sessions preserve client affinity when necessary.
- Modern cloud platforms and Kubernetes rely heavily on load balancing.

At this point, services can:

- Discover each other.
- Secure requests using API Gateways.
- Scale horizontally using Load Balancers.

The next question is:

> **What's the difference between a Load Balancer and a Reverse Proxy?**

Many developers confuse them, but they solve different problems.

---

# Next Chapter

## Chapter 77 — Reverse Proxy: The Smart Middleman Between Clients and Servers

We'll explore:

- What is a Reverse Proxy?
- Forward Proxy vs Reverse Proxy
- Reverse Proxy Architecture
- SSL Termination
- Request Routing
- Caching
- Compression
- Static File Serving
- NGINX
- Envoy
- Traefik
- Reverse Proxy vs Load Balancer

> **This chapter explains why NGINX, Envoy, Apache, and Traefik are called reverse proxies and how they differ from traditional load balancers.**