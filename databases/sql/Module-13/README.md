# 🗄️ SQL Mastery — 20+ Years Experience
# Module 13 — Query Execution Engine

> This module explains what actually happens *after* you hit "run" on a SQL query — before any row of data is touched. It covers parsing, semantic analysis, rewriting, planning, and how to read an execution plan.

---

# Chapter 72 — SQL Parsing: Lexer, Parser, Parse Tree & Syntax Validation

> Before the database touches a single row, it must answer a basic question: is this even valid SQL? That's the parser's job.

## Learning Objectives
- What a lexer/tokenizer does
- What a parse tree is
- Syntax vs semantic errors
- Reserved words vs identifiers
- Why parsing cost matters at scale

## 1️⃣ Tokenization (Lexing)

```sql
SELECT name, email FROM users WHERE age > 18;
```

breaks into tokens:

```text
SELECT   → keyword
name     → identifier
,        → punctuation
email    → identifier
FROM     → keyword
users    → identifier
WHERE    → keyword
age      → identifier
>        → operator
18       → literal
;        → terminator
```

This stage understands structure, not meaning.

## 2️⃣ Parsing Into a Tree

```text
SELECT
 ├── columns: [name, email]
 ├── FROM: users
 └── WHERE
       └── age > 18
```

This parse tree (or AST) captures grammar, not schema truth.

## 3️⃣ Syntax Validation

```sql
SELCT name FROM users;        -- typo, syntax error
SELECT name FROM users WHERE; -- incomplete, syntax error
SELECT name, FROM users;      -- trailing comma, syntax error
```

None of these ever check whether `users` exists — they fail before that.

## 🧠 Syntax Error vs Semantic Error

```text
Syntax Error   → "This isn't valid SQL grammar."
Semantic Error → "Valid grammar, but the table/column doesn't exist,
                   or types don't match."
```

```sql
SELECT nonexistent_column FROM users;
```

The parser accepts this. The *next* stage (Chapter 73) rejects it.

## 4️⃣ Reserved Words vs Identifiers

```sql
SELECT order FROM orders;      -- fails: 'order' is reserved
SELECT "order" FROM orders;    -- works if quoted
```

```text
Keyword    → has special grammatical meaning
Identifier → refers to a table/column name
```

## 5️⃣ Parsing Is Purely Structural

At parse time the database does **not** know:

```text
Does "users" exist?
Does "age" exist?
Is age numeric?
Does the caller have permission?
```

It only knows the text matches valid SELECT-statement shape.

## 6️⃣ Prepared Statements

```sql
PREPARE get_user AS
SELECT * FROM users WHERE id = $1;
```

```text
Parse once
 ↓
Plan once (or per parameter shape)
 ↓
Execute many times
```

## 7️⃣ Parsing Cost at Scale

```text
50,000 unique dynamic query strings/sec
→ 50,000 parses/sec (expensive)

vs.

A handful of prepared statement shapes reused
→ Parse cost amortized
```

ORMs that generate wildly different SQL text for logically identical queries can quietly hurt performance through repeated parse/plan overhead — not query logic.

## 🧪 Hands-on Lab

**Lab 1** — Run `SELECT FROM users;` and read the exact parser error.
**Lab 2** — Create a column named `order` (quoted), query it unquoted, observe failure.
**Lab 3** — Time 1,000 unique dynamic queries vs. 1 prepared statement executed 1,000 times.

## 🎯 Interview Questions

**What does a SQL parser do?**
Converts raw SQL text into tokens, then into a structured tree the later stages can process.

**Syntax error vs semantic error?**
Syntax = doesn't match grammar at all. Semantic = grammatically valid but references something invalid (nonexistent column, wrong type, etc.).

**Why do prepared statements help performance?**
They let the database skip repeated parsing (and often planning) for logically identical queries executed repeatedly.

## 📌 Summary Rules

