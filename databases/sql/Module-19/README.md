# 🗄️ SQL Mastery — 20+ Years Experience
# Module 19 — Replication

> This module moves from "one database instance" to "one database instance plus copies of it" — and the very real consistency problems that introduces.

---

# Chapter 107 — Why Replication? Read Scaling, Availability & Disaster Recovery

> Replication solves three genuinely different problems at once — and understanding which one you're actually solving for shapes every other decision in this module.

## Learning Objectives
- The three core motivations for replication
- Primary/replica topology basics
- Why replication is not a substitute for backups
- Replication as a foundation for other capabilities

## 1️⃣ The Basic Topology

```text
Primary
 ↓
Replica 1
Replica 2
Replica 3
```

The primary accepts writes; replicas receive a continuous stream of changes (commonly built on the WAL mechanism from Module 17) and apply them to maintain their own copy.

## 2️⃣ Motivation 1 — Read Scaling

```text
All reads → Primary
        ↓
Primary becomes the bottleneck for both reads and writes
```

```text
Reads spread across replicas
Writes still go to Primary
        ↓
Primary is freed up to focus on write throughput
```

## 3️⃣ Motivation 2 — High Availability

```text
Primary fails
        ↓
A replica can be promoted to become the new primary
   (Chapter 112 — Database Failover)
```

Without replication, a primary failure means the database is simply unavailable until it's repaired or restored from backup — likely a much longer outage.

## 4️⃣ Motivation 3 — Disaster Recovery

```text
Replica in a different data center / region
        ↓
Protects against a region-level failure,
   not just a single-machine failure
```

## 5️⃣ Replication Is Not a Backup

```text
Accidental DROP TABLE on Primary
        ↓
Replicated to replicas almost immediately
        ↓
Replicas ALSO lose the table
```

This is a critical, commonly misunderstood distinction: replication propagates mistakes just as faithfully as it propagates legitimate writes. Backups (Chapter 100) — ideally combined with WAL-based point-in-time recovery — are what protect against this kind of logical/human error, not replication.

## 🧠 Senior Mental Model

```text
Replication solves:
   "How do I survive losing one machine, and scale reads?"

Backup solves:
   "How do I survive losing (or corrupting) the DATA itself,
    including mistakes that get faithfully replicated?"

You need both. Neither substitutes for the other.
```

## 🧪 Hands-on Lab

**Lab 1** — Set up a primary with at least one replica in a test environment and confirm writes on the primary appear on the replica.
**Lab 2** — Simulate an accidental destructive operation on the primary and observe it propagate to the replica.
**Lab 3** — Discuss which of the three motivations (read scaling, HA, DR) applies to a specific real system you're familiar with, and why.

## 🎯 Interview Questions

**What are the three main motivations for database replication?**
Read scaling (offloading reads from the primary), high availability (surviving a primary failure via promotion), and disaster recovery (surviving a broader, region-level failure).

**Why is replication not a substitute for backups?**
Because replication faithfully propagates all changes, including destructive mistakes like an accidental DROP TABLE — a replica will replicate that mistake almost immediately, whereas a backup taken before the mistake remains a safe restore point.

## 📌 Summary

```text
Rule 1 — Replication scales reads, improves availability, and enables DR.
Rule 2 — Replicas receive a continuous stream of changes from the primary.
Rule 3 — Replication propagates mistakes just as faithfully as legitimate writes.
Rule 4 — Backups and replication solve different problems — you need both.
```

---

# Chapter 108 — Primary / Replica: Write and Read Routing

> The simplest replication topology, and the foundation everything else in this module builds on.

## Learning Objectives
- Basic write/read routing rules
- Why writes can't simply go to any replica
- Application-level vs proxy-level routing
- The risks of routing writes incorrectly

## 1️⃣ The Basic Rule

```text
Write → Primary

Read → Replicas (or Primary, when appropriate)
```

## 2️⃣ Why Writes Must Go to the Primary

```text
Replica
 ↓
Receives changes FROM the primary
 ↓
Is not itself the source of truth for new writes
```

In a standard primary/replica setup, a replica is a downstream consumer of changes, not an independent place to originate them — attempting to write directly to a replica typically either fails outright or, in misconfigured setups, creates serious consistency problems.

## 3️⃣ Where Routing Decisions Get Made

```text
Application-level routing
 ↓
The application code itself decides:
   "this query is a write → primary connection"
   "this query is a read → replica connection"

Proxy-level routing
 ↓
A database proxy (Chapter 165) sits between application
   and database, inspecting queries and routing automatically
```

