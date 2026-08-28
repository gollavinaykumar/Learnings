# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 15 — SQL Indexes Deep Dive: B-Tree, Composite Indexes, Covering Indexes & Query Performance

> Imagine you have a table containing:
>
> ```text
> 500,000,000 users
> ```
>
> And you execute:
>
> ```sql
> SELECT *
> FROM users
> WHERE email = 'vinay@example.com';
> ```
>
> Without an appropriate index, the database may need to inspect a huge number of rows:
>
> ```text
> users
>  ↓
> Row 1
> Row 2
> Row 3
> Row 4
> ...
> Row 500,000,000
> ```
>
> This is called:
>
> # Table Scan
>
> Now imagine an index:
>
> ```text
> email
>   ↓
> B-Tree
>   ↓
> vinay@example.com
>   ↓
> Row location
> ```
>
> The database can navigate the index rather than examining every row.
>
> But here's the important part:
>
> **An index is not magic.**
>
> It is another data structure that the database must:
>
> ```text
> Create
> Store
> Search
> Maintain
> Update
> Log
> Replicate
> ```
>
> So indexes create a fundamental trade-off:
>
> ```text
> Faster Reads
>      ↕
> More Write Cost
> ```
>
> A developer with basic SQL knowledge asks:
>
> > "Which columns should I index?"
>
> A senior database engineer asks:
>
> ```text
> What queries do we run?
> What predicates are selective?
> What joins are performed?
> What ordering is required?
> How large is the table?
> How frequently is the table modified?
> What is the workload?
> What does EXPLAIN say?
> Does the optimizer estimate the cardinality correctly?
> Is the index actually cheaper than a scan?
> ```
>
> This chapter builds that mental model.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What is an index?
* Why indexes exist
* Table scans
* Index scans
* Index lookups
* B-Tree indexes
* B-Tree structure
* Root, branch and leaf pages
* Index selectivity
* Cardinality
* Composite indexes
* Column order
* Leftmost-prefix behavior
* Covering indexes
* Unique indexes
* Partial / filtered indexes
* Clustered indexes
* Non-clustered indexes
* Hash indexes
* Expression indexes
* Index-only scans
* Index condition pushdown
* Indexes and JOINs
* Indexes and ORDER BY
* Indexes and GROUP BY
* Indexes and pagination
* Why indexes are sometimes ignored
* Functions on indexed columns
* `LIKE`
* Wildcards
* Statistics
* Query optimizers
* `EXPLAIN`
* `EXPLAIN ANALYZE`
* Index maintenance
* Write amplification
* Production index design

---

# 📖 Why Do Indexes Exist?

Imagine a physical book.

You want:

```text
"Transactions"
```

Without an index:

```text
Page 1
Page 2
Page 3
...
Page 1000
```

You search every page.

With a book index:

```text
Transactions
    ↓
Page 742
```

You jump directly to the relevant section.

A database index works on the same basic idea:

```text
Query
 ↓
Index
 ↓
Find candidate rows
 ↓
Fetch data
```

---

# 1️⃣ Table Scan

Suppose:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

and there is no useful index.

The database may choose:

```text
Table
 ↓
Scan rows
 ↓
Check email
 ↓
Return matching rows
```

Conceptually:

```text
Row 1 → email? ❌
Row 2 → email? ❌
Row 3 → email? ❌
Row 4 → email? ❌
...
Row 500M → email? ✅
```

This is expensive when the table is large and only a tiny number of rows match.

---

# 2️⃣ Index Scan

Now create:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

The database now has an additional structure:

```text
users table

+

idx_users_email
```

Query:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

can potentially use:

```text
Query
 ↓
Index
 ↓
Find email
 ↓
Find matching row(s)
 ↓
Fetch table data
```

---

# 🧠 Important

Creating an index does **not** guarantee the optimizer will use it.

The optimizer chooses a plan based on:

```text
Estimated cost
Statistics
Table size
Selectivity
Available indexes
Query predicates
Join conditions
Ordering
Database engine
```

---

# 3️⃣ What Is a B-Tree?

One of the most common index structures is:

```text
B-Tree
```

Many databases use a B-Tree variant such as:

```text
B+Tree
```

The exact implementation differs by database.

A simplified structure:

```text
                 Root
              /        \
            /            \
         Branch          Branch
        /    \           /    \
       /      \         /      \
    Leaf     Leaf     Leaf     Leaf
```

The tree remains balanced.

---

# 4️⃣ Why a Tree?

Suppose values are:

```text
10
20
30
40
50
60
70
80
90
```

Searching linearly:

```text
10 → 20 → 30 → 40 → ...
```

could require many comparisons.

A balanced tree narrows the search space:

```text
              50
            /    \
          <50    >50
          /        \
        ...        ...
```

Each step eliminates a large portion of possible values.

---

# 5️⃣ B-Tree Search

Suppose the index contains:

```text
10
20
30
40
50
60
70
80
90
```

Search:

```text
70
```

Conceptually:

```text
Root
 ↓
50
 ↓
70+
 ↓
70
```

You don't need to inspect every value.

---

# 🧠 Complexity Mental Model

A balanced tree typically gives search behavior roughly proportional to:

```text
O(log N)
```

rather than:

```text
O(N)
```

for a simple lookup model.

But real database performance also depends on:

```text
Disk I/O
Memory
Cache
Index depth
Random access
Row width
Concurrency
Storage engine
```

So don't treat Big-O alone as a complete performance model.

---

# 6️⃣ Index Pages

A database doesn't usually think of an index as:

```text
One giant object
```

Instead, storage is organized into pages or blocks.

Conceptually:

```text
B-Tree
 ↓
Pages
 ↓
Rows / keys / pointers
```

For example:

```text
Root Page
   ↓
Branch Page
   ↓
Leaf Page
```

---

# 7️⃣ Leaf Pages

In a simplified B-Tree:

```text
Leaf
 ↓
Indexed Key
 +
Reference / row location
```

For example:

```text
email
-------------------------
a@example.com → row
b@example.com → row
c@example.com → row
vinay@example.com → row
```

The exact row-reference behavior depends on the database engine.

---

# 8️⃣ Index Lookup

Suppose:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

The database may perform:

```text
Query
 ↓
B-Tree root
 ↓
Branch page
 ↓
Leaf page
 ↓
email found
 ↓
Row location
 ↓
Table access
 ↓
Return row
```

This is why indexes can dramatically reduce the amount of data that must be examined.

---

# 9️⃣ Index Does Not Usually Store the Whole Row

Suppose:

```text
users
```

contains:

```text
id
name
email
phone
address
date_of_birth
created_at
...
```

An index on:

```text
email
```

generally stores information needed to navigate by email plus a way to identify the underlying row, depending on the database.

