# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 13 — SQL INSERT, UPDATE, DELETE & UPSERT: Data Modification at Production Scale

> Until now, most of our SQL work has been about **reading data**.
>
> ```text
> SELECT
>   ↓
> Read
>   ↓
> Analyze
>   ↓
> Return
> ```
>
> But real software systems constantly change data.
>
> A customer registers:
>
> ```text
> INSERT
> ```
>
> A customer changes their address:
>
> ```text
> UPDATE
> ```
>
> An order is cancelled:
>
> ```text
> UPDATE
> ```
>
> An expired temporary record is removed:
>
> ```text
> DELETE
> ```
>
> A payment webhook arrives twice:
>
> ```text
> INSERT
>       ↓
> Duplicate?
>       ↓
> UPDATE existing record
> ```
>
> This creates a much harder problem.
>
> Reading data is usually:
>
> ```text
> "Tell me what exists."
> ```
>
> Writing data is:
>
> ```text
> "Change what exists."
> ```
>
> And changing production data is dangerous.
>
> A single missing `WHERE` can turn:
>
> ```sql
> UPDATE customers
> SET status = 'INACTIVE';
> ```
>
> into:
>
> ```text
> 50 million customers
>        ↓
> 50 million rows modified
> ```
>
> No syntax error.
>
> The database may happily execute it.
>
> That's why senior SQL engineering is not just about knowing:
>
> ```text
> INSERT
> UPDATE
> DELETE
> ```
>
> You must understand:
>
> ```text
> Transactions
> Constraints
> Locks
> Isolation
> Concurrency
> Atomicity
> Idempotency
> Upserts
> Bulk operations
> Deadlocks
> Write amplification
> WAL / redo logging
> MVCC
> Replication
> Failure recovery
> ```
>
> This chapter moves from:
>
> ```text
> "How do I modify a row?"
> ```
>
> to:
>
> ```text
> "How do I safely modify millions of rows
> in a concurrent production system?"
> ```

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* `INSERT`
* Multi-row INSERT
* INSERT from SELECT
* `UPDATE`
* Safe UPDATE patterns
* `DELETE`
* `TRUNCATE`
* Soft deletes
* Hard deletes
* UPSERT
* `ON CONFLICT`
* `ON DUPLICATE KEY`
* `MERGE`
* Primary key conflicts
* Unique constraints
* Transactions
* Atomic modifications
* Row locks
* MVCC
* Lost updates
* Optimistic concurrency
* Pessimistic locking
* Bulk modifications
* Batch processing
* Large UPDATE strategies
* Large DELETE strategies
* Deadlocks
* Write amplification
* WAL / redo logging
* Replication impact
* Production-safe data modifications

---

# 📖 Why Data Modification Is Hard

Suppose your application receives:

```text
POST /users
```

The backend executes:

```sql
INSERT INTO users (...);
```

Simple.

But what happens if:

```text
Client
   ↓
Request
   ↓
Database
   ↓
INSERT succeeds
   ↓
Network timeout
   ↓
Client retries
```

Now the application sends the same request again.

You may get:

```text
Duplicate user
```

or:

```text
Duplicate payment
```

or:

```text
Duplicate order
```

This is a distributed systems problem hiding inside a database write.

---

# 🧠 Database Writes Have More Than SQL Syntax

A production write involves:

```text
Application
    ↓
Connection Pool
    ↓
Transaction
    ↓
SQL
    ↓
Locks / MVCC
    ↓
Constraints
    ↓
Storage Engine
    ↓
Transaction Log
    ↓
Data Pages
    ↓
Indexes
    ↓
Replication
```

Understanding this pipeline is what separates basic SQL knowledge from senior database engineering.

---

# 1️⃣ INSERT

Basic:

```sql
INSERT INTO customers (
    name,
    email
)
VALUES (
    'Vinay',
    'vinay@example.com'
);
```

Conceptually:

```text
Application
    ↓
INSERT
    ↓
Validate constraints
    ↓
Write row
    ↓
Update indexes
    ↓
Commit
```

---

# 🧠 What Does INSERT Actually Do?

An INSERT is not simply:

```text
"Put a row into a table."
```

The database may need to:

```text
Validate data types
Validate NOT NULL constraints
Validate CHECK constraints
Validate UNIQUE constraints
Validate foreign keys
Generate IDs
Update indexes
Write transaction logs
Acquire locks
Make the change visible according to isolation rules
```

Therefore:

```text
INSERT cost
≠
just writing one row
```

---

# 2️⃣ INSERT and Primary Keys

Suppose:

```text
customers

id
101
102
103
```

Now:

```sql
INSERT INTO customers (
    id,
    name
)
VALUES (
    101,
    'Ravi'
);
```

If `id` is a primary key:

```text
101 already exists
```

The database rejects the operation.

This is good.

The constraint protects the data.

---

# 👑 Senior Principle

> Database constraints are part of your application's correctness model.

Do not rely only on application code to enforce uniqueness.

---

# 3️⃣ UNIQUE Constraint

Suppose email must be unique.

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    name VARCHAR(200)
);
```

Now:

```text
user 1
email = vinay@example.com
```

exists.

Another request:

```text
user 2
email = vinay@example.com
```

will fail.

The database guarantees:

```text
One unique email
```

---

# 🔥 Why Application-Only Checks Are Dangerous

Bad pattern:

```text
Application:

SELECT
    email
FROM users
WHERE email = 'vinay@example.com';

If not found:

INSERT user
```

Two requests arrive simultaneously:

```text
Request A                Request B

SELECT                   SELECT
   ↓                        ↓
Not found                Not found
   ↓                        ↓
INSERT                   INSERT
```

Both requests may believe the email is available.

Without a database uniqueness constraint:

```text
Duplicate data
```

can be created.

---

# 👑 Correct Design

Use:

```text
Application validation
        +
Database UNIQUE constraint
```

Application validation improves user experience.

Database constraints guarantee correctness.

---

# 4️⃣ Multi-Row INSERT

Instead of:

```sql
INSERT INTO products (...)
VALUES (...);

INSERT INTO products (...)
VALUES (...);

INSERT INTO products (...)
VALUES (...);
```

you can often use:

```sql
INSERT INTO products (
    name,
    price
)
VALUES
    ('Laptop', 50000),
    ('Phone', 20000),
    ('Monitor', 15000);
