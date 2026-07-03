Imagine you have three database servers.

```
Server A
```

```
Server B
```

```
Server C
```

A client wants to update a user's balance.

Question:

```
Which Server

Should Accept

The Write?
```

If all three servers accept writes independently...

```
Server A

↓

₹15,000
```

```
Server B

↓

₹20,000
```

```
Server C

↓

₹12,000
```

Now every server contains different data.

Chaos.

Distributed systems solve this problem by choosing:

# One Leader

Only one server is allowed to coordinate important operations.

Everyone else follows it.

This process is called:

# Leader Election

Almost every major distributed system uses it.

Examples:

- Kubernetes (etcd)
- ZooKeeper
- Kafka
- Elasticsearch
- Consul
- MongoDB Replica Sets
- CockroachDB
- TiDB

After this chapter, you'll understand how distributed systems decide **who is in charge.**

---

# 👑 Linux Kernel, Networking & Distributed Systems Mastery

# Module 12 — Distributed Systems & Cloud Networking

# Chapter 72 — Leader Election: How Distributed Systems Choose One Leader Among Many Servers

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Why Leader Election Exists
> - Split Brain
> - Leader & Followers
> - Leader Responsibilities
> - Heartbeats
> - Failure Detection
> - Leader Re-election
> - Quorum
> - Kubernetes Example
> - Complete Election Flow

---

# 📖 Why Do We Need a Leader?

Suppose we have:

```
Server A

Server B

Server C
```

Clients can send requests to any server.

If all servers modify data independently,

eventually their data diverges.

Distributed systems need one authority.

---

# Single Leader Architecture

```
            Leader

          Server A

         /        \

Follower B    Follower C
```

Only the leader accepts writes.

Followers replicate data.

---

# Responsibilities of the Leader

The leader typically performs:

✔ Accept client writes

✔ Replicate data

✔ Coordinate cluster state

✔ Resolve conflicts

✔ Send heartbeats

Followers mainly:

✔ Replicate

✔ Serve reads (depending on system)

✔ Monitor the leader

---

# Write Flow

```
Client

↓

Leader

↓

Followers

↓

Acknowledgement

↓

Client
```

All important writes go through the leader.

---

# Why Not Multiple Leaders?

Imagine:

```
Leader A

↓

Write

↓

₹15,000
```

At the same time:

```
Leader B

↓

Write

↓

₹20,000
```

Which value is correct?

No one knows.

---

# Split Brain

The worst situation is:

```
Leader A

×

Leader B
```

Both believe:

```
"I Am The Leader"
```

This is called:

# Split Brain

It can corrupt data.

---

# Visual Representation

```
Network Failure

↓

Cluster Splits

↓

Leader A

Leader B

↓

Both Accept Writes
```

Disaster.

---

# Heartbeats

How do followers know the leader is alive?

Leader periodically sends:

```
Heartbeat

↓

"I'm Alive"
```

Followers reset their timers whenever they receive one.

---

# Heartbeat Flow

```
Leader

↓

Heartbeat

↓

Follower

↓

Heartbeat

↓

Follower
```

Repeated continuously.

---

# Failure Detection

Suppose heartbeats stop.

Follower waits:

```
Timeout
```

Still nothing.

Follower concludes:

```
Leader Failed
```

Election begins.

---

# Election Timeout

Every follower maintains:

```
Election Timer
```

If no heartbeat arrives before the timer expires,

it starts an election.

---

# Why Random Timeouts?

Imagine:

```
Follower B

↓

Election
```

```
Follower C

↓

Election
```

At exactly the same moment.

Both may vote for themselves.

No leader is chosen.

To reduce this,

systems typically choose **random election timeouts**.

---

# Election Begins

Follower becomes:

```
Candidate
```

Candidate requests votes.

```
Candidate

↓

Vote Please
```

Other servers respond.

---

# Voting

Example:

```
Server B

↓

Vote

↓

Yes
```

```
Server C

↓

Vote

↓

Yes
```

Candidate collects votes.

---

# Quorum

Leader cannot be chosen by one server alone.

It needs:

```
Majority
```

Example:

```
5 Servers

↓

Need

3 Votes
```

This majority is called a:

```
Quorum
```

---

# Why Majority?

Suppose:

```
5 Servers
```