Conceptually:

```text
Index

email
  ↓
row reference
```

not necessarily:

```text
email
name
phone
address
...
```

---

# 🔟 Index Scan + Table Lookup

Suppose:

```sql
SELECT name
FROM users
WHERE email = 'vinay@example.com';
```

The database may:

```text
Index
 ↓
Find email
 ↓
Find row
 ↓
Read table
 ↓
Return name
```

There are therefore two conceptual structures:

```text
Index
 ↓
Find row
 ↓
Table
 ↓
Get remaining columns
```

This is sometimes called:

```text
Index lookup + heap/table fetch
```

Terminology differs by database.

---

# 1️⃣1️⃣ Covering Index

Now suppose the query is:

```sql
SELECT email, name
FROM users
WHERE email = 'vinay@example.com';
```

If the index contains all required data:

```text
email
name
```

the database may be able to answer the query using the index itself.

Conceptually:

```text
Query
 ↓
Index
 ↓
Everything needed
 ↓
Return
```

This is a:

# Covering Index

---

# 1️⃣2️⃣ Example Covering Index

Depending on your database:

```sql
CREATE INDEX idx_users_email_name
ON users(email, name);
```

Query:

```sql
SELECT name
FROM users
WHERE email = 'vinay@example.com';
```

The index contains:

```text
email
name
```

So a table lookup may not be necessary.

Some databases describe this as an:

```text
Index-only scan
```

when the engine's visibility and storage rules allow it.

---

# 🧠 Covering Index Trade-Off

Covering indexes can improve reads.

But:

```text
More columns
 ↓
Larger index
 ↓
More storage
 ↓
More cache usage
 ↓
More write maintenance
```

Therefore:

> Don't put every selected column into every index.

---

# 1️⃣3️⃣ Selectivity

Suppose:

```text
users = 100 million
```

Column:

```text
gender
```

has:

```text
Male
Female
```

Only two distinct values.

Searching:

```sql
WHERE gender = 'Male'
```

may return:

```text
50 million rows
```

An index may not provide much benefit compared with scanning a large portion of the table.

Now:

```text
email
```

might have:

```text
100 million mostly unique values
```

Searching:

```sql
WHERE email = 'vinay@example.com'
```

returns:

```text
1 row
```

This is highly selective.

---

# 👑 Selectivity

A simplified mental model:

```text
High selectivity
→ Small fraction of rows match
→ Index often attractive
```

```text
Low selectivity
→ Large fraction of rows match
→ Scan may be cheaper
```

---

# 1️⃣4️⃣ Cardinality

Cardinality describes the number of distinct values in a column or result, depending on context.

Examples:

```text
gender
→ 2 distinct values

country
→ ~200 values

email
→ potentially millions of distinct values
```

High cardinality often provides more useful filtering power.

But:

> High cardinality alone does not guarantee an index will be useful.

The actual query and data distribution matter.

---

# 1️⃣5️⃣ Index Selectivity vs Cardinality

Think:

```text
Cardinality
=
How many distinct values?
```

while:

```text
Selectivity
=
How much does the predicate narrow the result?
```

For example:

```text
100M rows

email:
100M distinct values

WHERE email = exact value
→ approximately 1 row
```

Very selective.

But:

```text
status:
3 distinct values

WHERE status = 'ACTIVE'
→ 90M rows
```

Low selectivity.

---

# 1️⃣6️⃣ Composite Index

Suppose you frequently query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND status = 'PAID';
```

You may create:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

This is a:

# Composite Index

It contains multiple columns.

---

# 1️⃣7️⃣ Why Column Order Matters

Consider:

```text
(customer_id, status)
```

This is not equivalent to:

```text
(status, customer_id)
```

The order affects how the index can efficiently support predicates.

Think of:

```text
(customer_id, status)
```

as:

```text
Customer 101
   ↓
PAID
PENDING
SHIPPED

Customer 102
   ↓
PAID
PENDING
SHIPPED
```

The index is primarily ordered by:

```text
customer_id
```

then:

```text
status
```

---

# 1️⃣8️⃣ Leftmost Prefix

For an index:

```sql
CREATE INDEX idx_orders
ON orders(customer_id, status, created_at);
```

think:

```text
(customer_id)
(customer_id, status)
(customer_id, status, created_at)
```

as natural leading prefixes.

Depending on the database optimizer, queries that constrain the leading columns can use the index effectively.

But a query only on:

```text
status
```

cannot generally exploit the same index as effectively as one beginning with:

```text
customer_id
```

---

# 🔥 Example

Index:

```text
(customer_id, status)
```

Query:

```sql
WHERE customer_id = 101
```

Good candidate.

Query:

```sql
WHERE customer_id = 101
AND status = 'PAID'
```

Good candidate.

Query:

```sql
WHERE status = 'PAID'
```

The index may not be the best access path.

---

# 🧠 Important Nuance

Don't memorize:

```text
"Second column can never be used."
```

Modern database optimizers have advanced techniques.

The correct principle is:

> The leading columns of a composite index strongly influence how the index can be navigated efficiently.

Always verify with:

```text
EXPLAIN
```

---

# 1️⃣9️⃣ Composite Index Design

Suppose API:

```text
GET /orders?customer_id=101&status=PAID
```

and another API:

```text
GET /orders?customer_id=101
```

A useful index might be:

```text
(customer_id, status)
```

because both queries constrain:

```text
customer_id
```

---

# 2️⃣0️⃣ Add ORDER BY

Now query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
ORDER BY created_at DESC;
```

Potential index:

```text
(customer_id, created_at)
```

The index may help both:

```text
WHERE customer_id = 101
```

and:

```text
ORDER BY created_at DESC
```

depending on the database and requested ordering.

---

# 🧠 Index Can Solve Multiple Problems

One carefully designed index can potentially support:

```text
Filtering
+
Ordering
```

rather than creating separate indexes for everything.

---

# 2️⃣1️⃣ Equality Before Range

Suppose query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND created_at >= '2026-01-01'
ORDER BY created_at;
```

A common index design is:

```text
(customer_id, created_at)
```

Why?

Because:

```text
customer_id = 101
```

is an equality condition.

Then:

```text
created_at >= ...
```

is a range condition.

Mental model:

```text
(customer_id)
       ↓
(created_at range)
```

This is a common index-design heuristic.

---

# 2️⃣2️⃣ Index and JOIN

Suppose:

```sql
SELECT *
FROM orders o
JOIN customers c
  ON o.customer_id = c.id;
```

An index on:

```text
customers.id
```

is normally already provided by the primary key.

But what about:

```text
orders.customer_id
```

?

An index there can be extremely useful for certain join strategies.

Example:

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

Now:

```text
customers
    ↓
customer_id
    ↓
orders index
    ↓