```

This can reduce:

```text
Network round trips
Parsing overhead
Transaction overhead
```

compared with issuing many independent statements.

Exact performance depends on the database and workload.

---

# 🔥 Real Production Problem

Suppose:

```text
100,000 rows
```

are being inserted.

Doing:

```text
100,000 network requests
```

is usually very different from:

```text
Batch
   ↓
Database
```

The application should consider:

```text
Batch size
Transaction size
Memory
Lock duration
Log volume
Error handling
```

---

# 5️⃣ INSERT ... SELECT

You can insert the result of another query.

Example:

```sql
INSERT INTO archived_orders (
    order_id,
    customer_id,
    total_amount
)
SELECT
    id,
    customer_id,
    total_amount
FROM orders
WHERE created_at < '2025-01-01';
```

Conceptually:

```text
orders
   ↓
Filter
   ↓
Rows
   ↓
INSERT
   ↓
archive table
```

---

# 🔥 Real Production Use Cases

`INSERT ... SELECT` is useful for:

```text
Archiving
Data migration
ETL
Backfills
Reporting tables
Data copies
Partition movement strategies
```

But large operations need careful planning.

---

# 6️⃣ UPDATE

Basic:

```sql
UPDATE customers
SET
    name = 'Vinay Kumar'
WHERE id = 101;
```

The critical part:

```sql
WHERE id = 101
```

---

# 💣 The Most Dangerous SQL Mistake

Imagine:

```sql
UPDATE customers
SET
    status = 'INACTIVE';
```

No `WHERE`.

The database may update:

```text
Every customer
```

If there are:

```text
50 million customers
```

then:

```text
50 million rows
```

may be modified.

---

# 👑 Production Rule

Before executing an important UPDATE:

```text
First write SELECT.
```

Instead of immediately:

```sql
UPDATE customers
SET status = 'INACTIVE'
WHERE last_login < '2024-01-01';
```

first test:

```sql
SELECT
    id,
    status,
    last_login
FROM customers
WHERE last_login < '2024-01-01';
```

Then check:

```text
How many rows?
Which rows?
Are they correct?
```

Only then perform the UPDATE.

---

# 7️⃣ UPDATE with Multiple Columns

```sql
UPDATE customers
SET
    name = 'Vinay Kumar',
    city = 'Hyderabad',
    updated_at = CURRENT_TIMESTAMP
WHERE id = 101;
```

One statement can modify multiple columns atomically within a transaction.

---

# 8️⃣ UPDATE Using Another Table

Suppose:

```text
customers
orders
```

You want to update customer status based on orders.

Database syntax varies, but the general idea is:

```text
customers
    ↓
match orders
    ↓
determine condition
    ↓
UPDATE customers
```

For example, in a database supporting UPDATE with FROM:

```sql
UPDATE customers c
SET
    status = 'VIP'
FROM customer_stats s
WHERE c.id = s.customer_id
  AND s.total_spent > 100000;
```

The exact syntax differs between database engines.

---

# 🧠 Senior Concern

Whenever UPDATE depends on another table:

```text
Ask:
```

```text
Can the JOIN match multiple rows?
```

If:

```text
customer
   ↓
10 matching rows
```

you must understand how your database handles that UPDATE.

Never assume the result is obvious.

---

# 9️⃣ DELETE

Basic:

```sql
DELETE FROM customers
WHERE id = 101;
```

Again:

```text
WHERE
```

is critical.

---

# 💣 Dangerous DELETE

```sql
DELETE FROM customers;
```

This can remove every row from the table.

The table itself may remain, but the data is gone.

---

# 👑 Safe DELETE Workflow

Use:

```text
SELECT
   ↓
Count
   ↓
Inspect
   ↓
Transaction
   ↓
DELETE
```

For example:

```sql
SELECT COUNT(*)
FROM customers
WHERE status = 'TEST';
```

Then:

```sql
DELETE FROM customers
WHERE status = 'TEST';
```

---

# 🔟 DELETE vs TRUNCATE

These are not the same operation.

## DELETE

```sql
DELETE FROM customers;
```

Conceptually:

```text
Delete rows
```

It is generally row-oriented and can support filtering:

```sql
DELETE FROM customers
WHERE status = 'TEST';
```

---

## TRUNCATE

```sql
TRUNCATE TABLE customers;
```

Conceptually:

```text
Remove table contents as a bulk operation
```

It is generally intended to remove all rows and has database-specific transactional, locking, identity-reset, and trigger behavior.

---

# 🧠 Important

Do not memorize:

```text
TRUNCATE = always faster
```

or:

```text
TRUNCATE = always cannot rollback
```

These details depend on the database engine.

Understand your specific database.

---

# 1️⃣1️⃣ DELETE vs TRUNCATE vs DROP

Think:

```text
DELETE
   ↓
Remove rows
```

```text
TRUNCATE
   ↓
Remove table contents
```

```text
DROP
   ↓
Remove database object
```

Conceptually:

```text
DELETE
→ data

TRUNCATE
→ table data

DROP
→ table/object itself
```

---

# 1️⃣2️⃣ Soft Delete

Many production systems don't physically delete records.

Instead:

```text
deleted_at
```

is added.

Example:

```sql
UPDATE customers
SET deleted_at = CURRENT_TIMESTAMP
WHERE id = 101;
```

Then normal queries use:

```sql
SELECT *
FROM customers
WHERE deleted_at IS NULL;
```

This is:

```text
Soft Delete
```

---

# 🧠 Why Soft Delete?

Useful when you need:

```text
Audit history
Recovery
Legal retention
Business history
Referential analysis
Undo functionality
```

But soft deletes create complexity.

Every query must correctly consider:

```text
deleted_at IS NULL
```

---

# 🔥 Soft Delete Problem

Developer writes:

```sql
SELECT *
FROM customers;
```

and accidentally includes deleted customers.

Or:

```sql
SELECT COUNT(*)
FROM customers;
```

returns:

```text
Active + deleted
```

instead of:

```text
Active only
```

Therefore soft deletion is not free.

---

# 👑 Senior Rule

> Soft delete is a data-modeling decision, not simply an UPDATE trick.

---

# 1️⃣3️⃣ UPSERT

Now we reach a very important production concept.

Suppose an external system sends:

```text
customer_id = 101
email = vinay@example.com
```

You want:

```text
If record doesn't exist
    ↓
INSERT

If record already exists
    ↓