Proxy-level routing centralizes this logic and avoids scattering write/read decisions throughout application code, but adds its own operational complexity (health checks, failover awareness).

## 4️⃣ The Risk of Misrouted Writes

```text
Application accidentally sends a write to a replica connection
        ↓
Fails (read-only replica) — the "safe" failure mode

OR, in some misconfigured/unusual topologies:
        ↓
Creates data that diverges from the primary
        ↓
Serious, hard-to-detect consistency problems
```

Correct read/write routing is a foundational correctness requirement, not just a performance optimization — this deserves careful testing, not just trust in configuration.

## 🧪 Hands-on Lab

**Lab 1** — Attempt a write against a replica connection in a test environment and observe the failure behavior.
**Lab 2** — Design (on paper or in code) simple application-level routing logic that distinguishes reads from writes.
**Lab 3** — If a database proxy is available, configure it to route reads and writes automatically and verify behavior under both cases.

## 🎯 Interview Questions

**Why can't writes simply be sent to any replica in a standard primary/replica setup?**
Because a replica is a downstream consumer of the primary's changes, not an independent source of truth — it isn't designed to originate new writes in this topology.

**What's the difference between application-level and proxy-level read/write routing?**
Application-level routing means the application code itself decides which connection to use for a given query; proxy-level routing centralizes that decision in an intermediary that inspects and routes queries automatically.

## 📌 Summary

```text
Rule 1 — Writes go to the primary; reads can be spread across replicas.
Rule 2 — Replicas are downstream consumers of change, not independent write targets.
Rule 3 — Routing can be handled at the application level or via a proxy.
Rule 4 — Correct routing is a correctness requirement, not just a performance choice.
```

---

# Chapter 109 — Replication Lag: The Classic Distributed-Systems Problem

> The moment you have more than one copy of your data, you inherit the fundamental question every distributed system must answer: what happens in the gap before all copies agree?

## Learning Objectives
- What replication lag actually is
- Why it's unavoidable, not a bug
- Common causes of lag spikes
- Real-world symptoms of lag
- Monitoring lag properly

## 1️⃣ The Scenario

```text
User updates profile
        ↓
Primary
        ↓
Replica  (not yet caught up)
```

```text
User immediately reads (routed to the replica)
        ↓
Old data
```

## 2️⃣ Why Lag Is Unavoidable, Not a Bug

```text
Change must:
   1. Happen on the primary
   2. Be transmitted to the replica (network time)
   3. Be applied on the replica (apply time)
```

Even a perfectly healthy, well-configured replication setup has *some* nonzero lag — the question is how much, and how consistently, not whether it's zero.

## 3️⃣ Common Causes of Lag Spikes

```text
Network issues between primary and replica
Replica under heavy read load, competing for resources
Large/bulk write operations on the primary
   generating a burst of changes to apply
Replica hardware weaker than the primary
Long-running queries on the replica blocking apply progress
   (depending on the specific replication technology)
```

## 4️⃣ Real-World Symptom: The "Where Did My Data Go?" Bug Report

```text
User submits a form
        ↓
Redirected to a page that reads from a replica
        ↓
Replica hasn't caught up yet
        ↓
User sees their own submission missing
        ↓
Confusing, alarming bug report — "my data disappeared!"
```

This is one of the most common real production symptoms of replication lag, and it directly motivates Chapter 111 (Read-After-Write Consistency).

## 5️⃣ Monitoring Lag Properly

```text
Track:
   Time-based lag (how far behind, in seconds)
   Byte/position-based lag (how much data hasn't been applied)
```

Both matter — a replica could be "1 second behind" in time but represent a huge, urgent volume of unapplied changes if the primary is under heavy write load; conversely a small volume of unapplied changes might still represent many seconds of lag during a quiet period.

## 🧠 Senior Mental Model

```text
Replication lag isn't a defect to eliminate.
It's a property to measure, bound, and design around.
```

## 🧪 Hands-on Lab

**Lab 1** — Write to a primary and immediately query the replica in a tight loop, measuring how long it takes for the change to appear.
**Lab 2** — Generate a large bulk write on the primary and observe replica lag increase during and after.
**Lab 3** — Identify (or set up) monitoring for both time-based and position-based replication lag in your environment.

## 🎯 Interview Questions

**Why is replication lag considered unavoidable rather than a bug to fix entirely?**
Because a change must occur on the primary, be transmitted over the network, and be applied on the replica — each step takes some nonzero time, so some lag is inherent to the architecture, even when healthy.