matching orders
```

---

# 👑 Foreign Key Indexing

A foreign key does not universally imply that the database automatically creates an index.

Database behavior varies.

Often you should consider indexing the referencing column:

```text
orders.customer_id
```

especially when queries frequently:

```text
JOIN
Filter
DELETE parent
UPDATE parent key
```

depending on the database and workload.

---

# 2️⃣3️⃣ Index and ORDER BY

Query:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 20;
```

Without a useful index:

```text
Read many rows
 ↓
Sort
 ↓
Take top 20
```

Potential index:

```text
(created_at)
```

can allow the database to retrieve rows in useful order without performing a full sort.

Again:

> Verify the actual execution plan.

---

# 2️⃣4️⃣ Index and GROUP BY

Query:

```sql
SELECT customer_id, COUNT(*)
FROM orders
GROUP BY customer_id;
```

An index on:

```text
customer_id
```

may help some execution strategies.

But don't assume:

```text
GROUP BY column
=
index always faster
```

The optimizer considers:

```text
Table size
Distinct values
Data distribution
Sort cost
Hash aggregation cost
Index cost
```

---

# 2️⃣5️⃣ Why Optimizer Sometimes Ignores an Index

Suppose:

```text
Table:
100 million rows
```

Query:

```sql
WHERE status = 'ACTIVE'
```

and:

```text
90 million rows
```

match.

Using the index might mean:

```text
Index
 ↓
90M index entries
 ↓
90M table accesses
```

A sequential scan may be cheaper:

```text
Table
 ↓
Sequential scan
 ↓
Filter
```

Therefore:

> The optimizer may correctly ignore the index.

---

# 2️⃣6️⃣ The Optimizer Is Cost-Based

Modern relational databases generally use a cost-based optimizer.

Conceptually:

```text
SQL
 ↓
Parse
 ↓
Rewrite
 ↓
Generate possible plans
 ↓
Estimate costs
 ↓
Choose plan
 ↓
Execute
```

Possible plans:

```text
Sequential Scan
Index Scan
Index-Only Scan
Nested Loop
Hash Join
Merge Join
Sort
Hash Aggregate
...
```

The optimizer estimates which plan is likely to be cheapest.

---

# 2️⃣7️⃣ Statistics

How does the optimizer estimate:

```text
90 million rows?
```

It uses statistics.

Statistics can describe things such as:

```text
Row counts
Distinct values
Value distributions
Histograms
Correlation
```

The exact statistics system differs by database.

---

# 🔥 Bad Statistics

Suppose optimizer thinks:

```text
Query returns 10 rows
```

but reality is:

```text
10 million rows
```

It may choose:

```text
Nested Loop
```

when:

```text
Hash Join
```

would be better.

Result:

```text
Query becomes extremely slow
```

This is why:

# Cardinality Estimation

is one of the most important topics in query optimization.

---

# 2️⃣8️⃣ EXPLAIN

Before optimizing a query, inspect its execution plan.

For example:

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

You may see something conceptually like:

```text
Index Scan
  Index: idx_users_email
  Condition: email = ...
```

The exact output differs by database.

---

# 2️⃣9️⃣ EXPLAIN ANALYZE

Some databases support:

```sql
EXPLAIN ANALYZE
```

which executes the query while reporting actual runtime information.

Conceptually:

```text
Estimated:
100 rows

Actual:
1,000,000 rows
```

This tells you:

```text
Optimizer estimate
vs
Reality
```

This is extremely valuable for performance debugging.

---

# ⚠️ EXPLAIN ANALYZE Runs the Query

Important:

```text
EXPLAIN
```

often means:

```text
Show plan
```

while:

```text
EXPLAIN ANALYZE
```

typically means:

```text
Execute + measure
```

For:

```text
SELECT
```

this may simply read data.

For:

```text
UPDATE
DELETE
INSERT
```

it can actually modify data unless the database provides a safe alternative.

Never blindly run it against production writes.

---

# 3️⃣0️⃣ Function on Indexed Column

Suppose:

```text
created_at
```

is indexed.

Query:

```sql
SELECT *
FROM orders
WHERE DATE(created_at) = '2026-08-28';
```

Depending on the database and optimizer, applying a function to the column may prevent efficient use of a normal index.

A range predicate is often more index-friendly:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-28'
  AND created_at <  '2026-08-29';
```

This allows the index to navigate directly to the relevant time range.

---

# 👑 General Rule

Instead of:

```text
FUNCTION(indexed_column)
```

consider whether you can rewrite as:

```text
indexed_column
```

with a searchable range or predicate.

But there are database-specific exceptions such as expression/function indexes.

---

# 3️⃣1️⃣ Expression Index

Some databases support indexes on expressions.

For example, conceptually:

```sql
CREATE INDEX idx_users_lower_email
ON users(LOWER(email));
```

Then:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'vinay@example.com';
```

may use the expression index.

This is useful when the query logically needs the transformed value.

Exact syntax and capabilities vary by database.

---

# 3️⃣2️⃣ LIKE and Indexes

Consider:

```sql
WHERE email LIKE 'vinay%'
```

A B-Tree index may often help because the pattern has a fixed prefix.

But:

```sql
WHERE email LIKE '%vinay%'
```

is much harder for a normal B-Tree to optimize because the beginning is unknown.

Mental model:

```text
'vinay%'
```

means:

```text
Starts with vinay
```

while:

```text
'%vinay%'
```

means:

```text
Contains vinay anywhere
```

---

# 🧠 For Search

If your application needs:

```text
Full-text search
Substring search
Fuzzy search
Typo tolerance
Relevance ranking
```

a traditional B-Tree may not be the correct tool.

You may need:

```text
Full-text indexes
Trigram indexes
Search engines
Specialized indexes
```

depending on your database and requirements.

---

# 3️⃣3️⃣ Unique Index

Suppose:

```text
email
```

must be unique.

A unique constraint may be backed by a unique index in many relational databases.

Conceptually:

```text
email
 ↓
Unique index
 ↓
Prevent duplicates
```

Example:

```sql
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

or, preferably when expressing a business rule:

```sql
ALTER TABLE users
ADD CONSTRAINT uq_users_email
UNIQUE(email);
```

The exact implementation differs by database.

---

# 👑 Constraint vs Index

Think:

```text
UNIQUE constraint
=
Business/data rule
```

while:

```text
Index
=
Access structure
```

A unique constraint may use an index internally, but conceptually they serve different purposes.

---

# 3️⃣4️⃣ Partial Index

Some databases support partial indexes.

Example:

```text
orders
```

contains:

```text
status
```

and most orders are:

```text
COMPLETED
```

but the application frequently queries:

```text
status = 'PENDING'
```

A partial index can target only the relevant subset.

For example in PostgreSQL:

```sql
CREATE INDEX idx_pending_orders
ON orders(created_at)
WHERE status = 'PENDING';
```

Now:

```text
Index
 ↓