UPDATE
```

This pattern is called:

```text
UPSERT
```

Meaning:

```text
UPDATE
+
INSERT
```

---

# 🔥 Real Production Problem

Payment webhook:

```text
Payment Provider
       ↓
Webhook
       ↓
Your API
```

The provider may retry the webhook.

First request:

```text
payment_id = PAY123
```

Second request:

```text
payment_id = PAY123
```

You don't want:

```text
2 payment records
```

You want:

```text
1 payment record
```

Therefore:

```text
Unique payment ID
        +
UPSERT / idempotent write
```

---

# 1️⃣4️⃣ PostgreSQL-Style UPSERT

In PostgreSQL:

```sql
INSERT INTO payments (
    payment_id,
    status,
    amount
)
VALUES (
    'PAY123',
    'SUCCESS',
    5000
)
ON CONFLICT (payment_id)
DO UPDATE SET
    status = EXCLUDED.status,
    amount = EXCLUDED.amount;
```

Conceptually:

```text
Try INSERT
    ↓
Conflict?
 ┌──┴──┐
No    Yes
 ↓      ↓
Insert Update
```

---

# 🧠 What Is EXCLUDED?

In PostgreSQL UPSERT:

```text
EXCLUDED
```

refers to the row that was proposed for insertion but conflicted.

So:

```sql
EXCLUDED.status
```

means:

```text
The incoming status value
```

---

# 1️⃣5️⃣ MySQL-Style UPSERT

MySQL has its own syntax, such as:

```sql
INSERT INTO payments (
    payment_id,
    status,
    amount
)
VALUES (
    'PAY123',
    'SUCCESS',
    5000
)
ON DUPLICATE KEY UPDATE
    status = VALUES(status),
    amount = VALUES(amount);
```

Modern MySQL versions also provide newer syntax patterns.

The exact syntax depends on the MySQL version.

---

# 🧠 Important

Do not memorize one UPSERT syntax as:

```text
"SQL standard."
```

Different database systems implement different forms.

Learn the concept first:

```text
Conflict
   ↓
Resolve
   ↓
Update or ignore
```

Then learn your database's syntax.

---

# 1️⃣6️⃣ UPSERT Requires a Conflict Definition

An UPSERT needs to know:

```text
What counts as duplicate?
```

For example:

```text
payment_id
```

or:

```text
email
```

or:

```text
external_system_id
```

This normally relies on:

```text
PRIMARY KEY
```

or:

```text
UNIQUE constraint
```

---

# 👑 Senior Principle

> Idempotency usually requires a stable uniqueness key.

Example:

```text
external_payment_id
```

is often more useful than:

```text
request timestamp
```

because the external identifier represents the same logical operation across retries.

---

# 1️⃣7️⃣ INSERT or UPDATE Race Condition

Bad pattern:

```text
SELECT
    WHERE payment_id = 'PAY123'
```

If not found:

```text
INSERT
```

But two requests can arrive simultaneously:

```text
Request A              Request B

SELECT                 SELECT
  ↓                      ↓
Not Found               Not Found
  ↓                      ↓
INSERT                  INSERT
```

Now you have a race.

---

# 🧠 Database Constraint Solves the Race

Create:

```sql
UNIQUE(payment_id)
```

Then:

```text
Request A
    ↓
INSERT
    ↓
Success

Request B
    ↓
INSERT
    ↓
Unique conflict
```

Then the application can:

```text
Ignore
Retry
Update
Return existing record
```

depending on the business rule.

---

# 1️⃣8️⃣ MERGE

Some databases support:

```text
MERGE
```

Conceptually:

```text
Source Data
     ↓
Match Target
     ↓
 ┌───┴────┐
Match   No Match
 ↓         ↓
UPDATE    INSERT
```

Example conceptual structure:

```sql
MERGE INTO customers c
USING incoming_customers i
ON c.external_id = i.external_id

WHEN MATCHED THEN
    UPDATE SET
        name = i.name

WHEN NOT MATCHED THEN
    INSERT (
        external_id,
        name
    )
    VALUES (
        i.external_id,
        i.name
    );
```

Exact syntax and behavior vary by database.

---

# 🔥 Real Use Case

Imagine:

```text
CRM System
     ↓
Nightly Customer Feed
     ↓
Database
```

Every night:

```text
Existing customers
New customers
Updated customers
```

A MERGE-like operation can synchronize the target table.

---

# 1️⃣9️⃣ Transactions

Now we reach the foundation of safe data modification.

Suppose bank transfer:

```text
Account A
₹10,000
```

Account B:

```text
₹5,000
```

Transfer:

```text
₹2,000
```

You need:

```text
A = ₹8,000
B = ₹7,000
```

What if:

```text
UPDATE A
   ↓
Success
   ↓
Database crashes
   ↓
UPDATE B
```

Now:

```text
A = ₹8,000
B = ₹5,000
```

₹2,000 disappeared.

That's unacceptable.

---

# 👑 Transaction

Wrap both operations:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 2000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 2000
WHERE id = 2;

COMMIT;
```

If something fails:

```text
ROLLBACK
```

Conceptually:

```text
BEGIN
  ↓
Operation A
  ↓
Operation B
  ↓
COMMIT
```

Either the transaction commits successfully, or it is rolled back according to the database's transaction semantics.

---

# 2️⃣0️⃣ ACID

Transactions are commonly described using:

```text
A — Atomicity
C — Consistency
I — Isolation
D — Durability
```

---

# Atomicity

Either:

```text
All operations
```

or:

```text
None
```

Conceptually:

```text
A + B + C
```

becomes:

```text
ALL
```

or:

```text
NONE
```

---

# Consistency

A committed transaction should preserve the database's defined integrity rules.

Examples:

```text
Primary keys
Foreign keys
CHECK constraints
Unique constraints
```

---

# Isolation

Concurrent transactions should not incorrectly interfere according to the database's isolation guarantees.

---

# Durability

Once a transaction commits, the database provides durability guarantees so committed data survives failures, subject to the database's recovery architecture.

---

# 2️⃣1️⃣ Atomic UPDATE

Consider:

```sql
UPDATE accounts
SET balance = balance - 2000
WHERE id = 1;
```

This is preferable to:

```text
SELECT balance
   ↓
Application calculates
   ↓
UPDATE balance = calculated value
```

Why?

Because:

```sql
SET balance = balance - 2000
```

expresses the modification relative to the current stored value.

