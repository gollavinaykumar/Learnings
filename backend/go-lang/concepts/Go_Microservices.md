# Go Microservices – Complete Interview Revision Guide

> Microservices are one of the most important backend architecture topics.
>
> Modern large-scale applications like:
>
> - Netflix
> - Uber
> - Amazon
> - Spotify
> - Swiggy
> - Zomato
>
> use microservices instead of a single large application.
>
> Common interview topics:
>
> - Microservices Architecture
> - Service Discovery
> - API Gateway
> - gRPC
> - Message Queues
> - Kafka
> - RabbitMQ

---

# 1. What are Microservices?

Microservices are an architectural style where an application is divided into small independent services.

---

# Monolith Architecture

Everything inside one application.

```text
Application
 ├── Users
 ├── Orders
 ├── Payments
 ├── Notifications
 └── Inventory
```

Single deployment.

Single codebase.

---

# Problems With Monoliths

As application grows:

```text
Large Codebase
Slow Deployments
Hard Scaling
Tight Coupling
```

---

# Microservices Architecture

Split into independent services.

```text
User Service

Order Service

Payment Service

Notification Service
```

Each service:

```text
Own Codebase
Own Deployment
Own Database
```

---

# Example

```text
User Service
      ↓
Order Service
      ↓
Payment Service
      ↓
Notification Service
```

---

# Benefits

---

## Independent Deployment

Deploy one service without affecting others.

---

## Independent Scaling

Scale only the busy service.

Example:

```text
Payment Service
```

needs 10 servers.

Others need 2.

---

## Smaller Codebases

Easier maintenance.

---

## Technology Flexibility

Different services may use:

```text
Go
Java
Node.js
Python
```

---

# Challenges

---

## Network Communication

Services communicate over network.

---

## Monitoring Complexity

Many services.

---

## Distributed Failures

One service may fail.

Others continue.

---

## Data Consistency

Harder than monoliths.

---

# Simple Interview Answer

```text
Microservices divide a large application into small independently deployable services.
```

---

# Example E-Commerce System

```text
API Gateway
      ↓

 ├── User Service
 ├── Product Service
 ├── Order Service
 ├── Payment Service
 └── Notification Service
```

---

# 2. Service Discovery

Most Asked Microservices Topic.

---

# Problem

Suppose:

```text
Order Service
```

needs:

```text
Payment Service
```

---

Hardcoding:

```go
http://10.1.1.5:8080
```

is bad.

---

Why?

Servers change.

Containers restart.

IPs change.

---

# Solution

Service Discovery.

---

# What is Service Discovery?

Mechanism for finding service locations dynamically.

---

# Visual

```text
Order Service
      ↓
Service Registry
      ↓
Payment Service IP
```

---

# Architecture

```text
Services
     ↓
Register
     ↓
Registry
     ↓
Lookup
```

---

# Popular Tools

---

## Consul

Very common.

---

## Eureka

Netflix ecosystem.

---

## Kubernetes DNS

Most common today.

---

# Kubernetes Example

Instead of:

```text
10.1.1.5
```

use:

```text
payment-service
```

---

Kubernetes resolves it automatically.

---

# Interview Answer

Service discovery allows services to locate each other dynamically without hardcoded addresses.

---

# 3. API Gateway

Extremely Important Topic.

---

# Problem

Frontend talking directly to:

```text
User Service
Order Service
Payment Service
Inventory Service
```

becomes messy.

---

# Solution

API Gateway.

---

# Architecture

```text
Client
   ↓
API Gateway
   ↓
Services
```

---

# Visual

```text
Frontend
    ↓

API Gateway

 ├── User Service
 ├── Order Service
 ├── Payment Service
 └── Inventory Service
```

---

# Responsibilities

---

## Routing

```text
/users → User Service
/orders → Order Service
```

---

## Authentication

Verify JWT.

---

## Rate Limiting

Prevent abuse.

---

## Logging

Track requests.

---

## Load Balancing

