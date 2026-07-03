Most developers build APIs like this:

```http
GET /users/101
```

or

```http
POST /orders
```

using:

```
HTTP/1.1

+

REST

+

JSON
```

This works perfectly.

But imagine Google.

One user opens YouTube.

The browser immediately requests:

- User Profile
- Recommendations
- Comments
- Ads
- Notifications
- Subtitles
- Video Metadata
- Analytics

Hundreds of requests happen almost simultaneously.

Question:

Can HTTP/1.1 efficiently handle thousands of concurrent requests between thousands of microservices?

Not very well.

Google needed something faster.

The solution became:

```
HTTP/2
```

and

```
gRPC
```

Today, many internal communications at Google, Netflix, Uber, Cloudflare, Kubernetes components, and service meshes rely heavily on HTTP/2 and gRPC.

After this chapter, you'll understand why modern distributed systems increasingly use gRPC instead of traditional REST APIs.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 79 — gRPC & HTTP/2: High-Performance Communication for Modern Distributed Systems

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why HTTP/1.1 becomes a bottleneck
> - HTTP/2 Architecture
> - Binary Framing
> - Multiplexing
> - HPACK Header Compression
> - What is gRPC?
> - Protocol Buffers (Protobuf)
> - Unary RPC
> - Streaming RPC
> - gRPC vs REST
> - Real-World Usage

---

# 📖 Why Was HTTP/2 Created?

Imagine loading a modern web page.

Browser requests:

```
HTML
```

```
CSS
```

```
JavaScript
```

```
Images
```

```
Fonts
```

```
Videos
```

Hundreds of requests.

HTTP/1.1 starts showing limitations.

---

# HTTP/1.1 Problem

Suppose one TCP connection.

Requests:

```
Request A

↓

Response A

↓

Request B

↓

Response B
```

One slow request delays others.

This is known as:

```
Head-of-Line Blocking

(at the HTTP layer)
```

---

# Multiple Connections

Browsers opened:

```
6

TCP Connections
```

to improve performance.

Still,

more connections mean:

- More memory
- More TCP handshakes
- More TLS handshakes

Not ideal.

---

# HTTP/2 Solution

HTTP/2 uses:

```
One TCP Connection
```

But allows:

```
Many Requests

At The Same Time
```

---

# Multiplexing

Instead of:

```
A

↓

B

↓

C
```

HTTP/2 sends:

```
A

B

C

Together
```

Responses can arrive independently.

---

# Visual Representation

HTTP/1.1

```
Connection

↓

A

↓

B

↓

C
```

HTTP/2

```
Connection

↓

A

B

C

↓

Mixed Frames
```

Much more efficient.

---

# Streams

Every request gets a:

```
Stream ID
```

Example:

```
Stream 1

GET /users
```

```
Stream 3

GET /orders
```

```
Stream 5

GET /payments
```

All share the same TCP connection.

---

# Frames

HTTP/2 doesn't send plain text messages.

Instead,

it divides communication into:

```
Frames
```

Small binary units.

---

# Binary Protocol

HTTP/1.1

```
Readable Text
```

HTTP/2

```
Binary Frames
```

Computers process binary formats more efficiently than textual protocols.

---

# Header Compression

Suppose every request sends:

```
Authorization

User-Agent

Accept

Cookie
```

Repeated again and again.

HTTP/2 uses:

```
HPACK
```

to compress repeated headers.

Bandwidth decreases significantly.

---

# Example

Instead of sending:

```
Authorization

Bearer ...

```

100 times,

HTTP/2 can reuse compressed header information.

---

# Server Push

HTTP/2 originally introduced:

```
Server Push
```

allowing servers to proactively send related resources.

Although supported by the protocol,

many browsers and deployments have reduced or removed its usage due to practical limitations.

---

# What is gRPC?

gRPC is a modern Remote Procedure Call (RPC) framework developed by Google.

Instead of thinking:

```
HTTP Request
```

Think:

```
Call Function

On Another Server
```

---

# REST Example

```
GET /users/101
```

Server returns:

```json
{
  "id":101,
  "name":"Vinay"
}
```

---

# gRPC Example

Application simply calls:

```text
GetUser(101)
```

It feels like calling a local function,

even though it executes on another machine.

---

# Remote Procedure Call

RPC means:

```
Local Function

↓

Network

↓

Remote Function
```

The networking details are hidden.

---

# Protocol Buffers

gRPC usually uses:

```
Protocol Buffers

(Protobuf)
```

Instead of JSON.

---

# Why Protobuf?

JSON:

```
Text

↓

Large
```

Protobuf:

```
Binary

↓

Compact
```

Messages are typically:

✔ Smaller

✔ Faster to serialize

✔ Faster to deserialize

---

# Example

Instead of:

```json
{
  "name":"Vinay",
  "age":25
}
```

Protobuf stores a compact binary representation based on the message schema.

Applications automatically convert between binary and objects.

---

# .proto File

Developers define messages.

Example:

```proto
message User {
  string name = 1;
  int32 age = 2;
}
```