Concurrency semantics still depend on transaction isolation and locking behavior, but this avoids a common read-modify-write race in application code.

---

# 💣 Lost Update Problem

Suppose:

```text
Balance = 10,000
```

Two requests:

```text
Request A
reads 10,000

Request B
reads 10,000
```

A calculates:

```text
10,000 - 2,000 = 8,000
```

B calculates:

```text
10,000 - 3,000 = 7,000
```

Then:

```text
A writes 8,000
B writes 7,000
```

Expected:

```text
5,000
```

Actual:

```text
7,000
```

A transaction or atomic database-side update strategy is needed to prevent this class of race.

---

# 2️⃣2️⃣ Optimistic Concurrency

A common pattern is:

```text
version
```

Example:

```text
id = 101
balance = 10000
version = 5
```

Application reads:

```text
version = 5
```

Then:

```sql
UPDATE accounts
SET
    balance = 8000,
    version = 6
WHERE id = 101
  AND version = 5;
```

If:

```text
Rows affected = 1
```

the update succeeded.

If:

```text
Rows affected = 0
```

someone else changed the record first.

This is:

```text
Optimistic Concurrency Control
```

---

# 🧠 Mental Model

```text
Read
 ↓
Remember version
 ↓
Modify
 ↓
UPDATE ... WHERE version = old_version
 ↓
 ┌─────────────┐
 │             │
Success       Conflict
 │             │
 ↓             ↓
Commit       Retry / Reject
```

---

# 2️⃣3️⃣ Pessimistic Locking

Another approach is to lock the row while working with it.

For databases that support it:

```sql
SELECT *
FROM accounts
WHERE id = 101
FOR UPDATE;
```

Conceptually:

```text
Transaction A
     ↓
Lock row
     ↓
Modify
     ↓
Commit
```

Another transaction attempting conflicting access may need to wait according to the database's locking rules.

---

# 🧠 Optimistic vs Pessimistic

```text
Optimistic
→ Assume conflicts are uncommon.

Pessimistic
→ Protect the row before modifying it.
```

---

# Compare

| Approach       | Idea                             |
| -------------- | -------------------------------- |
| Optimistic     | Detect conflict                  |
| Pessimistic    | Prevent conflicting access       |
| Version column | Detect stale update              |
| Row lock       | Serialize conflicting operations |

---

# 2️⃣4️⃣ Bulk UPDATE

Suppose:

```text
100 million customers
```

need migration:

```text
country_code
```

changes.

Naively:

```sql
UPDATE customers
SET country_code = 'IN';
```

could touch the entire table.

Potential consequences:

```text
Huge transaction
Long locks
Large transaction log
Replication lag
High I/O
Cache pressure
Long rollback
```

---

# 🔥 Production Approach

Instead of one enormous operation:

```text
100 million rows
```

consider batches:

```text
Batch 1
10,000 rows

Batch 2
10,000 rows

Batch 3
10,000 rows

...
```

Conceptually:

```text
100M
 ↓
10K
 ↓
Commit
 ↓
10K
 ↓
Commit
 ↓
...
```

Exact batch size depends on:

```text
Database
Hardware
Indexes
Replication
Locking
Transaction log capacity
Workload
```

---

# 2️⃣5️⃣ Batch UPDATE

A common strategy is to update by a stable key range.

For example:

```sql
UPDATE customers
SET migrated = TRUE
WHERE id > 100000
  AND id <= 110000;
```

Then:

```text
100001–110000
110001–120000
...
```

This makes progress measurable.

---

# 🧠 Why Stable Key Ranges?

Because:

```text
OFFSET
```

can become expensive or unstable for large datasets.

Prefer a stable progression such as:

```text
id > last_processed_id
```

when the schema and workload support it.

---

# 2️⃣6️⃣ Batch DELETE

Suppose:

```text
temporary_events
```

contains:

```text
500 million rows
```

and you need to delete old records.

A single:

```sql
DELETE FROM temporary_events
WHERE created_at < ...;
```

may create a huge transaction.

Instead:

```text
Find batch
   ↓
Delete batch
   ↓
Commit
   ↓
Repeat
```

Conceptually:

```text
500M
 ↓
10K
 ↓
Commit
 ↓
10K
 ↓
Commit
 ↓
...
```

---

# 🔥 Why Batch Deletes?

Potential benefits:

```text
Shorter transactions
Smaller rollback scope
Reduced lock duration
Controlled log generation
Less replication pressure
Better operational control
```

But batching can also increase total statement overhead.

The correct batch size must be measured.

---

# 2️⃣7️⃣ Write Amplification

A single UPDATE may affect more than one physical structure.

Suppose:

```text
customers
```

has indexes:

```text
email
status
city
created_at
```

If you modify an indexed column:

```text
status
```

the database may need to maintain:

```text
Table data
+
status index
```

Therefore:

```text
One logical UPDATE
```

can cause:

```text
Multiple physical writes
```

---

# 🧠 Index Trade-Off

Indexes improve:

```text
READ
```

but generally add cost to:

```text
INSERT
UPDATE
DELETE
```

because index structures must be maintained.

This is one of the most important database trade-offs.

---

# 2️⃣8️⃣ WAL / Redo Logging

Most production relational databases maintain some form of transaction log.

Examples include:

```text
PostgreSQL → WAL
Many other systems → redo/transaction logs
```

Conceptually:

```text
Application
   ↓
Transaction
   ↓
Log record
   ↓
Data changes
```

The log supports mechanisms such as:

```text
Crash recovery
Durability
Replication
```

depending on the database architecture.

---

# 🧠 Why Log First?

A database cannot safely assume:

```text
Disk write always succeeds
```

The transaction log provides a durable record of changes used for recovery.

This is a deep internal concept that becomes important when understanding:

```text
Replication
Recovery
Performance
Checkpointing
Crash consistency
```

---

# 2️⃣9️⃣ MVCC

Many modern databases use:

```text
MVCC
```

meaning:

```text
Multi-Version Concurrency Control
```

Instead of simply thinking:

```text
One row
```

think:

```text
Multiple row versions
```

Conceptually:

```text
Old Version
     ↓
New Version
```

Different transactions may observe different versions according to the isolation level.

---

# 🧠 Why MVCC?

It can allow:

```text
Readers
```

and:

```text
Writers
```

to interact with less blocking than a simplistic lock-everything model.

Exact behavior varies by database.

---

# 3️⃣0️⃣ DELETE Under MVCC

