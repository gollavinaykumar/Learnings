# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 4 — SQL vs Database Engine

> In the previous chapter, we learned the **relational model**:
>
> ```text
> Entities
>     ↓
> Relationships
>     ↓
> Tables
>     ↓
> Rows + Columns
>     ↓
> Keys + Constraints
> ```
>
> We also learned that SQL allows us to work with this relational model.
>
> But now comes a very important question:
>
> # When I write SQL, who actually executes it?
>
> When you write:
>
> ```sql
> SELECT *
> FROM users
> WHERE id = 100;
> ```
>
> the database does **far more** than simply read the `users` table.
>
> This chapter starts opening the black box.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What SQL actually is
* SQL as a declarative language
* SQL client vs database server
* Database engine
* Query parser
* Query analyzer
* Query rewriter
* Query optimizer
* Execution plan
* Query executor
* Storage engine
* Buffer/cache
* Disk I/O
* Client-server database communication
* Why the same SQL can have different execution plans
* Why SQL performance is not determined only by SQL syntax
* How to begin reading an execution plan
* How senior engineers debug slow SQL

---

# 📖 The Problem

Imagine you have an application:

```text
E-Commerce Application
```

A customer opens:

```text
My Orders
```

The application executes:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

The developer might think:

```text
SQL
 ↓
Database
 ↓
Result
```

But internally, something much more complicated happens.

Conceptually:

```text
Application
     ↓
SQL
     ↓
Database Connection
     ↓
Database Server
     ↓
Parser
     ↓
Analyzer
     ↓
Rewriter
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
Storage
     ↓
Result
     ↓
Application
```

This entire chapter is about understanding this pipeline.

---

# 🧠 First Principle

# SQL Does Not Tell the Database Exactly How to Execute the Query

Consider:

```sql
SELECT *
FROM users
WHERE id = 100;
```

You specified:

```text
WHAT DATA YOU WANT
```

You did not specify:

```text
WHICH FILE TO OPEN
WHICH PAGE TO READ
WHICH INDEX TO USE
WHICH MEMORY BUFFER TO USE
WHICH JOIN ALGORITHM TO USE
WHICH ORDER TO READ TABLES
```

The database decides those things.

This is called:

# Declarative Programming

---

# 🔄 Declarative vs Imperative

## Imperative Programming

You tell the computer:

```text
HOW
```

For example:

```text
Open file

↓

Read each record

↓

Check customer_id

↓

If customer_id = 1001

↓

Return record
```

---

## SQL

You tell the database:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

You specify:

```text
WHAT
```

The database decides:

```text
HOW
```

---

# 🧠 Why Is This Powerful?

Suppose the table contains:

```text
1,000 rows
```

A sequential scan may be perfectly fine.

But later:

```text
100 million rows
```

Now an index may be better.

Your SQL does not necessarily need to change:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

The database can potentially choose a different execution strategy.

That is one of the major strengths of declarative languages.

---

# 🏗️ Database Query Pipeline

Let's follow one query from beginning to end.

```sql
SELECT name
FROM users
WHERE id = 100;
```

Conceptually:

```text
                 SQL
                  │
                  ▼
               Parser
                  │
                  ▼
               Analyzer
                  │
                  ▼
              Rewriter
                  │
                  ▼
              Optimizer
                  │
                  ▼
           Execution Plan
                  │
                  ▼
               Executor
                  │
                  ▼
            Storage Access
                  │
             ┌────┴────┐
             ▼         ▼
           Cache      Disk
             │         │
             └────┬────┘
                  ▼
                Result
```

Let's examine each stage.

---

# 1️⃣ SQL Client

Before the database receives the SQL, something sends it.

For example:

```text
Application
```

could be written in:

```text
Java
Python
Go
Node.js
C#
PHP
Rust
```

The application uses a database driver.

For PostgreSQL, examples include:

```text
JDBC
psycopg
pgx
node-postgres
Npgsql
```

The application sends:

```sql
SELECT name
FROM users
WHERE id = 100;
```

to the database server.

---

