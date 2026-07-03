Imagine you're building an e-commerce application.

You have:

```
User Service
```

```
Order Service
```

```
Payment Service
```

```
Inventory Service
```

Everything works.

Now the application grows.

Instead of:

```
1 Order Server
```

You now have:

```
Order Server 1

Order Server 2

Order Server 3

Order Server 4
```

Question:

How does the User Service know:

- Which Order Service IP to call?
- What if Order Server 2 crashes?
- What if Kubernetes creates a new Pod with a different IP?
- Should developers manually update IP addresses every time?

Obviously not.

The solution is:

# Service Discovery

Service Discovery allows services to find each other **automatically**, even as servers start, stop, move, or scale.

Without Service Discovery:

- Kubernetes would not work.
- Microservices would constantly break.
- Cloud-native systems would be impossible to manage.

After this chapter, you'll understand how services discover each other in modern distributed systems.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 74 — Service Discovery: How Thousands of Microservices Find Each Other Automatically

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Service Discovery Exists
> - Static vs Dynamic Addresses
> - Service Registry
> - Registration
> - Health Checks
> - Client-side Discovery
> - Server-side Discovery
> - DNS-based Discovery
> - Kubernetes Services
> - Complete Discovery Flow

---

# 📖 The Problem

Imagine:

```
User Service

↓

10.1.1.20
```

Calls:

```
Order Service

↓

10.1.1.50
```

Everything works.

Now Kubernetes replaces the Pod.

New IP:

```
10.1.8.145
```

User Service still calls:

```
10.1.1.50
```

Connection fails.

---

# Static IPs Don't Work

Hardcoding:

```
10.1.1.50
```

works only until:

- Pod restarts
- Container restarts
- Node fails
- Autoscaling occurs

Modern systems need dynamic discovery.

---

# What is Service Discovery?

Service Discovery means:

```
Service Name

↓

Find Current Location

↓

Connect
```

Applications use names,

not IP addresses.

---

# Think of Your Phone Contacts

Instead of remembering:

```
+91-9876543210
```

you save:

```
Mom
```

When you call:

```
Mom
```

your phone resolves the number.

Service Discovery works the same way.

---

# Service Registry

At the center is:

```
Service Registry
```

It stores:

```
Service Name

↓

Current Instances
```

Example:

```
Order Service

↓

10.1.8.145

10.1.8.146

10.1.8.147
```

---

# Registration

When a service starts:

```
Order Service

↓

Register

↓

Registry
```

The registry now knows where it is.

---

# Deregistration

When a service stops:

```
Order Service

↓

Remove

↓

Registry
```

Clients no longer receive that instance.

---

# Health Checks

Suppose:

```
Order Service

↓

Crash
```

The registry periodically checks:

```
Healthy?
```

If not,

the instance is removed from discovery results.

---

# Complete Registry Flow

```
Service Starts

↓

Register

↓

Registry

↓

Healthy

↓

Clients Discover
```

---

# Client-side Discovery

In this model:

```
Client

↓

Registry

↓

Choose Instance

↓

Connect
```

The client decides which instance to use.

---

# Example

Registry returns:

```
Order 1

Order 2

Order 3
```

Client selects:

```
Order 2
```

Often using:

- Round Robin
- Random
- Least Connections
- Latency-based selection

---

# Server-side Discovery

Instead of the client selecting,

a load balancer handles it.

Flow:

```
Client

↓

Load Balancer

↓

Order Service
```

Client doesn't know how many servers exist.

---

# Compare

Client-side:

```
Client

↓

Registry

↓

Server
```

Server-side:

```
Client

↓

Load Balancer

↓

Server
```

Both approaches are widely used.

---

# DNS-Based Discovery

Many systems simply use:

```
DNS
```

Application requests:

```
order-service.company.local
```

DNS returns:

```
10.1.8.145
```

or multiple IP addresses.

---

# Kubernetes Example

Application calls:

```
http://order-service
```

Not:

```
http://10.244.2.17
```

Kubernetes DNS resolves:

```
order-service

↓

ClusterIP
```

Applications never need Pod IPs.

---

# Kubernetes Flow

```
Application

↓

DNS

↓

Service

↓

Pod
```

Simple.

Automatic.

Reliable.

---

# Service Registry Example