**What's a common real-world symptom of replication lag that causes confusing user-facing bugs?**
A user submits data, is then routed to read from a replica that hasn't caught up yet, and sees their own just-submitted data missing — appearing as if it was lost.

**Why should you monitor both time-based and volume-based replication lag?**
Because a small time-based lag can still represent a large, urgent volume of unapplied changes under heavy write load, while a small volume of unapplied changes could still represent significant time-based lag during quieter periods — each metric tells you something the other doesn't.

## 📌 Summary

```text
Rule 1 — Some replication lag is inherent, not necessarily a misconfiguration.
Rule 2 — Network issues, replica load, and bulk writes commonly spike lag.
Rule 3 — Lag directly causes real, confusing "my data disappeared" user reports.
Rule 4 — Monitor both time-based and volume-based lag.
```

---

# Chapter 110 — Synchronous vs Asynchronous Replication

> Whether a write "counts" as done the instant it's on the primary, or only once a replica confirms it too, is a fundamental trade-off between consistency, latency, and availability.

## Learning Objectives
- Synchronous vs asynchronous replication defined
- The latency cost of synchronous replication
- The data-loss risk of asynchronous replication
- Semi-synchronous as a middle ground
- Choosing based on workload requirements

## 1️⃣ Asynchronous Replication

```text
Write committed on Primary
        ↓
Acknowledged to the client IMMEDIATELY
        ↓
Change sent to replica(s) afterward, independently
```

```text
Trade-off:
   Lower write latency
   But: if the primary fails before the replica received
        the change, that change can be lost
```

## 2️⃣ Synchronous Replication

```text
Write committed on Primary
        ↓
Primary WAITS for at least one replica to confirm
   it has received (and possibly applied) the change
        ↓
THEN acknowledged to the client
```

```text
Trade-off:
   Stronger durability guarantee — the change survives
      even if the primary fails immediately after
   But: higher write latency (must wait for network round-trip
      plus replica processing)
   And: if the replica is unreachable, writes can stall entirely
      unless carefully designed around
```

## 3️⃣ Semi-Synchronous — A Common Middle Ground

```text
Primary waits for confirmation from AT LEAST ONE replica
   (not all of them)
        ↓
Balances some durability improvement
   against not fully blocking on every single replica
```

Exact terminology and guarantees vary significantly by database — always verify precisely what "synchronous" means for your specific system rather than assuming.

## 4️⃣ Choosing Based on Workload

```text
Financial transactions, critical writes
 ↓
May justify synchronous replication's latency cost
   for the stronger durability guarantee

High-volume, latency-sensitive writes where occasional
   loss of the very latest write is tolerable
 ↓
Asynchronous replication is usually the practical choice
```

## 🧠 Senior Mental Model

```text
Synchronous replication
 ↓
"I won't tell you it's done until I'm sure it can survive
 losing the primary right now."

Asynchronous replication
 ↓
"It's done as far as the primary is concerned —
 replicas will catch up shortly, usually."
```

## 🧪 Hands-on Lab

**Lab 1** — If your database supports it, configure asynchronous replication and measure write latency.
**Lab 2** — Reconfigure to synchronous (or semi-synchronous) and compare write latency under the same workload.
**Lab 3** — Simulate a replica becoming unreachable under synchronous replication and observe the effect on primary writes.

## 🎯 Interview Questions

**What's the core trade-off between synchronous and asynchronous replication?**
Synchronous replication offers stronger durability (a write is confirmed only after at least one replica has it) at the cost of higher write latency and potential stalls if a replica is unreachable; asynchronous replication offers lower latency but risks losing the most recent writes if the primary fails before replicating them.

**Why might a system use semi-synchronous replication instead of fully synchronous?**
To gain some improved durability from at least one replica confirming the write, without paying the full latency and availability cost of waiting on every single replica.

**How would you decide which replication mode to use for a given workload?**
Based on how much the workload can tolerate losing the very latest writes on a primary failure — critical, high-value writes may justify synchronous replication's cost, while high-volume, latency-sensitive writes usually favor asynchronous.

## 📌 Summary

```text
Rule 1 — Asynchronous replication: lower latency, possible data loss on failure.
Rule 2 — Synchronous replication: stronger durability, higher latency, possible stalls.
Rule 3 — Semi-synchronous is a common middle ground — verify exact guarantees per database.
Rule 4 — The right choice depends on the workload's tolerance for losing recent writes.
```

---

# Chapter 111 — Read-After-Write Consistency: Making Sure Users See Their Own Writes

