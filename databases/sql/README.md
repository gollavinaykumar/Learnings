# 🗄️ SQL Mastery — Complete Roadmap

### Beginner → Advanced → Database Internals → Performance → Production → Distributed Systems → Architecture

> A comprehensive roadmap to master SQL and relational databases from fundamentals to **20+ years of senior/principal-level database engineering**.
>
> The goal is not to memorize SQL syntax.
>
> The goal is to understand **how databases work internally, why queries become slow, how transactions and concurrency work, how storage engines operate, how databases scale, and how to design production systems handling billions of rows and thousands of concurrent requests.**

---

# 📚 Table of Contents

* [Course Overview](#-course-overview)
* [Learning Philosophy](#-learning-philosophy)
* [Module Index](#-module-index)
* [Module 1 — Database Fundamentals](#-module-1--database-fundamentals)
* [Module 2 — SQL Fundamentals](#-module-2--sql-fundamentals)
* [Module 3 — Filtering, Sorting & Expressions](#-module-3--filtering-sorting--expressions)
* [Module 4 — Joins & Relational Algebra](#-module-4--joins--relational-algebra)
* [Module 5 — Aggregation & Analytical SQL](#-module-5--aggregation--analytical-sql)
* [Module 6 — Subqueries & CTEs](#-module-6--subqueries--ctes)
* [Module 7 — Window Functions](#-module-7--window-functions)
* [Module 8 — Database Design](#-module-8--database-design)
* [Module 9 — Constraints & Data Integrity](#-module-9--constraints--data-integrity)
* [Module 10 — Transactions & ACID](#-module-10--transactions--acid)
* [Module 11 — Concurrency & Locking](#-module-11--concurrency--locking)
* [Module 12 — Indexes](#-module-12--indexes)
* [Module 13 — Query Execution Engine](#-module-13--query-execution-engine)
* [Module 14 — Query Optimization](#-module-14--query-optimization)
* [Module 15 — Storage Engine Internals](#-module-15--storage-engine-internals)
* [Module 16 — MVCC & Isolation](#-module-16--mvcc--isolation)
* [Module 17 — WAL, Logging & Recovery](#-module-17--wal-logging--recovery)
* [Module 18 — Partitioning](#-module-18--partitioning)
* [Module 19 — Replication](#-module-19--replication)
* [Module 20 — High Availability](#-module-20--high-availability)
* [Module 21 — Database Scaling](#-module-21--database-scaling)
* [Module 22 — Distributed Databases](#-module-22--distributed-databases)
* [Module 23 — Production Database Engineering](#-module-23--production-database-engineering)
* [Module 24 — Database Security](#-module-24--database-security)
* [Module 25 — Observability & Troubleshooting](#-module-25--observability--troubleshooting)
* [Module 26 — Cloud Databases](#-module-26--cloud-databases)
* [Module 27 — NoSQL & Polyglot Persistence](#-module-27--nosql--polyglot-persistence)
* [Module 28 — Database Architecture](#-module-28--database-architecture)
* [Module 29 — Large-Scale System Design](#-module-29--large-scale-system-design)
* [Module 30 — Staff / Principal Database Engineering](#-module-30--staff--principal-database-engineering)
* [Final Projects](#-final-projects)
* [Interview Preparation](#-interview-preparation)
* [Final Outcome](#-final-outcome)

---

## 🔗 Module Index

| Module | Link | Focus |
|--------|------|-------|
| Module 1 - 12 | [Learning set](./Module%201%20-%2012/) | SQL basics, joins, aggregation, filters, and subqueries |
| Module 13 | [README](./Module-13/README.md) | Data modification and production SQL writes |
| Module 14 | [README](./Module-14%20/README.md) | Database design and advanced SQL concepts |
| Module 15 | [README](./Module-15/README.md) | Indexing and query performance |
| Module 16 | [README](./Module-16/README.md) | Concurrency, isolation, and locking |
| Module 17 | [README](./Module-17/README.md) | Recovery, WAL, and crash consistency |
| Module 18 | [README](./Module-18/README.md) | Partitioning and table scaling |
| Module 19 | [README](./Module-19/README.md) | Replication and data distribution |
| Module 20 | [README](./Module-20/README.md) | High availability and production operations |
| Module 21 | [README](./Module-21/README.md) | Database scaling and architecture |

---

# 🎯 Course Overview

SQL is often taught as:

```text
SELECT
INSERT
UPDATE
DELETE
JOIN
GROUP BY
```

That is only the beginning.

A production database is much more complicated.

When an application sends:

```sql
SELECT *
FROM orders
WHERE user_id = 100;
```

the database may internally perform:

```text
SQL
 ↓
Parser
 ↓
Analyzer
 ↓
Query Rewriter
 ↓
Optimizer
 ↓
Execution Plan
 ↓
Executor
 ↓
Buffer Cache
 ↓
Storage Engine
 ↓
Indexes / Tables
 ↓
Disk
```

And when multiple users execute queries simultaneously:

```text
Transaction A
        ↓
      Database
        ↑
Transaction B
```

the database must deal with:

```text
Locks
MVCC
Isolation
Deadlocks
Transactions
WAL
Recovery
```

At large scale:

```text
Application
      ↓
Load Balancer
      ↓
Database Cluster
      ↓
Primary
 ↙    ↓    ↘
Replica Replica Replica
```

Eventually:

```text
One Database
      ↓
Partitioning
      ↓
Replication
      ↓
Sharding
      ↓
Distributed Database
```

This roadmap takes you through that entire journey.

---

# 🧠 Learning Philosophy

Don't learn SQL as a collection of commands.

For every topic ask:

### Question 1

```text
What problem does this solve?
```

### Question 2

```text
How does the database implement it?
```

### Question 3

```text
What happens internally?
```

### Question 4

```text
What happens when 1 million users do it simultaneously?
```

### Question 5

```text
What happens when the database crashes?
```

### Question 6

```text
How do we debug it in production?
```

### Question 7

```text
When does this approach stop scaling?
```

That mindset separates:

```text
SQL Developer
```

from:

```text
Database Engineer
```

and eventually:

```text
Database Architect
```

---

# 📦 Module 1 — Database Fundamentals

> Understand why databases exist and the fundamental problems they solve.

## Chapter 1 — Before Databases

Learn:

* File-based storage
* Flat files
* CSV
* JSON
* XML
* File locking
* Data duplication
* Data consistency problems
* Concurrent access problems

### Real Software Problem

Imagine an application stores:

```text
10 million users
```

inside:

```text
users.csv
```

Now the application needs:

```text
Find user by email
```

A naive implementation may scan:

```text
Row 1
Row 2
Row 3
...
Row 10,000,000
```

Questions:

* How do we make lookup fast?
* What happens when 100 users update simultaneously?
* What happens if the process crashes during an update?
* How do multiple applications share the same data?

These are the problems databases solve.

---

# Chapter 2 — What Is a Database?

Understand:

* Database
* DBMS
* RDBMS
* Relational database
* Table
* Row
* Column
* Schema
* Database instance

Learn the difference between:

```text
Database
```

and:

```text
Database Management System
```

---

# Chapter 3 — Relational Model

Learn:

```text
Relations
Tuples
Attributes
Domains
Keys
Relationships
```

Understand:

```text
Customer
   ↓
Orders
   ↓
Order Items
   ↓
Products
```

This becomes the foundation for relational database design.

---

# Chapter 4 — SQL vs Database Engine

Understand that:

```sql
SELECT *
FROM users;
```

is only a request.

The database engine decides:

```text
How to execute it
```

This distinction becomes extremely important later.

---

# 📦 Module 2 — SQL Fundamentals

> Master the SQL language before moving into database internals.

## Chapter 5 — SELECT

Learn:

```sql
SELECT
FROM
WHERE
ORDER BY
LIMIT
DISTINCT
```

---

# Chapter 6 — INSERT

Learn:

```sql
INSERT INTO
```

Single-row inserts:

```sql
INSERT INTO users(name, email)
VALUES ('Vinay', 'vinay@example.com');
```

Multiple-row inserts:

```sql
INSERT INTO users(name, email)
VALUES
('Vinay', 'vinay@example.com'),
('Rahul', 'rahul@example.com');
```

---

# Chapter 7 — UPDATE

Learn:

```sql
UPDATE
SET
WHERE
```

Understand the danger of:

```sql
UPDATE users
SET status = 'ACTIVE';
```

without:

```sql
WHERE
```

Real production problem:

```text
Developer forgot WHERE
        ↓
Millions of rows modified
```

Learn how to prevent such incidents.

---

# Chapter 8 — DELETE

Learn:

```sql
DELETE
TRUNCATE
DROP
```

Understand the difference.

---

# Chapter 9 — NULL

Understand:

```text
NULL
```

is not:

```text
0
```

and not:

```text
''
```

Learn:

```sql
IS NULL
IS NOT NULL
COALESCE
NULLIF
```

Understand SQL's three-valued logic:

```text
TRUE
FALSE
UNKNOWN
```

---

# 📦 Module 3 — Filtering, Sorting & Expressions

## Chapter 10 — Operators

Learn:

```text
=
<>
!=
>
<
>=
<=
BETWEEN
IN
LIKE
IS NULL
```

---

# Chapter 11 — Boolean Logic

Understand:

```text
AND
OR
NOT
```

and operator precedence.

Example:

```sql
WHERE
    status = 'ACTIVE'
    AND age > 18
    OR is_admin = true;
```

Understand why parentheses matter.

---

# Chapter 12 — CASE Expressions

Learn:

```sql
CASE
    WHEN ...
    THEN ...
    ELSE ...
END
```

Real problem:

```text
Convert numeric status codes
into business-readable categories.
```

---

# Chapter 13 — String / Date / Numeric Functions

Learn:

```text
String functions
Date functions
Numeric functions
Conversion functions
Conditional functions
```

Focus on writing portable SQL where practical, while learning database-specific features for the database you use.

---

# 📦 Module 4 — Joins & Relational Algebra

> Joins are one of the most important concepts in SQL.

## Chapter 14 — INNER JOIN

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

Understand:

```text
users
   +
orders
   ↓
Matching rows
```

---

# Chapter 15 — LEFT JOIN

Real problem:

> Show every customer, even customers who have never placed an order.

```sql
SELECT
    u.name,
    o.amount
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id;
```

---

# Chapter 16 — RIGHT JOIN

Understand why it exists and when it can be rewritten using `LEFT JOIN`.

---

# Chapter 17 — FULL OUTER JOIN

Useful for:

```text
Data reconciliation
```

Example:

```text
System A
   vs
System B
```

Find:

```text
Only in A
Only in B
In both
```

---

# Chapter 18 — CROSS JOIN

Understand:

```text
Cartesian Product
```

and why accidental Cartesian joins can destroy performance.

---

# Chapter 19 — SELF JOIN

Real problem:

```text
Employee
   ↓
Manager
   ↓
Department Head
```

Learn hierarchical relationships.

---

# Chapter 20 — Relational Algebra

Understand the concepts behind:

```text
Selection
Projection
Join
Union
Intersection
Difference
Cartesian Product
```

Now SQL stops being just syntax.

You begin understanding the mathematical model underneath relational databases.

---

# 📦 Module 5 — Aggregation & Analytical SQL

## Chapter 21 — GROUP BY

```sql
SELECT
    user_id,
    SUM(amount)
FROM orders
GROUP BY user_id;
```

Real problems:

* Revenue by customer
* Sales by product
* Orders by month
* Employees by department

---

# Chapter 22 — Aggregate Functions

Learn:

```text
COUNT
SUM
AVG
MIN
MAX
```

Understand:

```sql
COUNT(*)
```

vs

```sql
COUNT(column)
```

especially when `NULL` values exist.

---

# Chapter 23 — HAVING

Understand the difference:

```text
WHERE
```

filters rows.

```text
HAVING
```

filters groups.

---

# Chapter 24 — DISTINCT

Learn when:

```sql
DISTINCT
```

is useful and when it is hiding a modeling or join problem.

---

# Chapter 25 — Conditional Aggregation

Example:

```sql
SELECT
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE status = 'PAID') AS paid
FROM orders;
```

or database-specific equivalents.

Real problem:

```text
Generate dashboards with multiple metrics
in one query.
```

---

# 📦 Module 6 — Subqueries & CTEs

## Chapter 26 — Scalar Subqueries

```sql
SELECT
    name,
    (SELECT COUNT(*) FROM orders) AS total_orders
FROM users;
```

---

# Chapter 27 — IN / EXISTS

Understand:

```sql
IN
EXISTS
NOT EXISTS
```

Real problem:

> Find customers who have placed at least one order.

---

# Chapter 28 — Correlated Subqueries

Example:

```sql
SELECT *
FROM employees e
WHERE salary >
(
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

Question:

> Why can correlated subqueries become expensive?

This leads naturally into query optimization.

---

# Chapter 29 — Common Table Expressions

Learn:

```sql
WITH
```

Example:

```sql
WITH customer_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total
    FROM orders
    GROUP BY user_id
)
SELECT *
FROM customer_totals
WHERE total > 100000;
```

---

# Chapter 30 — Recursive CTEs

Real problem:

```text
CEO
 ↓
VP
 ↓
Manager
 ↓
Employee
```

Learn how SQL can traverse hierarchical structures.

---

# 📦 Module 7 — Window Functions

> Window functions are essential for advanced SQL and analytics.

## Chapter 31 — ROW_NUMBER

```sql
ROW_NUMBER() OVER (...)
```

Real problem:

> Get the latest order for every customer.

---

# Chapter 32 — RANK

```sql
RANK() OVER (...)
```

---

# Chapter 33 — DENSE_RANK

Understand the difference:

```text
ROW_NUMBER
RANK
DENSE_RANK
```

---

# Chapter 34 — PARTITION BY

Understand:

```text
GROUP BY
```

vs:

```text
PARTITION BY
```

---

# Chapter 35 — LAG / LEAD

Real problems:

```text
Compare today's sales
with yesterday's sales.
```

or:

```text
Find previous transaction
```

---

# Chapter 36 — Running Totals

Learn:

```sql
SUM(amount) OVER (
    ORDER BY created_at
)
```

---

# Chapter 37 — Advanced Window Frames

Learn:

```text
ROWS
RANGE
GROUPS
```

Understand exactly which rows belong to the window.

---

# 📦 Module 8 — Database Design

> Move from writing queries to designing systems.

## Chapter 38 — Entity Relationship Modeling

Learn:

```text
Entity
Attribute
Relationship
Cardinality
Optionality
```

---

# Chapter 39 — One-to-One

Example:

```text
User
 ↓
User Profile
```

---

# Chapter 40 — One-to-Many

Example:

```text
Customer
   ↓
Orders
```

---

# Chapter 41 — Many-to-Many

Example:

```text
Students
   ↕
Courses
```

Usually represented with:

```text
student_courses
```

---

# Chapter 42 — Primary Keys

Understand:

```text
Natural Keys
Surrogate Keys
UUID
Sequence
Identity
Composite Keys
```

---

# Chapter 43 — Normalization

Learn deeply:

```text
1NF
2NF
3NF
BCNF
```

But don't memorize definitions only.

Understand the actual problems:

```text
Update Anomaly
Insert Anomaly
Delete Anomaly
Data Duplication
```

---

# Chapter 44 — Denormalization

Now learn when breaking normalization can improve:

```text
Read Performance
```

Examples:

```text
Summary Tables
Materialized Views
Cached Aggregates
Duplicated Attributes
```

---

# Chapter 45 — Schema Evolution

Real production problem:

```text
Application is running
+
Database has 1 billion rows
```

You need to add:

```text
New Column
```

without taking the system down.

Learn:

```text
Backward-compatible migrations
Expand / Contract pattern
Online schema changes
Zero-downtime migrations
```

---

# 📦 Module 9 — Constraints & Data Integrity

## Chapter 46 — PRIMARY KEY

Guarantees row identity.

---

# Chapter 47 — FOREIGN KEY

Understand:

```text
Referential Integrity
```

Real problem:

```text
Order references customer
```

What happens when the customer is deleted?

Learn:

```text
CASCADE
RESTRICT
SET NULL
```

---

# Chapter 48 — UNIQUE

Prevent duplicates:

```sql
UNIQUE(email)
```

---

# Chapter 49 — CHECK

Example:

```sql
CHECK(balance >= 0)
```

---

# Chapter 50 — Constraints vs Application Validation

Important architectural question:

> Should validation happen in the application, database, or both?

Understand why critical invariants often belong in the database too.

---

# 💳 Module 10 — Transactions & ACID

> One of the most important modules for senior engineers.

## Chapter 51 — What Is a Transaction?

Example:

```text
Account A
₹10,000

Account B
₹5,000
```

Transfer:

```text
A - ₹1,000
B + ₹1,000
```

Both operations must succeed together.

---

# Chapter 52 — BEGIN / COMMIT / ROLLBACK

Learn:

```sql
BEGIN;

UPDATE ...

UPDATE ...

COMMIT;
```

and:

```sql
ROLLBACK;
```

---

# Chapter 53 — ACID

Understand deeply:

```text
Atomicity
Consistency
Isolation
Durability
```

Do not simply memorize definitions.

Ask:

> How does the database actually provide durability?

That leads to:

```text
WAL
↓
fsync
↓
Storage
↓
Recovery
```

---

# Chapter 54 — Savepoints

Learn:

```sql
SAVEPOINT
ROLLBACK TO SAVEPOINT
```

---

# Chapter 55 — Transaction Boundaries

Real application problem:

```text
HTTP Request
      ↓
Service
      ↓
Database
```

Where should the transaction begin?

Where should it end?

Learn transaction boundary design.

---

# 🔒 Module 11 — Concurrency & Locking

> This is where database engineering becomes significantly deeper.

## Chapter 56 — Concurrent Transactions

Imagine:

```text
Inventory = 1
```

Two users purchase simultaneously.

```text
User A reads 1
User B reads 1
```

Both purchase.

Potential result:

```text
Inventory = -1
```

How does the database prevent this?

---

# Chapter 57 — Row Locks

Learn:

```text
Shared Lock
Exclusive Lock
Row Lock
Table Lock
```

Database-specific syntax such as:

```sql
SELECT ...
FOR UPDATE;
```

---

# Chapter 58 — Lock Granularity

Understand:

```text
Row
Page
Table
Database
```

Trade-offs:

```text
Fine-grained locking
        ↓
More concurrency
        +
More lock-management overhead
```

---

# Chapter 59 — Deadlocks

Example:

```text
Transaction A

Lock Row 1
   ↓
Wait for Row 2
```

while:

```text
Transaction B

Lock Row 2
   ↓
Wait for Row 1
```

Result:

```text
A → B
↑   ↓
└───┘
```

Learn:

```text
Deadlock detection
Deadlock prevention
Lock ordering
Retry strategies
```

---

# Chapter 60 — Lost Updates

Understand how concurrent writes can overwrite each other.

---

# Chapter 61 — Dirty Reads

Understand:

```text
Transaction A writes
Transaction B reads
Transaction A rolls back
```

What did B see?

---

# Chapter 62 — Non-Repeatable Reads

Understand how the same query can return different results inside one transaction.

---

# Chapter 63 — Phantom Reads

Understand how new matching rows can appear during a transaction.

---

# 📇 Module 12 — Indexes

> Learn how databases find data efficiently.

## Chapter 64 — Why Indexes Exist

Without an index:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

may require:

```text
Table Scan

Row 1
Row 2
Row 3
...
Row 10,000,000
```

With an index:

```text
Index
 ↓
Matching key
 ↓
Row
```

---

# Chapter 65 — B-Tree

Understand:

```text
Root
 ↓
Internal Pages
 ↓
Leaf Pages
 ↓
Table Rows
```

Learn:

```text
Page
Node
Fan-out
Tree Height
Key
Pointer
```

---

# Chapter 66 — B+Tree

Understand why database indexes commonly use page-oriented balanced trees.

Think about:

```text
Disk I/O
```

not just:

```text
CPU comparisons
```

---

# Chapter 67 — Hash Indexes

Understand:

```text
Hash Function
 ↓
Bucket
 ↓
Row
```

Learn where hash indexing works well and where it does not.

---

# Chapter 68 — Composite Indexes

Example:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

Understand:

```text
Column Order
Selectivity
Cardinality
Leftmost Prefix
```

---

# Chapter 69 — Covering Indexes

Learn how an index can sometimes satisfy a query without reading the base table.

---

# Chapter 70 — Partial / Filtered Indexes

Learn how indexing only a subset of rows can reduce:

```text
Storage
Index maintenance
Lookup cost
```

---

# Chapter 71 — Index Trade-offs

Indexes provide:

```text
Faster Reads
```

but cost:

```text
Storage
Write Overhead
Maintenance
Memory
Vacuum / Cleanup Work
```

Therefore:

```text
More indexes ≠ Always better
```

---

# ⚙️ Module 13 — Query Execution Engine

> Now understand what happens after you send SQL.

## Chapter 72 — SQL Parsing

Flow:

```text
SQL
 ↓
Lexer / Parser
 ↓
Parse Tree
```

Learn:

```text
Syntax validation
Tokens
Parse tree
AST / internal representation
```

---

# Chapter 73 — Semantic Analysis

Database determines:

```text
Does table exist?
Does column exist?
Are types compatible?
Does user have permission?
```

---

# Chapter 74 — Query Rewriting

Understand:

```text
Query Rewrite
```

Examples may include:

```text
View expansion
Predicate transformations
Simplification
Subquery transformations
```

---

# Chapter 75 — Query Planner / Optimizer

The optimizer considers:

```text
Indexes
Join order
Join algorithms
Statistics
Cardinality
Costs
Sorting
Filtering
```

and generates a plan.

---

# Chapter 76 — Execution Plan

Learn:

```sql
EXPLAIN
```

and:

```sql
EXPLAIN ANALYZE
```

Learn to read:

```text
Seq Scan
Index Scan
Bitmap Scan
Nested Loop
Hash Join
Merge Join
Sort
Aggregate
HashAggregate
Materialize
```

---

# 🔬 Module 14 — Query Optimization

## Chapter 77 — Full Table Scans

Understand when:

```text
Sequential Scan
```

is actually the correct choice.

Important:

> An index is not automatically faster.

---

# Chapter 78 — Cardinality Estimation

Optimizer needs to estimate:

```text
How many rows will this operation produce?
```

If estimates are wrong:

```text
Wrong Cardinality
        ↓
Wrong Plan
        ↓
Slow Query
```

---

# Chapter 79 — Database Statistics

Learn:

```text
Statistics
Histograms
Distinct Values
Data Distribution
Correlation
```

Understand why statistics need maintenance.

---

# Chapter 80 — Join Algorithms

Deep dive into:

```text
Nested Loop Join
Hash Join
Merge Join
```

Question:

> Why did the optimizer choose Hash Join?

---

# Chapter 81 — Sort Algorithms

Understand:

```text
In-memory sort
External sort
Disk spill
```

Real production problem:

```text
Query works quickly in development
        ↓
Production dataset is huge
        ↓
Sort spills to disk
        ↓
Query becomes slow
```

---

# Chapter 82 — Predicate Pushdown

Understand why filtering data earlier can dramatically reduce work.

---

# Chapter 83 — Sargability

Learn why some predicates can use indexes efficiently and others cannot.

Example concept:

```sql
WHERE indexed_column = ?
```

versus applying a function to the indexed column.

---

# 🚨 Chapter 84 — Real Production Query Incident

Application:

```text
API latency = 100ms
```

Suddenly:

```text
API latency = 8 seconds
```

Investigate:

```text
API
 ↓
SQL
 ↓
Database
 ↓
EXPLAIN ANALYZE
```

Discover:

```text
Seq Scan
10,000,000 rows
```

Expected:

```text
Index Scan
10 rows
```

Investigate:

```text
Missing index?
Statistics?
Data distribution?
Query changed?
Wrong join order?
Parameter-specific behavior?
Memory pressure?
```

This is real database engineering.

---

# 💾 Module 15 — Storage Engine Internals

> Now go below SQL and understand how databases store data.

## Chapter 85 — Pages / Blocks

Database storage is commonly organized into:

```text
Pages / Blocks
```

Conceptually:

```text
Table
 ↓
Page 1
Page 2
Page 3
Page 4
...
```

Understand:

```text
Page size
Tuple / row
Page header
Free space
Pointers
```

---

# Chapter 86 — Heap Storage

Understand how table rows are physically stored.

---

# Chapter 87 — Buffer Cache

Database:

```text
Application
      ↓
Database
      ↓
Buffer Cache
      ↓
Disk
```

If data is cached:

```text
Cache Hit
```

Otherwise:

```text
Cache Miss
 ↓
Disk I/O
```

---

# Chapter 88 — Memory Management

Learn:

```text
Shared memory
Buffer pool
Sort memory
Hash memory
Connection memory
Working memory
```

Understand why memory configuration affects query performance.

---

# Chapter 89 — Disk I/O

Learn:

```text
Sequential I/O
Random I/O
IOPS
Throughput
Latency
Queue Depth
```

Understand:

```text
CPU-bound
Memory-bound
I/O-bound
```

---

# Chapter 90 — SSD vs HDD

Understand how storage characteristics affect:

```text
Database latency
Random reads
Random writes
Checkpointing
Recovery
```

---

# 🧬 Module 16 — MVCC & Isolation

> Understand how databases allow concurrent readers and writers.

## Chapter 91 — MVCC

MVCC:

```text
Multi-Version Concurrency Control
```

Conceptually:

```text
Row Version 1
      ↓
Row Version 2
      ↓
Row Version 3
```

Different transactions may see different versions according to their isolation semantics.

---

# Chapter 92 — Isolation Levels

Learn deeply:

```text
Read Uncommitted
Read Committed
Repeatable Read
Serializable
```

Understand what anomalies each level permits or prevents.

---

# Chapter 93 — Snapshots

Understand:

```text
Transaction Snapshot
```

and how consistent reads work in MVCC systems.

---

# Chapter 94 — Vacuum / Garbage Collection Concepts

Understand why old row versions need cleanup in MVCC databases.

Learn concepts such as:

```text
Dead tuples
Bloat
Vacuum
Autovacuum
Visibility
```

where applicable to the database engine you're studying.

---

# Chapter 95 — Long-Running Transactions

Real production problem:

```text
Transaction started
        ↓
Application hangs
        ↓
Transaction remains open
        ↓
Old versions cannot be cleaned
        ↓
Storage grows
        ↓
Performance degrades
```

---

# 💾 Module 17 — WAL, Logging & Recovery

## Chapter 96 — Write-Ahead Logging

Conceptually:

```text
Transaction
    ↓
WAL
    ↓
Durable Storage
    ↓
Data Pages
```

The core idea:

> Required log information is made durable before the corresponding data-page changes are considered safely persisted.

---

# Chapter 97 — Checkpoints

Learn:

```text
Checkpoint
```

and why databases periodically flush or coordinate durable state to reduce recovery work.

---

# Chapter 98 — Crash Recovery

Imagine:

```text
Transaction
 ↓
UPDATE
 ↓
Database Crash
```

On restart:

```text
Database starts
      ↓
Read recovery information
      ↓
Redo / undo as required
      ↓
Recover consistent state
```

---

# Chapter 99 — Durability

Understand:

```text
fsync
Flush
Write Cache
Disk
Storage Controller
```

and why "written by the application" is not automatically equivalent to "durably persisted."

---

# Chapter 100 — Backup vs WAL

Understand the difference between:

```text
Backup
```

and:

```text
Transaction Log / WAL
```

and how they work together for recovery.

---

# 🧩 Module 18 — Partitioning

> Learn how to divide massive tables.

## Chapter 101 — Why Partition?

Imagine:

```text
orders

5 billion rows
```

Queries mostly use:

```text
created_at
```

Instead of one huge table:

```text
orders
```

use partitions:

```text
orders_2026_01
orders_2026_02
orders_2026_03
...
```

---

# Chapter 102 — Range Partitioning

Common for:

```text
Dates
IDs
Numeric ranges
```

---

# Chapter 103 — List Partitioning

Useful for:

```text
Region
Country
Category
Tenant
```

---

# Chapter 104 — Hash Partitioning

Distribute rows using:

```text
hash(key)
```

---

# Chapter 105 — Partition Pruning

Real problem:

```sql
WHERE created_at >= '2026-08-01'
```

Database can potentially avoid reading unrelated partitions.

---

# Chapter 106 — Partition Management

Learn:

```text
Creating partitions
Dropping partitions
Attaching partitions
Detaching partitions
Partition maintenance
```

---

# 🔁 Module 19 — Replication

## Chapter 107 — Why Replication?

One database:

```text
Primary
```

with replicas:

```text
Primary
 ↓
Replica 1
Replica 2
Replica 3
```

Benefits:

```text
Read Scaling
High Availability
Disaster Recovery
```

---

# Chapter 108 — Primary / Replica

Understand:

```text
Write → Primary

Read → Replicas
```

---

# Chapter 109 — Replication Lag

Real production problem:

```text
User updates profile
        ↓
Primary
        ↓
Replica
```

If the replica hasn't received the update yet:

```text
User immediately reads
        ↓
Old data
```

This is a classic distributed-systems problem.

---

# Chapter 110 — Synchronous vs Asynchronous Replication

Understand trade-offs:

```text
Consistency
Latency
Availability
Durability
```

---

# Chapter 111 — Read-After-Write Consistency

Learn techniques for ensuring users see their own writes when reads may be routed to replicas.

---

# 🛡️ Module 20 — High Availability

## Chapter 112 — Database Failover

Architecture:

```text
Primary
   ↓
Replica
```

Primary fails:

```text
X Primary

Replica
   ↓
New Primary
```

---

# Chapter 113 — Leader Election

Understand the relationship between:

```text
Database Failover
```

and:

```text
Distributed Consensus
```

Learn concepts such as:

```text
Leader
Followers
Quorum
Election
Term / Epoch
```

---

# Chapter 114 — Split Brain

Understand:

```text
Node A thinks it is Primary
Node B thinks it is Primary
```

This can be catastrophic.

Learn how modern systems prevent or mitigate it.

---

# Chapter 115 — Quorum

Understand:

```text
N nodes
```

and:

```text
Majority
```

Why majority-based decisions matter in distributed systems.

---

# 📈 Module 21 — Database Scaling

> Learn what happens when one database is no longer enough.

## Chapter 116 — Vertical Scaling

```text
More CPU
More RAM
Faster Storage
```

Advantages:

```text
Simple
```

Limit:

```text
Physical ceiling
Cost
Single-system bottleneck
```

---

# Chapter 117 — Read Scaling

```text
Primary
 ↓
Replica
 ↓
Replica
 ↓
Replica
```

---

# Chapter 118 — Write Scaling

Eventually:

```text
One Primary
```

may not handle all writes.

Now you need:

```text
Partitioning
Sharding
Distributed SQL
```

---

# Chapter 119 — Connection Pooling

Learn why applications shouldn't create:

```text
1 database connection
per HTTP request
```

Understand:

```text
Connection Pool
```

and problems such as:

```text
Too many connections
Connection exhaustion
Pool starvation
```

---

# Chapter 120 — Backpressure

Real production problem:

```text
Traffic increases
 ↓
Connections increase
 ↓
Database overload
 ↓
Queries slow
 ↓
Requests remain open longer
 ↓
More connections accumulate
 ↓
Database collapses
```

Understand why application-level backpressure and load shedding matter.

---

# 🛰️ Module 22 — Distributed Databases

## Chapter 121 — Why Distributed SQL?

One machine:

```text
CPU Limit
RAM Limit
Storage Limit
```

Eventually you need:

```text
Multiple Machines
```

---

# Chapter 122 — Sharding

Example:

```text
Users 1–10M
     ↓
Shard 1

Users 10M–20M
     ↓
Shard 2

Users 20M–30M
     ↓
Shard 3
```

---

# Chapter 123 — Shard Keys

Choosing a shard key is one of the most important architecture decisions.

Learn:

```text
Cardinality
Distribution
Hotspots
Locality
Query Patterns
```

---

# Chapter 124 — Hot Partitions

Bad distribution:

```text
Shard 1 → 90% traffic
Shard 2 → 5%
Shard 3 → 5%
```

Technically distributed.

Practically broken.

---

# Chapter 125 — Cross-Shard Queries

Imagine:

```text
Query
 ↓
Shard 1
Shard 2
Shard 3
Shard 4
```

Then:

```text
Merge Results
```

Understand why distributed queries are expensive.

---

# Chapter 126 — Distributed Transactions

Learn the problems behind:

```text
Transaction across
multiple database nodes
```

Study concepts such as:

```text
Two-Phase Commit
Consensus
Distributed Commit
```

and understand their performance/availability trade-offs.

---

# Chapter 127 — CAP Theorem

Understand:

```text
Consistency
Availability
Partition Tolerance
```

and why network partitions make distributed database design fundamentally different from single-node databases.

---

# Chapter 128 — Consensus

Study concepts such as:

```text
Raft
Paxos
Quorum
Leader Election
Log Replication
```

The goal is understanding, not merely implementing the algorithms.

---

# 🏭 Module 23 — Production Database Engineering

> Learn how databases are actually operated in production.

## Chapter 129 — Production Database Architecture

Typical architecture:

```text
Users
 ↓
Load Balancer
 ↓
Application
 ↓
Connection Pool
 ↓
Database Proxy
 ↓
Primary
 ↓
Replicas
```

---

# Chapter 130 — Database Migrations

Learn:

```text
Schema migrations
Versioning
Rollback
Backward compatibility
Expand / Contract
```

---

# Chapter 131 — Zero-Downtime Migrations

Real problem:

```text
1 billion rows
```

You need to add:

```text
new_column
```

without:

```text
downtime
```

Learn safe migration patterns.

---

# Chapter 132 — Online Index Creation

Understand how indexes can be created while production traffic continues, subject to the capabilities and locking behavior of your database.

---

# Chapter 133 — Large Data Backfills

Bad approach:

```sql
UPDATE 1 billion rows;
```

Better strategies involve:

```text
Batching
Chunking
Rate limiting
Monitoring
Retrying
```

---

# Chapter 134 — Connection Management

Learn:

```text
Pool size
Timeouts
Idle connections
Connection limits
Retries
Circuit breakers
```

---

# Chapter 135 — Database Capacity Planning

Monitor:

```text
CPU
Memory
Disk
IOPS
Connections
QPS
TPS
Latency
Replication Lag
Storage Growth
```

---

# 🔐 Module 24 — Database Security

## Chapter 136 — Authentication

Learn:

```text
Users
Passwords
Certificates
IAM
Authentication plugins
```

---

# Chapter 137 — Authorization

Understand:

```text
Roles
Privileges
GRANT
REVOKE
```

---

# Chapter 138 — Least Privilege

Application should not normally have:

```text
DROP DATABASE
```

permissions.

Design:

```text
Application User
        ↓
Only Required Permissions
```

---

# Chapter 139 — SQL Injection

Understand how unsafe query construction leads to:

```text
SQL Injection
```

Learn parameterized queries:

```text
Prepared Statements
```

---

# Chapter 140 — Encryption

Understand:

```text
Encryption in Transit
Encryption at Rest
Key Management
Secrets Management
```

---

# Chapter 141 — Auditing

Learn:

```text
Who accessed data?
Who modified data?
When?
From where?
```

---

# 📊 Module 25 — Observability & Troubleshooting

> A senior database engineer must be able to diagnose production incidents.

## Chapter 142 — Database Metrics

Monitor:

```text
QPS
TPS
Latency
CPU
Memory
IOPS
Cache Hit Ratio
Connections
Locks
Replication Lag
```

---

# Chapter 143 — Slow Query Analysis

Find:

```text
Slow Queries
```

Then investigate:

```text
EXPLAIN ANALYZE
```

---

# Chapter 144 — Lock Monitoring

Find:

```text
Who is blocking whom?
```

Example:

```text
Transaction A
    ↓
Locks Row

Transaction B
    ↓
Waiting
```

---

# Chapter 145 — Deadlock Analysis

Learn how to inspect:

```text
Deadlock Logs
Transaction Graphs
Lock Ordering
```

---

# Chapter 146 — Database CPU Spikes

Possible causes:

```text
Bad Query
Missing Index
Wrong Plan
Too Many Connections
Large Sort
Hash Join
Aggregation
```

---

# Chapter 147 — Database Memory Problems

Investigate:

```text
Cache
Connection Memory
Sort Memory
Hash Memory
Workload
```

---

# Chapter 148 — Disk Full

Real production incident:

```text
Disk = 100%
```

Potential consequences:

```text
Writes fail
WAL/log growth
Temporary files fail
Database instability
```

Learn safe recovery procedures.

---

# Chapter 149 — Replication Lag Incident

Application:

```text
Write → Primary
Read → Replica
```

Replica falls behind.

Investigate:

```text
Network
Disk
CPU
Long Transactions
Replication Volume
Queries
```

---

# ☁️ Module 26 — Cloud Databases

## Chapter 150 — Why Managed Databases?

Traditional:

```text
Buy Server
↓
Install Database
↓
Configure Storage
↓
Configure Backups
↓
Configure Replication
↓
Monitor
```

Managed cloud database:

```text
Create Database
↓
Provider manages infrastructure
```

But:

> Managed doesn't mean you don't need database knowledge.

---

# Chapter 151 — AWS Databases

Study:

```text
Amazon RDS
Amazon Aurora
Amazon DynamoDB
Amazon Redshift
```

Understand where relational and non-relational services fit.

---

# Chapter 152 — Azure Databases

Study:

```text
Azure Database for PostgreSQL
Azure SQL Database
Azure Cosmos DB
```

---

# Chapter 153 — Google Cloud Databases

Study:

```text
Cloud SQL
AlloyDB
Spanner
BigQuery
```

---

# Chapter 154 — Cloud Database Networking

Understand:

```text
VPC
Private Subnet
Security Group / Firewall
Private Endpoint
Load Balancer
Application
Database
```

---

# Chapter 155 — Cloud Database Cost

Learn:

```text
Compute
Storage
IOPS
Backup
Network Transfer
Read Replicas
Provisioned Capacity
```

Senior engineers optimize:

```text
Performance
+
Reliability
+
Cost
```

---

# 🧩 Module 27 — NoSQL & Polyglot Persistence

> A senior database engineer must know when SQL is not the right tool.

## Chapter 156 — Why NoSQL?

Understand problems involving:

```text
Extreme Scale
Flexible Schema
High Throughput
Specialized Access Patterns
```

---

# Chapter 157 — Key-Value Databases

Examples:

```text
Redis
DynamoDB
```

Use cases:

```text
Caching
Sessions
Fast lookups
Counters
```

---

# Chapter 158 — Document Databases

Example:

```text
MongoDB
```

Understand:

```text
Document Model
Embedding
Referencing
Indexes
Aggregation
```

---

# Chapter 159 — Column-Family Databases

Study concepts from systems such as:

```text
Cassandra
```

---

# Chapter 160 — Graph Databases

Understand:

```text
Nodes
Edges
Relationships
Traversal
```

---

# Chapter 161 — Search Engines

Understand why systems such as:

```text
Elasticsearch / OpenSearch
```

may be used alongside SQL.

---

# Chapter 162 — Polyglot Persistence

Real architecture:

```text
PostgreSQL
   +
Redis
   +
Kafka
   +
Search Engine
   +
Object Storage
```

Question:

> Why use multiple data systems instead of one database?

---

# 🏗️ Module 28 — Database Architecture

## Chapter 163 — Monolithic Database Architecture

```text
Application
     ↓
Database
```

Understand when this is actually the best architecture.

---

# Chapter 164 — Read Replica Architecture

```text
                ┌── Replica
                │
Application → Primary
                │
                └── Replica
```

---

# Chapter 165 — Database Proxy Architecture

```text
Application
     ↓
Proxy
     ↓
Primary / Replicas
```

Understand:

```text
Read/Write Routing
Connection Pooling
Failover
Health Checks
```

---

# Chapter 166 — Microservices Database Architecture

Understand:

```text
User Service
     ↓
User DB

Order Service
     ↓
Order DB

Payment Service
     ↓
Payment DB
```

---

# Chapter 167 — Database Per Service

Understand why microservices often prefer:

```text
Service owns its data
```

rather than:

```text
Every service directly accesses
every other service's tables
```

---

# Chapter 168 — Distributed Transactions in Microservices

Instead of one giant transaction:

```text
Service A
 ↓
Service B
 ↓
Service C
```

study:

```text
Saga Pattern
Outbox Pattern
Eventual Consistency
Idempotency
Compensation
```

---

# 🌐 Module 29 — Large-Scale System Design

> Combine everything you've learned.

## Chapter 169 — Design an E-Commerce Database

Design:

```text
Users
Products
Inventory
Orders
Payments
Shipments
Reviews
```

Questions:

* What are the keys?
* What are the relationships?
* Where do transactions exist?
* What should be indexed?
* What should be partitioned?
* Which data can be cached?
* Which tables will grow fastest?

---

# Chapter 170 — Design a Banking Database

Requirements:

```text
Accounts
Transactions
Transfers
Audit
Balances
```

Focus on:

```text
ACID
Consistency
Concurrency
Locks
Recovery
Auditability
```

---

# Chapter 171 — Design a Food Delivery Database

```text
Users
Restaurants
Drivers
Orders
Payments
Locations
Tracking
```

Now think about:

```text
High writes
Real-time updates
Geo queries
Concurrency
Scaling
```

---

# Chapter 172 — Design a Social Media Database

```text
Users
Posts
Comments
Likes
Followers
Messages
Notifications
```

Think about:

```text
Huge reads
Huge writes
Hot users
Fan-out
Caching
Partitioning
```

---

# Chapter 173 — Design a Payment System

Focus on:

```text
Idempotency
Transactions
Double spending
Ledger
Audit
Retries
Exactly-once business effects
```

---

# Chapter 174 — Design a Multi-Tenant SaaS Database

Models:

```text
Shared Database
Shared Schema
```

versus:

```text
Shared Database
Separate Schema
```

versus:

```text
Database per Tenant
```

Understand the trade-offs.

---

# Chapter 175 — Design for Billion-Row Tables

Given:

```text
5 billion orders
```

decide:

```text
Partitioning
Indexes
Archival
Compression
Replication
Query patterns
Storage
```

---

# 👑 Module 30 — Staff / Principal Database Engineering

> This is where SQL knowledge becomes architecture and engineering leadership.

## Chapter 176 — Database Trade-offs

Every architecture decision has trade-offs.

Example:

```text
Consistency
vs
Latency
```

```text
Normalization
vs
Read Performance
```

```text
Indexes
vs
Write Performance
```

```text
Replication
vs
Operational Complexity
```

```text
Sharding
vs
Application Complexity
```

Learn to explain:

```text
Why?
```

not just:

```text
What?
```

---

# Chapter 177 — Database Capacity Planning

Given:

```text
10,000 requests/sec
```

estimate:

```text
Database QPS
Write QPS
Read QPS
Storage Growth
Network
Connections
CPU
Memory
```

---

# Chapter 178 — Performance Engineering

Learn to identify the bottleneck:

```text
CPU
Memory
Disk
Network
Lock
Connection Pool
Query Plan
Application
```

Don't optimize blindly.

---

# Chapter 179 — Reliability Engineering

Learn:

```text
SLA
SLO
SLI
RTO
RPO
```

Example:

```text
RPO = 5 minutes
RTO = 30 minutes
```

Now design:

```text
Backup
Replication
Failover
Recovery
```

around those requirements.

---

# Chapter 180 — Disaster Recovery

Study:

```text
Backup
Restore
Point-in-Time Recovery
Cross-Region Replication
Failover
Failback
Disaster Testing
```

Most importantly:

> A backup that has never been restored is only a theory.

---

# Chapter 181 — Database Migration at Scale

Scenario:

```text
Old Database
     ↓
New Database
```

while:

```text
1000 requests/sec
```

continue running.

Study:

```text
Dual Write
CDC
Backfill
Validation
Shadow Reads
Cutover
Rollback
```

---

# Chapter 182 — Change Data Capture

Understand:

```text
Database
   ↓
Change Log
   ↓
Kafka
   ↓
Consumers
```

Use cases:

```text
Search indexing
Analytics
Caches
Data warehouses
Event-driven systems
```

---

# Chapter 183 — Event-Driven Data Architecture

Example:

```text
Order Service
      ↓
Database
      ↓
CDC / Outbox
      ↓
Kafka
      ↓
Inventory
      ↓
Notification
      ↓
Analytics
```

Understand why databases often become the source of truth while events propagate changes to other systems.

---

# Chapter 184 — Data Consistency at Scale

Learn:

```text
Strong Consistency
Eventual Consistency
Read-Your-Writes
Monotonic Reads
Causal Consistency
```

Understand where each model is useful.

---

# Chapter 185 — Database Governance

At enterprise scale:

```text
Schema Standards
Naming Standards
Access Control
Data Retention
Auditing
Compliance
Backup Policies
Migration Policies
```

---

# Chapter 186 — Database Architecture Reviews

Learn to review a database design and ask:

```text
Will it scale?

What is the bottleneck?

What happens when traffic increases 10x?

What happens when the primary fails?

What happens during deployment?

What happens during a migration?

What happens when the network fails?

What happens when storage reaches 90%?

What happens when one tenant becomes huge?
```

---

# 🧪 Final Projects

Complete these projects in order.

---

# Project 1 — Banking System

Build:

```text
Users
Accounts
Transactions
Transfers
Audit Logs
```

Implement:

```text
ACID Transactions
Constraints
Indexes
Concurrency
Deadlock Handling
```

---

# Project 2 — E-Commerce Database

Build:

```text
Users
Products
Inventory
Orders
Payments
Shipments
Reviews
```

Implement:

```text
Normalization
Indexes
Transactions
Partitioning
Reporting Queries
```

---

# Project 3 — Production SaaS Database

Build:

```text
Authentication
Tenants
Users
Subscriptions
Invoices
Payments
Usage
Audit
```

Implement:

```text
Multi-tenancy
RBAC
Indexes
Migrations
Backups
Monitoring
```

---

# Project 4 — Billion-Row Simulation

Generate:

```text
1M
10M
100M
500M+
```

rows.

Measure:

```text
Query latency
Index size
Storage
CPU
Memory
I/O
```

Compare:

```text
Without Index
```

vs:

```text
With Index
```

vs:

```text
Partitioned
```

---

# Project 5 — Database Failure Lab

Simulate:

```text
Database Crash
Disk Full
Replication Lag
Long Transaction
Deadlock
Connection Exhaustion
Slow Query
Missing Index
```

Then diagnose each incident.

---

# Project 6 — Distributed Database

Build or study an architecture containing:

```text
Application
      ↓
Load Balancer
      ↓
Database Proxy
      ↓
Primary
 ↙    ↓    ↘
Replica Replica Replica
      ↓
     WAL
      ↓
Backup
```

Then introduce:

```text
Partitioning
```

and eventually:

```text
Sharding
```

---

# 🎯 Interview Preparation

## Beginner

Questions:

```text
What is SQL?

What is a database?

What is a primary key?

What is a foreign key?

What is normalization?

What is a JOIN?

What is an index?
```

---

# Intermediate

Questions:

```text
INNER JOIN vs LEFT JOIN?

WHERE vs HAVING?

GROUP BY vs Window Function?

DELETE vs TRUNCATE?

What is a transaction?

What is ACID?

What is a composite index?
```

---

# Senior

Questions:

```text
Why is this query slow?

How does the optimizer choose a plan?

How does a B-Tree work?

What is MVCC?

How do deadlocks happen?

How do you diagnose blocking?

How does replication work?

What causes replication lag?

How would you migrate a billion-row table?
```

---

# Staff / Principal

Questions:

```text
How would you design a database for 100M users?

How would you scale writes?

When would you shard?

How do you choose a shard key?

How do you handle cross-shard queries?

How do you design multi-region databases?

How do you guarantee durability?

How do you design disaster recovery?

How would you migrate from one database to another with zero downtime?

When should you choose SQL vs NoSQL?

How would you reduce database cost by 40% without hurting reliability?
```

---

# 🔥 Real-World Problem Ladder

Use this progression to develop engineering thinking.

---

## Problem 1

```text
Query takes 5 seconds.
```

Learn:

```text
EXPLAIN
Indexes
Query Plan
```

---

## Problem 2

```text
Query is fast in development
but slow in production.
```

Investigate:

```text
Data size
Statistics
Indexes
Hardware
Concurrency
```

---

## Problem 3

```text
Database CPU = 100%
```

Investigate:

```text
Queries
Connections
Execution Plans
Aggregations
Joins
```

---

## Problem 4

```text
Database connections exhausted.
```

Investigate:

```text
Connection Pool
Leaks
Long Queries
Traffic
Database Limits
```

---

## Problem 5

```text
Users see stale data.
```

Investigate:

```text
Read Replica
Replication Lag
Caching
Consistency Model
```

---

## Problem 6

```text
Database becomes slow after adding an index.
```

Investigate:

```text
Write overhead
Index size
Cache pressure
Maintenance
Query planner behavior
```

---

## Problem 7

```text
Database reaches 5 TB.
```

Investigate:

```text
Partitioning
Archival
Compression
Indexes
Storage
```

---

## Problem 8

```text
Database cannot handle write traffic.
```

Investigate:

```text
Batching
Connection Pool
Indexes
Partitioning
Sharding
Distributed SQL
```

---

## Problem 9

```text
Primary database fails.
```

Design:

```text
Detection
Failover
Promotion
Application Reconnection
Data Loss Handling
Recovery
```

---

## Problem 10

```text
One customer generates 40%
of all traffic.
```

Investigate:

```text
Hot Partition
Hot Shard
Caching
Rate Limiting
Tenant Isolation
Data Distribution
```

---

# 🧠 Database Mental Model

After completing this roadmap, you should be able to mentally trace:

```text
Application
      ↓
Connection Pool
      ↓
Network
      ↓
Database Connection
      ↓
SQL Parser
      ↓
Query Analyzer
      ↓
Query Rewriter
      ↓
Optimizer
      ↓
Execution Plan
      ↓
Executor
      ↓
Buffer Cache
      ↓
Index / Table
      ↓
Storage Engine
      ↓
WAL / Log
      ↓
Disk
```

And for concurrent requests:

```text
Transaction A
       ↓
       DB
       ↑
Transaction B

       ↓

Locks
MVCC
Isolation
Deadlocks
```

And for production:

```text
Application
      ↓
Load Balancer
      ↓
Connection Pool
      ↓
Database Proxy
      ↓
Primary
 ↙    ↓    ↘
R1    R2    R3
      ↓
Replication
      ↓
Backup
      ↓
Disaster Recovery
```

And at massive scale:

```text
Application
      ↓
Database Router
      ↓
Shard 1
Shard 2
Shard 3
Shard 4
      ↓
Replication
      ↓
Distributed Consensus
      ↓
Object Storage / Backup
```

---

# 🏆 What You'll Master

By completing this roadmap, you will understand:

### SQL

* SELECT
* INSERT
* UPDATE
* DELETE
* JOINs
* GROUP BY
* HAVING
* Subqueries
* CTEs
* Recursive CTEs
* Window Functions
* Advanced SQL

### Database Design

* ER Modeling
* Keys
* Relationships
* Normalization
* Denormalization
* Constraints
* Schema Evolution
* Multi-Tenancy

### Transactions

* ACID
* BEGIN
* COMMIT
* ROLLBACK
* Savepoints
* Transaction Boundaries

### Concurrency

* Locks
* Deadlocks
* Isolation
* MVCC
* Blocking
* Race Conditions

### Indexing

* B-Tree
* B+Tree
* Hash Index
* Composite Index
* Covering Index
* Partial Index
* Index Optimization

### Query Engine

* Parser
* Analyzer
* Rewriter
* Optimizer
* Execution Plans
* Join Algorithms
* Cardinality Estimation
* Statistics

### Storage Engine

* Pages
* Tuples
* Buffer Cache
* Disk I/O
* WAL
* Checkpoints
* Recovery
* MVCC

### Scaling

* Replication
* Read Replicas
* Partitioning
* Sharding
* Distributed SQL
* Connection Pooling
* Backpressure

### Production

* Monitoring
* Slow Query Analysis
* Capacity Planning
* Backups
* Disaster Recovery
* Migrations
* Zero-Downtime Deployment
* Security
* Auditing

### Distributed Systems

* CAP Theorem
* Consensus
* Raft
* Quorum
* Leader Election
* Distributed Transactions
* Eventual Consistency
* CDC
* Outbox Pattern
* Saga Pattern

### Architecture

* Monolith Database
* Read Replica Architecture
* Microservices Databases
* Database per Service
* Multi-Tenant Databases
* Multi-Region Architecture
* Distributed Databases

---

# 🧭 Recommended Learning Order

Do **not** jump directly to:

```text
Sharding
Kubernetes
Cloud Databases
Distributed SQL
```

Follow:

```text
SQL
 ↓
Relational Model
 ↓
Joins
 ↓
Aggregation
 ↓
CTEs
 ↓
Window Functions
 ↓
Database Design
 ↓
Constraints
 ↓
Transactions
 ↓
Concurrency
 ↓
Indexes
 ↓
Query Execution
 ↓
Query Optimization
 ↓
Storage Engine
 ↓
MVCC
 ↓
WAL
 ↓
Recovery
 ↓
Partitioning
 ↓
Replication
 ↓
High Availability
 ↓
Scaling
 ↓
Distributed Databases
 ↓
Production Engineering
 ↓
Cloud Databases
 ↓
Architecture
 ↓
Staff / Principal Level
```

---

# 🎓 Final Outcome

After completing this roadmap, you should no longer look at:

```sql
SELECT *
FROM orders
WHERE user_id = 100;
```

as simply:

> “A SQL query.”

Instead, you should think:

```text
SQL
 ↓
Parser
 ↓
Optimizer
 ↓
Execution Plan
 ↓
Index?
 ↓
Cardinality?
 ↓
Join?
 ↓
Buffer Cache?
 ↓
Disk I/O?
 ↓
Transaction?
 ↓
MVCC?
 ↓
Locks?
 ↓
WAL?
 ↓
Concurrency?
 ↓
Replication?
 ↓
Production Impact?
```

That is the difference between:

```text
Knowing SQL
```

and:

```text
Understanding Databases
```

---

# 🚀 Final Goal

> **Don't just learn SQL commands.**
>
> Understand what happens inside a database from the moment an application sends a query until the database reads pages, chooses an execution plan, uses indexes, manages transactions, handles concurrent users, writes durable logs, recovers from crashes, replicates data, and eventually scales across multiple machines.
>
> The goal is to develop the judgment required to design, optimize, troubleshoot, scale, and operate databases at **senior, staff, principal, and architect level**.

---

# 🔥 Next Step

After completing the SQL roadmap, continue into:

```text
Linux
 ↓
Operating Systems
 ↓
Networking
 ↓
Distributed Systems
 ↓
Kafka
 ↓
Redis
 ↓
Docker
 ↓
Kubernetes
 ↓
Cloud
 ↓
System Design
 ↓
Production Architecture
```

The database should not be studied in isolation.

A production database sits inside:

```text
Application
      ↓
API
      ↓
Networking
      ↓
Load Balancer
      ↓
Services
      ↓
Database
      ↓
Storage
      ↓
Operating System
      ↓
Hardware
```

Understanding those layers is what eventually turns a SQL developer into a **database/system architect**.
