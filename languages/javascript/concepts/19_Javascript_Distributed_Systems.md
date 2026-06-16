# Distributed Systems – Interview Revision Guide 🚀

⚠️ **Distributed Systems is a very important topic for Backend, System Design, Microservices, and Senior Developer interviews.**

Modern applications rarely run on a single server.

Examples:

```text
Netflix
Amazon
Uber
Google
Facebook
```

All run on hundreds or thousands of servers.

This introduces new challenges:

- Network failures
- Duplicate requests
- Partial failures
- Data synchronization
- Service outages

Distributed Systems concepts help solve these problems.

---

# Table of Contents

1. What is a Distributed System?
2. Eventual Consistency
3. Retries
4. Circuit Breaker
5. Idempotency
6. CAP Theorem
7. Common Interview Questions
8. Interview Quick Revision

---

# 1. What is a Distributed System?

A distributed system is a group of multiple machines working together as one system.

---

## Single Server

```text
Client
  ↓
Server
```

---

## Distributed System

```text
Client
  ↓
Load Balancer
  ↓
Server 1
Server 2
Server 3
```

---

Benefits:

✅ Scalability

✅ Fault Tolerance

✅ High Availability

✅ Better Performance

---

Problems:

❌ Network Failures

❌ Data Synchronization

❌ Partial Failures

❌ Consistency Issues

---

# 2. Eventual Consistency

⚠️ One of the most important distributed system concepts.

---

## Problem

Imagine:

```text
Database A
Database B
Database C
```

---

User updates profile:

```text
Name = Vinay
```

---

Update reaches:

```text
Database A
```

immediately.

---

But:

```text
Database B
Database C
```

may receive it later.

---

For a short period:

```text
Different Databases
Different Values
```

---

# What is Eventual Consistency?

A system guarantees that:

> If no new updates occur, all replicas will eventually have the same data.

---

Visualization

```text
Time T1

DB A → Vinay
DB B → Old
DB C → Old

       ↓

Time T2

DB A → Vinay
DB B → Vinay
DB C → Vinay
```

---

Eventually:

```text
All Become Consistent
```

---

# Real Examples

### Social Media

```text
Like Count
Followers
Views
```

---

### DNS

```text
DNS updates take time
```

---

### Distributed Databases

```text
Cassandra
DynamoDB
```

---

# Tradeoff

```text
Fast Availability
↓
Temporary Inconsistency
```

---

# 3. Retries

⚠️ Very common system design topic.

---

## Problem

Network requests can fail.

---

Example:

```text
Service A
     ↓
Service B
```

---

Request fails because:

```text
Network Issue
Temporary Outage
Timeout
```

---

# Retry Solution

Try again.

---

Example

```text
Attempt 1 ❌

Attempt 2 ❌

Attempt 3 ✅
```

---

# Retry Flow

```text
Request
   ↓
Failure
   ↓
Retry
   ↓
Success
```

---

# Example

```js
for (let i = 0; i < 3; i++) {

  try {

    await apiCall();

    break;

  } catch {}

}
```

---

# Exponential Backoff

⚠️ Frequently asked interview topic.

---

Bad:

```text
Retry Immediately
Retry Immediately
Retry Immediately
```

---

Can overload servers.

---

Better:

```text
Retry 1 → 1s

Retry 2 → 2s

Retry 3 → 4s

Retry 4 → 8s
```

---

Visualization

```text
1s
 ↓
2s
 ↓
4s
 ↓
8s
```

---

# Why?

Reduces load during outages.

---

# 4. Circuit Breaker

⚠️ One of the most asked microservice interview topics.

---

## Problem

Service B is down.

---

```text
Service A
     ↓
Service B ❌
```

---

Without protection:

```text
A keeps sending requests
```

---

Result:

```text
More failures
More load
Cascading failures
```

---

# Circuit Breaker Solution

Stop sending requests temporarily.

---

Visualization

```text
Too Many Failures
       ↓
Circuit Opens
       ↓
Requests Blocked
```

---

# States

### Closed

Normal operation.

```text
Requests Allowed
```

---

### Open

Service unhealthy.

```text
Requests Blocked
```

---

### Half-Open

Testing recovery.

```text
Allow Few Requests
```

---

# State Flow

```text
Closed
   ↓
Failures
   ↓
Open
   ↓
Timeout
   ↓
Half Open
   ↓
Success
   ↓
Closed
```

---

# Real Libraries

### Node.js

```text
opossum
```

---

### Java

```text
Resilience4j
Hystrix
```

---

# Benefits

✅ Prevent Cascading Failures

✅ Reduce Load

✅ Faster Recovery

---

# 5. Idempotency

⚠️ One of the MOST IMPORTANT interview topics.

---

## What is Idempotency?

Performing the same operation multiple times produces the same result.

---

# Example

```text
Set Account Status = Active
```

---

Run:

```text
1 time
```

or

```text
100 times
```

Result:

```text
Active
```

---

Still same result.

---

# Non-Idempotent Example

```text
Add ₹100 to account
```

---

Run once:

```text
100
```

---

Run twice:

```text
200
```

---

Different result.

---

Not idempotent.

---

# Why Important?

Imagine:

```text
Payment Request
```

---

Client sends:

```text
Pay ₹1000
```

---

Response lost.

Client retries.

---