In an MVCC database, DELETE does not necessarily mean:

```text
Immediately erase physical bytes
```

Instead, conceptually:

```text
Row becomes invisible
        ↓
Old version eventually cleaned up
```

PostgreSQL, for example, uses vacuuming to reclaim space from obsolete row versions.

This matters because:

```text
DELETE 1 billion rows
```

doesn't necessarily mean:

```text
Disk immediately becomes empty.
```

---

# 🔥 Production Consequence

Large DELETE operations can cause:

```text
Transaction log growth
Bloat
Vacuum pressure
Replication lag
I/O pressure
Long-running transactions
```

Therefore large data deletion requires operational planning.

---

# 3️⃣1️⃣ Foreign Keys and DELETE

Suppose:

```text
customers
    ↓
orders
```

with:

```text
orders.customer_id
```

referencing:

```text
customers.id
```

What happens if you try:

```sql
DELETE FROM customers
WHERE id = 101;
```

The database may reject the DELETE if dependent orders exist, depending on the foreign-key action.

Possible behaviors include:

```text
RESTRICT
NO ACTION
CASCADE
SET NULL
SET DEFAULT
```

Exact behavior depends on the database and constraint definition.

---

# 🔥 CASCADE

If defined as:

```text
ON DELETE CASCADE
```

deleting:

```text
Customer 101
```

may also delete:

```text
Orders for Customer 101
```

This can be powerful.

It can also be extremely dangerous.

---

# 👑 Senior Rule

Before deleting a parent row, understand:

```text
What references it?
```

Use database metadata to inspect:

```text
Foreign keys
Triggers
Cascades
```

---

# 3️⃣2️⃣ UPSERT and Idempotency

Suppose payment provider sends:

```text
PAY123
```

three times:

```text
Request 1
Request 2
Request 3
```

The desired database state is:

```text
PAY123
```

one time.

This is:

```text
Idempotent processing
```

A common architecture is:

```text
External ID
      ↓
UNIQUE constraint
      ↓
UPSERT / conflict handling
      ↓
One logical record
```

---

# 🧠 Why This Is Bigger Than SQL

Retries are normal in distributed systems.

You can have:

```text
Timeout
Retry
Duplicate request
Network failure
Message redelivery
Consumer restart
```

Therefore database design must assume:

> The same logical operation may arrive more than once.

---

# 3️⃣3️⃣ Idempotency Key

For an API:

```text
POST /payments
```

the client may send:

```text
Idempotency-Key:
ABC123
```

Database:

```text
idempotency_key UNIQUE
```

Then:

```text
First request
    ↓
Create result

Retry
    ↓
Find ABC123
    ↓
Return existing result
```

This prevents duplicate side effects.

---

# 🔥 Real Software Architecture

```text
Client
   ↓
API
   ↓
Idempotency Key
   ↓
Database UNIQUE constraint
   ↓
INSERT / UPSERT
   ↓
Commit
```

This pattern appears in:

```text
Payments
Orders
Bookings
Shipment creation
Message processing
Webhook handling
```

---

# 3️⃣4️⃣ Deadlocks

Now consider two transactions.

Transaction A:

```text
Lock Customer
   ↓
Lock Order
```

Transaction B:

```text
Lock Order
   ↓
Lock Customer
```

Possible sequence:

```text
A locks Customer
B locks Order

A waits for Order
B waits for Customer
```

Now:

```text
A → waiting for B
B → waiting for A
```

This is a:

# Deadlock

---

# 🧠 Database Response

Many databases detect deadlocks and abort one transaction.

Then the application should often:

```text
Retry transaction
```

with appropriate limits and backoff.

---

# 👑 Preventing Deadlocks

One strategy:

Always acquire locks in the same order.

For example:

```text
Customer
   ↓
Order
```

Every code path follows:

```text
Customer first
Order second
```

instead of:

```text
Flow A:
Customer → Order

Flow B:
Order → Customer
```

Consistent lock ordering can reduce deadlock risk.

---

# 3️⃣5️⃣ Transaction Duration

Long transaction:

```text
BEGIN
   ↓
5 minutes of work
   ↓
COMMIT
```

can be problematic.

Potential consequences:

```text
Long-held locks
Old MVCC versions
Large log usage
Replication effects
Reduced concurrency
```

Short transactions are generally easier to operate.

But:

> Don't split a transaction if doing so breaks business atomicity.

Correctness comes first.

---

# 3️⃣6️⃣ Transaction Boundary

Suppose an order creation requires:

```text
Create order
Create order items
Create payment record
Update inventory
```

Question:

Should all be in one transaction?

Potentially:

```text
BEGIN
   ↓
Create order
   ↓
Create items
   ↓
Create payment record
   ↓
Update inventory
   ↓
COMMIT
```

If these operations must be atomic within the same database, a transaction may be appropriate.

But if they span:

```text
Multiple microservices
```

a single database transaction may not be possible.

Then you enter:

```text
Distributed Transactions
Saga
Outbox Pattern
Eventual Consistency
```

which we'll cover later.

---

# 3️⃣7️⃣ Bulk INSERT Performance

Suppose:

```text
10 million rows
```

need to be inserted.

Potential strategies:

```text
Single-row INSERT
Multi-row INSERT
Batch INSERT
COPY / bulk loader
Staging table
INSERT ... SELECT
```

The best choice depends on:

```text
Database
Data format
Indexes
Constraints
Transaction size
Replication
Availability requirements
```

---

# 🔥 Staging Table Pattern

For large imports:

```text
External File
    ↓
Staging Table
    ↓
Validation
    ↓
Transformation
    ↓
Production Table
```

Example:

```text
CSV
 ↓
staging_customers
 ↓
validate
 ↓
deduplicate
 ↓
INSERT/UPSERT
 ↓
customers
```

This gives you a controlled place to validate incoming data before changing production records.

---

# 3️⃣8️⃣ Data Migration

Suppose you need:

```text
Add new column
Backfill 1 billion rows
```

Bad approach:

```sql
UPDATE customers
SET new_column = some_expression;
```

on a busy production system without analysis.

Potential problems:

```text
Huge transaction
Long locks
Replication lag
Storage growth
CPU pressure
I/O pressure
```

---

# 👑 Production Migration Pattern

Think:

```text
Phase 1
Add new nullable column
        ↓
Phase 2
Deploy code that can read old + new
        ↓
Phase 3
Backfill gradually
        ↓
Phase 4
Start writing new column
        ↓
Phase 5
Validate
        ↓
Phase 6
Enforce constraints if needed
        ↓
Phase 7
Remove old column later
```

This is the foundation of:

```text
Zero-downtime database migrations
```

---

# 3️⃣9️⃣ Update-Then-Read Verification

For critical modifications:

```sql
UPDATE customers
SET status = 'ACTIVE'
WHERE id = 101;
```

check:

```text
Rows affected = 1
```

If application expected:

```text
1 row
```

but gets:

```text
0
```

or:

```text
5000
```

something is wrong.

---

# 👑 Senior Rule

> The number of affected rows is a useful correctness signal.

For critical writes, validate:

```text
Expected rows
vs
Actual rows
```

---

# 4️⃣0️⃣ RETURNING / Generated Values

Some databases support returning values from INSERT/UPDATE/DELETE.

For example, PostgreSQL:

```sql
INSERT INTO customers (
    name
)
VALUES (
    'Vinay'
)
RETURNING id;
```

This can return the generated:

```text
id
```

without requiring another query.

Conceptually:

```text
INSERT
  ↓
Generate ID
  ↓
Return ID
```

This can reduce application round trips.

Syntax varies by database.

---

# 4️⃣1️⃣ UPDATE and Index Cost

Suppose:

```text
customers
```

has:

```text
INDEX(status)
```

and you run:

```sql
UPDATE customers
SET status = 'ACTIVE'
WHERE id = 101;
```

The database may need to maintain the index because the indexed value changed.

Therefore:

```text
UPDATE
+
Many indexes
=
More write work
```

---

# 🧠 Database Design Trade-Off

More indexes:

```text
Better reads
```

but potentially:

```text
More write cost
More storage
More maintenance
```

This is why:

> Index every column

is bad advice.

---

# 4️⃣2️⃣ UPSERT and Concurrent Requests

Suppose:

```text
Request A
PAY123
```

and:

```text
Request B
PAY123
```

arrive simultaneously.

With:

```text
UNIQUE(payment_id)
```

and proper conflict handling:

```text
Database
   ↓
Enforce uniqueness
   ↓
One logical record
```

This is much safer than:

```text
SELECT
   ↓
if not found
   ↓
INSERT
```

without a database-level constraint.

---

# 4️⃣3️⃣ Write Path Mental Model

A useful production mental model:

```text
Application
     ↓
Connection Pool
     ↓
Transaction
     ↓
SQL Statement
     ↓
Constraint Validation
     ↓
Lock / MVCC
     ↓
Modify Data
     ↓
Modify Indexes
     ↓
Transaction Log
     ↓
Commit
     ↓
Replication / Recovery
```

Every stage can affect performance and reliability.

---

# 4️⃣4️⃣ Production Write Checklist

Before deploying an INSERT:

```text
What identifies this record uniquely?

Can the request be retried?

What happens on duplicate?

Do I need UPSERT?

Are foreign keys valid?

How many rows will be inserted?

Should this be batched?
```

Before UPDATE:

```text
What exact rows will change?

Did I test the WHERE with SELECT?

How many rows should change?

Are indexed columns changing?

Could concurrent updates occur?

Do I need optimistic/pessimistic concurrency?

Is the transaction too large?
```

Before DELETE:

```text
Do I really need physical deletion?

Would soft delete be better?

What foreign keys reference this row?

Are cascades configured?

How many rows will be deleted?

Will this create replication or storage pressure?

Should deletion be batched?
```

---

# 🧪 Hands-on Lab

# Lab 1 — Basic INSERT

Create:

```text
customers
```

and insert:

```sql
INSERT INTO customers (
    name,
    email
)
VALUES (
    'Vinay',
    'vinay@example.com'
);
```

Verify:

```sql
SELECT *
FROM customers;
```

---

# Lab 2 — UNIQUE Constraint

Create:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    name VARCHAR(200)
);
```

Insert the same email twice.

Observe the database constraint.

---

# Lab 3 — Multi-Row INSERT

Insert 1,000 records using batches.

Compare against issuing 1,000 individual statements.

Measure:

```text
Execution time
Network calls
Transaction size
```

---

# Lab 4 — Safe UPDATE

First:

```sql
SELECT *
FROM customers
WHERE status = 'TEST';
```

Then:

```sql
UPDATE customers
SET status = 'INACTIVE'
WHERE status = 'TEST';
```

Check:

```text
Rows before
Rows affected
Rows after
```

---

# Lab 5 — Dangerous UPDATE

In a disposable test database:

```sql
UPDATE customers
SET status = 'TEST';
```

Observe how many rows change.

Understand why missing:

```text
WHERE
```

is dangerous.

---

# Lab 6 — DELETE

First:

```sql
SELECT COUNT(*)
FROM customers
WHERE status = 'TEST';
```

Then:

```sql
DELETE FROM customers
WHERE status = 'TEST';
```

Verify the count afterward.

---

# Lab 7 — Transaction

Create two accounts.

Execute:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

Then deliberately cause an error before COMMIT and test:

```sql
ROLLBACK;
```

---

# Lab 8 — Lost Update

Create:

```text
balance = 10000
```

Open two database sessions.

Read and modify the same row concurrently.

Observe how a naive read-modify-write pattern can lose an update.

Then test:

```sql
UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;
```

and compare behavior.

---

# Lab 9 — Optimistic Locking

Create:

```text
version
```

column.

Perform:

```sql
UPDATE accounts
SET
    balance = 9000,
    version = version + 1
WHERE id = 1
  AND version = 5;
```

Check:

```text
Rows affected
```

---

# Lab 10 — UPSERT

Create a unique key:

```text
payment_id
```

Insert:

```text
PAY123
```

again.

Implement the database-specific UPSERT syntax.

Verify:

```text
Only one logical payment
```

exists.

---

# Lab 11 — Batch UPDATE

Create:

```text
1 million test rows
```

Update them in:

```text
10,000-row batches
```

Measure:

```text
Transaction duration
Log generation
Execution time
Database load
```

---

# Lab 12 — Batch DELETE

Create:

```text
1 million old records
```

Delete them in batches.

Compare:

```text
One huge DELETE
```

against:

```text
Many smaller DELETEs
```

Measure operational impact.

---

# Lab 13 — Foreign Key Behavior

Create:

```text
customers
orders
```

with a foreign key.

Test:

```text
RESTRICT
CASCADE
SET NULL
```

where supported.

Understand exactly what happens when the parent is deleted.

---

# Lab 14 — EXPLAIN UPDATE

Use your database's execution-plan tooling to investigate:

```sql
UPDATE customers
SET status = 'ACTIVE'
WHERE id = 101;
```

Look for:

```text
Index usage
Rows examined
Rows affected
Lock behavior
```

---

# Lab 15 — Deadlock

Using two database sessions:

```text
Transaction A:
Lock Row 1
Then Row 2