Only pending orders
```

This can be much smaller than indexing every row.

---

# 🧠 Why Smaller Indexes Matter

Smaller:

```text
Index
```

means potentially:

```text
Less storage
Better cache residency
Less maintenance
Fewer pages to scan
```

But partial indexes are database-specific.

---

# 3️⃣5️⃣ Clustered Index

Some databases support the concept of a clustered index or organize table storage around a primary/index structure.

A common conceptual model:

```text
Clustered structure
 ↓
Index + row storage relationship
```

This can make certain range scans very efficient.

But:

> "Clustered index" means different things across database systems.

For example:

```text
SQL Server
MySQL InnoDB
PostgreSQL
```

have substantially different storage architectures.

Do not assume terminology maps one-to-one.

---

# 3️⃣6️⃣ InnoDB Mental Model

In MySQL InnoDB, the primary key is the clustered index.

Conceptually:

```text
Primary Key
     ↓
B-Tree
     ↓
Actual row data
```

Secondary indexes contain their indexed values plus the primary key needed to locate the clustered row.

Therefore a secondary-index lookup can conceptually become:

```text
Secondary Index
      ↓
Primary Key
      ↓
Clustered Index
      ↓
Row
```

This has important performance implications.

---

# 3️⃣7️⃣ Secondary Index Cost

Suppose:

```text
Primary Key:
id
```

and:

```text
Secondary Index:
email
```

Query:

```sql
WHERE email = 'vinay@example.com'
```

may involve:

```text
email index
    ↓
find primary key
    ↓
primary key index
    ↓
find row
```

This is one reason primary-key width matters in storage engines such as InnoDB.

---

# 3️⃣8️⃣ Primary Key Width

Suppose primary key is:

```text
BIGINT
```

versus:

```text
large VARCHAR
```

If secondary indexes store the primary key as part of their entries, a wide primary key can make every secondary index larger.

Conceptually:

```text
Small PK
 ↓
Smaller secondary indexes
```

versus:

```text
Large PK
 ↓
Larger secondary indexes
 ↓
More storage
 ↓
More cache pressure
```

This is an important schema-design consideration.

---

# 3️⃣9️⃣ Pagination and Indexes

Consider:

```sql
SELECT *
FROM orders
ORDER BY id
LIMIT 20 OFFSET 10000000;
```

The database may still need to walk past a huge number of rows before returning the requested page.

This is:

# OFFSET Pagination

It can become expensive at large offsets.

---

# 4️⃣0️⃣ Keyset Pagination

Instead of:

```text
OFFSET 10,000,000
```

use a known last key:

```sql
SELECT *
FROM orders
WHERE id > 10000000
ORDER BY id
LIMIT 20;
```

Mental model:

```text
Page 1
 ↓
last_id = 20

Page 2
 ↓
WHERE id > 20

Page 3
 ↓
WHERE id > 40
```

This is commonly called:

```text
Keyset Pagination
```

or:

```text
Cursor Pagination
```

when implemented with an opaque cursor.

---

# 🔥 Why Keyset Is Powerful

With a suitable index:

```text
id
```

the database can jump directly to:

```text
id > last_seen_id
```

instead of processing millions of earlier rows.

---

# 4️⃣1️⃣ Index and Range Queries

Suppose:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-01-01'
  AND created_at < '2026-02-01';
```

A B-Tree index on:

```text
created_at
```

is naturally suited to range navigation.

Conceptually:

```text
B-Tree
 ↓
January start
 ↓
Read relevant range
```

rather than:

```text
Entire table
 ↓
Check every row
```

---

# 4️⃣2️⃣ Index and Sorting

Query:

```sql
SELECT *
FROM orders
ORDER BY created_at;
```

Without index:

```text
Read
 ↓
Sort
 ↓
Return
```

With suitable index:

```text
Index
 ↓
Already ordered
 ↓
Return
```

But whether the index is cheaper depends on:

```text
How many rows?
LIMIT?
Direction?
Other predicates?
```

---

# 4️⃣3️⃣ Index and LIMIT

Query:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 10;
```

With a suitable index:

```text
Index
 ↓
Newest rows
 ↓
10 rows
```

The database may avoid sorting the entire table.

This is one of the most useful index patterns for APIs.

---

# 4️⃣4️⃣ API Index Design

Suppose API:

```text
GET /customers/101/orders
```

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
ORDER BY created_at DESC
LIMIT 20;
```

A strong candidate:

```text
(customer_id, created_at)
```

Mental model:

```text
customer_id
     ↓
created_at
     ↓
newest orders
     ↓
LIMIT 20
```

This can support:

```text
Filtering
+
Ordering
+
Pagination
```

---

# 👑 Senior Index Design Principle

> Design indexes from real query patterns, not from table columns in isolation.

---

# 4️⃣5️⃣ Bad Index Strategy

A beginner may create:

```text
Index(email)
Index(name)
Index(phone)
Index(city)
Index(status)
Index(country)
Index(created_at)
Index(updated_at)
...
```

for every column.

Looks safe.

It isn't.

Now every write may need to maintain many structures.

```text
INSERT
 ↓
Table
 +
Index 1
 +
Index 2
 +
Index 3
 +
Index 4
 +
...
```

---

# 4️⃣6️⃣ Write Amplification

Suppose a table has:

```text
10 indexes
```

and you insert:

```text
1 million rows
```

The database may need to maintain:

```text
Table
+
10 index structures
```

So:

```text
One logical INSERT
```

can result in substantial physical work.

This is:

# Write Amplification

---

# 4️⃣7️⃣ UPDATE and Indexes

Suppose:

```text
status
```

is indexed.

Query:

```sql
UPDATE orders
SET status = 'SHIPPED'
WHERE id = 101;
```

If `status` changes, the index may need maintenance.

So:

```text
UPDATE
 ↓
Table change
+
Index change
```

More indexes:

```text
Potentially more write work
```

---

# 4️⃣8️⃣ DELETE and Indexes

Suppose:

```text
orders
```

has:

```text
5 indexes
```

Delete:

```sql
DELETE FROM orders
WHERE id = 101;
```

The database must maintain the affected index structures as part of the deletion.

Therefore:

```text
DELETE
 ↓
Remove row
+
Maintain indexes
```

---

# 4️⃣9️⃣ Index Size

Suppose:

```text
Table = 500 GB
```

Indexes may add:

```text
100 GB
200 GB
500 GB
```

or more, depending on:

```text
Number of indexes
Column widths
Data types
Row count
Index structure
Included columns
```

Indexes are real storage.

They aren't free metadata.

---

