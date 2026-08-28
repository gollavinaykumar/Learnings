# 🗄️ SQL Mastery — 20+ Years Experience
# Module 18 — Partitioning

> This module is about what to do when a single table becomes too large to manage well as one physical structure — even with perfect indexing.

---

# Chapter 101 — Why Partition? The Problem With Massive Single Tables

> Indexes solve "find rows fast." Partitioning solves a different problem entirely: managing, maintaining, and pruning work against a table so large that even good indexes aren't enough on their own.

## Learning Objectives
- What problems partitioning actually solves
- Why indexing alone eventually isn't sufficient
- Partitioning vs sharding (the distinction)
- When partitioning is premature

## 1️⃣ The Scenario

```text
orders
 ↓
5 billion rows
```

Most queries filter primarily on:

```text
created_at
```

## 2️⃣ Problems at This Scale, Even With Good Indexes

```text
Index maintenance cost
 ↓
Every insert/update/delete still touches a massive index structure

Vacuum / cleanup cost (Chapter 94)
 ↓
Scanning for dead tuples across billions of rows is expensive

Maintenance operations
 ↓
Rebuilding an index, adding a column, archiving old data —
   all become extremely expensive against one giant table

Backup/restore time
 ↓
A single massive table extends backup and restore windows
```

## 3️⃣ The Partitioning Idea

```text
Instead of:
orders  (5 billion rows, one physical structure)

Use:
orders_2026_01
orders_2026_02
orders_2026_03
...
```

Each partition is a smaller, independently manageable physical structure, while still being queryable through the same logical table name in many implementations.

## 4️⃣ Partitioning vs Sharding — Don't Confuse Them

```text
Partitioning
 ↓
Splitting a table into multiple physical pieces
   WITHIN A SINGLE DATABASE INSTANCE

Sharding
 ↓
Splitting data across MULTIPLE DATABASE INSTANCES/MACHINES
   (Module 22)
```

Partitioning solves single-instance management problems. Sharding solves single-instance *capacity* problems (CPU/RAM/storage ceiling on one machine). They're related but distinct, and one doesn't automatically imply the other.

## 5️⃣ When Partitioning Is Premature

```text
Table = 500,000 rows
```

Partitioning this adds real complexity (query routing, partition management, cross-partition query cost) for very little benefit — indexing alone is almost certainly sufficient. Partitioning is a scale-driven decision, not a default best practice for every table.

## 🧠 Senior Mental Model

```text
Ask before partitioning:

Is the table large enough that maintenance operations
   (index rebuilds, vacuum, backup) are becoming genuinely painful?

Do queries naturally filter on a column that could
   serve as a clean partition boundary?

Is the added complexity actually justified by the problem
   we're solving, or are we partitioning "because big tables
   should be partitioned"?
```

## 🧪 Hands-on Lab

**Lab 1** — Estimate maintenance operation time (index rebuild, full backup) on a large unpartitioned table vs. an equivalent partitioned one, in a test environment.
**Lab 2** — Identify a real query pattern in your own schema that naturally filters on a column suitable for partitioning.
**Lab 3** — Discuss, for a small (sub-1M row) table, why partitioning would likely add more complexity than value.

## 🎯 Interview Questions

**What problems does partitioning solve that indexing alone doesn't?**
Maintenance costs at massive scale — index rebuilds, vacuum/cleanup, backups, and archival — all become expensive against one giant physical table; partitioning breaks that into smaller, independently manageable pieces.

**What's the difference between partitioning and sharding?**
Partitioning splits a table into multiple physical pieces within a single database instance; sharding splits data across multiple separate database instances/machines to scale beyond one machine's capacity.

**Why would partitioning a small table be a mistake?**
Because it adds real operational and query-routing complexity without a corresponding benefit — the maintenance and management problems partitioning solves only appear at genuinely large scale.

## 📌 Summary

```text
Rule 1 — Partitioning solves management/maintenance problems at massive scale.
Rule 2 — Partitioning is not the same as sharding.
Rule 3 — Partitioning is a scale-driven decision, not a default.
Rule 4 — A good partition key usually matches a natural, common query filter.
```

---

# Chapter 102 — Range Partitioning: Dates, IDs & Numeric Ranges

> The most common partitioning strategy — splitting data into contiguous ranges, usually by time or a monotonically increasing key.

## Learning Objectives
- What range partitioning is
- Why it's the default choice for time-series-like data
- Boundary design
- Common pitfalls (unbounded ranges, uneven partition sizes)

## 1️⃣ The Core Idea

