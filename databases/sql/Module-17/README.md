# 🗄️ SQL Mastery — 20+ Years Experience
# Module 17 — WAL, Logging & Recovery

> This module answers a deceptively simple question: when the database says "COMMIT," and the power goes out one millisecond later, how does it guarantee your data survives?

---

# Chapter 96 — Write-Ahead Logging: The Foundation of Durability

> The core idea behind almost every crash-safe database: never modify the actual data until you've first durably recorded what you're about to do.

## Learning Objectives
- What WAL actually is
- Why "log first, data second" matters
- The relationship between WAL and durability
- Sequential vs random write patterns for logging
- WAL and replication (preview)

## 1️⃣ The Core Idea

```text
Transaction
    ↓
WAL          (write-ahead log entry describing the change)
    ↓
Durable Storage
    ↓
Data Pages   (the actual table/index pages, updated later)
```

The essential rule:

> Required log information is made durable *before* the corresponding data-page changes are considered safely persisted.

## 2️⃣ Why Log First?

```text
If the database crashes AFTER the data page was modified
but BEFORE the log entry was durable:
    → No record of what happened, no way to verify consistency

If the database crashes AFTER the log entry was durable
but BEFORE the data page was modified:
    → The log entry can be replayed on restart to reconstruct the change
```

Logging first gives the database a reliable record to recover from, regardless of exactly when a crash interrupts the actual data-page write.

## 3️⃣ WAL Is (Mostly) Sequential

```text
Data pages
 ↓
Scattered across the table/index structure
 ↓
Random I/O to update

WAL
 ↓
Appended in order, one entry after another
 ↓
Sequential I/O to write
```

This is a major reason WAL-based durability is efficient in practice: even though the underlying change might touch scattered data pages, the *durable record* of that change is written sequentially — which is cheap, especially relative to random writes (Chapter 89).

## 4️⃣ Committing = Making the Log Durable

```text
COMMIT
 ↓
WAL entries for this transaction flushed to durable storage
 ↓
Only THEN is the commit acknowledged to the client
```

This is why `COMMIT` can have a real, measurable latency cost — it's not just an in-memory flag flip, it typically involves forcing data out to durable storage.

## 5️⃣ WAL Enables More Than Just Crash Recovery

```text
WAL
 ↓
Also commonly used for:
   Replication (streaming WAL to replicas)
   Point-in-time recovery
   Change data capture (Chapter 182)
```

The same durable, ordered record of changes that protects against crashes also becomes the backbone for keeping other systems in sync.

## 🧠 Senior Mental Model

```text
Data pages = "the current truth, eventually"
WAL         = "the durable, ordered history of how we got there"
```

## 🧪 Hands-on Lab

**Lab 1** — Research where your database's WAL/transaction log files are physically stored and how they're named/rotated.
**Lab 2** — Compare commit latency for a transaction with synchronous durability settings vs. a relaxed/asynchronous setting (where your database supports adjusting this) — read the safety trade-offs carefully first.
**Lab 3** — Read your database's documentation on what WAL is used for beyond crash recovery (replication, PITR, CDC).

## 🎯 Interview Questions

**What is the core principle behind write-ahead logging?**
Required information about a change is made durable in the log before the corresponding data-page change is considered safely persisted, ensuring a reliable basis for recovery regardless of exactly when a crash occurs.

**Why is WAL efficient despite protecting scattered, random data-page changes?**
Because the log itself is written sequentially/append-only, which is far cheaper than the random I/O the actual data-page updates might require — durability is achieved via a cheap sequential write, not by durably writing every scattered page synchronously.

**Why can COMMIT have measurable latency?**
Because committing typically requires flushing the relevant WAL entries to durable storage before acknowledging success to the client, which is a real I/O operation, not just an in-memory state change.

## 📌 Summary

```text
Rule 1 — WAL records changes durably before data pages are considered persisted.
Rule 2 — WAL writes are sequential; data-page writes are often random.
Rule 3 — COMMIT durability cost comes from flushing the log, not the data pages.
Rule 4 — WAL underpins replication, point-in-time recovery, and CDC — not just crash safety.
```

---

# Chapter 97 — Checkpoints: Bounding Recovery Time

