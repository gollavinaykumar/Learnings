# 🗄️ SQL Mastery — 20+ Years Experience
# Module 15 — Storage Engine Internals

> This module drops below SQL entirely. No more query text — this is about how a database actually stores and retrieves bytes: pages, heaps, buffer caches, memory, and the physical realities of disk I/O.

---

# Chapter 85 — Pages / Blocks: The Real Unit of Storage

> A database doesn't think in "rows." It thinks in fixed-size pages, and rows are just tenants living inside them.

## Learning Objectives
- What a page/block is
- Typical page sizes
- Page header, free space, row pointers
- Why databases read whole pages, not single rows
- How page size affects performance

## 1️⃣ What Is a Page?

```text
Table
 ↓
Page 1
Page 2
Page 3
Page 4
...
```

A page (also called a block) is the fixed-size unit the storage engine reads and writes — commonly something like 8KB, though exact defaults vary by database.

## 2️⃣ Anatomy of a Page

```text
Page
 ├── Page Header (metadata)
 ├── Row/Tuple Data
 ├── Free Space
 └── Pointers / Slot Array
```

The slot array lets the engine locate individual rows within the page without scanning byte-by-byte.

## 3️⃣ Why the Database Reads Whole Pages

```text
Need: 1 row
Reality: database reads the entire page containing that row
```

Disk I/O (especially on spinning disks, but even conceptually on SSDs) is far more efficient in fixed chunks than in tiny scattered reads. So even a query that "only needs one row" still pulls the whole page containing it into memory.

## 4️⃣ Implication: Row Packing Matters

```text
Small rows
 ↓
More rows per page
 ↓
Fewer pages read per query
```

```text
Wide rows (many/large columns)
 ↓
Fewer rows per page
 ↓
More pages read per query
```

This is one reason very wide tables (dozens of columns, large text/blob fields) can hurt performance even for queries that only touch a few columns — unless the storage engine supports separating large values out-of-line (TOAST-like mechanisms in some databases).

## 5️⃣ Page Size Trade-offs

```text
Larger pages
 ↓
Fewer page headers, more amortized overhead
 ↓
But more wasted I/O when only a tiny row is needed

Smaller pages
 ↓
Less wasted I/O per lookup
 ↓
But more overhead and more pages to manage overall
```

Most databases fix page size at the engine level, but it's a useful mental model when reasoning about storage behavior.

## 🧪 Hands-on Lab

**Lab 1** — Inspect your database's default page/block size via documentation or a system setting.
**Lab 2** — Create a table with wide rows (large text columns) vs. a table with narrow rows; compare storage size for the same row count.
**Lab 3** — If your database exposes it, inspect how many rows are packed per page for a real table.

## 🎯 Interview Questions

**Why does a database read a whole page instead of a single row?**
Because fixed-size chunked I/O is far more efficient than scattered single-row access, and the storage engine is built around page-level reads and writes as its fundamental unit.

**Why can wide rows hurt performance even when a query only selects a few columns?**
Because fewer wide rows fit per page, so more pages (and more I/O) are needed to retrieve the same number of logical rows.

## 📌 Summary

```text
Rule 1 — Pages, not rows, are the real unit of storage I/O.
Rule 2 — Row width directly affects how many rows fit per page.
Rule 3 — Wide tables can hurt performance even for narrow queries.
```

---

# Chapter 86 — Heap Storage: How Table Rows Are Physically Stored

> "Heap" describes the default, unordered way most relational databases physically store table rows.

## Learning Objectives
- What heap storage means
- Row location / physical addressing
- Why heap order isn't logical order
- Heap vs clustered storage (preview, ties to Ch. 35–36)
- Bloat and dead rows in heap storage

## 1️⃣ What "Heap" Means

```text
Heap
 ↓
Rows stored in no particular logical order
 ↓
Physical location ≠ any column's sorted order
```

New rows are generally appended wherever there's free space, not inserted "in order" by any column value.

## 2️⃣ Row Locators

```text
Row
 ↓
Physical location identifier
   (page number + slot/offset within the page)
```

An index doesn't store the row itself — it stores a pointer to this physical location so the engine can jump directly to the page and slot.