```text
Rule 1 — Parsing only checks grammar, not meaning.
Rule 2 — Reserved words can silently break unquoted identifiers.
Rule 3 — Prepared statements avoid repeated parse/plan cost.
Rule 4 — Parsing is the first of many stages a query passes through.
```

---

# Chapter 73 — Semantic Analysis: Table Existence, Column Resolution, Type Checking & Permissions

> The parse tree is grammatically valid. Now the database must check whether it's actually *true* — does this table exist, does this column exist, are the types compatible, is the user even allowed to do this?

## Learning Objectives
- Table/column resolution
- Type checking and implicit coercion
- Ambiguous column resolution in joins
- Permission checks
- Where semantic errors surface

## 1️⃣ What Semantic Analysis Checks

```text
Does table exist?
Does column exist?
Are types compatible?
Does user have permission?
Is the query even meaningful given the schema?
```

## 2️⃣ Table & Column Resolution

```sql
SELECT nam FROM users;
```

Parser: fine. Semantic analyzer: `nam` doesn't exist on `users` — error, often with a "did you mean `name`?" suggestion in modern databases.

## 3️⃣ Ambiguous Columns in Joins

```sql
SELECT id
FROM users u
JOIN orders o ON u.id = o.user_id;
```

If both `users` and `orders` have an `id` column, an unqualified `id` is ambiguous:

```text
users.id
   vs
orders.id
```

The analyzer must reject this or force qualification (`u.id`).

## 4️⃣ Type Checking

```sql
SELECT *
FROM orders
WHERE amount = 'abc';
```

`amount` is numeric; `'abc'` is text. Depending on the database, this may error outright or attempt implicit coercion — which itself can silently break index usage (related to Chapter 83, Sargability).

## 5️⃣ Implicit Coercion Danger

```sql
WHERE phone_number = 5551234
```

If `phone_number` is stored as text, this comparison may force a conversion on every row — turning a potentially indexed equality lookup into something the optimizer can't use efficiently.

## 6️⃣ Permission Checks

```sql
SELECT * FROM payroll;
```

Semantic analysis (or a closely related authorization stage) verifies the current user has `SELECT` privilege on `payroll`. If not:

```text
permission denied for table payroll
```

This happens before any row is read — not as a runtime filter.

## 7️⃣ View Resolution

```sql
SELECT * FROM active_customers;
```

If `active_customers` is a view, the analyzer must resolve its definition, effectively substituting the underlying query, then re-validate the combined structure.

## 🧠 Why This Stage Exists Separately From Parsing

```text
Parser
 ↓
"Is this valid SQL shape?"

Semantic Analyzer
 ↓
"Is this valid SQL *for this specific database, schema, and user*?"
```

The same syntactically perfect query can be semantically valid against one schema and invalid against another.

## 🧪 Hands-on Lab

**Lab 1** — Query a column that doesn't exist; read the error and any suggestion offered.
**Lab 2** — Create two tables sharing a column name, join them, select the shared column unqualified — observe ambiguity error.
**Lab 3** — Compare a numeric column filtered with a string literal vs. a properly typed literal; inspect `EXPLAIN` for any difference in plan.
**Lab 4** — Revoke `SELECT` on a table for a test user and attempt a query.

## 🎯 Interview Questions

**What does semantic analysis check that parsing doesn't?**
Whether referenced tables/columns exist, whether types are compatible, and whether the user has permission — all schema- and user-specific facts the parser can't know.

**Why can implicit type coercion hurt performance?**
Because comparing a column against a mismatched literal type may force a conversion on every row, which can prevent the optimizer from using a normal index on that column.

**Why must ambiguous column references be rejected?**
Because with multiple tables sharing a column name, the database cannot safely guess which one you meant.

## 📌 Summary Rules

```text
Rule 1 — Semantic analysis validates meaning against the real schema.
Rule 2 — Type mismatches can silently defeat indexes via coercion.
Rule 3 — Ambiguous column names in joins must be qualified.
Rule 4 — Permission checks happen before any row is read.
```