> If the database only ever appended to the log and never applied changes to the actual data pages, recovery after a crash would mean replaying the entire history since the beginning of time. Checkpoints prevent that.

## Learning Objectives
- What a checkpoint does
- Why recovery time would be unbounded without them
- The cost of checkpointing
- Checkpoint frequency trade-offs
- Checkpoint-related latency spikes

## 1️⃣ The Problem Checkpoints Solve

```text
Without checkpoints
 ↓
Crash recovery would need to replay EVERY WAL entry
   since the database was first created
 ↓
Recovery time grows unbounded over the life of the database
```

## 2️⃣ What a Checkpoint Does

```text
Checkpoint
 ↓
Periodically flush/coordinate durable state of data pages
   so that WAL entries before this point are no longer
   needed for crash recovery
```

Conceptually, a checkpoint says: "Everything up to here is now safely reflected in the actual data pages — recovery only needs to replay what happened after this point."

## 3️⃣ Recovery Time as a Function of Checkpoint Frequency

```text
Frequent checkpoints
 ↓
Less WAL to replay after a crash
 ↓
Faster recovery
 ↓
But more overhead from checkpointing itself, more often

Infrequent checkpoints
 ↓
More WAL to replay after a crash
 ↓
Slower recovery
 ↓
But less overhead from checkpointing itself
```

This directly ties into RTO planning (Chapter 179) — checkpoint frequency is a real, tunable lever on how long recovery actually takes.

## 4️⃣ The Cost of Checkpointing

```text
Checkpoint
 ↓
Must write out a potentially large volume of
   modified-but-not-yet-persisted data pages
 ↓
Real, sometimes bursty I/O load
```

On HDD-backed storage in particular (Chapter 90), this burst of write I/O can compete with concurrent read activity and cause visible, periodic latency spikes — a classic "why does my database stutter every few minutes" production symptom.

## 5️⃣ Smoothing Checkpoint I/O

```text
Aggressive/instant checkpoint
 ↓
Short burst, high peak I/O, visible latency spike

Spread-out checkpoint
 ↓
Same total I/O, spread over a longer window
 ↓
Lower peak, less visible impact
```

Many databases offer tuning to spread checkpoint I/O over time specifically to avoid this kind of stutter — worth investigating if periodic latency spikes correlate with checkpoint timing.

## 🧪 Hands-on Lab

**Lab 1** — Correlate periodic latency spikes (if observable in a test environment) with checkpoint timing/logs.
**Lab 2** — Research your database's checkpoint frequency/interval settings and how they trade off against recovery time.
**Lab 3** — If your database supports spreading checkpoint I/O over time, compare default vs. spread-out settings under a write-heavy workload.

## 🎯 Interview Questions

**Why are checkpoints necessary if WAL already guarantees durability?**
Because without checkpoints, crash recovery would need to replay the entire WAL history since the database began, making recovery time grow unbounded — checkpoints bound how far back recovery needs to look.

**What's the trade-off between frequent and infrequent checkpoints?**
Frequent checkpoints shorten recovery time but add more regular I/O overhead; infrequent checkpoints reduce that overhead but lengthen recovery time after a crash.

**Why might a database show periodic latency spikes correlated with checkpoint timing?**
Because a checkpoint can require writing out a large volume of modified data pages in a burst, which can compete with concurrent I/O and cause temporary latency increases, especially on storage less tolerant of write bursts.

## 📌 Summary

```text
Rule 1 — Checkpoints bound how much WAL must be replayed during recovery.
Rule 2 — Checkpoint frequency trades recovery time against ongoing overhead.
Rule 3 — Checkpoints can cause real, sometimes bursty write I/O load.
Rule 4 — Spreading checkpoint I/O over time can reduce visible latency spikes.
```

---

# Chapter 98 — Crash Recovery: What Actually Happens on Restart

> This is the payoff of everything in Chapters 96–97: watching the database put itself back together after an unclean shutdown.

## Learning Objectives
- The recovery sequence on restart
- Redo vs undo concepts
- Why recovery time varies
- What "consistent state" means after recovery
- Testing recovery in practice

## 1️⃣ The Scenario