## 3️⃣ Heap Order ≠ Query Order

```sql
SELECT * FROM orders ORDER BY created_at;
```

Just because rows were inserted roughly in time order doesn't guarantee they're physically stored that way after updates, deletes, page splits, and vacuum/reclaim activity over time. The database still needs an index or an explicit sort to guarantee true ordering.

## 4️⃣ Updates and Heap Storage

```text
UPDATE row
 ↓
Depending on the engine:
   - overwrite in place, or
   - write a new row version and mark the old one dead
```

The second pattern (common in MVCC-based heap storage) means updates can leave behind dead row versions that must eventually be reclaimed (Chapter 94).

## 5️⃣ Heap Bloat

```text
Many updates/deletes
 ↓
Dead row versions accumulate
 ↓
Heap physically larger than the "live" data would require
 ↓
More pages to scan, worse cache efficiency
```

This is one of the most common causes of "the table used to be fast, now it's slow" even when row count and query haven't meaningfully changed.

## 🧪 Hands-on Lab

**Lab 1** — Insert rows, then heavily update a subset repeatedly; compare table storage size before and after.
**Lab 2** — Run a maintenance/reclaim command (database-specific) and compare storage size afterward.
**Lab 3** — Compare a `SELECT *` scan's page count on a heavily-updated table vs. a freshly-loaded table with identical live row counts.

## 🎯 Interview Questions

**What does "heap storage" mean for a table?**
Rows are stored without any guaranteed logical ordering — physical location is generally determined by available free space, not by any column's sort order.

**Why can a table get slower over time even without growing in row count?**
Because updates and deletes can leave dead row versions behind in heap storage, inflating the physical size and hurting cache efficiency until maintenance reclaims that space.

## 📌 Summary

```text
Rule 1 — Heap storage has no inherent logical row ordering.
Rule 2 — Row locators point to physical page+slot, not logical position.
Rule 3 — Updates/deletes can leave dead rows behind, causing bloat.
Rule 4 — Bloat degrades performance independent of live row count.
```

---

# Chapter 87 — Buffer Cache: The Layer Between Query and Disk

> Almost every fast query is fast because it never actually touched disk. It hit memory instead.

## Learning Objectives
- What the buffer cache is
- Cache hit vs cache miss
- Why "working set" size matters
- Cold cache problems
- Cache eviction basics

## 1️⃣ The Layered Path to Data

```text
Application
      ↓
Database
      ↓
Buffer Cache
      ↓
Disk
```

The buffer cache is an in-memory copy of recently/frequently accessed pages, sitting between the query executor and physical storage.

## 2️⃣ Cache Hit vs Cache Miss

```text
Cache Hit
 ↓
Page already in memory → very fast

Cache Miss
 ↓
Page not in memory → disk I/O required → much slower
```

## 3️⃣ Working Set

```text
Working Set
 ↓
The subset of data actively/frequently accessed
```

```text
Working set fits in buffer cache
 ↓
Most queries hit memory
 ↓
Consistently fast

Working set exceeds buffer cache
 ↓
Constant eviction and reload
 ↓
Consistently slower, more variable latency
```

## 4️⃣ Cold Cache Problems

```text
Database restart
 ↓
Buffer cache empty
 ↓
First wave of queries all miss
 ↓
Temporary latency spike until cache "warms up"
```

This is a real operational concern after restarts, failovers, or deployments — sometimes worth deliberately "warming" critical tables/indexes before routing production traffic back.

## 5️⃣ Eviction

```text
Cache full
 ↓
New page needs to be loaded
 ↓
Some existing page must be evicted
   (commonly some form of least-recently-used policy)
```

If a large, rarely-needed query (e.g., a huge analytical scan) runs against a database also serving latency-sensitive transactional queries, it can evict genuinely "hot" pages and cause a temporary but real slowdown for unrelated queries — a classic noisy-neighbor problem within a single database.

## 🧪 Hands-on Lab

**Lab 1** — Run the same query twice in a row; compare timing (second run should generally be faster due to caching).
**Lab 2** — Restart the database (in a safe/dev environment) and compare the first execution of a query against subsequent executions.
**Lab 3** — Run a large scanning query alongside smaller point queries and observe whether the smaller queries' latency is affected.

