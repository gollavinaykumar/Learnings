# 🗄️ SQL Mastery — 20+ Years Experience
# Module 14 — Query Optimization

> This module goes deeper into *why* the optimizer picks the plan it picks: full scans, cardinality estimation, statistics, join algorithms, sorting, predicate pushdown, and sargability — closing with a real production incident that ties it all together.

---

# Chapter 77 — Full Table Scans: When Sequential Scan Is Actually Correct

> A beginner sees "Seq Scan" in a plan and assumes something is broken. A senior engineer asks: given this data distribution, was a sequential scan actually the cheapest option?

## Learning Objectives
- What a sequential/table scan actually does
- Why scans aren't inherently bad
- The break-even point between scan and index
- Small-table behavior
- Why "just add an index" is not always the fix

## 1️⃣ What a Table Scan Does

```text
Table
 ↓
Read every page
 ↓
Evaluate predicate on every row
 ↓
Return matches
```

Conceptually the simplest possible access path — no auxiliary structure required.

## 2️⃣ When a Scan Is Actually Cheaper

```sql
SELECT * FROM orders WHERE status = 'ACTIVE';
```

If:

```text
80–90% of rows = ACTIVE
```

then using an index means:

```text
Index
 ↓
Millions of index entries
 ↓
Millions of random table lookups
```

versus:

```text
Table
 ↓
One sequential pass
```

Sequential I/O is often dramatically cheaper per row than random I/O, so scanning the whole table can genuinely win.

## 3️⃣ Small Tables Almost Always Scan

```text
Table = 200 rows
```

Even with a perfect index on the filtered column, the optimizer will very often just scan — the overhead of using an index (root → branch → leaf → row fetch) isn't worth it at this scale.

## 4️⃣ The Break-Even Point

```text
Small result, large table   → Index usually wins
Large result, large table   → Scan usually wins
Any result, small table     → Scan usually wins
```

There's no fixed percentage threshold ("20% of rows") that applies universally — it depends on row width, page layout, cache state, and the specific cost model of your database.

## 5️⃣ Why "Just Add an Index" Isn't Always the Fix

```text
Symptom: Seq Scan, query is slow
Assumption: "Missing index"
Reality: Maybe. Or maybe the predicate matches most of the table,
         and no index would help without restructuring the query
         or the data itself (e.g., partial index, partitioning).
```

## 🧪 Hands-on Lab

**Lab 1** — Create a table with a low-cardinality column (2–3 distinct values), index it, run a query matching 80% of rows, and confirm the optimizer ignores the index.
**Lab 2** — Repeat with a query matching 0.001% of rows and confirm the optimizer uses the index.
**Lab 3** — Force the optimizer to use the index anyway (database-specific hint/setting) and compare actual runtime against the scan.

## 🎯 Interview Questions

**Why might the optimizer ignore a valid index?**
Because using the index would require many random row lookups that, in total, cost more than one sequential pass over the table — this is common when the predicate matches a large fraction of rows.

**Is a sequential scan always a performance problem?**
No — for small tables or low-selectivity predicates, it can be the genuinely cheapest plan.

## 📌 Summary

```text
Rule 1 — A sequential scan is a valid, sometimes optimal, plan choice.
Rule 2 — Low selectivity favors scans; high selectivity favors indexes.
Rule 3 — Small tables almost always scan, regardless of indexes.
Rule 4 — "Add an index" is a hypothesis, not an automatic fix.
```

---

# Chapter 78 — Cardinality Estimation: How the Optimizer Predicts Row Counts

> Every plan the optimizer considers is priced based on a prediction: "how many rows will this operation produce?" Get that number wrong, and everything downstream is wrong too.

## Learning Objectives
- What cardinality estimation is
- Why it's the foundation of cost-based optimization
- How estimates compound across joins
- Common causes of bad estimates
- How to detect bad estimates from EXPLAIN ANALYZE

## 1️⃣ What Cardinality Estimation Predicts

```text
How many rows will this WHERE clause match?
How many rows will this JOIN produce?
How many rows will this GROUP BY produce?
```

Every one of these numbers feeds directly into cost calculations for I/O, memory, and CPU.

## 2️⃣ Why Errors Compound