```text
Transaction
 ↓
UPDATE
 ↓
Database Crash
   (power loss, OS crash, process kill, etc.)
```

## 2️⃣ The Recovery Sequence on Restart

```text
Database starts
      ↓
Read recovery information (WAL, since the last checkpoint)
      ↓
Redo committed changes not yet reflected in data pages
      ↓
Undo changes from transactions that never committed
      ↓
Recover to a consistent state
```

## 3️⃣ Redo vs Undo, Conceptually

```text
Redo
 ↓
"This change was committed and durably logged,
 but the data page hadn't been updated yet — reapply it."

Undo
 ↓
"This change was in-progress but never committed —
 roll back any partial effect so it's as if it never happened."
```

The exact mechanisms differ significantly by database engine, but this redo/undo distinction is the conceptual core of most crash recovery implementations.

## 4️⃣ Why Recovery Time Varies

```text
Amount of WAL since the last checkpoint
        ↓
Directly determines how much work recovery has to do
```

This is exactly why Chapter 97's checkpoint frequency matters operationally — a database that checkpoints rarely will have a longer, more painful recovery after a crash than one that checkpoints more often.

## 5️⃣ What "Consistent State" Means

```text
After recovery completes:
   All committed transactions' effects are present
   No partial effects from uncommitted transactions remain
   The database is safe to accept new connections/queries
```

This is the concrete meaning behind the "Durability" and "Atomicity" letters in ACID (Chapter 53) — recovery is where those guarantees are actually enforced, not just promised.

## 6️⃣ Testing Recovery Is Not Optional

```text
"We have WAL, so we're durable."
```

is a claim, not a fact, until it's actually tested — deliberately killing a database process mid-transaction (in a non-production environment!) and confirming it recovers to the expected state is a legitimate and valuable exercise, directly related to the later principle in Chapter 180: "A backup that has never been restored is only a theory." The same applies to recovery itself.

## 🧪 Hands-on Lab

**Lab 1** — In a disposable test environment, forcibly kill the database process mid-transaction and observe the recovery process on restart.
**Lab 2** — Compare recovery time after a crash with a recent checkpoint vs. after a crash following a long checkpoint interval.
**Lab 3** — After a simulated crash, verify: committed transactions are present, and any deliberately uncommitted transaction's changes are absent.

## 🎯 Interview Questions

**What are the two conceptual phases of crash recovery?**
Redo — reapplying committed changes not yet reflected in data pages — and undo — rolling back partial effects from transactions that never committed.

**Why does recovery time depend on checkpoint frequency?**
Because recovery only needs to replay WAL entries since the last checkpoint; a longer interval between checkpoints means more log to replay and therefore longer recovery.

**Why is it valuable to actually test crash recovery rather than just trusting WAL exists?**
Because "we have WAL" is a claim about mechanism, not a verified guarantee — actually killing the process and confirming correct recovery validates that the mechanism works as intended in your specific configuration.

## 📌 Summary

```text
Rule 1 — Recovery redoes committed-but-unapplied changes and undoes uncommitted ones.
Rule 2 — Recovery time scales with WAL volume since the last checkpoint.
Rule 3 — "Consistent state" means committed effects present, uncommitted effects absent.
Rule 4 — Recovery should be tested deliberately, not assumed to work.
```

---

# Chapter 99 — Durability: fsync, Write Caches & What "Written" Actually Means

> "The database wrote it" and "the database durably wrote it" are two very different claims — and the gap between them is where real data loss incidents happen.

## Learning Objectives
- The layered write path
- What fsync actually guarantees
- Write caches (OS and disk-level)
- Why "written by the application" isn't automatically "durable"
- Durability configuration trade-offs

## 1️⃣ The Layered Write Path

```text
Application write() call
      ↓
OS page cache / buffer
      ↓
Disk controller write cache
      ↓
Physical storage medium
```

A write can be considered "successful" by an application long before it's actually durable on physical storage — it may simply be sitting in one of these intermediate caches.

## 2️⃣ What fsync Guarantees

```text
fsync (or equivalent)
 ↓
Forces buffered data through the OS and, ideally,
   the disk controller, down to physically durable storage
```