## 🎯 Interview Questions

**Why is buffer cache hit ratio such an important metric?**
Because cache hits avoid disk I/O entirely, and disk I/O is orders of magnitude slower than memory access — a low hit ratio directly signals frequent, expensive disk access.

**Why might a database be slow immediately after a restart even with no other changes?**
Because the buffer cache starts empty (cold), so the first wave of queries all miss and must read from disk until the cache "warms up" with frequently accessed pages.

**How can one large query hurt the performance of unrelated smaller queries?**
By evicting frequently-used ("hot") pages from the shared buffer cache to make room for its own large scan, forcing the smaller queries to suffer cache misses they wouldn't normally have.

## 📌 Summary

```text
Rule 1 — Buffer cache sits between the executor and disk.
Rule 2 — A cache hit is far cheaper than a cache miss.
Rule 3 — Working-set size relative to cache size drives overall latency consistency.
Rule 4 — Cold caches after restarts cause temporary latency spikes.
Rule 5 — Large queries can evict hot pages and hurt unrelated queries.
```

---

# Chapter 88 — Memory Management: Shared Buffers, Sort Memory & Connection Memory

> The database doesn't have one pool of memory — it has several, each serving a different purpose, and misconfiguring any one of them causes a different class of problem.

## Learning Objectives
- Shared memory / buffer pool
- Per-connection working memory
- Sort and hash memory
- Why memory configuration is a balancing act
- Symptoms of memory misconfiguration

## 1️⃣ The Major Memory Pools

```text
Shared memory / buffer pool
 ↓
Shared across all connections, holds cached pages
   (this is the buffer cache from Chapter 87)

Sort memory / work memory
 ↓
Per-operation memory for sorts, hashes, and similar work

Connection memory
 ↓
Per-connection overhead just for the connection existing
```

## 2️⃣ Why This Is a Balancing Act

```text
Total server RAM
 ↓
Split between:
   Shared buffer pool
   Per-connection working memory × number of connections
   OS-level file cache
   Everything else running on the machine
```

Over-allocating any one pool starves the others.

## 3️⃣ Sort/Hash Memory and Concurrency

```text
Sort memory setting: 4MB per operation
Concurrent connections running sorts: 1,000
```

```text
Worst case: 1,000 × 4MB = 4GB
just for sort memory, simultaneously
```

This is why "increase sort memory to fix slow sorts" is dangerous advice without also considering concurrency — a setting that's safe at low concurrency can exhaust server memory under real production load.

## 4️⃣ Connection Memory at Scale

```text
Per-connection overhead: a few MB
Connections: 5,000
```

```text
Total: potentially tens of GB
just to keep connections open,
before a single query even runs
```

This is one of the core reasons connection pooling (Chapter 119) exists — uncontrolled connection counts directly threaten memory stability.

## 5️⃣ Symptoms of Misconfiguration

```text
Buffer pool too small
 ↓
Constant eviction, poor cache hit ratio

Sort memory too small
 ↓
Frequent disk spills on sorts (Chapter 81)

Sort memory too large + high concurrency
 ↓
Out-of-memory pressure, possible crashes

Too many connections
 ↓
Connection memory overhead alone destabilizes the server
```

## 🧪 Hands-on Lab

**Lab 1** — Check your database's current memory configuration values (buffer pool size, sort/work memory, max connections).
**Lab 2** — Estimate worst-case sort memory usage: sort memory setting × max connections. Compare against total available RAM.
**Lab 3** — Simulate a high-connection-count scenario (via a connection pool test) and observe memory behavior.

## 🎯 Interview Questions

**Why can increasing sort memory make a database less stable rather than more?**
Because sort memory is typically allocated per operation, and under high concurrency, many simultaneous sorts can multiply that setting into far more total memory usage than the server actually has.

**Why does connection count matter for memory even if no queries are running?**
Because each open connection carries its own baseline memory overhead, so a very high connection count alone can consume significant memory before any query workload is considered.

## 📌 Summary