# 🌐 Client-Server Architecture

Conceptually:

```text
Application Server
       │
       │ Database Protocol
       ▼
Database Server
       │
       ▼
Database Engine
```

For a remote database:

```text
Application
     │
     │ TCP
     ▼
Network
     │
     ▼
Database Server
```

This means SQL performance can involve networking too.

---

# 🚨 Real Software Problem

Suppose your query execution inside the database takes:

```text
20 ms
```

But the API response takes:

```text
500 ms
```

Developers might immediately blame SQL.

But:

```text
500 ms
```

could be:

```text
Network = 100 ms
Connection acquisition = 200 ms
Database execution = 20 ms
Application processing = 180 ms
```

So:

> **Query time and end-to-end request time are not the same thing.**

This distinction becomes extremely important in production.

---

# 2️⃣ Parser

The database receives:

```sql
SELECT name
FROM users
WHERE id = 100;
```

The parser checks whether the SQL is syntactically valid.

Think:

```text
SQL Text
   ↓
Syntax
   ↓
Internal Representation
```

For example:

```sql
SELECT FROM users;
```

is syntactically invalid.

The parser can reject it.

---

# 🧠 Parsing Is Not Execution

This is an important distinction.

The parser doesn't:

```text
Search users
```

It primarily answers:

> "Does this SQL have valid syntax, and how should I represent its structure internally?"

---

# 3️⃣ Analyzer

Now the database needs to understand what the SQL refers to.

For:

```sql
SELECT name
FROM users
WHERE id = 100;
```

the database needs to resolve:

```text
users
name
id
```

Questions include:

```text
Does users exist?

Does users.name exist?

Does users.id exist?

Are the referenced objects accessible?

Are the expressions type-compatible?
```

Conceptually:

```text
SQL
 ↓
Names
 ↓
Objects
 ↓
Types
 ↓
Permissions
```

---

# 🚨 Real Software Problem

Imagine a developer renames:

```text
users.email
```

to:

```text
users.email_address
```

but an application still sends:

```sql
SELECT email
FROM users;
```

The database can detect that the referenced column no longer exists.

This is why schema changes can affect applications.

---

# 4️⃣ Query Rewriter

Some database systems perform transformations between parsing/analysis and optimization.

For example, the database may need to expand or transform:

```text
Views
Subqueries
Rules
Expressions
```

The goal is to transform the query into an equivalent internal representation that can be optimized.

Conceptually:

```text
Original Query
      ↓
Rewrite
      ↓
Equivalent Query Representation
```

The exact rewriting behavior is database-specific.

---

# 5️⃣ Query Optimizer

Now we reach one of the most important components.

Suppose:

```text
users
=
100 million rows
```

Query:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

The database could potentially execute it in multiple ways.

---

# Option A — Sequential Scan

```text
users
 ↓
Read row 1
 ↓
Read row 2
 ↓
Read row 3
 ↓
...
 ↓
Read 100 million rows
 ↓
Find matching email
```

---

# Option B — Index Scan

Suppose:

```text
INDEX users_email_idx
```

exists.

The database may do:

```text
Email Index
    ↓
Find vinay@example.com
    ↓
Locate matching row
    ↓
Fetch row
```

Potentially much less work.

---

# 🧠 The Optimizer's Question

The optimizer asks:

> **Which available execution strategy is expected to be cheapest?**

It considers information such as:

```text
Table Statistics
Indexes
Estimated Row Counts
Data Distribution
Join Conditions
Predicates
Available Operators
System Cost Estimates
```

The exact optimization process depends on the database engine.

---

# 🧠 Cost-Based Optimization

Many modern relational databases use cost-based optimization.

Conceptually:

```text
Possible Plan A
      ↓
Estimated Cost = 100

Possible Plan B
      ↓
Estimated Cost = 20

Possible Plan C
      ↓
Estimated Cost = 50
```

The optimizer may choose:

```text
Plan B
```

because its estimated cost is lowest.

Important:

> **Cost is usually an internal estimate, not milliseconds.**

---