Databases rely on this kind of explicit flush operation specifically because a plain write() call alone doesn't guarantee durability — it only guarantees the OS has accepted the data, not that it has survived a power loss.

## 3️⃣ Write Caches Can Lie

```text
Disk controller write cache enabled (without battery/capacitor backup)
      ↓
Controller reports "write complete"
      ↓
Data still only in the controller's volatile cache
      ↓
Power loss
      ↓
Data lost, despite having been "acknowledged" as written
```

This is a genuinely dangerous real-world failure mode — storage hardware/configuration that acknowledges writes before they're actually durable can silently violate a database's durability guarantees, no matter how correctly the database itself implements WAL and fsync.

## 4️⃣ "Written by the Application" ≠ "Durably Persisted"

```text
Application: "I called the write function, so it's saved."
Reality: depends entirely on what happens at every layer
         between that call and physical storage.
```

This is why database durability configuration (which flush/sync behavior is used, and at which layer) is a genuinely important production decision, not an implementation detail to ignore.

## 5️⃣ Durability Configuration Trade-offs

```text
Strict, fully synchronous durability
 ↓
Every commit waits for a real, verified flush to durable storage
 ↓
Strongest guarantee, but higher commit latency

Relaxed/asynchronous durability
 ↓
Commits acknowledged faster
 ↓
Small window of potential data loss on crash
   (recent commits not yet durably flushed)
```

Some databases expose this as a tunable setting specifically because different applications have different tolerance for this trade-off — but it must be a deliberate choice, not an accidental default.

## 🧠 Senior Mental Model

```text
Never assume "the write succeeded" means
"the write is safe from a power loss right now."

Understand, for your specific database AND your specific
storage hardware, exactly where the durability guarantee
actually terminates.
```

## 🧪 Hands-on Lab

**Lab 1** — Research your database's durability/sync configuration options and their documented trade-offs.
**Lab 2** — Investigate whether your storage hardware's write cache is battery/capacitor-backed (if physical hardware is involved) or how your cloud storage provider documents this guarantee.
**Lab 3** — Compare commit latency under strict vs. relaxed durability settings in a non-production environment, and discuss which workloads could tolerate the relaxed setting.

## 🎯 Interview Questions

**Why isn't a successful write() call sufficient to guarantee durability?**
Because the data may still be sitting in an OS page cache or disk controller write cache, neither of which guarantees survival of a power loss — an explicit flush/sync operation is needed to force it to physically durable storage.

**How can a disk controller's write cache silently violate database durability guarantees?**
If the controller acknowledges a write as complete while it's still only in volatile cache (without battery/capacitor backup), a power loss can lose that data even though the database correctly performed its fsync — the failure is below the database's control.

**Why might a database offer relaxed/asynchronous durability settings at all?**
Because some applications can tolerate a small window of potential data loss on crash in exchange for lower commit latency, and this can be a legitimate, deliberate trade-off for specific workloads.

## 📌 Summary

```text
Rule 1 — A successful write() call does not guarantee durability.
Rule 2 — fsync (or equivalent) is what forces data to truly durable storage.
Rule 3 — Misconfigured write caches can silently violate durability guarantees.
Rule 4 — Durability configuration is a deliberate trade-off, not a detail to ignore.
```

---

# Chapter 100 — Backup vs WAL: Two Different Tools for Two Different Failures

> Backups and transaction logs solve related but distinct problems — confusing them is a common and dangerous mistake.

## Learning Objectives
- What a backup actually protects against
- What WAL/transaction logs actually protect against
- Point-in-time recovery as the combination of both
- Why backups alone aren't enough
- Why WAL alone isn't enough

## 1️⃣ What a Backup Protects Against

```text
Backup
 ↓
A full (or incremental) copy of the database
   at some point in time
```

Protects against: catastrophic loss of the primary storage entirely — corruption, hardware failure, accidental `DROP TABLE`, a bad migration, ransomware, or a destroyed data center.

## 2️⃣ What WAL Protects Against

```text
WAL / Transaction Log
 ↓
A continuous, ordered record of changes
   since the last checkpoint/backup
```