```text
Rule 1 — Database memory is split across several distinct pools.
Rule 2 — Sort/hash memory settings multiply by concurrency, not just by operation.
Rule 3 — Connection count has a direct, often underestimated, memory cost.
Rule 4 — Memory tuning is a balancing act, not a single "increase this" knob.
```

---

# Chapter 89 — Disk I/O: Sequential vs Random, IOPS, Throughput & Latency

> Two queries reading the exact same number of bytes can have wildly different performance — because *how* those bytes are accessed matters as much as how many.

## Learning Objectives
- Sequential vs random I/O
- IOPS vs throughput vs latency
- Queue depth
- CPU-bound vs I/O-bound vs memory-bound queries
- Why "CPU is fine" doesn't mean "database is fine"

## 1️⃣ Sequential vs Random I/O

```text
Sequential I/O
 ↓
Reading consecutive blocks in order
 ↓
Generally much more efficient

Random I/O
 ↓
Reading scattered, non-consecutive blocks
 ↓
Generally much less efficient, especially historically on spinning disks
```

This is the physical reason a sequential scan can outperform an index-driven access pattern that requires many scattered row lookups (tying back to Chapter 77).

## 2️⃣ IOPS, Throughput, Latency

```text
IOPS
 ↓
I/O Operations Per Second — how many discrete reads/writes per second

Throughput
 ↓
Total bytes moved per second

Latency
 ↓
Time for a single I/O operation to complete
```

A storage system can be throughput-rich but IOPS-poor, or vice versa — which workload it's good for depends on which of these dominates your access pattern.

## 3️⃣ Queue Depth

```text
Queue Depth
 ↓
How many I/O requests are outstanding/in-flight at once
```

Higher queue depth can improve total throughput on modern storage (especially SSD/NVMe) by keeping the device busy, but can also increase per-request latency under contention.

## 4️⃣ CPU-bound vs I/O-bound vs Memory-bound

```text
CPU-bound
 ↓
CPU is the bottleneck (heavy computation, complex expressions, aggregation)

I/O-bound
 ↓
Waiting on disk reads/writes is the bottleneck

Memory-bound
 ↓
Insufficient memory forces spills, evictions, or swapping
```

## 5️⃣ Why Low CPU Doesn't Mean "Database Is Fine"

```text
Database CPU = 40%
Developer: "Database isn't the problem."
```

```text
Reality checked:
Lock waits
I/O
Buffer/cache misses
Query plans
Connection pool
```

```text
Discovery:
Index scan
 ↓
Millions of random table lookups
 ↓
Storage latency high, CPU mostly idle waiting
```

CPU utilization only tells you about CPU. A database can be almost entirely idle on CPU while being severely bottlenecked on I/O wait — these are different resources with different symptoms.

## 🧪 Hands-on Lab

**Lab 1** — Compare timing of a sequential scan vs. a query requiring many scattered random lookups over the same amount of data.
**Lab 2** — Monitor database-level I/O wait metrics (not just CPU) during a known-slow query.
**Lab 3** — Identify, from real metrics, a case where CPU is low but latency is high — trace it back to I/O or lock waits.

## 🎯 Interview Questions

**Why can random I/O be much slower than sequential I/O for the same amount of data?**
Because accessing scattered, non-consecutive blocks generally incurs more overhead per operation than reading consecutive blocks in one continuous pass, particularly pronounced on spinning disks and still meaningful on SSDs.

**Why is "database CPU is low" not sufficient evidence that the database isn't the bottleneck?**
Because CPU utilization only measures CPU usage — a database can be severely bottlenecked on I/O wait, lock contention, or cache misses while CPU sits mostly idle.

**What's the difference between IOPS and throughput?**
IOPS measures how many discrete I/O operations occur per second; throughput measures the total volume of data moved per second — a system can be strong in one and weak in the other depending on workload shape.

## 📌 Summary

```text
Rule 1 — Sequential I/O is generally cheaper per byte than random I/O.
Rule 2 — IOPS, throughput, and latency are distinct metrics that can diverge.
Rule 3 — A query's bottleneck can be CPU, I/O, or memory — diagnose accordingly.
Rule 4 — Low CPU usage does not mean the database has no problem.
```