# 5️⃣0️⃣ Index Cache

A frequently used index can remain in memory/cache.

If:

```text
Index
```

fits well in memory:

```text
Query
 ↓
Memory
 ↓
Fast
```

If it doesn't:

```text
Query
 ↓
Storage I/O
 ↓
Slower
```

Therefore index size affects more than storage cost.

It affects:

```text
Cache efficiency
I/O
Query latency
```

---

# 5️⃣1️⃣ Index Fragmentation / Maintenance

Index structures can accumulate internal inefficiencies depending on:

```text
Insert patterns
Updates
Deletes
Page splits
Storage engine
```

Maintenance requirements vary by database.

Examples may include:

```text
VACUUM
REINDEX
OPTIMIZE
ALTER INDEX
```

depending on the database.

Never blindly run maintenance commands in production.

---

# 5️⃣2️⃣ Page Splits

Imagine an index page is full:

```text
[10 20 30 40 50]
```

Now insert:

```text
25
```

The database may need to reorganize pages.

Conceptually:

```text
Before

[10 20 30 40 50]


After

[10 20 25]
[30 40 50]
```

This is a simplified illustration of a page split.

Frequent random inserts can cause additional write work.

---

# 5️⃣3️⃣ Sequential vs Random Primary Keys

Consider:

```text
1
2
3
4
5
6
```

versus random IDs:

```text
983472
17263
928374
48291
...
```

Sequential keys often have favorable locality for B-Tree insertion.

Random keys can cause more scattered insert locations and potentially more page splitting/cache churn.

But modern systems use many different ID strategies:

```text
UUID
UUIDv7
ULID
Snowflake-like IDs
Sequences
Auto-increment
```

The correct choice depends on:

```text
Ordering requirements
Distribution
Sharding
Storage engine
Index size
Generation architecture
```

---

# 5️⃣4️⃣ UUID and Index Size

Compare:

```text
BIGINT
```

with:

```text
UUID
```

A UUID generally consumes more bytes than a BIGINT.

If the primary key is also included in secondary indexes, this can increase index size.

At:

```text
100 million rows
```

the difference becomes significant.

Therefore:

> Identifier design is also index design.

---

# 5️⃣5️⃣ Query That Looks Fast but Isn't

Suppose:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'vinay@example.com';
```

and:

```text
email
```

has an index.

A developer may assume:

```text
"email is indexed, so this is fast."
```

Not necessarily.

Because the query asks for:

```text
LOWER(email)
```

not:

```text
email
```

Possible solutions:

```text
Normalize email during writes
```

or:

```text
Expression index
```

depending on the application and database.

---

# 5️⃣6️⃣ Normalize vs Expression Index

Option A:

```text
Store normalized email
```

Example:

```text
vinay@example.com
```

Then:

```sql
WHERE email = 'vinay@example.com'
```

Option B:

```text
Index LOWER(email)
```

Trade-offs include:

```text
Data model complexity
Application behavior
Index maintenance
Query consistency
Case sensitivity requirements
```

---

# 5️⃣7️⃣ Index Design From a Real Problem

Imagine:

```text
orders
```

has:

```text
500 million rows
```

API:

```text
GET /customers/101/orders?status=PAID
```

Query:

```sql
SELECT id, amount, created_at
FROM orders
WHERE customer_id = 101
  AND status = 'PAID'
ORDER BY created_at DESC
LIMIT 50;
```

A possible index:

```text
(customer_id, status, created_at)
```

Now think:

```text
customer_id
     ↓
status
     ↓
created_at
     ↓
50 newest rows
```

Potentially excellent.

But don't stop there.

Ask:

```text
How many customers?
How many PAID orders?
How often is this query executed?
How often do orders change status?
How large is the index?
Can it be covering?
What does EXPLAIN show?
```

---

# 5️⃣8️⃣ Covering Version

If query always needs:

```text
id
amount
created_at
```

you might consider an index that can cover those columns, depending on database support.

Conceptually:

```text
(customer_id, status, created_at, amount, id)
```

But this may be too large.

Therefore:

> Covering indexes are powerful, but they must be justified by workload.

---

# 5️⃣9️⃣ Indexing Is a Workload Decision

Suppose:

```text
Query A
runs 10 million times/day
```

and:

```text
Query B
runs once/day
```

If both benefit from an index:

```text
Query A
```

may justify substantial index cost.

This is why:

```text
Index design
```

must consider:

```text
Read frequency
Write frequency
Latency requirements
Storage
Concurrency
```

---

# 6️⃣0️⃣ Production Query Optimization Workflow

When a query is slow:

```text
Step 1
Observe actual query
```

```text
Step 2
Measure latency
```

```text
Step 3
Run EXPLAIN
```

```text
Step 4
Inspect estimated rows
```

```text
Step 5
Inspect actual rows
```

```text
Step 6
Check scans
```

```text
Step 7
Check joins
```

```text
Step 8
Check sorts
```

```text
Step 9
Check index usage
```

```text
Step 10
Check statistics
```

```text
Step 11
Design or modify index
```

```text
Step 12
Test again
```

---

# 👑 Don't Start With "Add an Index"

The correct process is:

```text
Measure
 ↓
Understand
 ↓
Explain
 ↓
Hypothesize
 ↓
Change
 ↓
Measure again
```

Not:

```text
Slow query
 ↓
Add index
 ↓
Hope
```

---

# 6️⃣1️⃣ EXPLAIN Mental Model

When looking at a plan, ask:

```text
What is the access path?

How many rows does the optimizer expect?

How many rows are actually processed?

Which indexes are used?

Which indexes are ignored?

Where does most time occur?

Is there a sort?

Is there a hash?

Is there a nested loop?

Are there repeated table lookups?

Is there a large scan?

Are estimates wrong?
```

---

# 6️⃣2️⃣ Query Plan Example

Imagine:

```text
Seq Scan on orders
Rows Removed by Filter: 499,999,000
Rows Returned: 1
```

This tells you:

```text
500M rows
 ↓
Almost entire table scanned
 ↓
Only 1 row returned
```

Potentially:

```text
Missing index
```

---

# 6️⃣3️⃣ Another Example

Suppose:

```text
Index Scan
Index Cond:
customer_id = 101