---

# Chapter 74 — Query Rewriting: View Expansion, Predicate Transformation & Simplification

> Before the optimizer chooses a physical plan, many databases first *rewrite* your query into a logically equivalent but more optimizer-friendly form.

## Learning Objectives
- View expansion
- Predicate transformation
- Constant folding and simplification
- Subquery-to-join transformations
- Why rewriting is invisible but important

## 1️⃣ What Is Query Rewriting?

```text
Original Query
 ↓
Rewriter
 ↓
Logically Equivalent Query
 ↓
(passed to the optimizer)
```

The rewritten query returns the same result — but may be structured very differently internally.

## 2️⃣ View Expansion

```sql
CREATE VIEW active_customers AS
SELECT * FROM customers WHERE status = 'ACTIVE';

SELECT name FROM active_customers WHERE country = 'IN';
```

The rewriter typically expands this into something conceptually like:

```sql
SELECT name
FROM customers
WHERE status = 'ACTIVE'
  AND country = 'IN';
```

The two predicates are merged, so the optimizer can consider them together — e.g., a composite index on `(status, country)` becomes usable, whereas neither the view nor the outer query alone made that obvious.

## 3️⃣ Predicate Pushdown (Preview)

```sql
SELECT *
FROM (
    SELECT * FROM orders
) sub
WHERE customer_id = 101;
```

Rather than materializing all of `orders` and then filtering, the rewriter can push `customer_id = 101` down into the inner query:

```sql
SELECT * FROM orders WHERE customer_id = 101;
```

(Full treatment in Chapter 82.)

## 4️⃣ Constant Folding / Simplification

```sql
WHERE 1 = 1 AND status = 'ACTIVE'
```

may be simplified to:

```sql
WHERE status = 'ACTIVE'
```

```sql
WHERE age > 10 + 8
```

may be folded to:

```sql
WHERE age > 18
```

so the expression doesn't need to be recomputed per row.

## 5️⃣ Subquery-to-Join Transformation

```sql
SELECT *
FROM customers
WHERE id IN (
    SELECT customer_id FROM orders WHERE status = 'PAID'
);
```

Many optimizers can rewrite this into a semantically equivalent join or semi-join form, opening up join algorithms (hash join, merge join) that a naive subquery execution wouldn't consider.

## 6️⃣ OR-to-UNION Transformations

```sql
SELECT * FROM orders
WHERE customer_id = 101 OR customer_id = 202;
```

Depending on the database, this might be handled directly, or in some cases rewritten toward a form that allows two separate index lookups combined, rather than one broad scan.

## 🧠 Why This Stage Matters to You

You rarely see the rewritten query directly. But it explains:

```text
Why does using a view sometimes perform identically
to writing the equivalent raw SQL?

Why does wrapping a query in a subquery sometimes
NOT hurt performance, even though it "looks" nested?
```

Answer: because the rewriter often flattens/merges these structures before the optimizer ever sees them.

## 🧠 Rewriting Has Limits

Not every logically equivalent transformation is guaranteed. Complex subqueries, certain aggregate placements, or side-effect-bearing expressions can block rewriting, forcing a less efficient literal execution. This is why two seemingly-equivalent queries can perform very differently — always check `EXPLAIN`, don't assume the rewriter caught it.

## 🧪 Hands-on Lab

**Lab 1** — Create a view with a `WHERE` clause, query it with an additional filter, and compare `EXPLAIN` output against the manually combined query.
**Lab 2** — Write a query with `IN (SELECT ...)` and compare its plan to an equivalent `JOIN`.
**Lab 3** — Nest a simple filter inside a subquery wrapper and confirm (via `EXPLAIN`) whether the predicate was pushed down.

## 🎯 Interview Questions

**What does the query rewriter do?**
Transforms a query into a logically equivalent form that's easier for the optimizer to reason about — expanding views, pushing down predicates, folding constants, and sometimes converting subqueries into joins.