Tools generate client and server code.

---

# Unary RPC

One request.

One response.

```
Client

↓

Request

↓

Server

↓

Response
```

Similar to a normal REST request.

---

# Server Streaming

One request.

Many responses.

```
Client

↓

Request

↓

Server

↓

Response

↓

Response

↓

Response
```

Useful for logs, notifications, or large datasets.

---

# Client Streaming

Many requests.

One response.

```
Client

↓

Data

↓

Data

↓

Data

↓

Server

↓

Response
```

Useful for uploads.

---

# Bidirectional Streaming

Both sides send messages simultaneously.

```
Client

⇄

Server
```

Excellent for:

- Chat applications
- Real-time collaboration
- Multiplayer games

---

# gRPC vs REST

| REST | gRPC |
|-------|------|
| HTTP/1.1 or HTTP/2 | Usually HTTP/2 |
| JSON | Protobuf |
| Text | Binary |
| Larger Messages | Smaller Messages |
| Resource-Oriented | RPC-Oriented |
| Browser Friendly | Backend Friendly |

---

# Performance

Because of:

✔ HTTP/2

✔ Multiplexing

✔ Binary Protocol

✔ Header Compression

✔ Protobuf

gRPC is often faster and more bandwidth-efficient for service-to-service communication.

---

# Browser Support

Browsers naturally understand:

```
REST

HTTP
```

gRPC is primarily designed for backend communication.

Direct browser support is limited,

although technologies like gRPC-Web bridge this gap.

---

# Kubernetes

Many Kubernetes components communicate using gRPC.

Examples include:

- Container Runtime Interface (CRI)
- CSI (Container Storage Interface)
- Some control-plane interactions

---

# Service Mesh

Envoy and Istio fully support:

```
HTTP/2

+

gRPC
```

Making them excellent choices for microservice communication.

---

# Complete Request Flow

```
Application

↓

gRPC Stub

↓

HTTP/2

↓

TCP

↓

Remote Server

↓

Function

↓

Response
```

---

# Why gRPC Matters

REST is excellent for:

✔ Public APIs

✔ Browser Clients

gRPC excels at:

✔ Internal Microservices

✔ High Throughput

✔ Low Latency

✔ Streaming

---

# Hands-on Lab

## Lab 1 — Create a Simple .proto File

```proto
syntax = "proto3";

service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
}
```

Observe how service definitions resemble function declarations.

---

## Lab 2 — Compare JSON vs Protobuf

Serialize the same data using JSON and Protobuf.

Compare message sizes.

---

## Lab 3 — Observe HTTP/2

Use browser developer tools or command-line tools to inspect HTTP/2 connections to supported websites.

---

# Interview Questions

## What is HTTP/2?

HTTP/2 is a newer version of HTTP that introduces binary framing, multiplexing, and header compression to improve performance.

---

## What is multiplexing?

Multiplexing allows multiple independent request-response streams to share a single TCP connection simultaneously.

---

## What is gRPC?

gRPC is Google's high-performance RPC framework built primarily on HTTP/2 and Protocol Buffers.

---

## Why does gRPC use Protocol Buffers?

Protocol Buffers produce compact binary messages that are typically faster and smaller than JSON.

---

## What are the four types of gRPC calls?

- Unary
- Server Streaming
- Client Streaming
- Bidirectional Streaming

---

## When should you choose REST?

Public APIs, browser clients, and scenarios where simplicity and broad compatibility are important.

---

## When should you choose gRPC?

Internal microservices, high-performance communication, streaming workloads, and strongly typed APIs.

---

# Summary

HTTP/2 and gRPC modernize service-to-service communication.

```
Application

↓

gRPC Stub

↓

Protobuf

↓

HTTP/2

↓

TCP

↓

Remote Service

↓

Response
```

Key concepts:

- HTTP/2 introduces multiplexing, binary framing, and header compression.
- gRPC builds on HTTP/2.
- Protocol Buffers replace JSON for compact binary messages.
- gRPC supports unary and streaming communication.
- Modern cloud-native platforms frequently use gRPC internally.
- REST remains an excellent choice for many public-facing APIs.

At this point, you've learned how modern distributed systems communicate efficiently.

The final chapter of this module expands from software architecture into **cloud networking**, showing how providers like AWS, Azure, and Google Cloud connect thousands of virtual machines and Kubernetes clusters.

---

# Next Chapter

## Chapter 80 — Cloud Networking: VPCs, Subnets, Internet Gateways, NAT Gateways, and Cloud Load Balancers

We'll explore:

- Why Cloud Networking Exists
- What is a VPC?
- Public vs Private Subnets
- Route Tables
- Internet Gateway (IGW)
- NAT Gateway
- Security Groups
- Network ACLs
- Cloud Load Balancers
- AWS, Azure, and GCP Networking
- Complete Cloud Network Architecture

> **This chapter connects everything you've learned about Linux networking and distributed systems to real cloud infrastructure used by AWS, Azure, and Google Cloud.**