```text
orders_2026_01   → created_at in January 2026
orders_2026_02   → created_at in February 2026
orders_2026_03   → created_at in March 2026
```

Each partition owns a contiguous range of values on the partitioning column.

## 2️⃣ Why This Fits Time-Series-Like Data So Well

```text
Most queries:
   WHERE created_at >= X AND created_at < Y
```

naturally map onto one or a small number of partitions — an extremely common access pattern for orders, logs, events, metrics, and similar append-heavy tables.

## 3️⃣ Boundary Design

```text
orders_2026_01  → [2026-01-01, 2026-02-01)
orders_2026_02  → [2026-02-01, 2026-03-01)
```

Half-open ranges (inclusive start, exclusive end) are the standard pattern — avoiding gaps and avoiding ambiguity about which partition a boundary value belongs to.

## 4️⃣ Pitfall: Forgetting to Create Future Partitions

```text
Today: 2026-08-28
Partitions exist through: 2026-08-31
        ↓
2026-09-01 arrives
        ↓
No partition exists for new rows
        ↓
Inserts fail or fall into an unintended default/catch-all partition
```

Range-partitioned schemes built around time need an operational process (automated, ideally) to keep creating future partitions ahead of need — this is a very common real-world operational gap.

## 5️⃣ Pitfall: Uneven Partition Sizes

```text
Numeric ID range partitioning:
   0 – 1,000,000        → Partition 1
   1,000,001 – 2,000,000 → Partition 2
```

If insert rate accelerates over time (very common for growing systems), later partitions can fill up much faster than earlier ones were designed for, leading to uneven partition sizes and uneven maintenance cost across partitions.

## 🧪 Hands-on Lab

**Lab 1** — Create a range-partitioned table by month; insert rows spanning several months and confirm they land in the correct partitions.
**Lab 2** — Attempt to insert a row with a date beyond any created partition and observe the failure (or default-partition behavior).
**Lab 3** — Design and implement (or script) a process for automatically creating future partitions ahead of time.

## 🎯 Interview Questions

**Why is range partitioning such a common default for time-series-like tables?**
Because most queries against such tables naturally filter by a date/time range, which maps cleanly onto one or a small number of contiguous partitions.

**What's a common operational failure mode with time-based range partitioning?**
Failing to proactively create future partitions ahead of need, causing inserts for new time periods to fail or fall into an unintended default partition.

**Why might numeric-ID range partitions become uneven in size over time?**
Because if insert rate accelerates as the system grows, later ID ranges fill up much faster than earlier ones, leading to unevenly sized and unevenly loaded partitions.

## 📌 Summary

```text
Rule 1 — Range partitioning splits data into contiguous value ranges.
Rule 2 — It's a natural fit for time-series-like, append-heavy tables.
Rule 3 — Half-open boundaries avoid gaps and ambiguity.
Rule 4 — Future partitions must be proactively created, ideally automated.
Rule 5 — Partition sizes can become uneven if growth rate isn't constant.
```

---

# Chapter 103 — List Partitioning: Region, Country, Category & Tenant

> When your data naturally splits into a small number of discrete, meaningful groups rather than a continuous range, list partitioning fits better than range partitioning.

## Learning Objectives
- What list partitioning is
- When it fits better than range partitioning
- Handling an unknown/growing set of list values
- Combining list partitioning with tenancy models

## 1️⃣ The Core Idea

```text
orders_us   → country = 'US'
orders_in   → country = 'IN'
orders_uk   → country = 'UK'
```

Each partition owns an explicit, enumerated set of values rather than a range.

## 2️⃣ When List Fits Better Than Range

```text
Range partitioning fits:
   Continuous, ordered values (dates, sequential IDs)

List partitioning fits:
   Discrete, categorical values with no natural ordering
   (country, region, category, tenant, status)
```

Trying to force categorical data into range partitioning (e.g., alphabetically ranging country codes) rarely aligns with actual query patterns and rarely produces balanced partitions.

## 3️⃣ Handling New/Unknown Values

```text
New country appears in the data
        ↓
No matching partition defined
        ↓
Insert fails, or falls into a default/catch-all partition
   (depending on database support and configuration)
```

Similar to range partitioning's "missing future partition" pitfall, list partitioning needs a process for handling genuinely new category values — either proactively defining new partitions or deliberately routing unknowns to a default partition.

## 4️⃣ Multi-Tenant List Partitioning

```text
orders_tenant_a
orders_tenant_b
orders_tenant_c
```