```sql
SELECT *
FROM a
JOIN b ON a.id = b.a_id
JOIN c ON b.id = c.b_id;
```

If the estimate for the `a ⋈ b` step is off by 10x, that error propagates into the estimate for `⋈ c`, potentially compounding into a 100x error by the final join — leading the optimizer to choose entirely wrong join algorithms.

## 3️⃣ Independence Assumption

Most optimizers assume predicates are statistically independent unless told otherwise:

```sql
WHERE country = 'IN' AND city = 'Hyderabad'
```

If the optimizer treats these as independent, it may badly overestimate the matching row count — `city = 'Hyderabad'` almost entirely implies `country = 'IN'`, so the true selectivity is much closer to the `city` predicate alone, not the product of both.

## 4️⃣ Correlated Columns

```text
Correlation
 ↓
Two columns move together in ways
the default independence assumption doesn't capture
```

Some databases support extended statistics or multi-column statistics specifically to correct for this.

## 5️⃣ Skewed Data

```text
status column:
  ACTIVE   → 90,000,000 rows
  DELETED  → 10 rows
```

A single average-based estimate can badly misjudge either query. This is why histograms (Chapter 79) exist — a single "average selectivity" number can't represent this shape.

## 6️⃣ Detecting Bad Estimates

```text
EXPLAIN ANALYZE
 ↓
Estimated rows: 100
Actual rows: 5,000,000
```

A gap of several orders of magnitude is the clearest signal that cardinality estimation — not the query logic — is the root problem.

## 🧪 Hands-on Lab

**Lab 1** — Create two correlated columns (e.g., `city` and `country`), query both together, and compare the optimizer's row estimate against reality.
**Lab 2** — Load heavily skewed data into a `status` column and compare estimates for a common value vs. a rare one.
**Lab 3** — If your database supports extended/multi-column statistics, create them and re-check the correlated-column estimate.

## 🎯 Interview Questions

**Why do cardinality estimation errors compound across joins?**
Because each join's estimate is used as an input to size the next join, so an error early in the plan propagates and often multiplies through subsequent steps.

**Why can correlated columns cause bad estimates?**
Because optimizers typically assume predicates are independent by default; when columns are correlated, that assumption overestimates or underestimates the true selectivity.

## 📌 Summary

```text
Rule 1 — Cardinality estimation underlies every cost-based decision.
Rule 2 — Estimate errors compound across multiple joins.
Rule 3 — The independence assumption breaks down for correlated columns.
Rule 4 — Large estimated-vs-actual gaps point directly at this stage.
```

---

# Chapter 79 — Database Statistics: Histograms, Distinct Values & Data Distribution

> Cardinality estimation is only as good as the statistics feeding it. This chapter is about where those numbers actually come from.

## Learning Objectives
- What statistics the optimizer collects
- Histograms and why averages aren't enough
- Distinct value counts
- Statistics staleness
- Why bulk operations require a statistics refresh

## 1️⃣ What Gets Collected

```text
Row counts
Distinct value counts (n_distinct)
Most common values (MCVs) and their frequencies
Histograms of value distribution
Null fraction
Column correlation with physical row order
```

## 2️⃣ Why Averages Aren't Enough

```text
Average selectivity for "status" = 33% (3 distinct values)
```

but reality:

```text
ACTIVE   → 89%
PENDING  → 10%
DELETED  → 1%
```

A single average badly misrepresents each individual value. Histograms bucket the actual distribution so the optimizer can estimate per-value, not per-column-average.

## 3️⃣ Most Common Values (MCVs)

Many databases separately track the most frequent values and their exact frequencies, rather than relying purely on a histogram — because a handful of very common values (e.g., `NULL`, `'ACTIVE'`) can dominate a distribution in ways a generic histogram bucket smooths over.

## 4️⃣ Statistics Are Samples, Not Exact

```text
Table = 500 million rows
```

Recomputing exact statistics on every row would itself be expensive. Most databases sample a subset of rows and extrapolate — which means statistics can be *approximately* right, and occasionally noticeably wrong for rare values.

## 5️⃣ Staleness

```text
Yesterday: status mostly PENDING
Today: bulk job marks 95% as SHIPPED
Statistics: not yet refreshed
```

