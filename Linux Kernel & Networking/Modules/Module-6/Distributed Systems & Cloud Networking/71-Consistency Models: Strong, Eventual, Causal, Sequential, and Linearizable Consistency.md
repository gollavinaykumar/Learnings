After learning the CAP Theorem, many developers think:

> "Consistency means everyone sees the same data."

That is true...

But **what does "same" actually mean?**

Imagine this.

You upload a profile picture to Instagram.

Your phone immediately shows:

```
✅ New Photo
```

Your friend's phone still shows:

```
❌ Old Photo
```

Five seconds later:

```
Everyone

↓

New Photo
```

Question:

Was the system:

- Consistent?
- Inconsistent?
- Eventually Consistent?

The answer depends on **which consistency model** the system guarantees.

Modern distributed systems don't all provide the same type of consistency.

Different applications choose different guarantees.

After this chapter, you'll understand why databases like **MongoDB**, **Cassandra**, **CockroachDB**, **etcd**, and **Spanner** behave differently.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 71 — Consistency Models: Strong, Eventual, Sequential, Linearizable, and Causal Consistency

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Consistency Models Exist
> - Strong Consistency
> - Eventual Consistency
> - Sequential Consistency
> - Linearizability
> - Causal Consistency
> - Read-Your-Writes
> - Monotonic Reads
> - Real-World Database Examples
> - Choosing the Right Model

---

# 📖 Why Do We Need Different Consistency Models?

Imagine:

```
Server A

↓

India
```

```
Server B

↓

USA
```

A user updates:

```
Name

↓

Vinay
```

Question:

When should every server show:

```
Vinay
```

Immediately?

Within one second?

Within one minute?

Eventually?

Different applications answer this differently.

---

# What is a Consistency Model?

A consistency model defines:

```
Rules

↓

For

↓

When Data Becomes Visible
```

after updates.

It tells applications what they can expect when reading data.

---

# Strong Consistency

Strong Consistency guarantees:

```
Write

↓

Complete

↓

Every Read

↓

Latest Value
```

Nobody can read stale data.

---

# Example

Initial value:

```
100
```

Write:

```
200
```

Immediately:

```
Server A

↓

200
```

```
Server B

↓

200
```

Every client sees:

```
200
```

---

# Timeline

```
Write

↓

Complete

↓

Read

↓

Latest Value
```

Always.

---

# Advantages

✔ Easy to understand

✔ No stale reads

✔ Perfect for banking

---

# Disadvantages

❌ Higher latency

❌ More coordination

❌ Slower writes across distant regions

---

# Eventual Consistency

Suppose:

```
Write

↓

Server A

↓

Updated
```

Other servers update:

```
Later
```

Eventually:

```
Everyone

↓

Same Data
```

No guarantee about exactly **when** that happens.

---

# Example

Time:

```
10:00

↓

Server A

↓

Photo Updated
```

```
10:01

↓

Server B

↓

Still Old
```

```
10:05

↓

Server B

↓

Updated
```

Eventually,

all replicas converge.

---

# Advantages

✔ Fast

✔ Highly available

✔ Easy to scale globally

---

# Disadvantages

❌ Temporary stale reads

---

# Sequential Consistency

Sequential Consistency guarantees:

All operations appear to execute in one global order,

and every client observes **the same order**.

However,

that order does **not** have to match real-world clock time.

---

# Example

Suppose two writes:

```
Write A

↓

Write B
```

Everyone sees:

```
A

↓

B
```

The order is consistent for all observers.

---

# Linearizability

Linearizability is stronger than Sequential Consistency.

It guarantees:

```
Real Time

↓

Operation Order

↓

Preserved
```

If a write completes,

every later read must observe it.

---

# Timeline

```
Write Finishes

↓

Read Starts

↓

Must Return

New Value
```

This closely matches how most people naturally think systems should behave.

---

# Strong vs Linearizable

Strong Consistency is a broad idea.

Linearizability is a specific formal consistency model that provides real-time ordering guarantees.

Many distributed databases advertise linearizable reads or writes.

---

# Causal Consistency

Imagine:

```
Message 1

↓

"Let's Meet"
```

Then:

```
Message 2

↓

"Tomorrow"
```

Readers should never see:

```
Tomorrow

↓

Before

↓

Let's Meet
```

Causal relationships must be preserved.

---

# Example

User posts:

```
"I got promoted!"
```

Then comments:

```
"Finally!"
```

Everyone should see:

```
Post

↓

Comment
```

Not the reverse.

---

# Read-Your-Writes

Suppose:

You update:

```
Username

↓

Vinay
```

Immediately afterwards,

you refresh the page.

You expect:

```
Vinay
```

Not the previous value.

This guarantee is called:

```
Read-Your-Writes
```

---

# Monotonic Reads

Suppose first read returns:

```
Version 10
```

Your next read should never return:

```
Version 8
```

Data should move forward,

not backward.

---

# Monotonic Writes

Suppose one client performs:

```
Write A

↓

Write B
```

The system guarantees:

```
A

Before

B
```

Writes from the same client are applied in order.

---

# Consistency Model Comparison

| Model | Latest Data Guaranteed? | Notes |
|--------|-------------------------|-------|
| Strong | ✅ Yes | Every completed write is immediately visible |
| Linearizable | ✅ Yes | Preserves real-time ordering |
| Sequential | ⚠️ Ordered | Same operation order, not necessarily real-time |
| Causal | ⚠️ Related Updates | Preserves cause-and-effect relationships |
| Eventual | ❌ Not Immediately | Replicas converge over time |

---

# Banking Example

Balance:

```
₹10,000
```

Deposit:

```
₹5,000
```

Immediately after,

everyone must see:

```
₹15,000
```

Typically requires very strong consistency guarantees.

---

# Social Media Example

Like Count:

```
1000
```

Friend sees:

```
999
```

for two seconds.

Usually acceptable.

Eventual consistency works well.

---

# Chat Example

Message:

```
Hello
```

↓

```
How are you?
```

Users should not see:

```
How are you?

↓

Hello
```

Causal ordering is important.

---

# Google Docs

When multiple people edit simultaneously,

the system must carefully preserve user intent.

Modern collaborative systems combine multiple consistency techniques and conflict-resolution algorithms.

---

# Database Examples

Common examples (general tendencies):

| Database | Typical Consistency Characteristics |
|----------|-------------------------------------|
| etcd | Strong / Linearizable |
| CockroachDB | Strong / Serializable transactions |
| Google Spanner | Strong Global Consistency |
| Cassandra | Tunable, often Eventual |
| DynamoDB | Eventual or Strong (configurable for many reads) |
| MongoDB | Configurable depending on read/write concern |

The exact guarantees depend on configuration and operation type.

---

# Choosing the Right Model

Ask:

```
What Happens

If Users

Read Old Data?
```

If unacceptable:

```
Choose Stronger Consistency
```

If acceptable:

```
Eventual Consistency

May Be Better
```

---

# Complete Picture

```
Write

↓

Replication

↓

Consistency Model

↓

Read
```

The consistency model defines what readers are allowed to observe.

---

# Hands-on Thought Experiment

Imagine building:

1. Banking App
2. Instagram
3. WhatsApp
4. Online Shopping Cart

Ask:

```
How Fresh

Must

The Data Be?
```

The answer determines the appropriate consistency model.

---

# Interview Questions

## What is a consistency model?

A consistency model defines the guarantees a distributed system provides about when updates become visible to readers.

---

## What is Strong Consistency?

Every read returns the latest successfully written value.

---

## What is Eventual Consistency?

Replicas may temporarily differ, but eventually converge to the same value if no new updates occur.

---

## What is Linearizability?

A consistency model where completed operations appear to occur instantly at a single point in time while preserving real-time ordering.

---

## What is Sequential Consistency?

All clients observe operations in the same order, although that order does not have to match real-time execution.

---

## What is Causal Consistency?

Operations that are causally related are observed in the same order by all clients.

---

## What is Read-Your-Writes?

After a client performs a write, subsequent reads by that same client will observe the updated value.

---

# Summary

Consistency models define **how and when data becomes visible** after updates.

```
Write

↓

Replication

↓

Consistency Rules

↓

Read
```

Key concepts:

- Not all distributed systems provide the same consistency guarantees.
- Strong consistency always returns the latest value.
- Eventual consistency prioritizes scalability and availability.
- Linearizability preserves real-time ordering.
- Sequential consistency preserves a global order of operations.
- Causal consistency preserves cause-and-effect relationships.
- Read-Your-Writes and Monotonic Reads improve user experience.
- Different applications require different consistency models.

Understanding consistency models prepares you for one of the hardest problems in distributed systems:

> **How do multiple servers actually agree on the same value?**

That is the problem solved by **consensus algorithms**.

---

# Next Chapter

## Chapter 72 — Leader Election: How Distributed Systems Choose One Leader Among Many Servers

We'll explore:

- Why Leader Election Exists
- Split Brain
- Single Leader Architecture
- Leader Responsibilities
- Heartbeats
- Leader Failure Detection
- Leader Re-election
- ZooKeeper
- etcd
- Kubernetes Control Plane
- Complete Leader Election Process

> **Leader Election is the foundation of consensus systems like Raft and is used by Kubernetes, etcd, ZooKeeper, and many distributed databases.**