This can be a legitimate multi-tenancy strategy (related to Chapter 174) — but be aware of the "hot partition" risk (Chapter 124): if one tenant is dramatically larger than the others, that single partition can become a bottleneck even though the overall scheme is "partitioned."

## 🧪 Hands-on Lab

**Lab 1** — Create a list-partitioned table by a categorical column (e.g., region), insert rows across several categories, and confirm correct partition placement.
**Lab 2** — Insert a row with a value not covered by any defined partition and observe the failure or default-partition behavior.
**Lab 3** — Simulate an uneven tenant-based list partitioning scheme (one tenant with far more rows than others) and discuss the resulting imbalance.

## 🎯 Interview Questions

**When would you choose list partitioning over range partitioning?**
When the partitioning column has discrete, categorical values with no natural ordering (country, region, tenant, category) rather than a continuous, ordered range.

**What's a common operational risk with list partitioning?**
New category values appearing in the data without a corresponding partition defined, causing inserts to fail or fall into an unintended default partition unless proactively managed.

**What's a specific risk of using list partitioning for multi-tenancy?**
A single disproportionately large tenant can create a "hot partition" that becomes a bottleneck, even though the table is technically partitioned across many tenants.

## 📌 Summary

```text
Rule 1 — List partitioning fits discrete, categorical values.
Rule 2 — Unknown/new category values need explicit handling.
Rule 3 — Multi-tenant list partitioning risks uneven, hot partitions.
```

---

# Chapter 104 — Hash Partitioning: Distributing Rows Evenly

> When there's no natural range or discrete category to partition by — but you still want to split a large table into manageable, roughly equal pieces — hash partitioning is the tool.

## Learning Objectives
- What hash partitioning is
- Why it optimizes for even distribution over query locality
- The trade-off vs range/list partitioning
- Rebalancing challenges

## 1️⃣ The Core Idea

```text
hash(key) % number_of_partitions
        ↓
Determines which partition a row belongs to
```

Unlike range or list partitioning, hash partitioning doesn't try to group logically related rows together — it deliberately scatters them evenly.

## 2️⃣ Why Even Distribution Is the Goal

```text
Range/List partitioning
 ↓
Optimizes for: queries naturally targeting one partition
 ↓
Risk: uneven partition sizes (Chapters 102–103)

Hash partitioning
 ↓
Optimizes for: even distribution of rows and write load
 ↓
Trade-off: a typical query can no longer target just one partition,
   because logically related rows are scattered by design
```

## 3️⃣ The Query-Locality Trade-off

```sql
SELECT * FROM orders WHERE customer_id = 101;
```

With hash partitioning on, say, `order_id` (not `customer_id`), a single customer's orders could be scattered across every partition — meaning this query might need to check all partitions rather than just one. Hash partitioning is best chosen when the *write distribution* problem matters more than *query locality*, or when the hash key matches the actual query pattern.

## 4️⃣ Rebalancing Is Hard

```text
Partitions: 4
        ↓
Add a 5th partition
        ↓
hash(key) % 5 ≠ hash(key) % 4 for most keys
        ↓
Most rows would need to move to different partitions
```

This is a genuinely difficult operational problem — changing the number of hash partitions typically requires significant data movement, unlike adding a new range or list partition, which is usually cheap and localized.

## 5️⃣ When Hash Partitioning Makes Sense

```text
No natural time-based or categorical partition key
Write load needs to be spread evenly to avoid hotspots
Query pattern is expected to hit the hash key directly
   (e.g., queries always filter by the exact hashed column)
```

## 🧪 Hands-on Lab

**Lab 1** — Create a hash-partitioned table and insert a large, varied dataset; confirm rows are distributed roughly evenly across partitions.
**Lab 2** — Run a query filtering on a column OTHER than the hash key and observe (via EXPLAIN) whether all partitions must be checked.
**Lab 3** — Discuss/estimate the data-movement cost of adding one more hash partition to an existing 4-partition scheme.

## 🎯 Interview Questions

**What problem does hash partitioning optimize for that range/list partitioning doesn't?**
Even distribution of rows and write load across partitions, avoiding hotspots that can occur with range or list schemes when data or traffic is unevenly distributed by time or category.

**What's the main trade-off of hash partitioning?**
Loss of query locality — because rows are deliberately scattered, a query that doesn't filter directly on the hash key may need to check every partition instead of just one.

**Why is changing the number of hash partitions operationally difficult?**
Because the hash-to-partition mapping changes for most keys when the partition count changes, typically requiring significant data movement to rebalance, unlike adding a new range or list partition.

## 📌 Summary

