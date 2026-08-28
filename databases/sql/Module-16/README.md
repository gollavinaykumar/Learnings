# 🗄️ SQL Mastery — 20+ Years Experience
# Module 16 — MVCC & Isolation

> This module explains how databases let many transactions read and write concurrently without constantly blocking each other — and the subtle correctness trade-offs that come with each isolation level.

---

# Chapter 91 — MVCC: Multi-Version Concurrency Control

> Instead of forcing readers to wait for writers (and vice versa), most modern relational databases let multiple *versions* of a row coexist, so readers can see a consistent snapshot without blocking writers.

## Learning Objectives
- What MVCC solves
- Row versions
- How readers and writers avoid blocking each other
- Version visibility rules
- The storage cost of MVCC

## 1️⃣ The Problem MVCC Solves

```text
Without MVCC (naive locking model)

Transaction A reads a row
        ↓
Transaction B wants to update it
        ↓
B must wait for A to finish reading
```

At scale, this creates constant contention between readers and writers — even when a reader only wants a consistent snapshot, not to block anyone.

## 2️⃣ The MVCC Idea

```text
Row Version 1  (created by Transaction X)
      ↓
Row Version 2  (created by Transaction Y's UPDATE)
      ↓
Row Version 3  (created by Transaction Z's UPDATE)
```

Instead of overwriting a row in place and forcing readers to wait, the database can keep multiple versions and let each transaction see the version appropriate to its own snapshot.

## 3️⃣ Readers Don't Block Writers (Generally)

```text
Transaction A (long-running SELECT)
        ↓
sees Version 2 (its snapshot)

Transaction B (concurrent UPDATE)
        ↓
creates Version 3

Both proceed without blocking each other
```

This is one of MVCC's biggest practical wins: read-heavy and write-heavy workloads can coexist far more gracefully than under strict locking alone.

## 4️⃣ Version Visibility

```text
Which version should THIS transaction see?
```

Answered by comparing the row version's creation/expiration information against the transaction's own snapshot — the specific mechanism (transaction IDs, timestamps, etc.) varies by database engine.

## 5️⃣ The Cost of MVCC

```text
Multiple versions of the same logical row
 ↓
More storage
 ↓
Old versions must eventually be cleaned up
   (Chapter 94 — Vacuum / Garbage Collection)
```

MVCC isn't free — it trades some storage and cleanup overhead for dramatically better concurrency behavior.

## 🧠 Senior Mental Model

```text
Locking-only model
 ↓
"Readers and writers fight over the same single copy."

MVCC model
 ↓
"Readers see a consistent version;
 writers create new versions;
 old versions are cleaned up later."
```

## 🧪 Hands-on Lab

**Lab 1** — Start a long-running transaction that reads a row, then in a separate session update that same row and commit. Confirm the first transaction still sees its original snapshot value.
**Lab 2** — Research how your specific database tracks row version visibility (transaction IDs, timestamps, etc.).
**Lab 3** — Compare table storage size before and after a large batch of updates, tying the growth back to row versioning.

## 🎯 Interview Questions

**What problem does MVCC solve?**
It allows readers and writers to operate concurrently without constantly blocking each other, by letting each transaction see a consistent snapshot via row versions rather than forcing serialized access to a single copy of each row.

**What's the trade-off of using MVCC?**
Increased storage usage from multiple row versions, and the need for ongoing cleanup (garbage collection) of versions no longer visible to any active transaction.

## 📌 Summary

```text
Rule 1 — MVCC lets transactions see consistent snapshots via row versions.
Rule 2 — Readers generally don't block writers, and vice versa, under MVCC.
Rule 3 — Multiple row versions cost extra storage.
Rule 4 — Old versions must eventually be cleaned up.
```

---

# Chapter 92 — Isolation Levels: Read Uncommitted, Read Committed, Repeatable Read & Serializable

> Isolation levels are a contract: how much "interference" from other concurrent transactions are you willing to tolerate, in exchange for how much concurrency?

## Learning Objectives
- The four standard isolation levels
- Which anomalies each level permits or prevents
- Why stricter isolation costs performance
- Default isolation levels differ by database
- Choosing an isolation level deliberately