# 🔥 Important Production Lesson

Suppose the optimizer chooses:

```text
Sequential Scan
```

and your application becomes slow.

Don't immediately conclude:

```text
"PostgreSQL is broken."
```

Ask:

```text
What statistics did the optimizer have?

What row count did it estimate?

What row count actually exists?

What indexes exist?

What selectivity does the predicate have?

What plan did it choose?

Why?
```

This is how experienced database engineers investigate.

---

# 6️⃣ Execution Plan

The optimizer produces an execution plan.

For example, conceptually:

```text
Index Scan
    ↓
users_email_idx
    ↓
email = 'vinay@example.com'
```

Or:

```text
Seq Scan
    ↓
users
    ↓
Filter
    ↓
email = 'vinay@example.com'
```

The plan describes:

```text
HOW
```

the database intends to execute the query.

---

# 🧠 SQL vs Execution Plan

This distinction is critical.

SQL:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

is:

```text
WHAT
```

Execution plan:

```text
Index Scan
```

is:

```text
HOW
```

Think:

```text
SQL
 ↓
WHAT

Optimizer
 ↓
HOW
```

---

# 7️⃣ Executor

The executor receives the chosen plan.

Example:

```text
Index Scan
    ↓
Search index
    ↓
Find matching entry
    ↓
Fetch row
    ↓
Return result
```

The executor actually performs the work.

---

# 8️⃣ Buffer Cache

Now we reach the storage side.

Suppose the database needs a page.

The first question is often:

```text
Is the required data already in memory?
```

Conceptually:

```text
Executor
   ↓
Buffer Cache
   ↓
Is page present?
```

If yes:

```text
CACHE HIT
   ↓
Use memory
```

If no:

```text
CACHE MISS
   ↓
Read from storage
   ↓
Load page into memory
```

---

# ⚡ Why Memory Matters

Memory is dramatically faster than persistent storage.

So databases try to keep frequently accessed data in memory.

Conceptually:

```text
CPU
 ↓
Memory
 ↓
SSD
 ↓
Disk
```

The farther down you go, generally the more expensive access becomes in terms of latency.

Exact performance depends heavily on hardware and workload.

---

# 🔥 Real Production Problem

Suppose:

```text
Database RAM = 64 GB
```

and your working dataset is:

```text
10 GB
```

A large portion may remain cached.

Now the workload grows:

```text
Working Dataset
=
500 GB
```

Suddenly:

```text
Cache Hit Rate
↓
More Storage Reads
↓
More I/O
↓
Higher Latency
```

The SQL hasn't changed.

But performance changed.

This is why:

> **Query performance is a system problem, not only a SQL syntax problem.**

---

# 9️⃣ Storage

If the required page isn't available in memory, the database may need to read from persistent storage.

Conceptually:

```text
Database
   ↓
Operating System
   ↓
Filesystem / Storage Layer
   ↓
SSD
```

The database generally works with pages/blocks rather than thinking in terms of individual application-level rows being fetched directly from disk.

---

# 🧱 Pages

Databases commonly organize table and index storage into fixed-size pages or blocks.

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

Each page contains database-managed data structures and records.

The exact page size and format vary by database.

For example, PostgreSQL commonly uses:

```text
8 KB
```

database pages by default.

Do not generalize this value to every database.

---

# 🧠 Why Pages?

Storage devices don't naturally operate like:

```text
"Give me customer 1001."
```

The database organizes data into manageable units.

Conceptually:

```text
Table
 ↓
Pages
 ↓
Records
```

This becomes important when understanding:

```text
Sequential Scan
Index Scan
I/O
Buffer Cache
Table Bloat
Vacuum
Storage Layout
```

---

# 🔥 Complete Query Journey

Let's follow:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

from application to disk.