```text
Rule 1 — Hash partitioning optimizes for even distribution, not query locality.
Rule 2 — Queries not filtering on the hash key may need to scan all partitions.
Rule 3 — Changing hash partition count typically requires significant rebalancing.
Rule 4 — Best suited when write-load distribution matters more than locality.
```

---

# Chapter 105 — Partition Pruning: Avoiding Unnecessary Partitions Entirely

> This is the actual performance payoff of partitioning — the optimizer's ability to skip entire partitions it can prove are irrelevant to a query.

## Learning Objectives
- What partition pruning is
- Static vs dynamic pruning
- Why pruning depends on the WHERE clause matching the partition key
- Pruning failures and how to detect them

## 1️⃣ The Core Idea

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-01';
```

Given range partitions by month, the database can potentially avoid reading unrelated partitions entirely:

```text
orders_2026_01  → skipped
orders_2026_02  → skipped
...
orders_2026_08  → read
orders_2026_09  → read (if it exists)
```

This is the actual mechanism that makes partitioning a *performance* feature, not just a management/maintenance feature.

## 2️⃣ Static vs Dynamic Pruning

```text
Static pruning
 ↓
Decided at planning time, when the filter value
   is a literal constant known in advance

Dynamic pruning
 ↓
Decided at execution time, when the filter value
   depends on a parameter, a join, or a subquery result
```

Dynamic pruning is generally harder for an optimizer to perform well and isn't universally supported — always verify with `EXPLAIN` rather than assuming it happened.

## 3️⃣ Pruning Requires the Predicate to Match the Partition Key

```sql
-- Partitioned by created_at

SELECT * FROM orders WHERE customer_id = 101;
```

This query doesn't filter on the partition key at all — pruning cannot help here, and every partition may need to be checked. Partitioning only pays off performance-wise for queries that actually filter on (or near) the partition key.

## 4️⃣ A Common Pruning Failure: Function-Wrapped Partition Key

```sql
WHERE DATE(created_at) = '2026-08-28'
```

Just like sargability (Chapter 83) for indexes, wrapping the partition key in a function can prevent the optimizer from pruning partitions, even though a logically equivalent range predicate would prune cleanly:

```sql
WHERE created_at >= '2026-08-28'
  AND created_at <  '2026-08-29'
```

## 5️⃣ Verifying Pruning Actually Happened

```text
EXPLAIN
 ↓
Look for confirmation that only the expected subset
   of partitions was considered/scanned,
   not the full list of all partitions
```

This should be a standard verification step after implementing partitioning — don't assume pruning is working just because partitions exist.

## 🧪 Hands-on Lab

**Lab 1** — Run a query filtering cleanly on the partition key and confirm via `EXPLAIN` that only relevant partitions are scanned.
**Lab 2** — Run a query filtering on a non-partition-key column and confirm all partitions are scanned.
**Lab 3** — Compare pruning behavior between a sargable range predicate and a function-wrapped equivalent on the partition key.

## 🎯 Interview Questions

**What is partition pruning?**
The optimizer's ability to skip reading partitions it can prove are irrelevant to a query, based on the query's filter conditions matching the partition key.

**Why doesn't partitioning help a query that filters on a column other than the partition key?**
Because pruning depends on the optimizer being able to map the filter condition to specific partitions; a filter on an unrelated column gives it no basis to skip any partition.

**How can wrapping the partition key in a function break pruning, and how do you fix it?**
The optimizer generally can't prune based on a function-wrapped predicate the same way it can with a direct comparison; rewriting the condition as an equivalent range predicate on the raw partition key column typically restores pruning.

## 📌 Summary

```text
Rule 1 — Pruning is the actual performance benefit of partitioning.
Rule 2 — Pruning requires the query to filter on (or near) the partition key.
Rule 3 — Function-wrapped partition keys can silently break pruning.
Rule 4 — Always verify pruning behavior with EXPLAIN, never assume it.
```

---

# Chapter 106 — Partition Management: Creating, Dropping, Attaching & Detaching

> Partitioning isn't a "set it up once and forget it" feature — it requires an ongoing operational process, and this is where most real-world partitioning implementations succeed or fail.

## Learning Objectives
- Creating new partitions proactively
- Dropping old partitions (archival/retention)
- Attach/detach as a maintenance technique
- Automating partition lifecycle management
- Real production partition management patterns

## 1️⃣ Creating Partitions Proactively

```text
Recurring job / automation
        ↓
Creates next period's partition(s)
   well BEFORE they're needed
        ↓
