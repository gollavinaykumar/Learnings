# Go Distributed Systems – Complete Interview Revision Guide

> Distributed Systems is a senior-level backend topic.
>
> Once applications become large enough, they run on:
>
> - Multiple Servers
> - Multiple Databases
> - Multiple Services
> - Multiple Regions
>
> At this scale, new problems appear:
>
> - Network failures
> - Duplicate requests
> - Partial failures
> - Data inconsistency
> - Service outages
>
> Common interview topics:
>
> - CAP Theorem
> - Eventual Consistency
> - Idempotency
> - Retries
> - Circuit Breakers
> - Distributed Locks

---

# 1. What is a Distributed System?

A distributed system is a collection of multiple computers working together as a single system.

---

# Example

```text
User Service
Order Service
Payment Service
Inventory Service
```

Running on different servers.

---

# Visual

```text
Client
  ↓

API Gateway

  ↓

 ├── User Service
 ├── Order Service
 ├── Payment Service
 └── Inventory Service
```

---

# Benefits

```text
Scalability
Fault Tolerance
High Availability
```

---

# Challenges

```text
Network Failures
Latency
Data Consistency
Partial Failures
```

---

# Simple Interview Answer

```text
A distributed system is a collection of independent machines that work together as a single application.
```

---

# 2. CAP Theorem

Most Asked Distributed Systems Interview Topic.

---

# What is CAP Theorem?

CAP theorem states:

A distributed system can guarantee only TWO of the following THREE properties at the same time:

```text
Consistency
Availability
Partition Tolerance
```

---

# CAP Components

---

## Consistency (C)

All nodes see the same data.

---

Example:

```text
Write Balance = 100
```

All servers immediately return:

```text
100
```

---

# Availability (A)

Every request receives a response.

Even if response is stale.

---

Example:

```text
Server responds
even during failures
```

---

# Partition Tolerance (P)

System continues working even if servers cannot communicate.

---

Example:

```text
Network Split
```

Some servers cannot talk to others.

System still runs.

---

# Visual

```text
       CAP

Consistency
     / \
    /   \
Availability
     \
      \
Partition Tolerance
```

---

# Important Rule

In distributed systems:

```text
Partition Tolerance is mandatory
```

because network failures happen.

---

Therefore:

Usually choose between:

```text
CP
or
AP
```

---

# CP Systems

Consistency + Partition Tolerance

Sacrifice availability.

---

Example:

```text
Banking Systems
```

Better to reject requests than return wrong balance.

---

# AP Systems

Availability + Partition Tolerance

Sacrifice immediate consistency.

---

Example:

```text
Social Media
```

Better to show slightly stale data.

---

# Interview Answer

CAP theorem states that during a network partition, a distributed system must choose between consistency and availability.

---

# 3. Eventual Consistency

Extremely Important Topic.

---

# What is Eventual Consistency?

Data may not be immediately consistent.

But eventually all nodes will converge to the same value.

---

# Example

User updates profile.

```text
Server A → Updated
```

---

Server B still has:

```text
Old Data
```

---

After synchronization:

```text
All Servers Updated
```

---

# Visual

```text
Write
  ↓
Server A Updated
  ↓
Server B Updated Later
  ↓
Consistent Eventually
```

---

# Why Needed?

Strong consistency is expensive.

Eventual consistency improves:

```text
Availability
Scalability
Performance
```

---

# Real Examples

---

## Social Media Likes

```text
100 Likes
```

might briefly show:

```text
99 Likes
```

on another server.

Acceptable.

---

## Product View Count

Slight delay is okay.

---

# Where Not To Use?

Bad for:

```text
Bank Balances
Payments
Financial Transactions
```

---

# Interview Answer

Eventual consistency means all replicas may not immediately agree, but they will eventually reach the same state.

---

# 4. Idempotency

Most Asked API Interview Topic.

---

# What is Idempotency?

Performing the same operation multiple times produces the same result.

---

# Example

Request:

```http
DELETE /users/1
```

---

First call:

```text
User Deleted
```

---

Second call:

```text
User Already Deleted
```

Final state remains:

```text
Deleted
```

---

Therefore:

```text
Idempotent
```

---

# Non-Idempotent Example

```http
POST /payments
```

---

First Request:

```text
Charge $100
```

---

Retry:

```text
Charge Another $100
```

---

Result:

```text
$200 Charged
```

Bad.

---

# Why Important?

Networks fail.

Clients retry requests.

Need protection against duplicates.

---

# Idempotency Key

Common Production Pattern.

---

Example:

```http
Idempotency-Key:
abc123
```

---

Server stores:

```text
abc123 → Payment Success
```

---

Retry arrives:

```text
Same Key
```

Return old result.

Don't process again.

---

# Real Example

Used heavily in:

```text
Payments
Orders
Financial Systems
```

---

# Interview Answer

Idempotency ensures repeated requests produce the same outcome and prevents duplicate processing.

---

# 5. Retries

Very Common Production Topic.

---

# Why Retries?

Networks fail temporarily.

---

Example:

```text
Timeout
Temporary DB Failure
Service Restart
```

---

Retry may succeed.

---

# Example

```text
Service A
    ↓
Service B
```

---

First call:

```text
Timeout
```

---

Retry:

```text
Success
```

---

# Retry Strategy

Bad:

```text
Retry Forever
```

---

Good:

```text
Retry Limited Times
```

Example:

```text
3 Attempts
```

---

# Exponential Backoff