Distribute traffic.

---

# Benefits

```text
Single Entry Point
Security
Centralized Control
```

---

# Popular API Gateways

---

## Kong

---

## NGINX

---

## Traefik

---

## AWS API Gateway

---

# Interview Answer

API Gateway acts as a single entry point that routes requests to appropriate microservices while handling authentication, logging, and rate limiting.

---

# 4. gRPC

Most Important Service-to-Service Communication Topic.

---

# What is gRPC?

High-performance RPC framework developed by :contentReference[oaicite:0]{index=0}.

---

# RPC

Remote Procedure Call.

---

Instead of:

```text
HTTP + JSON
```

Use:

```text
HTTP/2 + Protocol Buffers
```

---

# Why gRPC?

Benefits:

```text
Faster
Smaller Payloads
Strong Typing
Code Generation
```

---

# REST Example

```http
GET /users/1
```

Returns JSON.

---

# gRPC Example

```proto
rpc GetUser(UserRequest)
returns(UserResponse)
```

Looks like calling a function.

---

# Protocol Buffers

Data format used by gRPC.

---

Example:

```proto
message User {
    int32 id = 1;
    string name = 2;
}
```

---

# Generate Code

```bash
protoc
```

creates Go code automatically.

---

# Why Companies Use gRPC?

```text
Fast
Efficient
Strong Contracts
Ideal For Internal Services
```

---

# REST vs gRPC

| Feature | REST | gRPC |
|----------|------|------|
| Format | JSON | Protobuf |
| Speed | Good | Faster |
| Human Readable | Yes | No |
| Browser Friendly | Yes | Limited |
| Microservices | Good | Excellent |

---

# Interview Answer

Use:

```text
REST
```

for public APIs.

Use:

```text
gRPC
```

for internal service communication.

---

# 5. Message Queues

Very Important Microservices Topic.

---

# What is a Message Queue?

A system that allows services to communicate asynchronously.

---

# Synchronous Communication

```text
Order Service
      ↓
Payment Service
      ↓
Wait
```

---

Problem:

If Payment Service is slow:

```text
Order Service waits
```

---

# Solution

Message Queue.

---

# Architecture

```text
Producer
    ↓
 Queue
    ↓
Consumer
```

---

# Example

```text
Order Created
      ↓
Queue
      ↓
Notification Service
```

---

# Benefits

```text
Decoupling
Reliability
Scalability
```

---

# Visual

```text
Order Service
      ↓
 Message Queue
      ↓
Email Service
```

---

# Producer

Creates messages.

---

# Consumer

Processes messages.

---

# Real Example

```text
Order Created
```

---

Producer sends:

```text
Send Email
```

message.

---

Email Service consumes it later.

---

# Why Queues Matter?

If Email Service is down:

```text
Message waits
```

instead of being lost.

---

# 6. Kafka

Most Asked Queue Interview Topic.

---

# What is Kafka?

Distributed event streaming platform.

---

Entity: :contentReference[oaicite:1]{index=1}

---

# Architecture

```text
Producer
    ↓
Topic
    ↓
Consumer
```

---

# Topic

A stream of messages.

Example:

```text
orders
payments
notifications
```

---

# Example

```text
Order Service
      ↓
Orders Topic
      ↓
Consumers
```

---

# Benefits

```text
High Throughput
Durability
Scalability
Fault Tolerance
```

---

# Real Companies

Use Kafka for:

```text
Events
Analytics
Logs
Streaming
```

---

# Kafka Components

---

## Producer

Writes messages.

---

## Topic

Stores messages.

---

## Consumer

Reads messages.

---

## Broker

Kafka server.

---

# Example

```text
Order Created
```

Message goes into:

```text
orders topic
```

---

Consumers:

```text
Email Service
Analytics Service
Inventory Service
```

all receive it.

---

# Kafka Strength

One event can be consumed by many services.

---

# Example

```text
Order Created
      ↓
Kafka
      ↓
Email
Analytics
Inventory
```