```text
Application
     │
     ▼
Database Driver
     │
     ▼
Network
     │
     ▼
Database Server
     │
     ▼
Parser
     │
     ▼
Analyzer
     │
     ▼
Rewriter
     │
     ▼
Optimizer
     │
     ▼
Execution Plan
     │
     ▼
Executor
     │
     ▼
Buffer Cache
     │
     ├───────────────┐
     │               │
   HIT             MISS
     │               │
     │               ▼
     │             Storage
     │               │
     │               ▼
     │              Page
     │               │
     └───────┬───────┘
             ▼
          Result
             │
             ▼
          Network
             │
             ▼
        Application
```

This is the mental model you should remember.

---

# 🚨 Real Production Incident

Imagine your API endpoint:

```text
GET /customers/1001/orders
```

normally responds in:

```text
100 ms
```

One day:

```text
8 seconds
```

The developer looks at the SQL:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

and says:

> "The query looks simple."

That is not enough.

---

# 🔍 Step 1 — Measure Database Time

Determine:

```text
How long does the database actually take?
```

Suppose:

```text
Database = 7.5 sec
```

Now we know the database is involved.

---

# 🔍 Step 2 — Check Execution Plan

Use an appropriate database-specific tool such as:

```sql
EXPLAIN
```

and, where appropriate:

```sql
EXPLAIN ANALYZE
```

The exact syntax and behavior vary by database.

For PostgreSQL:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 1001;
```

---

# 🔍 Step 3 — Compare Estimates With Reality

Suppose the plan estimates:

```text
Estimated rows = 100
```

but execution finds:

```text
Actual rows = 10,000,000
```

That's a huge mismatch.

Potential causes include:

```text
Outdated statistics
Data distribution changes
Correlation
Skew
Poor cardinality estimates
```

Now we have a real investigation.

---

# 🔍 Step 4 — Check Access Path

Maybe the database chooses:

```text
Sequential Scan
```

because it believes that is cheaper.

But perhaps:

```text
customer_id
```

has highly selective values and a suitable index would be beneficial.

Now investigate:

```text
Does the index exist?

Is it usable?

Is the predicate selective?

Are statistics current?

Is the table small enough that sequential scan is cheaper?
```

---

# 🔍 Step 5 — Check Storage

Suppose the query reads:

```text
Millions of pages
```

Now investigate:

```text
Disk throughput
Latency
Cache hit ratio
Memory pressure
Concurrent I/O
```

---

# 🔍 Step 6 — Check Concurrency

Maybe the query is not doing huge amounts of work.

Maybe it is waiting.

For example:

```text
Query
 ↓
Lock
 ↓
WAIT
 ↓
3 seconds
```

This is fundamentally different from:

```text
Query
 ↓
CPU
 ↓
3 seconds
```

A senior engineer must distinguish:

```text
Work
```

from:

```text
Waiting
```

---

# 🧠 Slow Query ≠ Always Bad SQL

A slow query can result from:

```text
Bad Query
Bad Plan
Missing Index
Wrong Index
Stale Statistics
Lock Contention
CPU Saturation
Memory Pressure
Storage Latency
Network Latency
Data Growth
Concurrency
```

This is why database troubleshooting requires systems thinking.

---

# 🔥 Same SQL, Different Performance

Suppose:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

Yesterday:

```text
10 ms
```

Today:

```text
2 seconds
```

The SQL text is identical.

What changed?

Possibilities:

```text
10,000 rows → 10 million rows

Statistics changed

Data distribution changed

Cache became cold

Index changed

Execution plan changed

Storage became slower

Concurrent workload increased
```

This is one of the most important ideas in production SQL engineering.

---

# 🧠 SQL Is Not the Whole Database

Think in layers:

```text
                 SQL
                  ↓
            Query Processing
                  ↓
             Optimization
                  ↓
              Execution
                  ↓
               Cache
                  ↓
              Storage
                  ↓
            Operating System
                  ↓
              Hardware