Two fail.

Remaining:

```
3 Servers
```

Still enough to form:

```
Majority
```

The system continues operating.

---

# Example

Cluster:

```
A

B

C
```

Votes:

```
A

↓

B

↓

Leader
```

Majority:

```
2

of

3
```

---

# New Leader

Once majority is reached:

```
Candidate

↓

Leader
```

Immediately begins sending:

```
Heartbeats
```

Followers stop their elections.

---

# Leader Failure

Timeline:

```
Leader

↓

Heartbeat

↓

Heartbeat

↓

Crash
```

Followers wait.

Timeout expires.

Election begins.

New leader chosen.

Cluster continues.

---

# Complete Election Flow

```
Leader Alive

↓

Heartbeats

↓

Leader Fails

↓

Timeout

↓

Candidate

↓

Vote Request

↓

Majority

↓

New Leader
```

---

# Kubernetes Example

Kubernetes stores cluster state in:

```
etcd
```

etcd uses:

```
Raft Consensus
```

Raft includes leader election.

Only the elected leader coordinates writes.

---

# Kafka Example

Kafka partitions also have leaders.

```
Producer

↓

Leader Replica

↓

Follower Replicas
```

Followers replicate data from the leader.

---

# ZooKeeper Example

ZooKeeper elects a leader to coordinate updates and maintain cluster consistency.

---

# MongoDB Example

MongoDB Replica Sets elect:

```
Primary
```

The primary accepts writes.

Secondaries replicate.

If the primary fails,

a new primary is elected.

---

# Why Leader Election Matters

Without it:

```
Multiple Writers

↓

Conflicts

↓

Data Corruption
```

Leader election prevents conflicting updates.

---

# Complete Cluster

```
Clients

↓

Leader

↓

Replication

↓

Followers
```

Simple architecture.

Reliable coordination.

---

# Hands-on Thought Experiment

Imagine:

```
Leader

↓

Power Failure
```

Ask yourself:

- Who notices?
- How long should they wait?
- Who becomes the new leader?
- How many votes are required?

These questions are exactly what consensus algorithms answer.

---

# Interview Questions

## What is Leader Election?

Leader Election is the process by which distributed systems choose one node to coordinate important operations.

---

## Why do distributed systems need a leader?

To prevent conflicting writes and coordinate shared state.

---

## What is Split Brain?

Split Brain occurs when multiple nodes incorrectly believe they are the leader at the same time.

---

## What are heartbeats?

Periodic messages sent by the leader to inform followers that it is still alive.

---

## What happens when heartbeats stop?

Followers wait for an election timeout and then begin a leader election.

---

## What is a quorum?

A quorum is the minimum majority of nodes required to make decisions such as electing a leader.

---

## Does Kubernetes use Leader Election?

Yes.

The Kubernetes control plane relies on etcd, which uses the Raft consensus algorithm and leader election.

---

# Summary

Leader Election ensures only one node coordinates important operations in a distributed system.

```
Leader

↓

Heartbeats

↓

Failure

↓

Election

↓

Votes

↓

Majority

↓

New Leader
```

Key concepts:

- One leader coordinates writes.
- Followers monitor the leader using heartbeats.
- Random election timeouts reduce vote collisions.
- A quorum (majority) is required to elect a leader.
- Split Brain is dangerous because multiple leaders can create conflicting updates.
- Leader election is used by Kubernetes, Kafka, MongoDB, ZooKeeper, etcd, and many distributed databases.

Leader Election answers:

> **Who should be in charge?**

The next question is even more important:

> **How do all servers agree on exactly the same sequence of operations?**

That is the job of a **consensus algorithm**.

---

# Next Chapter

## Chapter 73 — Raft Consensus Algorithm: How Distributed Systems Agree on the Same Data

We'll explore:

- Why Consensus Exists
- What is Consensus?
- Raft Architecture
- Leader, Followers, and Candidates
- Terms
- Log Replication
- Commit Index
- Heartbeats
- Leader Election in Raft
- Safety Guarantees
- Kubernetes & etcd
- Complete Consensus Flow

> **Raft is one of the most important distributed systems algorithms. Understanding it gives you a deep understanding of Kubernetes, etcd, CockroachDB, TiKV, Consul, and many modern distributed databases.**