```text
Optimizer still believes old distribution
 ↓
Chooses a plan suited to yesterday's data
 ↓
Today's query performs badly
```

## 6️⃣ Bulk Operations and Statistics

```text
Bulk INSERT of 50 million rows
 ↓
Table shape changed dramatically
 ↓
Statistics from before the load
 ↓
No longer representative
```

Best practice: explicitly refresh statistics after large bulk loads, rather than waiting for automatic/scheduled maintenance.

## 7️⃣ How to Inspect Statistics

Most databases expose statistics through system views/commands (implementation-specific). Checking these directly — rather than only looking at `EXPLAIN` output — helps confirm whether a bad plan is caused by stale statistics vs. something else entirely.

## 🧪 Hands-on Lab

**Lab 1** — Load skewed data, check the optimizer's row estimate for the common value vs. the rare value before any manual statistics refresh.
**Lab 2** — Bulk-update a large fraction of a table's rows, run a query, and observe estimate quality before and after manually refreshing statistics.
**Lab 3** — Inspect your database's statistics/histogram system view directly for a real table.

## 🎯 Interview Questions

**Why aren't average-based selectivity estimates good enough for skewed data?**
Because a single average obscures the actual shape of the distribution; histograms and most-common-value tracking let the optimizer estimate selectivity per actual value rather than per column-wide average.

**Why can statistics be wrong even without a bug?**
Because they are usually built from a sample, not the full table, and because data distribution can shift after the last statistics refresh (staleness).

**When should you manually refresh statistics?**
After large bulk loads, bulk updates, or any operation that dramatically changes the table's data distribution faster than automatic maintenance would catch it.

## 📌 Summary

```text
Rule 1 — Statistics are the raw material for every cost estimate.
Rule 2 — Histograms and MCVs capture distribution shape averages can't.
Rule 3 — Statistics are typically sampled, not exact.
Rule 4 — Bulk operations should trigger a manual statistics refresh.
```

---

# Chapter 80 — Join Algorithms: Nested Loop, Hash Join & Merge Join

> The optimizer doesn't just decide *what* to join — it decides *how*. Three classic algorithms dominate almost every relational database.

## Learning Objectives
- Nested Loop Join
- Hash Join
- Merge Join
- When each is chosen
- Why the "wrong" algorithm can be catastrophic

## 1️⃣ Nested Loop Join

```text
For each row in outer table:
    For each row in inner table:
        If join condition matches → emit
```

```text
Outer rows: small
Inner side: indexed lookup per outer row
 ↓
Very efficient
```

```text
Outer rows: large
Inner side: no index, scanned repeatedly
 ↓
Catastrophically slow (essentially O(outer × inner))
```

Nested Loop shines when the outer side is small and the inner side has a good index to probe.

## 2️⃣ Hash Join

```text
Build phase:
    Build a hash table from the smaller input, keyed on the join column

Probe phase:
    Scan the larger input, probe the hash table for matches
```

```text
Good for large, unsorted inputs
Requires enough memory for the build side
```

If the build side doesn't fit in memory, the database may spill to disk in batches — still usually far better than a bad Nested Loop, but slower than an in-memory hash join.

## 3️⃣ Merge Join

```text
Requires both inputs sorted (or sortable) on the join key
 ↓
Walk both sorted streams together, matching as you go
```

```text
Excellent when both sides are already sorted
(e.g., via an index that provides that order)
```

If neither side is naturally sorted, the database must sort both first — which can offset the benefit.

## 4️⃣ Why the Optimizer Picks One Over Another

```text
Small outer + indexed inner        → Nested Loop
Two large, unsorted inputs         → Hash Join
Both inputs already sorted         → Merge Join
```

But this is driven entirely by cost estimates — which loop back to cardinality estimation and statistics (Chapters 78–79). A bad row-count estimate on the "outer" side can make the optimizer choose Nested Loop when a Hash Join was actually far cheaper.

## 5️⃣ Real Failure Mode: Nested Loop on a Misestimated Outer

