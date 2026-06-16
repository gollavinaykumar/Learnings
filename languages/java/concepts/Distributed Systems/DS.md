# Distributed Systems - Complete Interview Revision Notes

> 🔴 VERY IMPORTANT FOR BACKEND / MICROSERVICES INTERVIEWS
>
> Asked in:
>
> - Java Backend
> - Spring Boot
> - Microservices
> - System Design
> - Senior Developer Interviews
>
> Topics Covered:
>
> - What is Distributed System?
> - CAP Theorem
> - Consistency Models
> - Eventual Consistency
> - Distributed Locks
> - Idempotency
> - Retries
> - Exponential Backoff
> - Real World Examples

---

# What is a Distributed System?

A Distributed System is a collection of multiple servers/services working together as a single system.

---

# Example

Netflix

```text
User Service

Payment Service

Recommendation Service

Video Service

Notification Service
```

All run on different servers.

---

# Why Distributed Systems?

Single Server Problems:

```text
Limited CPU

Limited Memory

Single Point Of Failure
```

---

Distributed System Benefits:

```text
Scalability

High Availability

Fault Tolerance

Better Performance
```

---

# Example

E-Commerce Application

```text
User Service

Order Service

Inventory Service

Payment Service
```

Each service can scale independently.

---

# CAP Theorem

Most Asked Distributed Systems Interview Question.

---

# What is CAP Theorem?

CAP Theorem states:

A distributed system can guarantee only TWO out of THREE:

```text
Consistency

Availability

Partition Tolerance
```

---

# C = Consistency

All nodes see same data at same time.

---

Example:

User updates profile.

```text
Node A -> New Value

Node B -> New Value

Node C -> New Value
```

All return same result.

---

# A = Availability

Every request gets a response.

---

Example:

```text
Server A Down
```

Still:

```text
Server B Responds
```

System remains available.

---

# P = Partition Tolerance

System continues working even if network communication fails.

---

Example:

```text
Mumbai Server
     X
Delhi Server
```

Network disconnected.

System still operates.

---

# CAP Diagram

```text
         CAP

      /   |   \

Consistency

Availability

Partition Tolerance
```

Can choose only two.

---

# CP System

Consistency + Partition Tolerance

Sacrifice Availability.

---

Example:

```text
HBase

MongoDB (Certain Modes)

ZooKeeper
```

---

Scenario:

Network Partition Happens.

Instead of returning stale data:

```text
Request Rejected
```

Consistency maintained.

---

# AP System

Availability + Partition Tolerance

Sacrifice Consistency.

---

Example:

```text
Cassandra

DynamoDB
```

---

Scenario:

Data not fully synchronized.

Still:

```text
System Returns Response
```

Availability maintained.

---

# CA System

Consistency + Availability

No Partition Tolerance.

---

Possible only in:

```text
Single Server Systems
```

Not realistic for distributed systems.

---

# CAP Interview Answer

```text
When network partition occurs,
we must choose between:

Consistency
or
Availability
```

---

# Consistency Models

Important System Design Topic.

---

# What is Consistency?

Defines how quickly updates become visible across nodes.

---

# Strong Consistency

After write completes:

```text
All Reads Return Latest Data
```

---

Example

Bank Account

Balance:

```text
1000
```

Withdraw:

```text
200
```

Immediately:

```text
All Servers Show 800
```

---

# Advantages

```text
Accurate Data
```

---

# Disadvantages

```text
Slower
```

---

# Eventual Consistency

Most Common Cloud Model.

---

# Definition

Data becomes consistent eventually.

Not immediately.

---

Example:

```text
Node A Updated
```

Node B:

```text
Old Data
```

for few seconds.

Later:

```text
Updated Data
```

---

# Real World Example

Instagram Likes

You like a photo.

Immediately:

```text
Server A = 101 Likes
```

Other servers:

```text
100 Likes
```

After few seconds:

```text
101 Likes Everywhere
```

---

# Why Use Eventual Consistency?

Provides:

```text
High Availability

High Scalability
```

---

# Systems Using Eventual Consistency

```text
DynamoDB

Cassandra

MongoDB

DNS
```

---

# Consistency Models Summary

| Model | Guarantee |
|---------|---------|
| Strong Consistency | Latest Data Always |
| Eventual Consistency | Latest Data Eventually |
| Weak Consistency | No Guarantee |

---

# Distributed Locks

Very Important Microservices Topic.

---

# Problem

Multiple servers updating same resource.

Example:

```text
Inventory = 1 Item
```

Server A:

```text
Buy Product
```

Server B:

```text
Buy Product
```

Same time.

Result:

```text
Overselling
```

---

# Solution

Distributed Lock

Only one server allowed.

---

# Example

```text
Server A
Obtains Lock
```

Server B:

```text
Wait
```

---

# Lock Flow

```text
Acquire Lock
     ↓
Perform Work
     ↓
Release Lock
```

---

# Redis Distributed Lock

Common Interview Example.

---

Acquire Lock:

```text
SET product_lock value NX EX 30
```

Meaning:

```text
Create Lock

If Not Exists

Expire In 30 Seconds
```

---