Protects against: brief interruptions (a crash, a restart) where the database needs to recover to a consistent state using recent, granular history — not a full restore from scratch.

## 3️⃣ Why Neither Alone Is Enough

```text
Backup alone
 ↓
Only restores you to the moment the backup was taken
 ↓
Everything since then is lost
   (could be hours or a full day's worth of data, depending on backup frequency)

WAL alone
 ↓
Useless without a base backup/starting point to apply it against
 ↓
An ever-growing, unbounded log with nothing to anchor it to
```

## 4️⃣ Point-in-Time Recovery (PITR): The Combination

```text
Base Backup
   (taken at some point in time)
        +
WAL / Transaction Log
   (everything that happened since that backup)
        ↓
Restore the base backup, then replay WAL
   up to any desired point in time
```

This is what lets you recover to, for example, "5 minutes before the accidental DROP TABLE" rather than only to "the last nightly backup, losing everything since."

## 5️⃣ Backup Frequency and Recovery Point Objective (RPO)

```text
Backup taken once per day
 ↓
Worst case, without WAL: up to 24 hours of data loss

Backup taken once per day + continuous WAL archiving
 ↓
Worst case: only whatever WAL wasn't yet archived —
   potentially seconds to minutes of data loss
```

This directly ties to RPO planning (Chapter 179) — the combination of backup frequency and WAL archiving determines your real, achievable recovery point objective, not backup frequency alone.

## 🧠 Senior Mental Model

```text
Backup
 ↓
"Where can I start restoring from?"

WAL
 ↓
"What happened since that starting point,
 and how far forward can I replay it?"

Together
 ↓
"I can recover to almost any specific moment in time,
 not just to the last backup."
```

## 🧪 Hands-on Lab

**Lab 1** — Research your database's point-in-time recovery capability and what it requires (base backup + continuous WAL archiving, typically).
**Lab 2** — In a test environment, take a base backup, make additional changes, then perform a full restore using only the backup — observe the data loss window.
**Lab 3** — Repeat, this time also replaying WAL/logs since the backup, and confirm recovery to a more recent point in time.

## 🎯 Interview Questions

**What's the key difference between what a backup protects against and what WAL protects against?**
A backup protects against catastrophic loss of the primary storage/database entirely, restoring to a specific point-in-time snapshot; WAL protects against brief interruptions by providing a granular, ordered record for recovering to a consistent state without needing a full restore.

**Why is a backup alone not sufficient for a robust recovery strategy?**
Because a backup only restores you to the exact moment it was taken — everything that happened between that backup and the failure is lost unless combined with a continuous log like WAL.

**What is point-in-time recovery, and what two things does it require?**
The ability to recover a database to a specific moment in time, requiring both a base backup (a starting point) and a continuous transaction log (WAL) to replay forward from that starting point.

## 📌 Module 17 Critical Rules

```text
Rule 1  — WAL durably records changes before data pages are considered persisted.
Rule 2  — WAL writes are sequential, which makes durability efficient.
Rule 3  — Checkpoints bound recovery time by limiting how much WAL must be replayed.
Rule 4  — Checkpoint frequency trades recovery speed against ongoing I/O overhead.
Rule 5  — Recovery redoes committed-but-unapplied changes and undoes uncommitted ones.
Rule 6  — Recovery behavior should be tested, not assumed.
Rule 7  — A successful write() call does not guarantee durability — fsync/equivalent does.
Rule 8  — Misconfigured write caches can silently break durability guarantees.
Rule 9  — Backups and WAL protect against different classes of failure.
Rule 10 — Point-in-time recovery requires both a base backup and continuous log archiving.
```

---

# 🎉 Module 17 Complete

You now understand the full durability story, from a single COMMIT to full disaster recovery:

```text
Transaction
 ↓
WAL (durable, sequential record)
 ↓
Checkpoints (bounding recovery time)
 ↓
Crash Recovery (redo/undo to consistent state)
 ↓
Durability guarantees (fsync, write caches)
 ↓
Backups + WAL = Point-in-Time Recovery
```

# 🚀 Next Module

# Module 18 — Partitioning
Why partition massive tables, range/list/hash partitioning strategies, partition pruning, and ongoing partition management.