```

A query can be affected by any layer.

---

# 🧪 Hands-on Lab

## Lab 1 — Create a Table

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

---

# Lab 2 — Insert Data

Start with:

```text
100 rows
```

Then:

```text
10,000 rows
```

Then:

```text
1,000,000 rows
```

Observe how execution behavior changes.

---

# Lab 3 — Run EXPLAIN

Run:

```sql
EXPLAIN
SELECT *
FROM users
WHERE id = 100;
```

Look for:

```text
Scan Type
Estimated Rows
Estimated Cost
```

Don't worry about understanding every field yet.

---

# Lab 4 — Add an Index

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Then:

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

Compare the plan.

---

# Lab 5 — Use EXPLAIN ANALYZE

Run:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

Now compare:

```text
Estimated Rows
```

with:

```text
Actual Rows
```

This is your first step toward real query-performance analysis.

---

# 🧪 Lab 6 — Change Data Distribution

Insert many rows with the same value:

```text
department = Engineering
```

and a small number with:

```text
department = HR
```

Now ask:

> Would an index be equally useful for both values?

This introduces:

```text
Selectivity
Data Distribution
Cardinality
```

which will become critical later.

---

# 🎯 Interview Questions

## Beginner

### What is SQL?

SQL is a language used to define, query, manipulate, and control data in relational database systems.

---

### Is SQL procedural or declarative?

SQL is primarily declarative: you describe the desired result, while the database determines an execution strategy.

---

### What is a query optimizer?

A database component that evaluates possible execution strategies and selects an execution plan based on estimated costs and available information.

---

### What is an execution plan?

A representation of how the database intends to execute a query.

---

### What is a sequential scan?

An access method that examines table pages/rows sequentially to find qualifying records.

---

### What is an index scan?

An access method that uses an index to locate qualifying rows, potentially reducing the amount of table data that must be examined.

---

# 🧠 Intermediate Questions

### Why can the same SQL query have different execution plans?

Because the optimizer may have different:

```text
Statistics
Indexes
Data Volumes
Data Distribution
Configuration
Cost Estimates
```

and database versions may also change optimizer behavior.

---

### What is the difference between estimated rows and actual rows?

Estimated rows are what the optimizer expects an operation to produce.

Actual rows are what execution actually produces.

A large mismatch can indicate estimation problems.

---

### Why are statistics important?

The optimizer uses statistics to estimate:

```text
Row Counts
Selectivity
Data Distribution
```

and choose execution plans.

---

### Why can a query be slow even when it uses an index?

Because:

```text
Index access has overhead
```

and the query may still need to fetch many table rows.

If a large fraction of the table qualifies, another access path may be cheaper.

---

# 👑 Senior Questions

### Why isn't "using an index" automatically good?

Because the goal is not:

```text
Use Index
```

The goal is:

```text
Minimize total work
```

For some queries:

```text
Sequential Scan
```

is cheaper.

For others:

```text
Index Scan
```

is better.

---

### What is more important than knowing SQL syntax?

Understanding:

```text
Data Model
Query Semantics
Execution Plans
Indexes
Statistics
Transactions
Concurrency
Storage
```

---

### Why should you compare estimated and actual rows?

Because a large difference can explain why the optimizer chose a poor plan.

For example:

```text
Estimated:
100 rows

Actual:
10,000,000 rows
```

can dramatically affect plan quality.

---

# 👑 20+ Year Experience Questions

## Question 1

Your query changed from:

```text
20 ms
```

to:

```text
5 seconds
```

but:

```text
SQL text = identical
```

What do you investigate?

A senior engineer should think:

```text
Execution Plan
Statistics
Data Growth
Data Distribution
Indexes
Cache
Locks
I/O
CPU
Memory
Concurrency
```

---

# Question 2

The optimizer chooses a sequential scan even though an index exists.

Is that necessarily wrong?

No.

Ask:

```text
How many rows qualify?

How large is the table?

How selective is the predicate?

What is the estimated cost?