---

# 7. RabbitMQ

Another Very Common Interview Topic.

---

Entity: :contentReference[oaicite:2]{index=2}

---

# What is RabbitMQ?

Message broker focused on reliable message delivery.

---

# Architecture

```text
Producer
    ↓
Exchange
    ↓
Queue
    ↓
Consumer
```

---

# Benefits

```text
Simple
Reliable
Easy To Use
```

---

# Common Use Cases

```text
Email Processing
Background Jobs
Task Queues
Notifications
```

---

# Example

```text
Order Service
      ↓
RabbitMQ
      ↓
Email Worker
```

---

# RabbitMQ vs Kafka

Most Asked Interview Question.

---

| Feature | Kafka | RabbitMQ |
|----------|--------|----------|
| Purpose | Event Streaming | Message Queue |
| Throughput | Very High | High |
| Message Retention | Long-term | Usually Short-term |
| Replay Messages | Yes | Limited |
| Complexity | Higher | Lower |
| Real-Time Analytics | Excellent | Limited |

---

# Interview Answer

Use:

```text
RabbitMQ
```

for task processing.

Use:

```text
Kafka
```

for event streaming and large-scale systems.

---

# Synchronous vs Asynchronous Communication

Very Important Question.

---

# Synchronous

```text
Service A
     ↓
Service B
     ↓
Wait
```

Examples:

```text
REST
gRPC
```

---

# Asynchronous

```text
Service A
     ↓
Queue
     ↓
Service B
```

No waiting.

---

Examples:

```text
Kafka
RabbitMQ
```

---

# Real Microservice Architecture

```text
Client
   ↓
API Gateway
   ↓

 ├── User Service
 ├── Order Service
 ├── Payment Service
 └── Inventory Service

        ↓
      Kafka

 ├── Analytics
 ├── Notifications
 └── Reporting
```

---

# Most Asked Interview Questions

---

## What are Microservices?

Small independently deployable services that work together.

---

## Why Microservices?

- Scalability
- Independent Deployment
- Better Maintainability

---

## What is Service Discovery?

Mechanism for dynamically locating services.

---

## What is API Gateway?

Single entry point for all client requests.

---

## What is gRPC?

High-performance RPC framework using HTTP/2 and Protocol Buffers.

---

## REST vs gRPC?

REST:

```text
Public APIs
```

gRPC:

```text
Internal Services
```

---

## What is a Message Queue?

System for asynchronous communication between services.

---

## What is Kafka?

Distributed event streaming platform.

---

## What is RabbitMQ?

Reliable message broker for task queues and background jobs.

---

## Kafka vs RabbitMQ?

Kafka:

```text
Event Streaming
Analytics
Large Scale
```

RabbitMQ:

```text
Task Queues
Simple Messaging
```

---

# Quick Revision Cheat Sheet

## Microservices

```text
Independent Services
```

---

## Service Discovery

```text
Consul
Kubernetes DNS
```

---

## API Gateway

```text
Single Entry Point
```

---

## gRPC

```text
HTTP/2
Protobuf
```

---

## Message Queue

```text
Producer
Queue
Consumer
```

---

## Kafka

```text
Producer
Topic
Consumer
```

---

## RabbitMQ

```text
Producer
Exchange
Queue
Consumer
```

---

## Synchronous

```text
REST
gRPC
```

---

## Asynchronous

```text
Kafka
RabbitMQ
```

---

# Golden Interview Answer

### How Do Microservices Communicate?

Microservices communicate in two main ways:

### Synchronous

```text
REST
gRPC
```

Used when an immediate response is needed.

---

### Asynchronous

```text
Kafka
RabbitMQ
```

Used for events, notifications, and background processing.

A typical production architecture includes:

```text
API Gateway
      ↓
Microservices
      ↓
Kafka/RabbitMQ
```

along with service discovery mechanisms to allow services to find and communicate with each other dynamically.

---

# End of Go Microservices Revision