```text
Optimizer estimates outer = 100 rows
 ↓
Chooses Nested Loop
 ↓
Actual outer = 5,000,000 rows
 ↓
5,000,000 × inner-side cost
 ↓
Query takes minutes instead of milliseconds
```

This is one of the most common real production incidents tied to cardinality estimation errors.

## 🧪 Hands-on Lab

**Lab 1** — Join a small table to a large indexed table; confirm Nested Loop is chosen and observe its speed.
**Lab 2** — Join two large, unsorted tables with no useful index and observe Hash Join being chosen.
**Lab 3** — Join two tables where both sides already have a matching sorted index; check whether Merge Join is chosen.
**Lab 4** — Deliberately create a misestimated outer side (e.g., via stale statistics) and observe the optimizer wrongly choose Nested Loop; measure the runtime cost.

## 🎯 Interview Questions

**When is Nested Loop Join a good choice, and when is it terrible?**
Good when the outer side is small and the inner side can be probed via an efficient index; terrible when the outer side is large and the inner side requires a repeated scan per outer row.

**What does a Hash Join need to perform well?**
Enough memory to build a hash table from the smaller input; if it doesn't fit, the database spills to disk in batches, which is slower but usually still better than a bad Nested Loop.

**Why would a Merge Join be chosen over a Hash Join?**
When both inputs are already sorted on the join key (e.g., via existing indexes), avoiding the need to build a hash table or explicitly sort either side.

## 📌 Summary

```text
Rule 1 — Nested Loop favors small outer + indexed inner.
Rule 2 — Hash Join favors large, unsorted inputs with sufficient memory.
Rule 3 — Merge Join favors inputs that are already sorted.
Rule 4 — Wrong join algorithm choice is often a symptom of bad cardinality estimates.
```

---

# Chapter 81 — Sort Algorithms: In-Memory Sort, External Sort & Disk Spill

> A query that flies in development can crawl in production for one specific reason: the sort that fit comfortably in memory locally now has to spill to disk against real data volume.

## Learning Objectives
- In-memory sort
- External (disk-based) sort
- Why sort memory limits matter
- How ORDER BY, GROUP BY, and DISTINCT can all trigger sorts
- Diagnosing sort-related slowness

## 1️⃣ In-Memory Sort

```text
Rows fit within configured sort memory
 ↓
Sort entirely in RAM
 ↓
Fast
```

## 2️⃣ External Sort (Disk Spill)

```text
Rows exceed configured sort memory
 ↓
Sort in chunks
 ↓
Write sorted chunks to disk
 ↓
Merge chunks back together
 ↓
Much slower than in-memory sort
```

## 3️⃣ Why Development Doesn't Catch This

```text
Dev dataset: 10,000 rows → fits in memory easily
Production dataset: 50,000,000 rows → spills to disk
```

The query logic never changed. Only the data volume did — and sort memory configuration didn't scale with it.

## 4️⃣ Operations That Can Trigger a Sort

```text
ORDER BY (without a matching index)
GROUP BY (if not using hash aggregation)
DISTINCT
Merge Join (if inputs aren't already sorted)
Window functions with ORDER BY
```

## 5️⃣ Avoiding Unnecessary Sorts

```text
Query needs: ORDER BY created_at DESC LIMIT 20

Without a matching index:
    Read → Sort entire matching set → take top 20

With a matching index:
    Read already in order → take top 20 → no sort needed
```

This is one of the most impactful, low-risk optimizations available: matching an index to a common `ORDER BY ... LIMIT` pattern.

## 6️⃣ Diagnosing From EXPLAIN ANALYZE

```text
Sort
  Sort Method: external merge  Disk: 1,204,000kB
  Actual Time: 4,500ms
```

`external merge` with a nonzero disk figure is a direct signal: this sort spilled to disk. `Sort Method: quicksort` (in-memory) with no disk usage is the healthy case.

## 🧪 Hands-on Lab

**Lab 1** — Run an `ORDER BY` on a small table and confirm an in-memory sort in the plan.
**Lab 2** — Run the same style of query against a much larger table (or with reduced sort memory settings) and observe the switch to an external/disk-based sort.
**Lab 3** — Add an index matching the `ORDER BY` + `LIMIT` pattern and confirm the sort step disappears entirely from the plan.

## 🎯 Interview Questions