Most Important Retry Pattern.

---

# Problem

Immediate retries can overload system.

---

# Solution

Wait longer after each failure.

---

Example:

```text
Attempt 1 → Wait 1 sec
Attempt 2 → Wait 2 sec
Attempt 3 → Wait 4 sec
Attempt 4 → Wait 8 sec
```

---

# Visual

```text
1s
2s
4s
8s
```

---

# Benefits

```text
Reduced Load
Better Recovery
More Stable Systems
```

---

# Interview Answer

Retries help recover from temporary failures and should usually use exponential backoff.

---

# 6. Circuit Breaker

Extremely Important Senior-Level Topic.

---

# Problem

Service B is down.

---

```text
Service A
    ↓
Service B
```

---

Without protection:

```text
A keeps calling B
```

---

Result:

```text
More failures
More load
Cascading failure
```

---

# Solution

Circuit Breaker.

---

# What is Circuit Breaker?

Stops requests to failing services temporarily.

---

# States

---

## Closed

Normal operation.

Requests allowed.

---

## Open

Too many failures.

Requests blocked.

---

## Half-Open

Testing if service recovered.

---

# Visual

```text
Closed
   ↓
Failures
   ↓
Open
   ↓
Wait
   ↓
Half Open
   ↓
Success
   ↓
Closed
```

---

# Example

```text
Payment Service Down
```

Circuit opens.

---

Instead of:

```text
1000 failing requests
```

System immediately rejects calls.

---

# Benefits

```text
Protects System
Reduces Load
Prevents Cascading Failures
```

---

# Popular Go Library

```go
github.com/sony/gobreaker
```

---

# Interview Answer

Circuit breaker prevents repeated calls to failing services and helps systems recover gracefully.

---

# 7. Distributed Locks

Very Important Advanced Topic.

---

# Problem

Single server lock:

```go
mutex.Lock()
```

works only inside one process.

---

What if:

```text
Server A
Server B
Server C
```

all update same resource?

---

Need:

```text
Distributed Lock
```

---

# What is a Distributed Lock?

A lock shared across multiple machines.

---

# Example

```text
Inventory Count
```

Only one service should update it at a time.

---

# Visual

```text
Server A
      ↓
 Acquire Lock
      ↓
 Update Inventory
      ↓
 Release Lock
```

---

Other servers wait.

---

# Common Implementations

---

## Redis Lock

Most common.

---

## ZooKeeper

---

## etcd

---

# Redis Example

```text
SET key value NX EX 30
```

Meaning:

```text
Create Lock
Expire In 30 Seconds
```

---

# Why Expiration?

Prevent deadlocks.

---

# Example Use Cases

```text
Scheduled Jobs
Inventory Updates
Leader Election
Payment Processing
```

---

# Distributed Lock Risks

If lock never releases:

```text
System Stuck
```

---

Always use:

```text
Timeout
Expiration
```

---

# Real Example

Two servers processing same order.

Without lock:

```text
Order Processed Twice
```

---

With lock:

```text
Only One Server Processes
```

---

# Distributed Systems Patterns

Important Summary.

---

## Retry

Recover temporary failures.

---

## Circuit Breaker

Protect failing services.

---

## Idempotency

Prevent duplicate processing.

---

## Distributed Lock

Prevent concurrent updates.

---

# Real Production Flow

```text
Client
   ↓
API Gateway
   ↓
Order Service
   ↓
Retry
   ↓
Circuit Breaker
   ↓
Payment Service
   ↓
Distributed Lock
   ↓
Database
```

---

# Most Asked Interview Questions

---

## What is CAP Theorem?

A distributed system can only guarantee two of:

```text
Consistency
Availability
Partition Tolerance
```

during a partition.

---

## What is Eventual Consistency?

Replicas may temporarily differ but eventually converge.

---

## What is Idempotency?

Multiple identical requests produce the same final result.

---

## Why Are Idempotency Keys Used?

Prevent duplicate processing during retries.

---

## What is Exponential Backoff?

Increasing wait time between retries.

---

## What is a Circuit Breaker?

Stops requests to failing services temporarily.

---

## Why Use Circuit Breakers?

Prevent cascading failures.

---

## What is a Distributed Lock?

Lock shared across multiple servers.

---

## Why Can't Mutex Be Used Across Servers?

Mutex works only inside a single process.

---

## Common Distributed Lock Tools?

```text
Redis
ZooKeeper
etcd
```

---

# Quick Revision Cheat Sheet

## CAP

```text
Consistency
Availability
Partition Tolerance
```

---

## Eventual Consistency

```text
Consistent Later
```

---

## Idempotency

```text
Same Request
Same Result
```

---

## Idempotency Key

```http
Idempotency-Key
```

---

## Retry

```text
Retry Failed Requests
```

---

## Exponential Backoff

```text
1s
2s
4s
8s
```

---

## Circuit Breaker

```text
Closed
Open
Half-Open
```

---

## Distributed Lock

```text
Shared Lock Across Servers
```

---

## Redis Lock

```text
SET NX EX
```

---

# Golden Interview Answer

### What Problems Do Distributed Systems Introduce?

When applications run across multiple services and servers, failures become inevitable.

To build reliable systems, engineers use:

- CAP theorem tradeoffs
- Eventual consistency
- Idempotency
- Retries with exponential backoff
- Circuit breakers
- Distributed locks

These patterns help maintain availability, consistency, and reliability in large-scale distributed applications.

---

# End of Distributed Systems Revision