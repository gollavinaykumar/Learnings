Suppose you have three servers.

```
Server A

Server B

Server C
```

A client sends:

```
Transfer

₹5,000
```

Question:

How can all three servers guarantee they record **exactly the same transaction** in **exactly the same order**, even if:

- One server crashes?
- The network becomes slow?
- Messages arrive late?
- The leader changes?

If one server stores:

```
A

↓

B

↓

C
```

while another stores:

```
A

↓

C

↓

B
```

The databases become inconsistent.

Distributed systems need a way for **every server to agree on one history**.

This problem is called:

# Consensus

One of the most popular consensus algorithms today is:

# Raft

Raft is used by:

- Kubernetes (etcd)
- Consul
- TiKV
- CockroachDB (Raft replication)
- RethinkDB
- Many cloud-native systems

After this chapter, you'll understand how multiple servers agree on exactly the same sequence of operations.

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 73 — Raft Consensus Algorithm: How Distributed Systems Agree on the Same Data

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Consensus Exists
> - What is Raft?
> - Leader, Followers & Candidates
> - Terms
> - Log Replication
> - Commit Index
> - Leader Election
> - Heartbeats
> - Safety Guarantees
> - Kubernetes & etcd
> - Complete Raft Flow

---

# 📖 Why Consensus Exists

Imagine:

```
3 Servers
```

Client sends:

```
Write

↓

Balance = ₹15,000
```

Question:

How do all servers store:

```
Exactly

The Same

History?
```

Consensus solves this problem.

---

# What is Consensus?

Consensus means:

```
Multiple Servers

↓

Agree

↓

One Decision
```

Every node eventually reaches the same agreed state.

---

# Think of a Classroom

Teacher asks:

```
Who Won?

↓

Vote

↓

Everyone Agrees

↓

Final Answer
```

Consensus works similarly.

---

# Raft Architecture

Raft has three possible roles.

```
Leader
```

```
Follower
```

```
Candidate
```

Every server is always in one of these states.

---

# Leader

The leader is responsible for:

✔ Accepting writes

✔ Replicating logs

✔ Sending heartbeats

✔ Coordinating the cluster

Only one leader exists per term.

---

# Followers

Followers do not accept normal client writes.

They:

✔ Receive replicated logs

✔ Respond to leader requests

✔ Vote during elections

✔ Monitor heartbeats

---

# Candidate

If a follower stops receiving heartbeats:

```
Follower

↓

Candidate
```

It starts an election.

If it wins,

it becomes the new leader.

---

# Terms

Raft divides time into:

```
Term 1

↓

Term 2

↓

Term 3
```

Each election starts a new term.

Every message includes its term number.

---

# Why Terms Matter

Suppose:

```
Old Leader

↓

Term 5
```

New leader:

```
Term 6
```

Servers immediately recognize:

```
Term 6

>

Term 5
```

The newer leader is authoritative.

---

# Log

Raft stores operations inside a log.

Example:

```
Index

1

Create User
```

```
2

Deposit ₹1000
```

```
3

Transfer ₹500
```

The log is the source of truth.

---

# Log Replication

Client sends:

```
Deposit ₹500
```

Leader appends it:

```
Leader Log

↓

Entry Added
```

Then sends it to followers.

---

# Replication Flow

```
Client

↓

Leader

↓

Follower A

Follower B
```

Everyone receives the same log entry.

---

# AppendEntries RPC

The leader communicates with followers using:

```
AppendEntries
```

This RPC serves two purposes:

✔ Replicate log entries

✔ Send heartbeats

---

# Heartbeats

Even when there are no writes,

leader periodically sends:

```
AppendEntries

↓

Empty Log

↓

Heartbeat
```

Followers know the leader is alive.

---

# Acknowledgement

Follower replies:

```
Entry Received
```

Leader counts acknowledgements.

---

# Commit Index

Suppose cluster has:

```
5 Servers
```

Leader receives acknowledgements from:

```
3 Servers
```

Majority reached.

Entry becomes:

```
Committed
```

---

# Visual Representation

```
Leader

↓

Entry

↓

Followers

↓

Majority

↓

Commit
```

Now the write is considered durable.

---

# Why Majority?

Imagine:

```
5 Servers
```

Only:

```
2 Servers
```

receive the write.

Leader crashes.