Avoids the "missing partition" failure from Chapter 102
```

This should never be a manual, reactive process in a production system with ongoing time-based partitioning — it should be automated and monitored.

## 2️⃣ Dropping Old Partitions — Cheap Retention/Archival

```text
Without partitioning:
   DELETE FROM orders WHERE created_at < '2020-01-01';
      → Expensive: scans/deletes potentially billions of rows,
        generates massive WAL, heavy vacuum burden afterward

With partitioning:
   DROP the old partition entirely (e.g., orders_2019_12)
      → Cheap: removes an entire physical structure at once,
        essentially a metadata operation
```

This is one of the single biggest practical wins of time-based partitioning for any system with a data retention policy — turning an expensive, slow bulk delete into a near-instant structural operation.

## 3️⃣ Attach / Detach

```text
Detach a partition
 ↓
Removes it from the logical table without physically deleting it
 ↓
Can be archived elsewhere, inspected, or re-attached later

Attach a partition
 ↓
Adds an existing physical table as a new partition
   of the logical table
```

This pattern is useful for staged data loading (build/populate a table separately, then attach it as a partition) and for archival workflows (detach old data before eventually dropping or moving it).

## 4️⃣ Real Production Pattern: Rolling Window Retention

```text
Policy: keep 24 months of order history

Monthly automated job:
   1. Create next month's partition
   2. Drop the partition older than 24 months
```

This keeps the table's total size roughly bounded over time, regardless of how much new data continues to flow in — a critical property for long-lived, high-volume systems.

## 5️⃣ Partition Maintenance Still Has Costs

```text
Individual partitions still need:
   Their own indexes maintained
   Their own statistics kept current
   Their own vacuum/cleanup activity (Chapter 94)
```

Partitioning reduces the *scope* of these operations per partition, but doesn't eliminate the need for them — a partitioned table with dozens of small, neglected partitions can still accumulate problems, just distributed differently.

## 🧠 Senior Mental Model

```text
Partitioning is not a one-time schema decision.
It's an ongoing operational responsibility:
   creating partitions ahead of need,
   retiring old ones on schedule,
   and keeping every individual partition healthy.
```

## 🧪 Hands-on Lab

**Lab 1** — Implement (or script) an automated job that creates the next period's partition ahead of time.
**Lab 2** — Compare the cost/time of `DROP`-ing an old partition vs. running an equivalent `DELETE` on the same volume of unpartitioned data.
**Lab 3** — Practice detaching a partition, inspecting it independently, and re-attaching it.
**Lab 4** — Design a rolling-window retention policy (create + drop) for a hypothetical dataset with a defined retention period.

## 🎯 Interview Questions

**Why is dropping a partition so much cheaper than deleting the equivalent rows from an unpartitioned table?**
Because dropping a partition removes an entire physical structure as essentially a metadata operation, whereas deleting rows from an unpartitioned table requires scanning and removing them individually, generating significant WAL and vacuum burden.

**What are attach/detach used for in partition management?**
Detach removes a partition from the logical table without deleting its data, useful for archival or inspection; attach adds an existing table as a new partition, useful for staged data loading.

**Why is partitioning considered an ongoing operational responsibility rather than a one-time setup?**
Because it requires continuously creating new partitions ahead of need, retiring old ones on a retention schedule, and keeping each individual partition's indexes, statistics, and cleanup healthy over time.

## 📌 Module 18 Critical Rules

```text
Rule 1  — Partitioning solves management/maintenance problems at massive scale.
Rule 2  — Partitioning ≠ sharding — one instance vs. many instances.
Rule 3  — Range partitioning fits continuous, ordered data like dates.
Rule 4  — List partitioning fits discrete, categorical data.
Rule 5  — Hash partitioning optimizes for even distribution, sacrificing locality.
Rule 6  — Partition pruning is the actual performance win — verify it with EXPLAIN.
Rule 7  — Function-wrapped partition keys can silently break pruning.
Rule 8  — Dropping old partitions is dramatically cheaper than bulk deletes.
Rule 9  — Partition creation must be proactive, ideally automated.
Rule 10 — Partitioning is an ongoing operational responsibility, not a one-time decision.
```

---

# 🎉 Module 18 Complete

You now understand how to break a massive table into manageable pieces, and how the database exploits that structure for performance:

```text
Why Partition?
 ↓
Range / List / Hash Partitioning
 ↓
Partition Pruning (the performance payoff)
 ↓
Ongoing Partition Management (create, drop, attach, detach)
```

# 🚀 Next Module

# Module 19 — Replication
Why replication exists, primary/replica architecture, replication lag, synchronous vs asynchronous replication, and read-after-write consistency.