**Why can a query be fast in development but slow in production purely due to sorting?**
Because sort memory is fixed by configuration, and a sort that fits comfortably in memory against a small dev dataset can spill to disk against a much larger production dataset, which is dramatically slower.

**How can an index eliminate the need for an explicit sort step?**
If the index already stores rows in the order the query requests, the database can read directly in that order instead of reading unordered data and sorting it afterward.

**What in EXPLAIN ANALYZE tells you a sort spilled to disk?**
A sort method indicating external/disk-based merging along with a nonzero disk usage figure, as opposed to an in-memory sort method with no disk usage.

## 📌 Summary

```text
Rule 1 — Sorts that fit in memory are fast; sorts that spill to disk are not.
Rule 2 — Data volume growth alone can push a sort from memory to disk.
Rule 3 — ORDER BY, GROUP BY, DISTINCT, and some joins can all trigger sorts.
Rule 4 — A well-matched index can eliminate an explicit sort entirely.
```

---

# Chapter 82 — Predicate Pushdown: Filtering Data as Early as Possible

> One of the most powerful and least visible optimizations a database performs is moving your filters as close to the raw data as it possibly can.

## Learning Objectives
- What predicate pushdown means
- Pushdown through subqueries
- Pushdown through views
- Pushdown limits
- Predicate pushdown in joins

## 1️⃣ The Core Idea

```text
Filter late
 ↓
Read everything, then discard most of it

Filter early
 ↓
Never read what you don't need
```

## 2️⃣ Pushdown Through a Subquery

```sql
SELECT *
FROM (
    SELECT * FROM orders
) sub
WHERE customer_id = 101;
```

Rather than materializing all of `orders` and filtering afterward, the rewriter/optimizer can push the filter down:

```sql
SELECT * FROM orders WHERE customer_id = 101;
```

so the filter is applied at the earliest possible point — ideally letting an index be used directly.

## 3️⃣ Pushdown Through a View

```sql
CREATE VIEW paid_orders AS
SELECT * FROM orders WHERE status = 'PAID';

SELECT * FROM paid_orders WHERE customer_id = 101;
```

can conceptually become:

```sql
SELECT * FROM orders
WHERE status = 'PAID' AND customer_id = 101;
```

letting the optimizer consider both predicates together — potentially matching a composite index on `(customer_id, status)`.

## 4️⃣ Pushdown Into Joins

```sql
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.country = 'IN';
```

The `country = 'IN'` filter can often be applied to `customers` *before* the join, rather than joining everything and filtering afterward — dramatically reducing the size of one side of the join up front.

## 5️⃣ When Pushdown Is Blocked

```text
Aggregates
Window functions
Certain outer joins
Non-deterministic functions
LIMIT/OFFSET interacting with ordering
```

can all limit how aggressively a predicate can be pushed down, because moving the filter would change the result, not just the performance.

```sql
SELECT customer_id, SUM(amount)
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 1000;
```

`HAVING` filters *after* aggregation by definition — it cannot be pushed below the `GROUP BY` the way a `WHERE` clause on a raw column could.

## 🧠 Why This Matters to You Practically

```text
Wrapping a query in a subquery or view
does NOT automatically mean "slow" —
the optimizer will very often push filters through it.
```

But you should verify with `EXPLAIN`, not assume — some query shapes genuinely do block pushdown (Chapter 74 covered rewriting limits).

## 🧪 Hands-on Lab

**Lab 1** — Wrap a filtered query in a subquery and confirm via `EXPLAIN` whether the filter was pushed into the base table access.
**Lab 2** — Create a view with an embedded filter, query it with an additional filter, and check whether both predicates are applied together at the base table.
**Lab 3** — Construct a query where an aggregate or window function blocks pushdown, and observe the difference in the plan.

## 🎯 Interview Questions

**What is predicate pushdown?**
Moving a filter condition as close as possible to the raw data access, so unnecessary rows are eliminated as early as possible rather than being read and discarded later.

**Why doesn't wrapping a query in a subquery or view automatically hurt performance?**
Because the optimizer can often push outer filters down into the inner query, effectively flattening the structure before choosing a physical plan.