Transaction B:
Lock Row 2
Then Row 1
```

Observe the deadlock.

Then change both transactions to:

```text
Row 1
   ↓
Row 2
```

and observe the difference.

Only perform this in a controlled test environment.

---

# 🎯 Interview Questions

# Beginner

### What is INSERT?

It adds rows to a table.

---

### What is UPDATE?

It modifies existing rows.

---

### What is DELETE?

It removes rows from a table.

---

### What is UPSERT?

A write operation that inserts a row when it doesn't exist and updates or otherwise resolves the conflict when it does.

---

### What is a transaction?

A unit of database work governed by transactional guarantees such as atomicity and isolation.

---

### What is a primary key?

A constraint that uniquely identifies rows in a table.

---

### What is a UNIQUE constraint?

A constraint that prevents duplicate values according to the database's uniqueness semantics.

---

# 🧠 Intermediate Questions

### Why is a UNIQUE constraint important?

Because application-level checks alone can race under concurrent requests.

The database constraint provides authoritative uniqueness enforcement.

---

### Why can an UPDATE be dangerous?

Because an incorrect or missing WHERE clause can modify many more rows than intended.

---

### How do you safely perform a large UPDATE?

Common strategies include:

```text
Batching
Stable key ranges
Small transactions
Monitoring
Throttling
Validation
```

---

### Why can indexes slow INSERT?

Because indexes must also be maintained when rows are inserted.

---

### Why can indexes slow UPDATE?

If indexed values change, the database may need to update the corresponding index entries.

---

### What is soft delete?

Marking a row as deleted, for example using:

```text
deleted_at
```

instead of physically removing it.

---

### What is the difference between DELETE and TRUNCATE?

DELETE removes rows and can normally use a WHERE predicate.

TRUNCATE is a bulk table-content operation with database-specific transactional and locking semantics.

---

# 👑 Senior Questions

### Why isn't SELECT-then-INSERT safe for uniqueness?

Because two concurrent requests can both observe that the row doesn't exist before either inserts it.

A database uniqueness constraint closes this race.

---

### What is idempotency?

Processing the same logical request multiple times results in the same intended state rather than repeated side effects.

---

### How can a database enforce idempotency?

A common pattern is:

```text
Idempotency Key
       ↓
UNIQUE constraint
       ↓
UPSERT / conflict handling
```

---

### What is optimistic concurrency?

Detecting conflicting updates using a version, timestamp, or similar condition rather than holding a lock for the entire application workflow.

---

### What is pessimistic concurrency?

Using database locking to prevent conflicting operations from proceeding concurrently.

---

### What is a deadlock?

A situation where transactions wait on each other's resources in a cycle.

---

### How can deadlocks be reduced?

Use consistent lock ordering, keep transactions appropriately short, access resources predictably, and implement safe transaction retries where appropriate.

---

# 👑 20+ Year Experience Questions

## Question 1

A payment API receives the same webhook five times.

How do you prevent five payment records?

Think:

```text
External Payment ID
        ↓
UNIQUE constraint
        ↓
UPSERT / conflict handling
        ↓
Idempotent processing
```

---

# Question 2

A developer says:

> "I check if the email exists before inserting, so I don't need a UNIQUE constraint."

What's wrong?

Concurrency.

Two requests can execute:

```text
SELECT
```

at the same time and both see:

```text
Not Found
```

Then both attempt:

```text
INSERT
```

The database constraint is the authoritative protection.

---

# Question 3

You need to update:

```text
500 million rows
```

on a production table.

Would you immediately run:

```sql
UPDATE table
SET ...
```

No.

First investigate:

```text
Table size
Indexes
Transaction log
Replication
Locking
Maintenance windows
Batch size
Database load
Rollback requirements
```

Then design a controlled migration.

---

# Question 4

Why might one UPDATE create much more work than expected?

Because the database may need to maintain:

```text
Table data
+
Indexes
+
Transaction log
+
MVCC versions
+
Replication stream
```

The logical operation is one UPDATE statement.

The physical work can be much larger.

---

# Question 5

Why can a huge DELETE be dangerous?

Potential consequences include:

```text
Large transaction
Large log generation
Long locks
MVCC cleanup pressure
Replication lag
Storage/bloat issues
Slow rollback
```

---

# Question 6

How would you safely delete 500 million expired events?

Potential strategy:

```text
Find eligible rows
       ↓
Delete small batch
       ↓
Commit
       ↓
Monitor
       ↓
Repeat
```

Use a stable indexed predicate where possible.

---

# Question 7

Two transactions deadlock.

Should the application assume:

```text
Database is broken
```

No.

Deadlocks can be normal in concurrent systems.

A robust application should:

```text
Detect transaction failure
        ↓
Retry when safe
        ↓
Use backoff
        ↓
Limit retry attempts
```

while engineers investigate why the deadlock occurs.

---

# Question 8

Why shouldn't you hold a transaction open while calling an external API?

Imagine:

```text
BEGIN
  ↓
UPDATE database
  ↓
Call payment API
  ↓
Wait 5 seconds
  ↓
Response
  ↓
COMMIT
```

During the external call:

```text
Transaction remains open
```

Potential consequences:

```text
Locks held longer
MVCC versions retained
Connections occupied
Reduced concurrency
```

A better architecture often avoids holding database transactions across slow external network operations.

---

# 🔥 Real Production Scenario

Consider an order service:

```text
Client
   ↓
POST /orders
   ↓
Order Service
```

Business operation:

```text
1. Create order
2. Create order items
3. Reserve inventory
4. Charge payment
5. Send notification
```

A naive implementation might put everything into:

```text
ONE DATABASE TRANSACTION
```

and hold it while calling:

```text
Payment Service
```

This is dangerous.

Why?

Because:

```text
Database Transaction
        ↓
Network Call
        ↓
External Service
        ↓