**Why might a view not hurt performance even though it wraps another query?**
Because the rewriter can expand the view and merge its predicates with the outer query before optimization, rather than executing it as a separate materialized step.

**Is query rewriting guaranteed to happen for every logically equivalent transformation?**
No — rewriting has limits; certain query shapes block it, so identical-looking queries can still perform differently.

## 📌 Summary Rules

```text
Rule 1 — Rewriting produces a logically equivalent, optimizer-friendly query.
Rule 2 — Views are often expanded and merged with outer predicates.
Rule 3 — Predicate pushdown moves filters as close to the data as possible.
Rule 4 — Rewriting isn't guaranteed for every query shape — verify with EXPLAIN.
```

---

# Chapter 75 — Query Planner / Optimizer: Turning a Rewritten Query Into a Plan

> The rewritten query says *what* you want. The optimizer decides *how* to get it — which indexes, which join order, which join algorithm, and in what sequence.

## Learning Objectives
- Logical plan vs physical plan
- What inputs the optimizer uses
- Join order search space
- Why optimization is a search problem, not a lookup
- Plan caching basics

## 1️⃣ Logical Plan vs Physical Plan

```text
Logical Plan
 ↓
"Filter orders, join to customers, aggregate."
(what, not how)

Physical Plan
 ↓
"Index Scan on orders.customer_id,
 Hash Join with customers,
 HashAggregate."
(concrete execution strategy)
```

The same logical plan can map to many different physical plans.

## 2️⃣ What the Optimizer Considers

```text
Indexes
Join order
Join algorithms
Statistics
Cardinality estimates
Sort requirements
Filtering
Available memory
Parallelism
```

## 3️⃣ Join Order Is a Search Problem

```sql
SELECT *
FROM a
JOIN b ON a.id = b.a_id
JOIN c ON b.id = c.b_id
JOIN d ON c.id = d.c_id;
```

Even with just 4 tables, there are many possible join orders:

```text
(((a ⋈ b) ⋈ c) ⋈ d)
((a ⋈ b) ⋈ (c ⋈ d))
(((a ⋈ c) ⋈ b) ⋈ d)
...
```

As table count grows, the number of possible orderings grows combinatorially. Real optimizers use heuristics, dynamic programming, or genetic/greedy search to avoid exhaustively trying every combination.

## 4️⃣ Cost Estimation Drives the Choice

For each candidate plan, the optimizer estimates:

```text
Estimated rows processed
Estimated I/O
Estimated CPU
Estimated memory
```

and picks the plan with the lowest estimated total cost — not necessarily the "obviously correct" one to a human.

## 5️⃣ Why Two "Equivalent" Queries Can Get Different Plans

```sql
SELECT * FROM orders WHERE customer_id = 101;
```

vs.

```sql
SELECT * FROM orders WHERE customer_id = ANY(ARRAY[101]);
```

Even though these are logically the same, different databases may or may not treat them identically during planning, depending on rewrite/normalization rules. Always verify — don't assume syntactic variations plan identically.

## 6️⃣ Plan Caching & Parameter Sniffing (Preview)

```sql
PREPARE q AS
SELECT * FROM orders WHERE customer_id = $1;
```

Some databases cache a plan after the first execution and reuse it for subsequent calls with different parameter values. If the first parameter value was atypical (e.g., a customer with 1 order vs. one with 10 million), the cached plan may be poorly suited to later calls.

```text
First execution
 ↓
Plan cached based on that parameter's characteristics
 ↓
Later execution with very different parameter
 ↓
Potentially bad plan reused
```

This is called **parameter sniffing** and is covered in depth alongside statistics (Chapter 78/79).

## 7️⃣ The Optimizer Is Not Omniscient

```text
Good statistics → good estimates → good plan
Bad statistics  → bad estimates  → bad plan
```