**Give an example of something that blocks predicate pushdown.**
A `HAVING` clause filtering on an aggregate result — since the filter depends on the aggregation having already happened, it cannot be pushed below the `GROUP BY`.

## 📌 Summary

```text
Rule 1 — Predicate pushdown filters data as early as possible.
Rule 2 — Views and subqueries are often transparent to pushdown.
Rule 3 — Aggregates, window functions, and some joins can block pushdown.
Rule 4 — Always verify pushdown behavior with EXPLAIN rather than assuming it.
```

---

# Chapter 83 — Sargability: Why Some Predicates Can Use Indexes and Others Can't

> "Sargable" (Search ARGument ABLE) is one of the most important — and most misunderstood — concepts in practical query tuning.

## Learning Objectives
- What sargability means
- Function-wrapped columns
- Implicit type coercion (revisited)
- Leading wildcards
- Rewriting non-sargable predicates into sargable ones

## 1️⃣ What "Sargable" Means

```text
Sargable predicate
 ↓
The database can navigate an index directly
using the predicate as a search boundary
```

```text
Non-sargable predicate
 ↓
The database must evaluate the expression
for every row, defeating a normal index
```

## 2️⃣ Function on the Indexed Column

```sql
WHERE DATE(created_at) = '2026-08-28';
```

Even with an index on `created_at`, the database generally can't use it directly, because it would have to compute `DATE(created_at)` for every row before it can compare — a normal B-Tree index doesn't store precomputed function results.

Sargable rewrite:

```sql
WHERE created_at >= '2026-08-28'
  AND created_at <  '2026-08-29';
```

Now the raw indexed column is compared directly.

## 3️⃣ Implicit Type Coercion (Revisited From Ch. 73)

```sql
WHERE phone_number = 5551234
```

If `phone_number` is text, this may force a per-row conversion, breaking sargability the same way a function call would.

## 4️⃣ Leading Wildcards

```sql
WHERE email LIKE '%vinay%';
```

Because the match can start anywhere in the string, a B-Tree index (ordered by leading characters) can't use its ordering to narrow the search — this typically falls back to a full scan.

```sql
WHERE email LIKE 'vinay%';
```

is sargable — the fixed prefix lets the index be searched like a range.

## 5️⃣ OR Conditions Across Different Columns

```sql
WHERE customer_id = 101 OR email = 'x@example.com';
```

Depending on the optimizer, this may or may not efficiently use separate indexes on each column — some databases combine index scans (bitmap-style), others fall back to a broader scan. Always verify.

## 6️⃣ Arithmetic on the Column

```sql
WHERE price * 1.18 > 1000;
```

is non-sargable for the same reason as a function call — rewrite as:

```sql
WHERE price > 1000 / 1.18;
```

so the raw column stays untouched and the constant absorbs the computation.

## 🧠 General Principle

```text
FUNCTION(indexed_column)  →  often non-sargable
indexed_column FUNCTION(constant)  →  often still sargable,
                                       because only the constant is computed
```

## 🧪 Hands-on Lab

**Lab 1** — Compare `WHERE DATE(created_at) = ...` against the equivalent range predicate; inspect `EXPLAIN` for both.
**Lab 2** — Compare `LIKE 'prefix%'` against `LIKE '%substring%'` on an indexed text column.
**Lab 3** — Rewrite an arithmetic-on-column predicate to move the computation to the constant side, and compare plans.

## 🎯 Interview Questions

**What does "sargable" mean?**
A predicate is sargable if the database can use it directly as a search boundary against an index, without having to evaluate a function or expression per row first.

**Why does `WHERE DATE(created_at) = '2026-08-28'` often fail to use an index on created_at?**
Because the database would need to compute `DATE(created_at)` for every row before comparing, since the index stores raw `created_at` values, not precomputed function results.

**How do you generally fix a non-sargable predicate?**
Move the computation to the constant/literal side of the comparison, or rewrite the condition as an equivalent range predicate on the raw column, so the indexed column itself is compared directly.

## 📌 Summary

```text
Rule 1 — Sargable predicates let the database navigate an index directly.
Rule 2 — Functions/arithmetic on the indexed column usually break sargability.
Rule 3 — Leading wildcards in LIKE usually break sargability.
Rule 4 — Prefer rewriting expressions to keep the raw column untouched.
```