Unknown Latency
```

could keep database resources locked for seconds.

Instead, modern architectures often separate:

```text
Local database transaction
```

from:

```text
Distributed workflow
```

using patterns such as:

```text
Outbox
Saga
Events
Idempotency
Compensation
```

These concepts become critical when SQL meets distributed systems.

---

# 🧠 Production Write Architecture

A mature write path often looks like:

```text
                 CLIENT
                   │
                   ▼
                  API
                   │
                   ▼
             Validation
                   │
                   ▼
            Idempotency Key
                   │
                   ▼
             DB Transaction
                   │
          ┌────────┼─────────┐
          ▼        ▼         ▼
       INSERT    UPDATE    UPSERT
          │        │         │
          └────────┼─────────┘
                   ▼
              Constraints
                   │
                   ▼
              MVCC / Locks
                   │
                   ▼
             Transaction Log
                   │
                   ▼
                COMMIT
                   │
          ┌────────┴─────────┐
          ▼                  ▼
     Application         Replication
```

---

# 👑 Senior SQL Mental Model

When you see:

```sql
UPDATE ...
```

don't only think:

```text
"Change this column."
```

Think:

```text
Which rows?

How many rows?

Which indexes change?

What locks are required?

What happens concurrently?

How large is the transaction?

How much log is generated?

Will replicas fall behind?

Can the operation be retried?

What happens if the connection dies?

What happens if the application crashes?

What happens if the statement partially progresses internally?

What is the rollback cost?

Is the operation idempotent?
```

That is the mindset required for production database engineering.

---

# 🔥 Production Data Modification Checklist

```text
INSERT
------
□ What uniquely identifies this entity?
□ Can requests be retried?
□ Is there a UNIQUE constraint?
□ Do I need UPSERT?
□ Are foreign keys valid?
□ How many rows?
□ Should I batch?


UPDATE
------
□ Did I test the WHERE with SELECT?
□ How many rows should change?
□ How many rows actually changed?
□ Could concurrent updates occur?
□ Do I need version checking?
□ Are indexed columns changing?
□ Is the transaction too large?
□ Will replicas be affected?


DELETE
------
□ Do I really need physical deletion?
□ Should I use soft delete?
□ What foreign keys reference this row?
□ Are cascades enabled?
□ How many rows will disappear?
□ Should deletion be batched?
□ What happens to replication?


TRANSACTIONS
------------
□ What must be atomic?
□ How long will the transaction remain open?
□ What locks are acquired?
□ Can deadlocks occur?
□ Is retry safe?
□ Does the transaction cross service boundaries?


PERFORMANCE
-----------
□ Row count
□ Index maintenance
□ Transaction log
□ I/O
□ Memory
□ Lock duration
□ Replication lag
□ Batch size
```

---

# 📌 Chapter Summary

Data modification is more than:

```text
INSERT
UPDATE
DELETE
```

Production database engineering requires understanding:

```text
Constraints
Transactions
Concurrency
Locks
MVCC
Idempotency
UPSERT
Bulk operations
Logging
Replication
```

Core operations:

```text
INSERT
UPDATE
DELETE
UPSERT
MERGE
```

Important safety concepts:

```text
WHERE validation
Affected-row validation
Transactions
Unique constraints
Foreign keys
Optimistic locking
Pessimistic locking
```

Important scalability concepts:

```text
Batching
Bulk loading
Stable key ranges
Transaction size
Write amplification
Index maintenance
Log volume
Replication impact
```

---

# 🔥 Critical Rules

```text
Rule 1
------
Never run a production UPDATE without understanding
exactly which rows will change.


Rule 2
------
Never run a production DELETE without validating
the target set first.


Rule 3
------
Use database constraints for correctness,
not only application validation.


Rule 4
------
Assume requests can be retried.


Rule 5
------
Design writes to be idempotent where retries are possible.


Rule 6
------
Use UNIQUE constraints to protect logical uniqueness.


Rule 7
------
Use UPSERT/conflict handling where the business operation
requires insert-or-update behavior.


Rule 8
------
Don't hold database transactions open across slow
external network calls unless there is a deliberate reason.


Rule 9
------
Large UPDATE and DELETE operations need operational planning.


Rule 10
------
More indexes improve some reads but increase write work.


Rule 11
------
Understand transaction log and replication impact.


Rule 12
------
Deadlocks are a concurrency problem, not simply a SQL syntax problem.


Rule 13
------
Use consistent lock ordering where possible.


Rule 14
------
Validate affected-row counts for critical writes.


Rule 15
------
Understand the database engine's exact transaction and
locking behavior instead of relying on generic SQL folklore.


Rule 16
------
Correctness comes before performance.


Rule 17
------
Production SQL must be designed for retries, failures,
concurrency, and large data volumes.
```

---

# 🎉 Module 1 — Database Fundamentals Continues

You now understand:

```text
SELECT
   ↓
Filtering
   ↓
Expressions
   ↓
Aggregation
   ↓
GROUP BY / HAVING
   ↓
JOINs
   ↓
Subqueries
   ↓
CTEs
   ↓
INSERT
   ↓
UPDATE
   ↓
DELETE
   ↓
UPSERT
   ↓
Transactions
   ↓
Concurrency
```

Now we are ready to understand the mechanism that makes all of these operations **safe under concurrent users**.

---

# 🚀 Next Chapter

# Chapter 14 — SQL Transactions & Concurrency: ACID, Locks, MVCC, Isolation Levels & Race Conditions

We will go much deeper into:

```text
Transactions
ACID
Atomicity
Consistency
Isolation
Durability

Autocommit
BEGIN / COMMIT / ROLLBACK
Savepoints

Locks
Row Locks
Table Locks
Shared Locks
Exclusive Locks

MVCC
Snapshots
Transaction IDs
Visibility

Isolation Levels
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE

Dirty Reads
Non-Repeatable Reads
Phantom Reads
Lost Updates
Write Skew

Optimistic Locking
Pessimistic Locking

Deadlocks
Lock Ordering
Deadlock Detection
Transaction Retries

Real Banking Problems
Inventory Race Conditions
Seat Booking Problems
Payment Processing
Concurrent Order Creation
Stock Decrement Problems
```

The central question will be:

> **What actually happens inside the database when 1,000 users try to modify the same data at exactly the same time?**

We'll move from:

```text
"BEGIN and COMMIT"
```

to:

```text
"How does a database guarantee correctness
when thousands of transactions execute concurrently?"
```