## 1️⃣ The Anomaly Ladder

```text
Dirty Read
 ↓
Reading another transaction's uncommitted changes

Non-Repeatable Read
 ↓
Re-reading the same row within one transaction
returns different data because another transaction committed a change

Phantom Read
 ↓
Re-running the same query within one transaction
returns different ROWS (new matching rows appeared)
```

## 2️⃣ Read Uncommitted

```text
Permits: Dirty Reads, Non-Repeatable Reads, Phantom Reads
```

You might read data that another transaction later rolls back — meaning you saw something that, from the database's perspective, never actually happened. Rarely used in serious production systems; some databases don't even implement it distinctly from Read Committed.

## 3️⃣ Read Committed

```text
Prevents: Dirty Reads
Permits: Non-Repeatable Reads, Phantom Reads
```

You only ever see committed data — but if you query the same row twice within one transaction, you might see two different committed values, because another transaction committed a change in between.

This is the default isolation level for many popular relational databases.

## 4️⃣ Repeatable Read

```text
Prevents: Dirty Reads, Non-Repeatable Reads
Permits (in the standard definition): Phantom Reads
```

Within a transaction, re-reading the same row is guaranteed to return the same value. New rows matching a broader query, however, may still appear depending on the exact database's implementation (some databases' Repeatable Read implementations also prevent phantoms in practice, even though the SQL standard doesn't strictly require it at this level).

## 5️⃣ Serializable

```text
Prevents: Dirty Reads, Non-Repeatable Reads, Phantom Reads
```

The strongest level — transactions behave as though they executed one at a time, in some serial order, even though they actually ran concurrently. This is the strongest guarantee, and also generally the most expensive in terms of concurrency (more blocking, more retries on conflict, or more overhead depending on implementation).

## 6️⃣ Why You Don't Always Want Serializable

```text
Stronger isolation
 ↓
Fewer anomalies
 ↓
But more blocking / more retries / less throughput
```

```text
Weaker isolation
 ↓
More throughput, less blocking
 ↓
But more anomalies possible
```

Choosing an isolation level is a deliberate trade-off based on what your application actually needs — not a "always pick the strongest" default.

## 7️⃣ Defaults Differ by Database

```text
Different relational databases ship with different
DEFAULT isolation levels.
```

Never assume — check your specific database's default, and set it explicitly for transactions where correctness genuinely depends on a particular level.

## 🧠 Senior Mental Model

```text
Ask, for each transaction:

Can I tolerate seeing slightly stale committed data?
Do I need every read within this transaction to be consistent?
Do I need to prevent new rows from appearing mid-transaction?
Is the performance cost of stronger isolation worth it here?
```

## 🧪 Hands-on Lab