---

# Chapter 84 — Real Production Query Incident: From 100ms to 8 Seconds

> Everything in this module comes together in a single realistic incident.

## The Incident

```text
API latency baseline: 100ms
Sudden change: 8,000ms (8 seconds)
```

## The Investigation Path

```text
API
 ↓
SQL
 ↓
Database
 ↓
EXPLAIN ANALYZE
```

## What Was Found

```text
Seq Scan
Rows: 10,000,000
```

Expected:

```text
Index Scan
Rows: 10
```

## Working Through the Possibilities

```text
Missing index?
 → Checked: index exists on the filtered column.

Statistics?
 → Checked: statistics were recently refreshed.

Data distribution changed?
 → Found: a recent bulk migration shifted a huge portion
   of rows into the value being filtered on, changing the
   column from high-selectivity to low-selectivity almost overnight.

Query changed?
 → Checked: no application code deploy occurred.

Wrong join order?
 → Not applicable — single table query.

Parameter-specific behavior?
 → Confirmed: a cached plan built for a rare parameter value
   was being reused for a now-common value.

Memory pressure?
 → Checked: no unusual pressure at time of incident.
```

## Root Cause

```text
Bulk data migration
 ↓
Column selectivity flipped
    (previously rare value → now common value)
 ↓
Optimizer's cached/previous plan assumption
   no longer matched reality
 ↓
Sequential scan became genuinely correct for the NEW distribution,
   but the endpoint was written assuming an index-driven fast path
```

## The Fix

```text
1. Refresh/validate statistics against the new distribution
2. Reconsider whether a plain index still made sense at the new selectivity
3. Consider a partial index targeting the now-rare complementary value
4. Add monitoring on selectivity/estimate drift, not just query latency
```

## 🧠 The Real Lesson

```text
The problem was never
"the optimizer is broken."

The problem was
"the data distribution changed underneath a query
 whose performance assumptions depended on that distribution."
```

This is why senior engineers treat query performance as a property of *data plus query plus statistics*, not the query text alone.

## 🎯 Interview Questions

**Why can a query become slow without any code changes at all?**
Because data distribution, statistics, or cardinality can shift underneath an unchanged query — a plan that was optimal for yesterday's data distribution may be poorly suited to today's.

**What's the first step in a real production slow-query investigation?**
Get the actual query and run `EXPLAIN ANALYZE` (or check historical plan data) to compare estimated vs. actual behavior before guessing at causes.

**Why isn't "add an index" always the right response to this kind of incident?**
Because the underlying cause might be a selectivity shift, stale statistics, or a stale cached plan — adding an index without diagnosing the actual cause can fail to fix the problem or even make writes worse without helping reads.

## 📌 Module 14 Critical Rules

```text
Rule 1  — Sequential scans are sometimes genuinely the cheapest plan.
Rule 2  — Cardinality estimation underlies every cost-based decision.
Rule 3  — Statistics quality determines estimate quality.
Rule 4  — Join algorithm choice depends heavily on row-count estimates.
Rule 5  — Sorts that spill to disk are a common prod-only slowdown.
Rule 6  — Predicate pushdown often makes subqueries/views performance-neutral.
Rule 7  — Non-sargable predicates silently defeat otherwise-good indexes.
Rule 8  — Data distribution changes can break a previously-fine query.
Rule 9  — Always diagnose with EXPLAIN ANALYZE before changing schema.
Rule 10 — Query performance is a property of data + query + statistics together.
```

---

# 🎉 Module 14 Complete

You now understand not just *that* an index exists, but the full reasoning chain the optimizer goes through:

```text
Query
 ↓
Scan vs Index decision
 ↓
Cardinality estimate
 ↓
Statistics quality
 ↓
Join algorithm choice
 ↓
Sort strategy
 ↓
Predicate pushdown
 ↓
Sargability
 ↓
Final plan
```

# 🚀 Next Module

# Module 15 — Storage Engine Internals
Pages/blocks, heap storage, buffer cache, memory management, disk I/O (sequential vs random), and SSD vs HDD implications for database performance.