---

# Chapter 90 — SSD vs HDD: How Storage Hardware Shapes Database Behavior

> The underlying storage medium changes which optimizations matter and which production incidents are even possible.

## Learning Objectives
- Core HDD vs SSD differences
- Random read/write implications
- Checkpointing behavior
- Recovery time implications
- Why "just use SSD" doesn't eliminate the need for good design

## 1️⃣ Core Differences

```text
HDD
 ↓
Mechanical, spinning platters + moving read/write head
 ↓
Random I/O is dramatically slower than sequential I/O

SSD
 ↓
No moving parts, electronic access
 ↓
Random I/O is much closer to sequential I/O performance
   (though still not identical)
```

## 2️⃣ Why This Matters for Index Design

```text
On HDD
 ↓
Random row lookups via an index are especially punishing
 ↓
Sequential scans win more often at moderate selectivity

On SSD
 ↓
Random lookups are much cheaper relative to sequential scans
 ↓
Indexes remain favorable across a wider selectivity range
```

The core index-design principles (Module 12) don't change, but the exact break-even point between "use the index" and "just scan" shifts significantly depending on the underlying storage medium.

## 3️⃣ Checkpointing Behavior

```text
Checkpoint
 ↓
Periodically flush in-memory changes to durable storage
   to bound crash-recovery time
```

On HDD, large checkpoint I/O bursts can cause visible latency spikes because sequential writes compete with concurrent random reads for the same mechanical resource. SSDs handle these bursts far more gracefully, though write amplification (Module 12, Chapter 46) and wear-related behavior become their own concerns.

## 4️⃣ Recovery Time Implications

```text
Crash recovery
 ↓
Must read/replay logs, potentially touching many pages
   scattered across storage
```

Faster random access on SSDs generally translates directly into faster recovery, which matters for RTO (Recovery Time Objective, Chapter 179) planning.

## 5️⃣ "Just Use SSD" Isn't a Substitute for Good Design

```text
SSD makes random I/O cheaper
 ↓
It does NOT make random I/O free
```

A billion-row table with no useful indexes still benefits enormously from proper indexing and query design — SSD narrows the gap between good and bad design, it doesn't eliminate it.

## 🧪 Hands-on Lab

**Lab 1** — If you have access to both HDD-backed and SSD-backed storage, compare the same random-lookup-heavy query across both.
**Lab 2** — Research your storage medium's documented random vs. sequential I/O performance characteristics.
**Lab 3** — Discuss (or document) how a missing index's real-world impact might differ between HDD-backed and SSD-backed production systems.

## 🎯 Interview Questions

**Why does the "sequential scan vs index" break-even point shift between HDD and SSD?**
Because SSDs handle random I/O much more efficiently relative to sequential I/O than HDDs do, making index-driven random lookups relatively more attractive on SSD across a wider range of selectivity.

**Does moving to SSD storage remove the need for good indexing?**
No — it narrows the performance gap between well-indexed and poorly-indexed access patterns, but random I/O on SSD is still not free, and good index/query design still matters significantly at scale.

## 📌 Summary

```text
Rule 1 — HDDs penalize random I/O far more heavily than SSDs do.
Rule 2 — The scan-vs-index break-even point shifts with storage medium.
Rule 3 — Checkpointing and recovery both benefit from SSD's random-access speed.
Rule 4 — Good indexing and query design remain necessary regardless of storage medium.
```

---

# 🎉 Module 15 Complete

You now understand what happens physically beneath every SQL query:

```text
Query
 ↓
Buffer Cache (hit or miss?)
 ↓
Pages / Blocks
 ↓
Heap Storage (rows, physically)
 ↓
Disk I/O (sequential or random?)
 ↓
Storage Medium (HDD or SSD?)
```

And how memory is divided to make all of this work:

```text
Shared Buffer Pool
Sort / Work Memory
Connection Memory
```

# 🚀 Next Module

# Module 16 — MVCC & Isolation
Multi-Version Concurrency Control, isolation levels (Read Uncommitted → Serializable), snapshots, vacuum/garbage collection, and the dangers of long-running transactions.