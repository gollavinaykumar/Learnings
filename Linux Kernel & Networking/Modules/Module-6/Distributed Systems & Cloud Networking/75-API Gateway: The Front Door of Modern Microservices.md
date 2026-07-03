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

```
Notification Service
```

A mobile app needs to access all of them.

Without thinking much, you expose every service directly.

```
Internet

↓

User Service

↓

Order Service

↓

Payment Service

↓

Inventory Service

↓

Notification Service
```

It works...

But now new problems appear.

Questions:

- Where should authentication happen?
- Where should JWT tokens be verified?
- Where should rate limiting happen?
- Where should logging happen?
- Where should SSL certificates be managed?
- What if service URLs change?

Duplicating this logic in every service becomes a nightmare.

The solution is:

# API Gateway

An API Gateway acts as the **single front door** for all client requests.

Instead of talking directly to every microservice,

clients communicate with **one gateway**.

After this chapter, you'll understand why nearly every modern cloud architecture starts with an API Gateway.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 75 — API Gateway: The Front Door of Modern Microservices

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why API Gateways Exist
> - Problems Without a Gateway
> - Request Routing
> - Authentication
> - Authorization
> - Rate Limiting
> - SSL/TLS Termination
> - Logging
> - API Aggregation
> - Popular API Gateways
> - Complete Request Flow

---

# 📖 The Problem

Imagine a mobile application.

It needs:

```
Login

↓

User Service
```

```
Orders

↓

Order Service
```

```
Payments

↓

Payment Service
```

Every request goes to a different service.

---

# Without an API Gateway

```
                Mobile App

      ↙       ↓        ↘

User     Order     Payment

↓

Inventory

↓

Notification
```

Every service must implement:

- Authentication
- Logging
- Rate Limiting
- SSL
- Error Handling

Repeated everywhere.

---

# Problems

Every service now needs:

✔ JWT Validation

✔ Logging

✔ Security

✔ Monitoring

✔ Rate Limiting

✔ CORS

This creates duplicated code and inconsistent behavior.

---

# What is an API Gateway?

An API Gateway is a server that sits between clients and backend services.

```
Client

↓

API Gateway

↓

Microservices
```

Clients only know the gateway.

---

# Think of a Hotel Reception

Guests don't walk directly into:

- Kitchen
- Laundry
- Manager's Office

Instead:

```
Guest

↓

Reception

↓

Correct Department
```

The API Gateway works the same way.

---

# High-Level Architecture

```
Internet

↓

API Gateway

↓

User Service

Order Service

Payment Service

Inventory Service
```

Everything enters through one place.

---

# Request Routing

Suppose request:

```
/orders
```

Gateway decides:

```
Order Service
```

Another request:

```
/payments
```

Gateway forwards it to:

```
Payment Service
```

---

# Example

```
GET /users

↓

User Service
```

```
GET /orders

↓

Order Service
```

Routing happens automatically.

---

# Authentication

Instead of every service checking JWT tokens,

the gateway performs authentication.

```
Client

↓

JWT

↓

Gateway

↓

Verified?
```

If authentication fails,

the request never reaches backend services.

---

# Authorization

Authentication answers:

```
Who Are You?
```

Authorization answers:

```
What Are You Allowed To Do?
```

Gateway can reject requests before they reach services.

---

# Example

User Role:

```
Customer
```

Request:

```
DELETE /users
```

Gateway responds:

```
403 Forbidden
```

No backend processing required.

---

# Rate Limiting

Suppose one client sends:

```
100,000 Requests

Per Minute
```

Gateway enforces limits.

Example:

```
100 Requests

Per Minute
```

Extra requests are rejected.

---

# Visual Representation

```
Client

↓

101st Request

↓

Gateway

↓

429 Too Many Requests
```

Backend stays protected.

---

# SSL/TLS Termination

HTTPS requires encryption.

Instead of every service handling certificates,

gateway manages them.

```
HTTPS

↓

Gateway

↓

HTTP

↓

Internal Services
```

Services communicate within a trusted network.

---

# Logging

Every request passes through:

```
Gateway
```

Perfect place to record:

- URL
- Client IP
- Response Time
- Status Code

Centralized logging becomes easy.

---

# Monitoring

Gateway measures:

✔ Latency

✔ Request Count

✔ Error Rate

✔ Throughput

One dashboard can monitor the entire platform.

---

# API Aggregation

Suppose mobile app needs:

- User
- Orders
- Recommendations

Without aggregation:

```
3 Requests
```

Gateway can combine them.