The optimizer only reasons as well as the information it's given. This is why "the optimizer picked a bad plan" is very often actually "the statistics were stale or the estimate was wrong," not a bug.

## 🧠 Senior Mental Model

```text
SQL
 ↓
Rewriter
 ↓
Logical Plan
 ↓
Search over physical plan candidates
 ↓
Cost estimation per candidate
 ↓
Cheapest plan chosen
 ↓
Executor
```

## 🧪 Hands-on Lab

**Lab 1** — Join 4+ tables and inspect `EXPLAIN` for the chosen join order; try forcing a different order (where your database allows join-order hints) and compare estimated cost.
**Lab 2** — Prepare a statement with a skewed parameter distribution, execute once with an atypical value, then with a typical value — check whether the plan changes or gets reused.
**Lab 3** — Deliberately go stale on statistics (insert a huge skewed batch without updating stats) and observe a bad plan choice; then refresh statistics and re-check.

## 🎯 Interview Questions

**What's the difference between a logical plan and a physical plan?**
A logical plan describes *what* operations are needed (filter, join, aggregate); a physical plan specifies *how* — which concrete algorithms and access paths are used.

**Why is join ordering considered a search problem?**
Because the number of valid orderings grows combinatorially with the number of joined tables, so the optimizer must use heuristics or bounded search rather than exhaustive enumeration.

**What is parameter sniffing and why can it cause problems?**
When a database caches a plan based on the first parameter value seen, and that plan is later reused for very different parameter values whose optimal plan would differ, causing poor performance.

## 📌 Summary Rules

```text
Rule 1 — The optimizer searches for the cheapest plan, not the "obvious" one.
Rule 2 — Join order matters and grows combinatorially with table count.
Rule 3 — Cost estimates depend entirely on the quality of statistics.
Rule 4 — Cached plans can go stale relative to new parameter values.
```

---

# Chapter 76 — Execution Plans: Reading EXPLAIN and EXPLAIN ANALYZE

> This is the single most important practical skill in this module: reading a real execution plan and understanding what the database actually did.

## Learning Objectives
- EXPLAIN vs EXPLAIN ANALYZE
- Common plan operators
- Reading estimated vs actual rows
- Spotting the expensive operator
- A repeatable workflow for diagnosing slow queries

## 1️⃣ EXPLAIN vs EXPLAIN ANALYZE

```text
EXPLAIN
 ↓
Shows the plan the optimizer WOULD use
 ↓
Does not execute the query

EXPLAIN ANALYZE
 ↓
Actually executes the query
 ↓
Shows the plan AND real runtime numbers
```

⚠️ `EXPLAIN ANALYZE` on `UPDATE`/`DELETE`/`INSERT` actually performs the write. Never run it blindly against production data-modifying statements.

## 2️⃣ Common Plan Operators

```text
Seq Scan / Table Scan   → reads all/most rows of a table
Index Scan              → navigates an index, then fetches rows
Index-Only Scan         → answers entirely from the index
Bitmap Scan             → builds a row-location bitmap, then fetches
Nested Loop             → for each outer row, probe inner side
Hash Join               → build a hash table, probe with the other side
Merge Join              → both sides sorted, merged together
Sort                    → explicit sorting step
Aggregate / HashAggregate → grouping/aggregation
Materialize             → buffers intermediate results for reuse
```

## 3️⃣ Reading a Simple Plan

```text
Seq Scan on orders
Rows Removed by Filter: 499,999,000
Rows Returned: 1
```

Translation:

```text
500M rows scanned
 ↓
Only 1 row matched
 ↓
Likely missing a useful index
```

## 4️⃣ A Healthier Plan

```text
Index Scan
Index Cond: customer_id = 101
Rows: 50
```

```text
500M rows
 ↓
narrowed to 50 candidates via the index
```

This is the filtering behavior an index should provide.

## 5️⃣ Estimated vs Actual Rows — The Most Important Comparison