Rows:
50
```

This is very different.

The index narrowed:

```text
500M rows
```

to:

```text
50 candidates
```

That's exactly the kind of filtering an index can provide.

---

# 6️⃣4️⃣ But Index Scan Can Still Be Slow

Suppose:

```text
Index Scan
```

returns:

```text
20 million rows
```

The index exists.

Yet the query may still be slow.

Why?

Because:

```text
Index exists
≠
Query is automatically fast
```

Potential causes:

```text
Low selectivity
Large result set
Many table lookups
Poor clustering
Wrong index order
Bad statistics
High I/O
```

---

# 6️⃣5️⃣ Index vs Full Scan

Think:

```text
Small result
+
Large table
→ Index often attractive
```

But:

```text
Huge result
+
Large portion of table
→ Sequential scan may be better
```

The optimizer makes this choice based on estimated cost.

---

# 6️⃣6️⃣ Database Cache Changes Everything

Suppose table:

```text
500 GB
```

but frequently accessed index:

```text
2 GB
```

If the index stays in memory:

```text
Index lookup
→ memory
```

can be very fast.

Therefore:

```text
Index size
```

has operational importance beyond disk space.

---

# 6️⃣7️⃣ Indexing and Distributed Databases

Now imagine:

```text
10 database shards
```

Each shard has:

```text
Orders
+
Indexes
```

A query:

```text
customer_id = 101
```

is efficient if the system knows:

```text
Which shard?
```

Otherwise:

```text
Query
 ↓
Shard 1
Shard 2
Shard 3
...
Shard 10
```

This creates:

```text
Scatter-Gather
```

So:

> Local indexes cannot solve a bad distributed data-placement strategy.

---

# 6️⃣8️⃣ Shard Key + Index

In distributed SQL systems, you often need to think about:

```text
Partition key
+
Local indexes
+
Global indexes
```

Example:

```text
customer_id
```

might determine the shard.

Then:

```text
customer_id
```

also becomes an important query predicate.

This is where:

```text
Index design
+
Partitioning
+
Sharding
```

become one problem.

---

# 6️⃣9️⃣ Real Production Problem

Your API suddenly becomes slow.

Metrics:

```text
p50 = 20ms
p95 = 100ms
p99 = 5s
```

Database:

```text
CPU = 40%
```

Developer says:

> "Database CPU is low, so database isn't the problem."

Wrong.

You inspect:

```text
Lock waits
I/O
Buffer/cache misses
Query plans
Connection pool
```

and discover:

```text
Index scan
 ↓
Millions of random table lookups
```

CPU is only:

```text
40%
```

but storage latency is high.

The lesson:

> Database performance is not just CPU.

---

# 7️⃣0️⃣ Another Production Problem

Query:

```sql
SELECT *
FROM orders
WHERE status = 'PENDING';
```

There is an index:

```text
status
```

but:

```text
80% of rows = PENDING
```

The optimizer chooses:

```text
Sequential Scan
```

Developer says:

> "The optimizer isn't using my index. The optimizer is broken."

Not necessarily.

The sequential scan may genuinely be cheaper.

---

# 7️⃣1️⃣ Another Production Problem

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
ORDER BY created_at DESC
LIMIT 20;
```

Indexes:

```text
customer_id
created_at
```

Developer assumes:

```text
Two indexes
=
Very fast
```

But the database may need to:

```text
Find customer rows
 ↓
Sort by created_at
 ↓
Return 20
```

A composite index:

```text
(customer_id, created_at)
```

may better match the actual query pattern.

---

# 👑 This Is the Key Insight

Indexes should match:

```text
Access Pattern
```

not merely:

```text
Column Existence
```

---

# 🧪 Hands-on Lab

# Lab 1 — Table Scan

Create:

```text
1 million users
```

Query:

```sql
SELECT *
FROM users
WHERE email = 'user500000@example.com';
```

Run:

```text
EXPLAIN
```

Observe the plan.

---

# Lab 2 — Add Index

Create:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Run the same query.

Compare:

```text
Before
After
```

Observe:

```text
Scan type
Estimated rows
Execution time
```

---

# Lab 3 — Composite Index

Create:

```text
orders
```

with:

```text
customer_id
status
created_at
```

Run:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND status = 'PAID';
```

Create:

```text
(customer_id, status)
```

and inspect the plan.

---

# Lab 4 — Column Order

Create:

```text
(status, customer_id)
```

and:

```text
(customer_id, status)
```

Compare queries:

```sql
WHERE customer_id = 101;
```

and:

```sql
WHERE status = 'PAID';
```

Observe how column order affects plan choices.

---

# Lab 5 — ORDER BY

Run:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
ORDER BY created_at DESC
LIMIT 20;
```

Compare:

```text
Index(customer_id)
```

against:

```text
Index(customer_id, created_at)
```

---

# Lab 6 — Covering Index

Create:

```text
users
```

and:

```text
INDEX(email, name)
```

Run:

```sql
SELECT name
FROM users
WHERE email = 'user500000@example.com';
```

Inspect whether your database can use an index-only/covering strategy.

---

# Lab 7 — Function on Column

Create:

```text
INDEX(created_at)
```

Compare:

```sql
WHERE DATE(created_at) = '2026-08-28';
```

with:

```sql
WHERE created_at >= '2026-08-28'
AND created_at < '2026-08-29';
```

Compare execution plans.

---

# Lab 8 — LIKE

Compare:

```sql
WHERE email LIKE 'vinay%';
```

with:

```sql
WHERE email LIKE '%vinay%';
```

Inspect:

```text
Index usage
Execution time
Rows scanned
```

---

# Lab 9 — OFFSET Pagination

Create:

```text
10 million orders
```

Compare:

```sql
LIMIT 20 OFFSET 1000000;
```

with:

```sql
WHERE id > 1000000
ORDER BY id
LIMIT 20;
```

Measure the difference.

---

# Lab 10 — Statistics

Load highly skewed data.

Compare:

```text
Estimated rows
```

with:

```text
Actual rows
```

Use the database's statistics update mechanism and compare plans before and after statistics refresh.

---

# Lab 11 — Write Cost

Create:

```text
orders
```

with:

```text
0 indexes
```

Insert:

```text
100,000 rows
```

Measure.

Then create:

```text
5 indexes
```

Insert another:

```text
100,000 rows
```

Compare write performance.

---

# Lab 12 — Dead Index

Create an index that no production-style query uses.

Inspect database-specific index usage statistics.

Determine whether the index is actually providing value.

Do not drop indexes in production solely from a short observation window.

---

# Lab 13 — Query Plan Investigation

Take a deliberately slow query.

Perform:

```text
1. EXPLAIN
2. EXPLAIN ANALYZE
3. Compare estimated vs actual rows
4. Identify expensive operator
5. Design index
6. Test
7. Compare
```

Document:

```text
Before
After
```

---

# 🎯 Interview Questions

# Beginner

### What is an index?

An index is an auxiliary data structure that helps the database locate and process rows efficiently for supported access patterns.

---

### Why are indexes used?

Primarily to reduce the amount of data the database must examine for certain queries and to support efficient ordering or joining.

---

### What is a B-Tree?

A balanced tree-based data structure commonly used for ordered indexes.

---

### What is a table scan?