```
Client

↓

Gateway

↓

3 Services

↓

One Response
```

Fewer network round trips.

---

# Request Transformation

Gateway can modify:

- Headers
- Query Parameters
- Request Body
- Response Body

Clients and services remain independent.

---

# Load Balancing

Gateway receives:

```
Order Service

↓

3 Instances
```

Gateway distributes requests across:

```
Order 1

Order 2

Order 3
```

---

# Caching

Frequently requested data can be cached.

```
Client

↓

Gateway Cache

↓

Response
```

Backend isn't contacted every time.

---

# Circuit Breaker

Suppose:

```
Payment Service

↓

Down
```

Gateway immediately returns:

```
Service Unavailable
```

Instead of waiting for long timeouts.

This improves overall system resilience.

---

# API Versioning

Gateway can route:

```
/v1/orders
```

↓

Old Service

```
/v2/orders
```

↓

New Service

Clients migrate gradually.

---

# Popular API Gateways

Common choices include:

- Kong
- NGINX
- Envoy
- AWS API Gateway
- Azure API Management
- Google API Gateway
- Spring Cloud Gateway
- Traefik

---

# Kubernetes Example

Typical request flow:

```
Internet

↓

Ingress Controller

↓

API Gateway

↓

Service

↓

Pod
```

Some deployments combine the Ingress Controller and Gateway,

while others keep them separate.

---

# API Gateway vs Load Balancer

Load Balancer:

```
Distributes Traffic
```

API Gateway:

```
Routing

+

Authentication

+

Rate Limiting

+

Logging

+

Caching

+

Transformation
```

An API Gateway often includes load balancing,

but does much more.

---

# Complete Request Flow

```
Client

↓

HTTPS

↓

API Gateway

↓

Authentication

↓

Authorization

↓

Rate Limiting

↓

Routing

↓

Service

↓

Response
```

---

# Why API Gateways Matter

Without one:

```
Clients

↓

Every Service
```

Complex.

With one:

```
Clients

↓

Gateway

↓

Everything Else
```

Simple.

---

# Hands-on Lab

## Lab 1 — Observe Kubernetes Services

```bash
kubectl get svc
```

Notice how clients normally communicate through stable service endpoints.

---

## Lab 2 — Install Kong (Optional)

```bash
helm install kong kong/kong
```

Explore gateway resources inside Kubernetes.

---

## Lab 3 — View Ingress

```bash
kubectl get ingress
```

Observe how incoming requests are routed into the cluster.

---

# Interview Questions

## What is an API Gateway?

An API Gateway is the single entry point through which clients access backend services.

---

## Why do microservices use an API Gateway?

To centralize authentication, authorization, routing, rate limiting, logging, monitoring, and other cross-cutting concerns.

---

## What is the difference between authentication and authorization?

Authentication verifies identity.

Authorization determines what actions that identity is allowed to perform.

---

## What is rate limiting?

Restricting how many requests a client can make during a given period.

---

## Why terminate SSL at the gateway?

It centralizes certificate management and reduces complexity for backend services.

---

## What is API aggregation?

Combining responses from multiple backend services into a single client response.

---

## Name some popular API Gateways.

- Kong
- NGINX
- Envoy
- AWS API Gateway
- Spring Cloud Gateway
- Traefik

---

# Summary

The API Gateway becomes the front door of a microservice architecture.

```
Client

↓

API Gateway

↓

Authentication

↓

Authorization

↓

Rate Limiting

↓

Routing

↓

Microservices

↓

Response
```

Key concepts:

- Clients communicate with one gateway instead of many services.
- Authentication and authorization are centralized.
- Rate limiting protects backend services.
- SSL termination simplifies certificate management.
- Logging and monitoring become centralized.
- API aggregation reduces client requests.
- Modern cloud-native systems almost always include an API Gateway.

Now clients can securely reach your services.

The next challenge is deciding **which backend server should actually receive the request**.

That's the job of a **Load Balancer**.

---

# Next Chapter

## Chapter 76 — Load Balancers: Distributing Millions of Requests Across Thousands of Servers

We'll explore:

- Why Load Balancers Exist
- L4 vs L7 Load Balancers
- Round Robin
- Least Connections
- Weighted Load Balancing
- Health Checks
- Sticky Sessions
- Active-Active vs Active-Passive
- HAProxy
- NGINX
- AWS Elastic Load Balancer (ELB)
- Complete Request Distribution Flow

> **This chapter explains how Google, Netflix, Amazon, and Kubernetes distribute billions of requests efficiently across thousands of servers.**