> This chapter directly solves the "my data disappeared" problem from Chapter 109 — a set of concrete techniques for ensuring users never see a version of the world that's older than what they just wrote.

## Learning Objectives
- What read-after-write consistency guarantees
- Routing recent reads to the primary
- Session-based consistency techniques
- Lag-aware read routing
- Trade-offs of each technique

## 1️⃣ The Problem, Restated

```text
User writes
        ↓
User immediately reads, routed to a lagging replica
        ↓
Sees stale/missing data
        ↓
Confusing, broken-feeling user experience
```

## 2️⃣ Technique 1 — Route Recent Reads to the Primary

```text
After a write, for THIS user/session,
   route subsequent reads to the primary
   for some short window of time
```

```text
Simple and reliable
But: partially defeats the purpose of read replicas
   for that specific user during that window
```

## 3️⃣ Technique 2 — Session Consistency via Tracked Position

```text
After a write, record the replication position
   (e.g., a log sequence number) associated with that write
        ↓
When this session reads from a replica,
   only use a replica that has caught up to AT LEAST
   that position
        ↓
Otherwise, wait briefly, or fall back to the primary
```

This is more precise than blanket "route to primary for N seconds," but requires replication-position awareness in the application or proxy layer.

## 4️⃣ Technique 3 — Lag-Aware Load Balancing

```text
Proxy/load balancer monitors each replica's current lag
        ↓
Avoids routing reads to replicas currently lagging
   beyond an acceptable threshold
```

This improves general staleness across all users, though it doesn't guarantee a *specific* user always sees their *own* specific write unless combined with session-based tracking.

## 5️⃣ Choosing a Technique

```text
Strict per-user guarantee needed
   (e.g., "user must see their own comment immediately")
 ↓
Session-based / position-tracked routing

General staleness reduction across the whole system
   acceptable
 ↓
Lag-aware load balancing alone may be sufficient

Simplicity over precision
 ↓
Route-to-primary-briefly-after-write
```

## 🧠 Senior Mental Model

```text
Read-after-write consistency isn't automatic —
it requires a deliberate architectural decision
about where "recent" reads for a given user should go.
```

## 🧪 Hands-on Lab

**Lab 1** — Reproduce the "data disappeared" scenario deliberately (write to primary, immediately read from a deliberately lagging replica).
**Lab 2** — Implement a simple "route to primary for N seconds after a write" rule and confirm it resolves the scenario.
**Lab 3** — If your replication technology exposes a position/log-sequence-number, design (on paper) a session-consistency scheme using it.

## 🎯 Interview Questions

**What does read-after-write consistency guarantee?**
That a user who just performed a write will see that write reflected in their subsequent reads, rather than potentially seeing stale data from a replica that hasn't caught up yet.

**Name two different techniques for achieving read-after-write consistency, and a trade-off of each.**
(1) Routing recent reads to the primary after a write — simple but reduces the benefit of read replicas for that user temporarily. (2) Session-based consistency using a tracked replication position — more precise, but requires replication-position awareness in the application or proxy layer.

**Why doesn't lag-aware load balancing alone fully solve read-after-write consistency?**
Because it reduces general staleness across the system but doesn't specifically guarantee that a particular user will see their own specific just-made write, unless combined with per-session tracking.

## 📌 Module 19 Critical Rules

```text
Rule 1  — Replication solves read scaling, availability, and disaster recovery.
Rule 2  — Replication is not a substitute for backups — it propagates mistakes too.
Rule 3  — Writes go to the primary; replicas are downstream consumers of change.
Rule 4  — Some replication lag is inherent to the architecture, not a bug.
Rule 5  — Monitor both time-based and volume-based lag.
Rule 6  — Synchronous replication trades write latency for stronger durability.
Rule 7  — Asynchronous replication trades some durability for lower latency.
Rule 8  — Read-after-write consistency requires a deliberate routing strategy.
Rule 9  — Different consistency techniques trade off precision vs. simplicity.
Rule 10 — Replication topology decisions should be driven by workload requirements.
```

---

# 🎉 Module 19 Complete

You now understand how a single database becomes a resilient, read-scalable system — and the very real consistency questions that introduces:

```text
Why Replication? (scaling, HA, DR)
 ↓
Primary / Replica Routing
 ↓
Replication Lag (inherent, must be measured)
 ↓
Sync vs Async (the core durability/latency trade-off)
 ↓
Read-After-Write Consistency (making it feel correct to users)
```

# 🚀 Next Module

# Module 20 — High Availability
Database failover, leader election, split-brain scenarios, and quorum-based decision making in distributed database systems.