Remaining servers never saw the update.

Using a majority prevents this inconsistency.

---

# Applying the Log

Once committed:

```
Log Entry

↓

State Machine

↓

Database Updated
```

Applications observe the new state.

---

# Example

Client:

```
Transfer ₹1000
```

Leader:

```
Log Index 15
```

Followers replicate:

```
Index 15
```

Majority acknowledges.

Commit.

Every server applies:

```
Transfer ₹1000
```

in the same order.

---

# Leader Failure

Leader crashes.

Followers stop receiving heartbeats.

```
Follower

↓

Candidate

↓

Election

↓

New Leader
```

The new leader continues using the replicated log.

---

# Log Matching Property

If two servers contain:

```
Log Entry 25
```

with the same term,

Raft guarantees:

```
Everything Before

Entry 25

Is Identical
```

This is one of Raft's key safety properties.

---

# Leader Completeness

If an entry is committed,

every future leader must contain that entry.

Committed history is never lost.

---

# Safety Guarantee

Raft guarantees:

```
Committed Entry

↓

Never Lost
```

Even if leaders change.

---

# Complete Raft Flow

```
Client

↓

Leader

↓

Append Log

↓

Replicate

↓

Majority

↓

Commit

↓

Apply

↓

Reply Client
```

---

# Kubernetes Example

Kubernetes stores cluster state inside:

```
etcd
```

etcd uses:

```
Raft
```

Every Kubernetes object:

- Pods
- Deployments
- Services
- Secrets
- ConfigMaps

is replicated using Raft.

---

# Consul Example

Consul stores service discovery information using Raft.

Every node agrees on the same cluster state.

---

# CockroachDB Example

CockroachDB replicates data using Raft.

Each data range has its own Raft group.

---

# Why Raft Became Popular

Older consensus algorithm:

```
Paxos
```

Very powerful,

but difficult to understand.

Raft was designed to be:

✔ Easier to learn

✔ Easier to implement

✔ Easier to verify

while providing similar safety guarantees.

---

# Hands-on Thought Experiment

Imagine:

```
Leader

↓

Accepts Write

↓

Crashes

↓

Before Majority
```

Question:

Should the write survive?

Answer:

No.

It was never committed.

Now imagine:

```
Leader

↓

Majority

↓

Commit

↓

Crash
```

The write survives because Raft guarantees committed entries are preserved.

---

# Interview Questions

## What is consensus?

Consensus is the process by which distributed systems agree on a single sequence of operations.

---

## What is Raft?

Raft is a distributed consensus algorithm that provides leader election, log replication, and fault tolerance.

---

## What are the three Raft roles?

- Leader
- Follower
- Candidate

---

## What is a term?

A term is a logical period of leadership. Each election begins a new term.

---

## What is log replication?

The leader copies log entries to followers so every node maintains the same ordered history.

---

## What is the commit index?

The commit index identifies log entries that have been safely replicated to a majority and can be applied to the state machine.

---

## Why is a majority required?

A majority ensures committed data survives leader failures and prevents conflicting histories.

---

## Which Kubernetes component uses Raft?

etcd.

---

# Summary

Raft enables multiple servers to agree on one consistent history.

```
Client

↓

Leader

↓

Log

↓

Followers

↓

Majority

↓

Commit

↓

State Machine
```

Key concepts:

- Consensus ensures every server agrees on the same sequence of operations.
- Raft uses Leaders, Followers, and Candidates.
- Time is divided into Terms.
- Leaders replicate log entries using AppendEntries RPCs.
- Entries become committed only after reaching a majority.
- Committed entries are never lost.
- Kubernetes, etcd, Consul, and many distributed databases rely on Raft.

Raft answers one of the hardest questions in distributed systems:

> **How can many independent servers behave like one reliable system?**

---

# Next Chapter

## Chapter 74 — Service Discovery: How Microservices Find Each Other Automatically

We'll explore:

- Why Service Discovery Exists
- Static vs Dynamic Addresses
- Service Registry
- Client-side Discovery
- Server-side Discovery
- DNS-based Discovery
- Consul
- etcd
- Eureka
- Kubernetes Services
- Complete Service Discovery Architecture

> **This chapter explains how thousands of microservices automatically locate each other without hardcoding IP addresses, forming the foundation of modern cloud-native applications.**