Reading many or all rows of a table and evaluating a predicate against them.

---

### What is a composite index?

An index containing multiple columns.

---

### What is a covering index?

An index containing enough information to answer a query without needing to fetch additional table data, when the database can use that strategy.

---

# 🧠 Intermediate Questions

### Why can too many indexes be bad?

Because indexes consume:

```text
Storage
Memory/cache
Insert cost
Update cost
Delete cost
Maintenance
```

---

### What is selectivity?

How strongly a predicate narrows the candidate row set.

---

### What is cardinality?

The number of distinct values in a column or the size of a distinct-value set, depending on context.

---

### Why does composite index column order matter?

Because the index is ordered by its leading columns, which strongly determines which predicates can efficiently navigate the index.

---

### Why might an optimizer ignore an index?

Because:

```text
Large result set
Low selectivity
High lookup cost
Poor statistics
Small table
Sequential scan cheaper
```

---

### Why can `LIKE '%abc%'` be slow with a B-Tree?

Because the leading wildcard prevents the database from using the beginning of the B-Tree key as a straightforward search boundary.

---

### Why can functions on indexed columns hurt performance?

Because the database may not be able to use a normal index on the raw column for the transformed expression.

---

# 👑 Senior Questions

### How would you design an index for:

```sql
WHERE customer_id = ?
AND status = ?
ORDER BY created_at DESC
LIMIT 50;
```

A strong candidate is often:

```text
(customer_id, status, created_at)
```

But verify using:

```text
EXPLAIN
```

and actual workload.

---

### Why might this index be bad?

```text
(status)
```

if:

```text
90% of rows = ACTIVE
```

Because the predicate may have low selectivity.

---

### Why is:

```text
(customer_id, status)
```

different from:

```text
(status, customer_id)
```

Because the leading column changes how the index can be navigated and which query predicates it naturally supports.

---

### What is write amplification?

The additional physical work generated by a logical write because the database must maintain indexes, logs, MVCC versions, replication state, and other structures.

---

### What is cardinality estimation?

The optimizer's prediction of how many rows an operation will produce.

Bad estimates can lead to poor query plans.

---

# 👑 20+ Year Experience Questions

## Question 1

A table has:

```text
500 million rows
```

and:

```sql
WHERE status = 'ACTIVE';
```

There is an index on:

```text
status
```

but the database chooses a sequential scan.

Is that necessarily wrong?

No.

If:

```text
80–90%
```

of rows are ACTIVE, scanning the table may be cheaper than performing millions of index-driven row accesses.

---

# Question 2

You have:

```text
Index(customer_id)
Index(status)
Index(created_at)
```

but this query is slow:

```sql
WHERE customer_id = 101
AND status = 'PAID'
ORDER BY created_at DESC
LIMIT 20;
```

What might be missing?

A composite index matching the access pattern:

```text
(customer_id, status, created_at)
```

may be appropriate.

But verify with the execution plan before changing production schema.

---

# Question 3

Why can a query using an index still be slow?

Because:

```text
Index
 ↓
Millions of matching entries
 ↓
Millions of table lookups
```

can still be expensive.

An index only changes the access path.

It doesn't guarantee a small result set.

---

# Question 4

Why can a covering index improve performance?

It can eliminate additional table/heap lookups when the index contains everything required and the database can perform an index-only strategy.

---

# Question 5

Why isn't:

```text
"Index every WHERE column"
```

a good strategy?

Because query predicates are not independent.

You must consider:

```text
Composite patterns
Selectivity
Ordering
Joins
Read frequency
Write frequency
Storage
Cache
```

---

# Question 6

A query is slow.

What do you do first?

Not:

```text
CREATE INDEX
```

Instead:

```text
Measure
 ↓
EXPLAIN
 ↓
Inspect actual plan
 ↓
Check cardinality estimates
 ↓
Identify bottleneck
 ↓
Design solution
 ↓
Measure again
```

---

# Question 7

Why can a wrong index order cause performance problems?

Suppose:

```text
Index(status, customer_id)
```

but most important query is:

```text
WHERE customer_id = ?
AND status = ?
```

The index may still have some utility, but it may not provide the same efficient navigation and ordering properties as:

```text
(customer_id, status)
```

especially for queries primarily constrained by `customer_id`.

The correct order depends on the complete workload.

---

# Question 8

Why can adding an index make the system slower?

Because:

```text
INSERT
UPDATE
DELETE
```

must maintain the additional structure.

If you add:

```text
20 indexes
```

to a heavily written table:

```text
Write throughput
 ↓
may decrease
```

while:

```text
Storage
 ↓
increases
```

---

# Question 9

Why is keyset pagination often better than large OFFSET pagination?

Because:

```text
OFFSET 10,000,000
```

may require processing or skipping a huge number of earlier rows.

Keyset:

```sql
WHERE id > last_seen_id
ORDER BY id
LIMIT 20;
```

can navigate from the known boundary when supported by an appropriate index.

---

# Question 10

A query suddenly became slow after a data distribution change.

Indexes haven't changed.

What could have changed?

Possibilities:

```text
Statistics
Data distribution
Selectivity
Cardinality estimates
Table size
Cache behavior
Query plan
```

The optimizer may now choose a different plan.

---

# 🔥 Real Production Incident

Imagine your order system:

```text
Orders:
1 billion rows
```

API:

```text
GET /customers/101/orders
```

Current query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
ORDER BY created_at DESC
LIMIT 20;
```

Existing indexes:

```text
INDEX(customer_id)
INDEX(created_at)
```

Latency:

```text
p95 = 2.5 seconds
```

You inspect:

```text
EXPLAIN ANALYZE
```

and discover:

```text
Index on customer_id
 ↓
Find 10 million orders
 ↓
Sort by created_at
 ↓
Return 20
```

The problem is not:

```text
"No index."
```

The problem is:

```text
Wrong access path for this query.
```

A candidate:

```text
(customer_id, created_at)
```

can allow:

```text
customer_id = 101
       ↓
created_at DESC
       ↓
first 20
```

Potential result:

```text
2.5 seconds
 ↓
tens of milliseconds
```

The exact improvement depends on data distribution, storage, database engine, and workload.

---

# 🧠 Production Index Architecture

Think of a mature system like this:

```text
                   APPLICATION
                        │
                        ▼
                      QUERY
                        │
                        ▼
                QUERY OPTIMIZER
                        │
             ┌──────────┼──────────┐
             │          │          │
             ▼          ▼          ▼
          Statistics   Indexes    Table
             │          │          │
             └──────────┼──────────┘
                        ▼
                    Query Plan
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
         Index Scan   Seq Scan   Join
             │
             ▼
          Row Fetch
             │
             ▼
           Result
```

The key is:

```text
SQL
 ↓