```text
Estimated rows: 100
Actual rows: 1,000,000
```

A 10,000x gap between estimate and reality is a strong signal:

```text
Stale statistics
Bad cardinality estimate
Wrong join algorithm chosen
Wrong plan overall
```

Small gaps are normal and not worth chasing. Large orders-of-magnitude gaps are worth investigating.

## 6️⃣ Index Scan Doesn't Mean Fast

```text
Index Scan
Rows: 20,000,000
```

The index exists and is being used — but returning 20 million rows through an index, with a table lookup per row, can still be very slow. The presence of "Index Scan" in a plan is not a guarantee of good performance.

## 7️⃣ Finding the Expensive Operator

In a real `EXPLAIN ANALYZE` output, look for:

```text
Which operator has the largest actual time?
Which operator processes the most rows?
Is there an unexpected Sort spilling to disk?
Is there a Nested Loop over a large row count?
```

The slowest single operator is usually where your optimization effort should go first — not the query as a whole.

## 8️⃣ A Repeatable Diagnostic Workflow

```text
Step 1 — Get the real, exact query (not an approximation)
Step 2 — Run EXPLAIN ANALYZE (read-only queries only)
Step 3 — Compare estimated vs actual rows at each node
Step 4 — Identify the single most expensive operator
Step 5 — Form a hypothesis (missing index? bad stats? wrong join?)
Step 6 — Make one change
Step 7 — Re-run and compare
Step 8 — Document before/after
```

## 🧠 Senior Mental Model

```text
Don't read a plan top-to-bottom like prose.
Read it looking for the biggest actual-time contributor,
then work outward from there.
```

## 🧪 Hands-on Lab

**Lab 1** — Run `EXPLAIN` (no ANALYZE) on a query and describe, in plain English, what the plan claims it will do — before ever running it for real.
**Lab 2** — Run `EXPLAIN ANALYZE` on the same query and compare estimated vs. actual rows at each step.
**Lab 3** — Take a query with a large actual-time Sort operator; add an index that could satisfy the ordering; compare plans before/after.
**Lab 4** — Deliberately create a query with a Nested Loop over a large outer row count; identify why it's expensive from the plan alone.

## 🎯 Interview Questions

**What's the key difference between EXPLAIN and EXPLAIN ANALYZE?**
`EXPLAIN` shows the planned strategy without running the query; `EXPLAIN ANALYZE` actually executes it and reports real timing and row counts alongside the plan.

**Why is "Index Scan" in a plan not automatically a sign of good performance?**
Because an index scan can still return a huge number of rows, each requiring a table lookup — the presence of an index doesn't guarantee a small or cheap result set.

**What should you look for first when reading an EXPLAIN ANALYZE plan?**
The operator with the largest actual time/row count, and the size of the gap between estimated and actual rows — both point toward where the real problem lives.

## 📌 Summary Rules

```text
Rule 1 — EXPLAIN plans; EXPLAIN ANALYZE executes and measures.
Rule 2 — Never run EXPLAIN ANALYZE on production writes casually.
Rule 3 — Large estimated-vs-actual gaps signal bad statistics or bad plans.
Rule 4 — An index scan is not automatically fast.
Rule 5 — Diagnose by finding the single most expensive operator, not by reading top-to-bottom.
```

---

# 🎉 Module 13 Complete

You now understand the full journey from raw SQL text to an executable plan:

```text
SQL
 ↓
Lexer / Parser
 ↓
Parse Tree
 ↓
Semantic Analysis
 ↓
Query Rewriter
 ↓
Optimizer (Logical → Physical Plan)
 ↓
Execution Plan
 ↓
EXPLAIN / EXPLAIN ANALYZE
```

# 🚀 Next Module

# Module 14 — Query Optimization
Full table scans, cardinality estimation, statistics/histograms, join algorithms (Nested Loop, Hash, Merge), sort algorithms and disk spill, predicate pushdown, and sargability.