What is the actual cost?
```

---

# Question 3

The optimizer estimates:

```text
10 rows
```

but execution returns:

```text
10 million rows
```

What could cause this?

Think:

```text
Statistics
Data Skew
Correlated Columns
Predicate Selectivity
Cardinality Estimation
```

---

# Question 4

Your query takes:

```text
10 seconds
```

but CPU usage is only:

```text
5%
```

What does that suggest?

Possibly:

```text
Waiting
I/O
Locks
Network
Resource Contention
```

The important lesson:

> Low CPU does not automatically mean the database is healthy.

---

# Question 5

Two identical SQL queries run on two different databases.

Database A:

```text
20 ms
```

Database B:

```text
2 seconds
```

Why?

Think:

```text
Indexes
Statistics
Data Size
Data Distribution
Execution Plan
Hardware
Memory
Cache
Storage
Configuration
Database Version
Concurrency
```

---

# Question 6

Why does SQL allow the database to choose the execution strategy instead of forcing the developer to specify it?

Because the database has access to information the application usually doesn't want to manage directly:

```text
Indexes
Statistics
Storage
Data Distribution
Current Resource State
Concurrency
```

This allows the database to optimize execution without exposing all physical implementation details to the application.

---

# 🔥 Senior Mental Model

When someone says:

> "This SQL query is slow."

Don't immediately ask:

```text
"What SQL should we change?"
```

Ask:

```text
1. What SQL is actually executing?

2. What execution plan was chosen?

3. Why was that plan chosen?

4. What did the optimizer estimate?

5. What actually happened?

6. Where is time being spent?

7. Is the query doing work or waiting?

8. How much data is involved?

9. What changed?
```

This mindset separates:

```text
SQL Developer
```

from:

```text
Database Engineer
```

---

# 🧠 The 7 Most Important Questions

Whenever you investigate a production SQL problem, ask:

```text
1. WHAT query is running?

2. HOW is it executing?

3. WHY did the optimizer choose that plan?

4. HOW MUCH data is being processed?

5. WHERE is the time being spent?

6. WHAT changed?

7. CAN the workload or architecture be improved?
```

---

# 📌 Chapter Summary

SQL is primarily a:

```text
Declarative Language
```

You describe:

```text
WHAT
```

you want.

The database determines:

```text
HOW
```

to obtain it.

The conceptual pipeline is:

```text
SQL
 ↓
Parser
 ↓
Analyzer
 ↓
Rewriter
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
Storage
 ↓
Result
```

The optimizer uses information such as:

```text
Statistics
Indexes
Data Distribution
Estimated Costs
```

to choose an execution plan.

The execution plan is the bridge between:

```text
SQL
```

and:

```text
Actual Work
```

And production performance depends on much more than SQL syntax:

```text
SQL
+
Plan
+
Data
+
Indexes
+
Statistics
+
Cache
+
CPU
+
Memory
+
I/O
+
Locks
+
Concurrency
+
Network
```

---

# 🔥 Final Mental Model

Remember this:

```text
                SQL
                 │
                 │ WHAT?
                 ▼
          Query Processing
                 │
                 ▼
             Optimizer
                 │
                 │ HOW?
                 ▼
          Execution Plan
                 │
                 ▼
             Executor
                 │
        ┌────────┴────────┐
        ▼                 ▼
      Index              Table
        │                 │
        └────────┬────────┘
                 ▼
            Buffer Cache
                 │
          ┌──────┴──────┐
          ▼             ▼
       Memory          Disk
          │             │
          └──────┬──────┘
                 ▼
               Result
                 │
                 ▼
             Application
```

The fundamental principle is:

> **SQL describes the result. The database engine determines the work required to produce it.**

Once you understand this, SQL performance stops being a collection of tricks and starts becoming a problem of **query semantics, algorithms, data structures, statistics, and system resources**.

---

# 🚀 Next Chapter

# Chapter 5 — SQL Language Fundamentals

We will finally start building SQL from the ground up:

```text
SELECT
FROM
WHERE
ORDER BY
LIMIT
DISTINCT
NULL
Expressions
Aliases
Operators
Data Types
```

But we will **not** learn them as isolated commands.

We will build a real application database and use every SQL concept to solve actual software problems.

The progression will be:

```text
SQL Syntax
   ↓
Query Semantics
   ↓
Relational Operations
   ↓
Execution Plans
   ↓
Performance
```

That is the foundation required for eventually reaching **senior/principal-level SQL and database engineering**.