Optimizer
 ↓
Plan
 ↓
Storage access
```

not:

```text
SQL
 ↓
Index
```

automatically.

---

# 👑 Senior Index Mental Model

Whenever you see:

```sql
SELECT ...
FROM ...
WHERE ...
ORDER BY ...
LIMIT ...
```

ask:

```text
What rows are we looking for?

How selective is the predicate?

Which columns are equality predicates?

Which columns are range predicates?

What ordering is required?

Can one composite index support filtering + ordering?

Can the query be covered?

Are joins involved?

How many rows are expected?

How many rows are actually returned?

What does EXPLAIN say?

What does EXPLAIN ANALYZE say?

Is the optimizer estimating correctly?

What is the write cost of this index?

How large will this index become?

Will it fit in cache?

How frequently is the table modified?

Will this index help enough to justify its cost?
```

---

# 🔥 Production Index Checklist

```text
QUERY
-----
□ What exact query are we optimizing?
□ How frequently does it run?
□ What is p50/p95/p99 latency?
□ What is the result-set size?


FILTERING
---------
□ Equality predicates?
□ Range predicates?
□ Selectivity?
□ Cardinality?


COMPOSITE INDEX
---------------
□ Which column should lead?
□ Are equality predicates first?
□ Is there a range predicate?
□ Can ORDER BY be supported?


JOIN
----
□ Which columns are joined?
□ Are foreign-key columns indexed where useful?
□ What join strategy is chosen?


ORDERING
--------
□ Can the index provide the requested order?
□ Can LIMIT stop the scan early?


COVERING
--------
□ Can the index contain all required columns?
□ Is the larger index worth the cost?


OPTIMIZER
---------
□ What does EXPLAIN show?
□ What are estimated rows?
□ What are actual rows?
□ Are statistics accurate?


WRITE COST
----------
□ INSERT impact?
□ UPDATE impact?
□ DELETE impact?
□ Index maintenance?


STORAGE
-------
□ Index size?
□ Cache impact?
□ Page count?
□ Maintenance requirements?


PRODUCTION
----------
□ Can the index be created online/concurrently?
□ How long will creation take?
□ What locks are involved?
□ What happens to replication?
□ How will rollback/removal work?
```

---

# 📌 Chapter Summary

Indexes are additional data structures designed to make certain access patterns efficient.

The fundamental idea:

```text
Without index

Query
 ↓
Table Scan
 ↓
Many rows examined
```

With a suitable index:

```text
Query
 ↓
Index
 ↓
Narrow candidate set
 ↓
Fetch required data
```

Important index concepts:

```text
B-Tree
Selectivity
Cardinality
Composite Index
Covering Index
Unique Index
Partial Index
Expression Index
Clustered Index
Secondary Index
```

Important query concepts:

```text
Table Scan
Index Scan
Index-Only Scan
JOIN
ORDER BY
GROUP BY
LIMIT
Pagination
```

Important optimizer concepts:

```text
Statistics
Cardinality Estimation
Cost-Based Optimization
EXPLAIN
EXPLAIN ANALYZE
```

Important production concepts:

```text
Write Amplification
Index Size
Cache Efficiency
Page Splits
Index Maintenance
Replication
Sharding
```

---

# 🔥 Critical Rules

```text
Rule 1
------
An index is a data structure, not magic.


Rule 2
------
Index real query patterns, not every column.


Rule 3
------
Always consider selectivity.


Rule 4
------
Composite index column order matters.


Rule 5
------
The leading columns of a composite index strongly influence
how efficiently the index can be navigated.


Rule 6
------
More indexes improve some reads but increase write cost.


Rule 7
------
A query using an index can still be slow.


Rule 8
------
An optimizer ignoring an index is not necessarily a bug.


Rule 9
------
Always inspect EXPLAIN before changing indexes.


Rule 10
------
Use EXPLAIN ANALYZE carefully because it can execute the query.


Rule 11
------
Covering indexes can remove table lookups but increase index size.


Rule 12
------
Avoid unnecessarily applying functions to indexed columns
when a searchable predicate can express the same condition.


Rule 13
------
Large OFFSET pagination can become expensive.


Rule 14
------
Keyset pagination is often better for large ordered datasets.


Rule 15
------
Statistics are critical to query-plan quality.


Rule 16
------
Index design is a workload decision.


Rule 17
------
A good index matches filtering, ordering, joining,
and pagination requirements together.


Rule 18
------
Primary-key design affects secondary-index size in
storage engines that include the primary key there.


Rule 19
------
Local indexes cannot fix poor shard-key/data-placement design.


Rule 20
------
Measure before and after every performance change.
```

---

# 🎉 Module 1 — Database Fundamentals Continues

You now understand:

```text
SQL
 ↓
SELECT
 ↓
Filtering
 ↓
Aggregation
 ↓
JOINs
 ↓
Subqueries
 ↓
CTEs
 ↓
INSERT / UPDATE / DELETE
 ↓
Transactions
 ↓
Concurrency
 ↓
ACID
 ↓
Locks
 ↓
MVCC
 ↓
Isolation Levels
 ↓
Indexes
 ↓
B-Trees
 ↓
Composite Indexes
 ↓
Query Plans
```

But an index alone does not tell us **why the database chooses one execution strategy over another**.

For that, we need to understand the SQL optimizer and execution engine.

---

# 🚀 Next Chapter

# Chapter 16 — SQL Query Optimizer & Execution Plans: EXPLAIN, Cost-Based Optimization, Join Algorithms & Cardinality Estimation

We will answer:

```text
What happens after the database receives SQL?

How is SQL parsed?

What is a parser?

What is a logical query plan?

What is a physical query plan?

What is a cost-based optimizer?

How does the optimizer choose an index?

How does it choose between Index Scan and Sequential Scan?

How does it estimate row counts?

What are table statistics?

What are histograms?

Why do cardinality estimates become wrong?

What is a Nested Loop Join?

What is a Hash Join?

What is a Merge Join?

When is each join algorithm useful?

Why can a query suddenly become slow without code changes?

How do bad statistics cause bad plans?

Why can parameter values change the best execution plan?

What is plan caching?

What is parameter sniffing?

What is predicate pushdown?

What is projection pushdown?

How does LIMIT change query planning?

How does ORDER BY affect execution?

How does GROUP BY affect execution?

How does the database execute a query operator by operator?

How do you read a real EXPLAIN ANALYZE plan?

How do senior database engineers debug a 5-second query?

How do you fix a query that is slow only in production?

How do you distinguish:
CPU-bound
I/O-bound
Lock-bound
Network-bound
and plan-bound queries?
```

The central question will be:

> **When you send SQL to a database, how does the database transform that human-readable query into the exact sequence of operations that actually runs on the machine?**