```
Inventory

↓

10.1.5.10

10.1.5.11

10.1.5.12
```

Clients receive healthy instances only.

---

# Scaling Example

Autoscaler creates:

```
Inventory 13
```

Immediately:

```
Register

↓

Registry
```

Clients begin using it automatically.

No configuration changes.

---

# Failure Example

Suppose:

```
Payment Service

↓

Crash
```

Health check fails.

Registry removes it.

Clients never receive that instance again.

---

# Real Systems

Popular service discovery systems include:

- Consul
- etcd
- ZooKeeper
- Eureka
- Kubernetes DNS

Each solves the same problem differently.

---

# Consul

Consul provides:

✔ Service Registry

✔ Health Checks

✔ DNS Interface

✔ Configuration Storage

Widely used outside Kubernetes.

---

# Eureka

Netflix developed Eureka for microservice discovery.

Spring Cloud integrates closely with it.

---

# etcd

etcd stores distributed configuration and cluster state.

Applications can also use it for service registration and discovery patterns.

---

# Kubernetes DNS

Kubernetes automatically creates DNS names.

Example:

```
payment-service.default.svc.cluster.local
```

Applications usually use the short name:

```
payment-service
```

---

# Packet Flow

```
Application

↓

DNS Query

↓

Registry

↓

Current Instance

↓

TCP Connection

↓

Request
```

---

# Why Service Discovery Matters

Without it:

```
Server Changes

↓

Broken Applications
```

With Service Discovery:

```
Server Changes

↓

Automatic Updates

↓

Applications Continue Working
```

---

# Complete Architecture

```
Services

↓

Register

↓

Registry

↓

DNS / API

↓

Clients

↓

Connect
```

---

# Hands-on Lab

## Lab 1 — Kubernetes Services

```bash
kubectl get svc
```

Observe service names and ClusterIP addresses.

---

## Lab 2 — DNS Resolution

Enter a Pod:

```bash
kubectl exec -it <pod> -- sh
```

Run:

```bash
nslookup kubernetes.default
```

Observe Kubernetes DNS.

---

## Lab 3 — Scale Deployment

```bash
kubectl scale deployment order-service --replicas=5
```

Notice that applications continue using the service name without any code changes.

---

# Interview Questions

## What is Service Discovery?

Service Discovery enables applications to automatically locate other services without hardcoding IP addresses.

---

## Why is Service Discovery necessary?

Because service instances frequently start, stop, restart, or change IP addresses.

---

## What is a Service Registry?

A Service Registry stores information about currently available service instances.

---

## What is the difference between client-side and server-side discovery?

Client-side discovery lets the client query the registry and choose an instance.

Server-side discovery delegates instance selection to a load balancer or proxy.

---

## Why are health checks important?

They prevent failed or unhealthy instances from receiving traffic.

---

## How does Kubernetes implement Service Discovery?

Using Services and an internal DNS system that resolves service names to stable virtual IPs (ClusterIPs).

---

## Name some popular Service Discovery systems.

- Consul
- Eureka
- ZooKeeper
- etcd
- Kubernetes DNS

---

# Summary

Service Discovery removes the need for hardcoded server addresses.

```
Service

↓

Register

↓

Registry

↓

DNS

↓

Client

↓

Current Instance
```

Key concepts:

- Services should communicate using names, not IP addresses.
- A Service Registry tracks active instances.
- Health checks keep discovery information accurate.
- Client-side and server-side discovery use different lookup strategies.
- Kubernetes provides built-in DNS-based service discovery.
- Service Discovery is a fundamental building block of microservices and cloud-native systems.

Now services can **find each other automatically**.

The next question becomes:

> **How do we control, secure, and manage all incoming requests to those services?**

That is the job of an **API Gateway**.

---

# Next Chapter

## Chapter 75 — API Gateway: The Front Door of Modern Microservices

We'll explore:

- Why API Gateways Exist
- Problems Without an API Gateway
- Authentication
- Authorization
- Rate Limiting
- Request Routing
- SSL/TLS Termination
- Logging
- Load Balancing
- API Aggregation
- Kong
- NGINX
- Envoy
- AWS API Gateway
- Complete Request Flow

> **This chapter explains why nearly every modern microservice architecture places an API Gateway in front of its services and how it simplifies security, routing, and traffic management.**