**Lab 1** — Run two concurrent transactions under Read Committed; from one, update and commit a row while the other has already read it; observe the second transaction's next read reflects the change.
**Lab 2** — Repeat under Repeatable Read (or your database's equivalent) and observe the difference.
**Lab 3** — Attempt to reproduce a phantom read: query a range of rows, have another transaction insert a new matching row and commit, then re-run the same range query in the first transaction.
**Lab 4** — Check and document your database's default isolation level.

## 🎯 Interview Questions

**What's the difference between a non-repeatable read and a phantom read?**
A non-repeatable read is re-reading the *same* row and getting a different value; a phantom read is re-running the *same query* and getting different (new) matching rows.

**Why wouldn't you always use Serializable isolation?**
Because stronger isolation generally costs more in blocking, retries, or overhead, reducing throughput — the right level depends on how much anomaly risk the application can actually tolerate.

**Is Read Committed the default for every relational database?**
No — default isolation levels vary by database, so it should always be explicitly checked rather than assumed.

## 📌 Summary

```text
Rule 1 — Isolation levels trade correctness guarantees for concurrency/performance.
Rule 2 — Read Committed is a common default but prevents only dirty reads.
Rule 3 — Repeatable Read guarantees consistent re-reads of the same row.
Rule 4 — Serializable is the strongest but most expensive guarantee.
Rule 5 — Never assume the default isolation level — verify per database.
```

---

# Chapter 93 — Snapshots: How Consistent Reads Actually Work

> "Consistent read" sounds abstract until you understand what a snapshot actually is: a frozen point-in-time view a transaction carries with it.

## Learning Objectives
- What a transaction snapshot is
- When a snapshot is taken (per-transaction vs per-statement)
- How snapshots relate to isolation levels
- Long-running transactions and snapshot cost

## 1️⃣ What a Snapshot Represents

```text
Transaction Snapshot
 ↓
"As of this point, which row versions are visible to me?"
```

Every read within the scope of that snapshot sees a consistent view, ignoring changes made by transactions that committed after the snapshot was taken.

## 2️⃣ Per-Transaction vs Per-Statement Snapshots

```text
Repeatable Read / Serializable (typical behavior)
 ↓
Snapshot taken once, at the start of the transaction
 ↓
Every statement within it sees the same frozen view

Read Committed (typical behavior)
 ↓
Snapshot taken fresh for EACH statement
 ↓
Different statements within the same transaction
   can see different, more recent committed data
```

This directly explains the non-repeatable read behavior discussed in Chapter 92 — it's a consequence of *when* the snapshot is refreshed, not an arbitrary rule.

## 3️⃣ Why Snapshots Enable Non-Blocking Reads

```text
Reader
 ↓
Uses its own snapshot to determine visible row versions
 ↓
Never needs to wait for a concurrent writer's lock
   just to perform a read
```

This is the direct payoff of MVCC (Chapter 91) combined with snapshot-based visibility.

## 4️⃣ The Cost of Long-Running Snapshots

```text
Transaction opens a snapshot
        ↓
Stays open for a long time (minutes, hours)
        ↓
Row versions still needed by that snapshot
        ↓
Cannot be cleaned up yet, even if no longer
   visible to any OTHER transaction
```

This directly ties into Chapter 95 (Long-Running Transactions) — an old open snapshot is one of the most common real causes of storage bloat and vacuum/cleanup backlog in production systems.

## 🧠 Senior Mental Model

```text
A snapshot isn't just "a moment in time" in the abstract —
it's a concrete constraint on how long old row versions
must be kept around for cleanup purposes.
```

## 🧪 Hands-on Lab

**Lab 1** — Open a transaction under Read Committed, read a row, have another session update+commit it, then read again in the first transaction — observe the value change.
**Lab 2** — Repeat under Repeatable Read/Serializable and observe the value staying constant across both reads.
**Lab 3** — Open a long-running transaction (don't commit), perform many updates to the same row from other sessions, and inspect whether cleanup/vacuum activity is blocked or delayed.

## 🎯 Interview Questions

**What is a transaction snapshot, conceptually?**
A frozen view of which row versions are visible to a transaction, used to provide consistent reads without requiring locks against concurrent writers.

**Why can Read Committed show different data across two SELECTs in the same transaction, while Repeatable Read doesn't?**
Because Read Committed typically takes a fresh snapshot for each statement, while Repeatable Read (and stricter levels) typically take one snapshot for the entire transaction.

**Why can a long-running transaction cause storage bloat elsewhere in the database?**
Because its open snapshot may still need old row versions that would otherwise be eligible for cleanup, preventing garbage collection from reclaiming that space until the transaction ends.

## 📌 Summary

```text
Rule 1 — A snapshot defines which row versions a transaction can see.
Rule 2 — Snapshot refresh timing (per-transaction vs per-statement) explains isolation-level behavior differences.
Rule 3 — Snapshots let readers avoid blocking on concurrent writers.
Rule 4 — Long-open snapshots delay cleanup of old row versions elsewhere.
```

---

# Chapter 94 — Vacuum / Garbage Collection: Cleaning Up Old Row Versions

> MVCC's flexibility comes at a price: someone has to clean up all those old row versions eventually. This chapter is about that cleanup process.

## Learning Objectives
- Dead tuples / dead row versions
- Why cleanup can't happen immediately
- Vacuum / autovacuum concepts
- Bloat as a consequence of delayed cleanup
- Visibility and cleanup eligibility

## 1️⃣ Dead Tuples

```text
UPDATE row
 ↓
Old version marked dead (no longer the "current" version)
 ↓
New version becomes current
```

```text
DELETE row
 ↓
Row marked dead, not immediately physically removed
```

The old/dead version isn't instantly erased — it lingers until nothing could possibly still need to see it.

## 2️⃣ Why Cleanup Can't Happen Immediately

```text
Old row version
 ↓
Might still be needed by:
   a long-running transaction's snapshot (Chapter 93)
   a replica still catching up
   other visibility rules specific to the engine
```

Cleanup must wait until it's provably safe — i.e., no transaction could possibly still need that version.

## 3️⃣ Vacuum / Autovacuum (Concept)

```text
Vacuum
 ↓
Scans for dead tuples no longer visible to anyone
 ↓
Reclaims that space for reuse
```

Many databases run this automatically in the background (often called autovacuum or an equivalent internal process), but it can fall behind under heavy write load or when blocked by long-running transactions.

## 4️⃣ Bloat as a Consequence

```text
Cleanup falls behind
 ↓
Dead tuples accumulate
 ↓
Table/index physically larger than the live data requires
 ↓
More pages to scan, worse cache efficiency (ties back to Ch. 86)
```

## 5️⃣ Never Blindly Run Maintenance in Production

```text
Manual maintenance commands
   (VACUUM, REINDEX, OPTIMIZE, etc., database-specific)
 ↓
Can themselves consume significant I/O and even
   briefly lock resources depending on the mode used
```

Always understand the specific behavior and locking implications of a maintenance command for your database before running it against production, especially at large table sizes.

## 🧠 Senior Mental Model

```text
MVCC gives you concurrency
      +
Vacuum/cleanup is the bill that eventually comes due
```

Ignoring cleanup health is one of the most common ways a database that "used to be fine" quietly degrades over months.

## 🧪 Hands-on Lab

**Lab 1** — Perform heavy repeated updates on a small set of rows; inspect dead-tuple/bloat statistics if your database exposes them.
**Lab 2** — Run your database's cleanup/maintenance command manually and compare bloat statistics before and after.
**Lab 3** — Open a long-running transaction, generate heavy update activity elsewhere, and observe whether cleanup falls behind while that transaction remains open.

## 🎯 Interview Questions

**Why can't a database immediately delete an old row version after an UPDATE?**
Because other transactions' snapshots, or replicas, might still legitimately need to see that old version under MVCC visibility rules — it can only be removed once it's provably invisible to everyone.

**What is bloat, and what commonly causes it?**
Bloat is the accumulation of dead row versions that haven't yet been cleaned up, inflating a table's or index's physical size beyond what the live data requires — commonly caused by heavy update/delete activity combined with cleanup falling behind, often due to long-running transactions.

**Why should maintenance commands not be run blindly in production?**
Because they can consume significant I/O and, depending on the mode, may briefly lock resources — their impact should be understood in advance rather than assumed to be harmless.

## 📌 Summary

```text
Rule 1 — Dead row versions must be cleaned up eventually.
Rule 2 — Cleanup can't happen until no transaction could possibly need the old version.
Rule 3 — Vacuum/cleanup processes reclaim that space, often automatically.
Rule 4 — Delayed cleanup causes bloat, which degrades performance over time.
Rule 5 — Understand a maintenance command's locking behavior before running it in production.
```

---

# Chapter 95 — Long-Running Transactions: A Real Production Failure Pattern

> This single anti-pattern — a transaction that stays open far longer than intended — is responsible for an outsized share of real MVCC-related production incidents.

## Learning Objectives
- How a long-running transaction forms in practice
- The chain reaction it triggers
- Why it's easy to miss until it's severe
- How to detect and prevent it

## 1️⃣ How It Happens

```text
Transaction started
        ↓
Application makes an external call
   (API request, email send, file upload, etc.)
   WHILE the transaction is still open
        ↓
External call is slow or hangs
        ↓
Transaction remains open far longer than intended
```

This is an extremely common real-world root cause: transactions left open around slow, unrelated I/O rather than being scoped tightly around the actual database work.

## 2️⃣ The Chain Reaction

```text
Transaction remains open
        ↓
Its snapshot still needs old row versions (Chapter 93)
        ↓
Cleanup/vacuum cannot reclaim those versions (Chapter 94)
        ↓
Dead tuples continue accumulating from OTHER concurrent activity
        ↓
Storage grows
        ↓
Cache efficiency degrades (more bloat to scan through)
        ↓
Overall database performance degrades — for everyone,
   not just the slow transaction
```

## 3️⃣ Why It's Easy to Miss

```text
Symptom noticed: "The database feels generally slower lately."
Root cause: one connection, opened hours ago, never committed or rolled back.
```

Because the effect is diffuse (general slowdown) rather than a single obvious failed query, this pattern often goes undiagnosed far longer than a straightforward slow query would.

## 4️⃣ Detection

```text
Check for:
   Transactions open far longer than your normal transaction duration
   Connections that are "idle in transaction"
      (connected, transaction open, but not actively running anything)
```

Most databases expose some form of active-session/transaction view that surfaces transaction start time and state — this should be a standard part of production monitoring, not something checked only after an incident.

## 5️⃣ Prevention

```text
Design principle:
   Keep transactions as short as possible.
   Never perform slow external I/O
      (network calls, third-party APIs, file writes)
   while a database transaction is open.
```

```text
Application
 ↓
Do external I/O BEFORE or AFTER the transaction,
   not DURING it.
```

Set statement/transaction timeouts where your database and application framework support them, as a safety net against this pattern recurring.

## 🧠 Senior Mental Model

```text
A transaction is not just "a unit of correctness."
It is also "a hold on cleanup for every row version
it might still need to see."

Treat transaction duration as a resource to be minimized,
not an implementation detail to ignore.
```

## 🧪 Hands-on Lab

**Lab 1** — Open a transaction, deliberately leave it open (simulating a hung external call), and generate update activity elsewhere; observe bloat/dead-tuple growth while it remains open.
**Lab 2** — Check your database's active-session view for "idle in transaction" or equivalent state, and identify how transaction start time is exposed.
**Lab 3** — Configure (where supported) a statement or idle-in-transaction timeout, and confirm it actually terminates a deliberately-hung transaction.

## 🎯 Interview Questions

**How does a single long-running transaction cause a general, diffuse performance degradation across a database?**
Its open snapshot prevents cleanup of old row versions it might still need, so dead tuples from unrelated concurrent activity accumulate unchecked, causing bloat and cache inefficiency that affects the whole system, not just that transaction.

**Why is this failure pattern often hard to diagnose?**
Because the symptom is a general slowdown rather than a single obviously failing query, and the root cause (one long-open connection/transaction) is easy to overlook without specifically monitoring transaction duration.

**What's the core prevention principle for this pattern?**
Never perform slow, unrelated I/O (external API calls, file operations, etc.) while a database transaction is open — keep transactions as short and tightly scoped as possible, and use timeouts as a safety net.

## 📌 Module 16 Critical Rules

```text
Rule 1 — MVCC lets readers and writers avoid blocking each other via row versions.
Rule 2 — Isolation levels trade anomaly prevention against concurrency/performance.
Rule 3 — A snapshot determines which row versions a transaction can see.
Rule 4 — Snapshot refresh timing explains why isolation levels behave differently.
Rule 5 — Old row versions must be cleaned up, but only once provably safe to do so.
Rule 6 — Cleanup delays cause bloat, which degrades performance over time.
Rule 7 — Long-running transactions are one of the most common real causes of bloat.
Rule 8 — Never hold a transaction open across slow external I/O.
Rule 9 — Monitor transaction duration and "idle in transaction" state proactively.
Rule 10 — Treat transaction duration itself as a resource to minimize.
```

---

# 🎉 Module 16 Complete

You now understand how a database supports many concurrent transactions without collapsing into constant blocking:

```text
MVCC
 ↓
Row Versions
 ↓
Isolation Levels (what anomalies you accept)
 ↓
Snapshots (what each transaction actually sees)
 ↓
Vacuum / Cleanup (reclaiming what's no longer needed)
 ↓
Long-Running Transactions (the failure mode that breaks all of this)
```

# 🚀 Next Module

# Module 17 — WAL, Logging & Recovery
Write-ahead logging, checkpoints, crash recovery, durability (fsync and friends), and the relationship between backups and transaction logs.