# Real World Use Cases

```text
Inventory Management

Payment Processing

Order Processing

Job Scheduling
```

---

# Idempotency

Most Asked Microservices Question.

---

# What is Idempotency?

Performing same operation multiple times produces same result.

---

# Example

Good:

```http
PUT /users/1
```

Update Name:

```json
{
  "name":"Vinay"
}
```

Calling:

```text
1 Time
10 Times
100 Times
```

Result:

```text
Same State
```

---

# Non-Idempotent Example

```http
POST /orders
```

Call twice:

```text
Order 1 Created

Order 2 Created
```

Duplicate.

---

# Payment Example

User clicks:

```text
Pay Now
```

twice.

Without idempotency:

```text
Charged Twice
```

---

# Solution

Use:

```text
Idempotency Key
```

Example:

```http
Idempotency-Key:
abc123
```

Server stores:

```text
Processed Request
```

Duplicate request:

```text
Ignored
```

---

# Real World Examples

```text
Stripe Payments

Razorpay

PayPal
```

Use idempotency keys.

---

# Retries

Very Important Reliability Topic.

---

# Why Retries?

Distributed systems fail.

Example:

```text
Network Timeout

Temporary DB Failure

Service Unavailable
```

---

Instead of failing:

```text
Retry Request
```

---

# Example

```text
Attempt 1 ❌

Attempt 2 ✅
```

Success.

---

# Retry Example

```java
for(int i=1;i<=3;i++){

    try{

        callPaymentService();

        break;

    }catch(Exception e){

    }
}
```

---

# Retry Risks

Blind retries can cause:

```text
Duplicate Payments

Duplicate Orders

Overloaded Services
```

---

# Solution

```text
Idempotency
```

before retries.

---

# Exponential Backoff

Most Asked Follow-Up Question.

---

# Problem

Service is down.

Thousands of clients retry immediately.

---

Result:

```text
System Overload
```

---

# Bad Retry

```text
Retry Every Second
```

```text
1s
1s
1s
1s
```

---

# Exponential Backoff

Increase delay after each failure.

---

Example

```text
Retry 1 -> 1 second

Retry 2 -> 2 seconds

Retry 3 -> 4 seconds

Retry 4 -> 8 seconds

Retry 5 -> 16 seconds
```

---

# Diagram

```text
1s
 ↓
2s
 ↓
4s
 ↓
8s
 ↓
16s
```

---

# Why Better?

Prevents:

```text
Retry Storm

Server Overload
```

---

# Real World Example

Payment Gateway Down

Instead of:

```text
1 Million Requests
Every Second
```

Use:

```text
Exponential Backoff
```

Traffic reduces.

---

# Retry + Backoff + Idempotency

Production Standard Pattern.

---

```text
Request
   ↓
Failure
   ↓
Retry
   ↓
Exponential Backoff
   ↓
Idempotency Check
   ↓
Success
```

---

# Real Microservices Example

Order Service

```text
Create Order
```

↓

Payment Service

```text
Timeout
```

↓

Retry

```text
After 1s
```

↓

Retry

```text
After 2s
```

↓

Retry

```text
After 4s
```

↓

Payment Success

No duplicate payment because:

```text
Idempotency Key Used
```

---

# Frequently Asked Interview Questions

## Q1. What is CAP Theorem?

```text
A Distributed System Can Guarantee
Only Two Of:

Consistency
Availability
Partition Tolerance
```

---

## Q2. What is Eventual Consistency?

```text
Data Becomes Consistent
After Some Time
```

---

## Q3. Difference Between Strong Consistency and Eventual Consistency?

Strong:

```text
Latest Data Immediately
```

Eventual:

```text
Latest Data Eventually
```

---

## Q4. Why Distributed Locks?

```text
Prevent Multiple Services
Modifying Same Resource
```

---

## Q5. What is Idempotency?

```text
Same Request
Same Result
```

even if executed multiple times.

---

## Q6. Why Are Retries Needed?

```text
Temporary Failures
Network Issues
Timeouts
```

---

## Q7. Why Exponential Backoff?

```text
Avoid Retry Storms
Reduce Load
```

---

## Q8. Can Retries Cause Problems?

```text
Yes

Duplicate Orders
Duplicate Payments
```

Solution:

```text
Idempotency
```

---

# One-Minute Revision

```text
Distributed System
-------------------
Multiple Servers Working Together

CAP Theorem
-------------------
Consistency
Availability
Partition Tolerance

Choose Any Two

Consistency Models
-------------------
Strong
Eventual
Weak

Eventual Consistency
-------------------
Data Syncs Eventually

Distributed Lock
-------------------
Only One Service Can Update Resource

Redis Lock
-------------------
SET key value NX EX 30

Idempotency
-------------------
Same Request
Same Result

Retries
-------------------
Handle Temporary Failures

Exponential Backoff
-------------------
1s
2s
4s
8s
16s

Production Pattern
-------------------
Retry
+
Backoff
+
Idempotency

Interview Favorites
-------------------
CAP Theorem
Eventual Consistency
Distributed Locks
Idempotency
Retries & Backoff
```

# End of Distributed Systems Notes