Without idempotency:

```text
₹1000 charged twice
```

---

Problem.

---

# Idempotency Key

Common solution.

---

Client sends:

```text
Request-ID: abc123
```

---

Server stores:

```text
abc123
```

---

Retry arrives:

```text
Same Request-ID
```

---

Server returns old result.

---

No duplicate payment.

---

# Real Uses

```text
Payments
Orders
Bank Transfers
Ticket Booking
```

---

# HTTP Methods

### Idempotent

```text
GET
PUT
DELETE
```

---

### Not Naturally Idempotent

```text
POST
```

---

# Example

```text
GET /users
```

Run 100 times.

Same result.

---

Idempotent.

---

# 6. CAP Theorem

⚠️ One of the MOST FAMOUS distributed system interview questions.

---

## What is CAP Theorem?

A distributed system can guarantee only TWO of:

```text
Consistency
Availability
Partition Tolerance
```

---

# Consistency (C)

Every node returns same data.

---

Example

```text
User Name = Vinay
```

All servers return:

```text
Vinay
```

---

# Availability (A)

Every request receives a response.

Even if response may be outdated.

---

# Partition Tolerance (P)

System continues working despite network failures.

---

Example

```text
Server A
   X
Server B
```

Network disconnected.

---

System still operates.

---

# CAP Rule

When partition occurs:

```text
Must choose:

Consistency

OR

Availability
```

---

Cannot have both.

---

# CP System

Choose:

```text
Consistency
+
Partition Tolerance
```

---

May reject requests.

---

Example:

```text
HBase
MongoDB (certain modes)
ZooKeeper
```

---

Visualization

```text
Network Failure
       ↓
Block Requests
       ↓
Maintain Consistency
```

---

# AP System

Choose:

```text
Availability
+
Partition Tolerance
```

---

Always respond.

May return stale data.

---

Examples:

```text
Cassandra
DynamoDB
Riak
```

---

Visualization

```text
Network Failure
       ↓
Return Data
       ↓
Sync Later
```

---

# Important Interview Point

In real distributed systems:

```text
Partition Tolerance
```

is mandatory.

---

Therefore choice is usually:

```text
CP

or

AP
```

---

# CAP Triangle

```text
        C
       / \
      /   \
     /     \
    /       \
   A ------- P
```

---

Choose only two during network partitions.

---

# Common Interview Questions

---

## Q1: What is Eventual Consistency?

Data may be temporarily inconsistent but eventually becomes consistent.

---

## Q2: Why Are Retries Important?

Handle temporary failures.

---

## Q3: What is Exponential Backoff?

Increasing delay between retries.

Example:

```text
1s → 2s → 4s → 8s
```

---

## Q4: What is a Circuit Breaker?

Stops requests to unhealthy services.

---

## Q5: What is Idempotency?

Performing the same operation multiple times produces the same result.

---

## Q6: Why Is Idempotency Important?

Prevents duplicate payments and duplicate operations.

---

## Q7: What is CAP Theorem?

Choose only two:

```text
Consistency
Availability
Partition Tolerance
```

during a network partition.

---

## Q8: What Are CP Systems?

Consistency + Partition Tolerance.

---

## Q9: What Are AP Systems?

Availability + Partition Tolerance.

---

# Interview Quick Revision 🚀

## Eventual Consistency

```text
Eventually All Nodes Match
```

---

## Retries

```text
Try Again On Failure
```

---

## Exponential Backoff

```text
1s → 2s → 4s → 8s
```

---

## Circuit Breaker

```text
Stop Calling Failed Service
```

---

## Idempotency

```text
Same Request
Same Result
```

---

## Idempotency Key

```text
Unique Request ID
```

---

## CAP Theorem

```text
Consistency
Availability
Partition Tolerance
```

---

## CP

```text
Consistency + Partition Tolerance
```

---

## AP

```text
Availability + Partition Tolerance
```

---

# Most Important Interview Diagram ⭐

### Retry with Backoff

```text
Request
   ↓
Fail
   ↓
1s
   ↓
Retry
   ↓
2s
   ↓
Retry
   ↓
4s
```

---

### Circuit Breaker

```text
Closed
   ↓
Failures
   ↓
Open
   ↓
Half Open
   ↓
Closed
```

---

### Idempotency

```text
Request
   ↓
Request-ID
   ↓
Stored
   ↓
Retry
   ↓
Same Result
```

---

### CAP Theorem

```text
       Consistency
            ▲
           / \
          /   \
         /     \
        /       \
Availability --- Partition Tolerance
```

---

# Golden Interview Tips ⭐

1. Eventual Consistency is common in distributed databases.
2. Always use retries for transient failures.
3. Use exponential backoff instead of immediate retries.
4. Circuit Breakers prevent cascading failures.
5. Idempotency is critical for payment systems.
6. Use idempotency keys for POST requests.
7. CAP theorem is almost guaranteed in system design interviews.
8. In real systems, Partition Tolerance is non-negotiable.
9. Most systems choose between:
   ```text
   CP or AP
   ```
10. If asked:

```text
How do large systems survive failures?
```

Answer:

```text
Retries
+
Circuit Breakers
+
Idempotency
+
Eventual Consistency
```

🚀 These concepts form the foundation of modern microservices, cloud platforms, distributed databases, and large-scale system design.