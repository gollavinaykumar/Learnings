# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 14 — SQL Transactions & Concurrency: ACID, Locks, MVCC, Isolation Levels & Race Conditions

> Imagine an e-commerce application during a major sale.
>
> ```text
> Product:
> iPhone
>
> Available Stock:
> 1
> ```
>
> At exactly the same moment:
>
> ```text
> Customer A → Buy
> Customer B → Buy
> Customer C → Buy
> ```
>
> All three requests reach the backend.
>
> If the application simply does:
>
> ```text
> Read stock
>     ↓
> stock = 1
>     ↓
> Check stock > 0
>     ↓
> Reduce stock
> ```
>
> multiple requests may read:
>
> ```text
> stock = 1
> ```
>
> before anyone writes the new value.
>
> Now the system may accidentally sell:
>
> ```text
> 3 phones
> ```
>
> when only:
>
> ```text
> 1 phone
> ```
>
> exists.
>
> This is not primarily a SQL syntax problem.
>
> It is a:
>
> # Concurrency Problem
>
> Databases solve these problems using mechanisms such as:
>
> ```text
> Transactions
> Locks
> MVCC
> Isolation Levels
> Atomic Operations
> Constraints
> ```
>
> The goal is not simply:
>
> ```text
> "Make SQL work."
> ```
>
> The real goal is:
>
> ```text
> "Make concurrent SQL operations
> produce correct results."
> ```
>
> This is one of the most important concepts separating basic SQL developers from senior database engineers.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What is a transaction?
* ACID
* Atomicity
* Consistency
* Isolation
* Durability
* Autocommit
* `BEGIN`
* `COMMIT`
* `ROLLBACK`
* Savepoints
* Transactions and application code
* Concurrent transactions
* Race conditions
* Lost updates
* Dirty reads
* Non-repeatable reads
* Phantom reads
* Write skew
* Locks
* Shared locks
* Exclusive locks
* Row-level locking
* Table-level locking
* MVCC
* Snapshots
* Isolation levels
* Read Committed
* Repeatable Read
* Serializable
* Optimistic locking
* Pessimistic locking
* Deadlocks
* Deadlock detection
* Transaction retries
* Inventory race conditions
* Seat booking
* Payment concurrency
* Database connection pools
* Long-running transactions

---

# 📖 Why Transactions Exist

Consider a bank transfer.

Account A:

```text
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

The system needs:

```text
A = ₹8,000
B = ₹7,000
```

There are two database modifications:

```sql
UPDATE accounts
SET balance = balance - 2000
WHERE id = 1;
```

and:

```sql
UPDATE accounts
SET balance = balance + 2000
WHERE id = 2;
```

What happens if the first succeeds but the second fails?

```text
A = ₹8,000
B = ₹5,000
```

₹2,000 has effectively disappeared from the application's logical state.

Transactions solve this class of problem.

---

# 1️⃣ What Is a Transaction?

A transaction is a logical unit of database work.

Conceptually:

```text
BEGIN
   ↓
Operation 1
   ↓
Operation 2
   ↓
Operation 3
   ↓
COMMIT
```

If something goes wrong:

```text
ROLLBACK
```

Mental model:

```text
Transaction
     │
     ├── Operation A
     ├── Operation B
     ├── Operation C
     │
     ▼
   COMMIT
```

The database treats the transaction according to its transactional guarantees.

---

# 2️⃣ Basic Transaction

Example:

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

If the application encounters an error:

```sql
ROLLBACK;
```

---

# 🧠 Transaction Mental Model

Think:

```text
BEGIN
  ↓
"I am starting a unit of work."
```

Then:

```text
SQL
SQL
SQL
```

Then:

```text
COMMIT
```

means:

```text
"Make this transaction durable/committed
according to the database's guarantees."
```

And:

```text
ROLLBACK
```

means:

```text
"Abort the transaction's changes."
```

---

# 3️⃣ ACID

Transactions are commonly explained using:

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

These four concepts form the foundation of transactional database reasoning.

---

# 4️⃣ Atomicity

Atomicity means the transaction behaves as a unit.

For:

```text
A
B
C
```

you don't want:

```text
A succeeds
B succeeds
C fails

→ A and B permanently remain
```

when the business operation requires all three to succeed together.

Instead:

```text
A
B
C
 ↓
COMMIT
```

or:

```text
A
B
C
 ↓
FAIL
 ↓
ROLLBACK
```

---

# 🔥 Real Example — Order Creation

Suppose creating an order requires:

```text
1. Create order
2. Create order items
3. Create payment record
```

If step 3 fails:

```text
Order exists
Items exist
Payment missing
```

That may create inconsistent application state.

If all three belong to one database transaction:

```text
BEGIN
   ↓
Create order
   ↓
Create items
   ↓
Create payment
   ↓
COMMIT
```

then failure can trigger:

```text
ROLLBACK
```

---

# 5️⃣ Consistency

Consistency means transactions preserve the database's defined integrity rules.

For example:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
CHECK
NOT NULL
```

Suppose:

```text
accounts.balance >= 0
```

is enforced through a constraint.

A transaction that attempts:

```text
balance = -500
```

may fail.

The database prevents a committed state that violates that constraint.

---

# 🧠 Important Distinction

Database consistency does **not** mean:

```text
"Everything always looks logically perfect."
```

It means the database maintains the consistency rules you have actually defined.

Business correctness may require additional application-level logic.

---

# 6️⃣ Isolation

Now imagine:

```text
Transaction A
```

and:

```text
Transaction B
```

running simultaneously.

A might be:

```text
Updating account
```

while B is:

```text
Reading account
```

Question:

> What should B be allowed to see?

Possible answers depend on the database's isolation model.

This is:

# Isolation

---

# 7️⃣ Durability

Suppose:

```text
Transaction
   ↓
COMMIT
```

immediately followed by:

```text
Server crash
```

A durable database is designed so committed changes survive crash/recovery according to its durability guarantees.

This involves mechanisms such as:

```text
Transaction logs
WAL / redo logs
Recovery
Checkpoints
Storage
```

The exact implementation differs between database systems.

---

# 8️⃣ Autocommit

Many database clients operate in:

```text
AUTOCOMMIT
```

mode.

Conceptually:

```text
INSERT
   ↓
COMMIT
```

automatically.

Then:

```text
UPDATE
   ↓
COMMIT
```

automatically.

This means:

```text
One statement
≈
One transaction
```

for many common configurations.

---

# 🧠 Why Autocommit Matters

Suppose you execute:

```sql
UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;
```

under autocommit.

You may effectively have:

```text
Transaction 1
    ↓
UPDATE A
    ↓
COMMIT

Transaction 2
    ↓
UPDATE B
    ↓
COMMIT
```

That's not the same as:

```text
Transaction
    ↓
UPDATE A
    ↓
UPDATE B
    ↓
COMMIT
```

If both operations must succeed together, you need an appropriate explicit transaction.

---

# 9️⃣ Transaction Boundary

A critical senior-level question is:

> Where should the transaction begin and end?

Suppose:

```text
HTTP Request
   ↓
Validate
   ↓
Database Update
   ↓
Call Payment API
   ↓
Database Update
   ↓
Response
```

Bad design can accidentally create:

```text
BEGIN
  ↓
UPDATE
  ↓
Call external API
  ↓
Wait 5 seconds
  ↓
UPDATE
  ↓
COMMIT
```

Now the transaction may remain open during:

```text
Network latency
External service delay
Retries
Timeouts
```

This can hold database resources longer than necessary.

---

# 👑 Senior Rule

> Keep transactions as short as possible while preserving the required business atomicity.

Do not sacrifice correctness merely to shorten a transaction.

---

# 🔟 Savepoints

Sometimes you need partial rollback inside a larger transaction.

Example:

```sql
BEGIN;

INSERT INTO orders (...);

SAVEPOINT before_optional_step;

INSERT INTO optional_data (...);
```

If the optional operation fails:

```sql
ROLLBACK TO SAVEPOINT before_optional_step;
```

Then the transaction can continue.

Finally:

```sql
COMMIT;
```

Conceptually:

```text
BEGIN
  ↓
Operation A
  ↓
SAVEPOINT
  ↓
Operation B
  ↓
FAIL
  ↓
Rollback to SAVEPOINT
  ↓
Continue
  ↓
COMMIT
```

---

# 1️⃣1️⃣ Concurrency

Concurrency means multiple operations are active at the same time.

Imagine:

```text
Request A
```

and:

```text
Request B
```

both modifying:

```text
Product 101
```

at the same time.

The database must decide:

```text
Who sees what?
Who waits?
Who wins?
Can both proceed?
```

---

# 🔥 Race Condition

Suppose:

```text
Stock = 1
```

Request A:

```text
SELECT stock
```

gets:

```text
1
```

Request B:

```text
SELECT stock
```

also gets:

```text
1
```

Both think:

```text
stock > 0
```

Then:

```text
A → stock = 0
B → stock = 0
```

Two customers may receive confirmation.

This is a race condition.

---

# 1️⃣2️⃣ Lost Update

Suppose:

```text
balance = 10000
```

Transaction A:

```text
READ → 10000
```

Transaction B:

```text
READ → 10000
```

A calculates:

```text
10000 - 2000 = 8000
```

B calculates:

```text
10000 - 3000 = 7000
```

A writes:

```text
8000
```

B writes:

```text
7000
```

Expected:

```text
5000
```

Actual:

```text
7000
```

A's update has effectively been lost.

---

# 👑 Why This Happens

The application performed:

```text
READ
 ↓
Calculate
 ↓
WRITE
```

instead of using an appropriately synchronized database operation.

A common safer pattern is:

```sql
UPDATE accounts
SET balance = balance - 2000
WHERE id = 1;
```

combined with suitable transactional/concurrency rules.

---

# 1️⃣3️⃣ Atomic Database Operation

Compare:

```text
SELECT balance
```

followed by:

```text
UPDATE balance = calculated_value
```

with:

```sql
UPDATE accounts
SET balance = balance - 2000
WHERE id = 1;
```

The second keeps the arithmetic in the database statement.

This is often a much better concurrency primitive.

But you still need to consider:

```text
Constraints
Isolation
Business rules
Concurrent transactions
```

---

# 1️⃣4️⃣ Locks

Databases use locks and/or MVCC mechanisms to control concurrent access.

A lock can conceptually mean:

```text
"This transaction currently owns
a particular concurrency right over this resource."
```

Locks can apply to:

```text
Rows
Pages
Tables
Indexes
Other database resources
```

The exact lock implementation differs between database engines.

---

# 1️⃣5️⃣ Shared Lock

Conceptually:

```text
Shared Lock
```

means:

```text
Multiple readers may be allowed.
```

A shared lock is commonly associated with reading under certain locking operations.

---

# 1️⃣6️⃣ Exclusive Lock

Conceptually:

```text
Exclusive Lock
```

means:

```text
The transaction needs exclusive access
for a conflicting operation.
```

Updates and deletes commonly require exclusive locking behavior on affected data.

---

# 🧠 Simple Mental Model

```text
READ
 ↓
Shared-like access

WRITE
 ↓
Exclusive-like access
```

But do not reduce real database locking systems to this oversimplified model.

Modern databases have many lock modes and MVCC interactions.

---

# 1️⃣7️⃣ Row-Level Locking

Some databases can lock individual rows.

Example:

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
Lock Account 101
     ↓
Modify
```

Another transaction trying to acquire a conflicting lock on that row may have to wait.

---

# 🔥 Seat Booking Example

Imagine:

```text
Seat A10
status = AVAILABLE
```

Two users:

```text
User A
User B
```

both click:

```text
Book A10
```

A locking approach can be:

```sql
BEGIN;

SELECT *
FROM seats
WHERE seat_number = 'A10'
FOR UPDATE;
```

Then:

```text
Check status
   ↓
AVAILABLE?
   ↓
UPDATE
   ↓
COMMIT
```

Transaction B may wait for A's conflicting lock.

After A commits, B re-evaluates according to the database's concurrency semantics.

---

# 1️⃣8️⃣ Pessimistic Locking

The above pattern is commonly called:

```text
Pessimistic Locking
```

Mental model:

```text
"I expect contention.
I'll lock the resource before modifying it."
```

Useful when:

```text
Contention is high
Correctness requires serialization
The protected operation is short
```

---

# 1️⃣9️⃣ Optimistic Locking

Another strategy is:

```text
"I don't expect many conflicts.
I'll detect them when they happen."
```

Use:

```text
version
```

column.

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
    version = version + 1
WHERE id = 101
  AND version = 5;
```

If:

```text
Rows affected = 1
```

success.

If:

```text
Rows affected = 0
```

someone else changed the row.

---

# 🧠 Optimistic Locking Flow

```text
READ
 ↓
version = 5
 ↓
Application modifies data
 ↓
UPDATE ... WHERE version = 5
 ↓
 ┌─────────────┐
 │             │
Success      Conflict
 │             │
 ↓             ↓
Commit       Retry / Reject
```

---

# 2️⃣0️⃣ Optimistic vs Pessimistic

| Optimistic                 | Pessimistic                |
| -------------------------- | -------------------------- |
| Detect conflict            | Prevent conflict           |
| Version/check condition    | Lock resource              |
| No long-held lock required | Holds lock                 |
| Good for low contention    | Useful for high contention |
| Retry may be needed        | Waiting may occur          |

Neither is universally better.

---

# 2️⃣1️⃣ Isolation Levels

Now we reach one of the most important database concepts.

Isolation levels define how concurrent transactions are allowed to observe one another.

Common levels include:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Exact behavior varies by database engine.

---

# 2️⃣2️⃣ Dirty Read

Suppose:

```text
Transaction A
```

updates:

```text
balance = 5000
```

but has not committed.

Transaction B reads:

```text
5000
```

Then A rolls back.

Actual committed value:

```text
10000
```

B read data that was never committed.

This is:

# Dirty Read

---

# 2️⃣3️⃣ READ UNCOMMITTED

Conceptually permits the weakest isolation.

Potentially:

```text
Dirty Reads
```

may be possible.

This level is uncommon for correctness-sensitive application workflows.

---

# 2️⃣4️⃣ READ COMMITTED

A common default isolation level.

General idea:

> A transaction should only observe committed data according to the database's READ COMMITTED semantics.

Example:

```text
Transaction A
UPDATE
not committed
```

Transaction B:

```text
SELECT
```

should not read A's uncommitted change.

---

# 🧠 But Something Interesting Can Happen

Transaction B:

```text
SELECT balance
```

gets:

```text
10000
```

A commits:

```text
balance = 8000
```

B runs the same SELECT again:

```text
8000
```

The result changed.

This is:

# Non-Repeatable Read

---

# 2️⃣5️⃣ Non-Repeatable Read

Definition:

```text
Same transaction
+
Same row
+
Same query
=
Different committed result
```

under an isolation level that allows concurrent commits to become visible.

---

# 2️⃣6️⃣ REPEATABLE READ

A stronger isolation level.

Conceptually:

```text
Transaction B
   ↓
Creates/uses a stable view
   ↓
Repeated reads see a consistent snapshot
```

The exact semantics differ by database.

For example, PostgreSQL's implementation is MVCC-based and differs in important ways from some other database systems.

---

# 2️⃣7️⃣ Phantom Read

Suppose:

```text
SELECT *
FROM orders
WHERE amount > 10000;
```

returns:

```text
10 rows
```

Another transaction inserts:

```text
Order 11
amount = 20000
```

Then the first transaction repeats:

```sql
SELECT *
FROM orders
WHERE amount > 10000;
```

and gets:

```text
11 rows
```

The new row is a:

# Phantom

---

# 🧠 Why Phantom?

The first query didn't just read one row.

It read a:

```text
Set of rows
```

Another transaction changed the set.

---

# 2️⃣8️⃣ SERIALIZABLE

This is the strongest standard isolation level commonly exposed by relational databases.

The goal is to make concurrent execution behave as though transactions were executed in some serial order.

Conceptually:

```text
Transaction A
     ↓
Transaction B
```

rather than allowing certain unsafe interleavings.

But:

```text
SERIALIZABLE
```

does not mean:

```text
"Everything becomes single-threaded."
```

Modern databases can use sophisticated mechanisms to provide serializable behavior while retaining concurrency.

---

# ⚠️ SERIALIZABLE Can Cause Failures

Under serializable execution, the database may detect that transactions cannot safely coexist.

Then a transaction may fail with a serialization error.

The application may need to:

```text
Retry transaction
```

This is normal in some workloads.

---

# 2️⃣9️⃣ Isolation Level Comparison

Conceptually:

| Isolation        | Dirty Read | Non-Repeatable Read |                     Phantom |
| ---------------- | ---------: | ------------------: | --------------------------: |
| READ UNCOMMITTED |   Possible |            Possible |                    Possible |
| READ COMMITTED   |         No |            Possible |                    Possible |
| REPEATABLE READ  |         No | Generally prevented | Database-specific semantics |
| SERIALIZABLE     |         No |                  No |                   Prevented |

**Important:** exact guarantees and implementation details differ by database engine.

---

# 3️⃣0️⃣ MVCC

Many modern relational databases use:

# Multi-Version Concurrency Control

or:

```text
MVCC
```

Instead of thinking:

```text
One physical row
```

think:

```text
Multiple versions of row state
```

Conceptually:

```text
Row Version 1
     ↓
Row Version 2
     ↓
Row Version 3
```

Transactions determine which version is visible to them.

---

# 🧠 Why MVCC?

Suppose:

```text
Transaction A
```

is updating a row.

At the same time:

```text
Transaction B
```

wants to read it.

With MVCC, the database can often allow B to see an appropriate earlier committed version rather than simply blocking all reads.

This improves concurrency.

Exact behavior depends on the database.

---

# 3️⃣1️⃣ MVCC Mental Model

Imagine:

```text
Account

Version 1
balance = 10000

Version 2
balance = 8000
```

Transaction A may see:

```text
Version 2
```

while another transaction may still be operating against:

```text
Version 1
```

depending on when the snapshots were established and the database's isolation rules.

---

# 3️⃣2️⃣ MVCC Does Not Mean "No Locks"

This is a common misunderstanding.

MVCC:

```text
≠
No locks
```

Databases can use:

```text
MVCC
+
Locks
```

together.

For example:

```text
Readers
   ↓
MVCC visibility

Writers
   ↓
Locks / conflict control
```

The exact mechanism depends on the database engine.

---

# 3️⃣3️⃣ PostgreSQL Example

PostgreSQL heavily relies on MVCC.

Conceptually:

```text
UPDATE
```

creates a new row version rather than simply overwriting the old version in place.

The old version may remain until it is safe to reclaim.

Maintenance processes such as:

```text
VACUUM
```

help reclaim space from obsolete row versions.

---

# 3️⃣4️⃣ Why Long Transactions Are Dangerous

Suppose:

```text
Transaction A
BEGIN
```

and remains open for:

```text
2 hours
```

During this period:

```text
Other transactions
      ↓
Create newer row versions
```

But the old transaction may still need older versions.

Consequences can include:

```text
MVCC cleanup pressure
Table/index bloat
Long-held locks
Connection pool exhaustion
Replication effects
Vacuum delays
```

---

# 👑 Senior Rule

> An idle transaction can be more dangerous than a busy transaction.

Why?

Because:

```text
Busy transaction
→ doing work
```

while:

```text
Idle transaction
→ holding transactional state
without making progress
```

---

# 3️⃣5️⃣ Long Transaction Example

Bad:

```text
BEGIN
  ↓
UPDATE
  ↓
Application does other work
  ↓
User waits
  ↓
External API call
  ↓
More application processing
  ↓
COMMIT
```

Better:

```text
Prepare required data
  ↓
BEGIN
  ↓
Database work
  ↓
COMMIT
  ↓
External work
```

when business semantics allow that separation.

---

# 3️⃣6️⃣ Deadlock

Consider:

```text
Transaction A
```

locks:

```text
Row 1
```

Then needs:

```text
Row 2
```

Transaction B locks:

```text
Row 2
```

Then needs:

```text
Row 1
```

Now:

```text
A → waiting for B
B → waiting for A
```

This is:

# Deadlock

---

# 3️⃣7️⃣ Deadlock Detection

Database systems can detect cycles in lock dependencies.

Conceptually:

```text
A
↓
waiting for B

B
↓
waiting for A
```

The database aborts one transaction.

Then:

```text
Transaction A → rollback
Transaction B → continue
```

depending on which transaction is selected as the victim.

---

# 3️⃣8️⃣ Application Retry

A robust application may do:

```text
Transaction
   ↓
Deadlock / Serialization Failure
   ↓
ROLLBACK
   ↓
Wait
   ↓
Retry
```

Example:

```text
Attempt 1 → failure
Attempt 2 → failure
Attempt 3 → success
```

But retries must be:

```text
Bounded
Safe
Idempotent
```

---

# 👑 Never Retry Blindly

Suppose:

```text
Payment charge
```

fails at the database layer.

You cannot blindly retry the entire workflow if the external payment operation may already have succeeded.

This is why:

```text
Database transactions
```

and:

```text
Distributed system idempotency
```

must be designed together.

---

# 3️⃣9️⃣ Deadlock Prevention

One important strategy:

```text
Consistent lock ordering
```

For example:

```text
Always lock Customer first
Then Order
```

Never:

```text
Flow A:
Customer → Order

Flow B:
Order → Customer
```

Instead:

```text
Flow A:
Customer → Order

Flow B:
Customer → Order
```

This reduces the possibility of cyclic waiting.

---

# 4️⃣0️⃣ Inventory Race Condition

Suppose:

```text
Product
stock = 1
```

Naive application:

```text
SELECT stock
```

then:

```text
if stock > 0:
    UPDATE stock = stock - 1
```

Two users:

```text
A → reads 1
B → reads 1
```

Both pass:

```text
stock > 0
```

---

# 🔥 Better Pattern

Use an atomic conditional update:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 101
  AND stock > 0;
```

Then inspect:

```text
Rows affected
```

If:

```text
1
```

the reservation succeeded.

If:

```text
0
```

the stock was unavailable or the row didn't match.

This is a powerful pattern.

---

# 🧠 Why This Works

Instead of:

```text
READ
 ↓
Application decision
 ↓
WRITE
```

we create:

```text
Database-side condition
        +
Database-side modification
```

The database can serialize the conflicting update according to its concurrency mechanisms.

---

# 4️⃣1️⃣ Seat Booking

Seat:

```text
A10
```

Status:

```text
AVAILABLE
```

Possible atomic approach:

```sql
UPDATE seats
SET status = 'BOOKED'
WHERE seat_number = 'A10'
  AND status = 'AVAILABLE';
```

Then:

```text
Rows affected = 1
```

means:

```text
Booking won
```

while:

```text
Rows affected = 0
```

means:

```text
Someone else already changed it
```

This can be simpler than:

```text
SELECT
 ↓
check
 ↓
UPDATE
```

---

# 4️⃣2️⃣ Unique Constraint as Concurrency Control

Suppose:

```text
username = vinay
```

must be unique.

Two requests:

```text
A → vinay
B → vinay
```

Both arrive simultaneously.

With:

```sql
UNIQUE(username)
```

the database guarantees that both cannot commit duplicate usernames.

One succeeds.

The other gets a uniqueness conflict.

---

# 👑 Senior Principle

> Constraints are concurrency-control tools as well as data-validation tools.

---

# 4️⃣3️⃣ Write Skew

This is a more advanced concurrency problem.

Suppose two doctors are on call:

```text
Doctor A → ON CALL
Doctor B → ON CALL
```

Business rule:

```text
At least one doctor must remain on call.
```

Transaction A:

```text
Reads:
Doctor B is ON CALL
```

Then:

```text
Doctor A → OFF CALL
```

Transaction B simultaneously reads:

```text
Doctor A is ON CALL
```

Then:

```text
Doctor B → OFF CALL
```

Both transactions individually appear valid.

But after both commit:

```text
Doctor A → OFF
Doctor B → OFF
```

Business invariant is broken.

This is:

# Write Skew

---

# 🧠 Why Is This Hard?

Neither transaction necessarily modified the same row.

Instead:

```text
Transaction A
checks B
updates A

Transaction B
checks A
updates B
```

The conflict is logical rather than a simple same-row update.

This is where:

```text
Isolation level
Constraints
Locking strategy
Serializable transactions
```

become extremely important.

---

# 4️⃣4️⃣ READ COMMITTED Isn't Magic

A common mistake:

> "We use READ COMMITTED, so concurrency is safe."

No.

READ COMMITTED prevents certain anomalies such as dirty reads, but it does not automatically prevent:

```text
Lost updates
Write skew
All race conditions
All business invariant violations
```

Your transaction logic still matters.

---

# 4️⃣5️⃣ SERIALIZABLE for Business Invariants

If the business rule depends on a set of rows:

```text
At least one doctor remains on call
```

or:

```text
No overlapping booking
```

or:

```text
Account balance constraints
```

a stronger isolation strategy may be appropriate.

But:

```text
SERIALIZABLE
```

can reduce concurrency and cause serialization failures.

Therefore:

```text
Correctness
+
Performance
```

must be balanced.

---

# 4️⃣6️⃣ Connection Pools and Transactions

Application servers commonly use:

```text
Connection Pool
```

Example:

```text
Application
    ↓
Connection Pool
    ↓
Database
```

Suppose pool size:

```text
100 connections
```

and 100 requests begin long transactions.

Now:

```text
100 connections
=
busy
```

Request 101 may wait for a connection.

---

# 🔥 Long Transactions Can Exhaust Pools

Example:

```text
Request
 ↓
BEGIN
 ↓
External API call
 ↓
5 seconds
 ↓
COMMIT
```

If many requests do this simultaneously:

```text
Connections
   ↓
Held longer
   ↓
Pool exhausted
   ↓
Requests queue
   ↓
Latency increases
```

This can cause cascading failures.

---

# 4️⃣7️⃣ Transaction Timeout

Production systems often need transaction timeouts.

Conceptually:

```text
Transaction starts
      ↓
Maximum allowed duration
      ↓
Exceeded
      ↓
Abort
```

This prevents one transaction from remaining open indefinitely.

Exact implementation depends on the application framework and database.

---

# 4️⃣8️⃣ Transaction Monitoring

Senior database engineers monitor:

```text
Transaction duration
Lock wait time
Deadlocks
Active sessions
Idle transactions
Blocked queries
Commit rate
Rollback rate
```

For PostgreSQL, useful operational information can be obtained from system views such as:

```text
pg_stat_activity
```

and lock-related catalog views.

Other databases provide equivalent monitoring facilities.

---

# 4️⃣9️⃣ Blocking

Suppose:

```text
Transaction A
```

holds a lock.

Transaction B:

```text
needs same resource
```

B may become:

```text
WAITING
```

Conceptually:

```text
A
 ↓
LOCK

B
 ↓
WAIT
```

This is:

# Lock Blocking

Blocking is not necessarily a bug.

Some blocking is normal.

The problem is:

```text
Unexpected
Long
Frequent
Cascading
```

blocking.

---

# 5️⃣0️⃣ Lock Wait Chain

Imagine:

```text
Transaction A
     ↓
holds Row 1
```

Transaction B:

```text
waits for Row 1
```

Transaction C:

```text
waits for B
```

Now:

```text
A
↓
B
↓
C
```

A slow transaction can therefore create a queue.

This is why a single slow transaction can affect many requests.

---

# 🔥 Production Incident

Imagine:

```text
10:00 AM
```

A report starts:

```text
BEGIN
```

Then performs a long operation.

Meanwhile:

```text
API requests
   ↓
Need same table/rows
   ↓
Wait
```

Suddenly:

```text
API latency ↑
Database connections ↑
CPU ↑
Requests timeout
```

The original problem might be:

```text
One long transaction
```

not the API code itself.

---

# 5️⃣1️⃣ Transaction Isolation vs Locking

These are related but not identical.

```text
Isolation Level
```

defines:

```text
What concurrent effects are allowed?
```

while:

```text
Locks
```

are one mechanism databases can use to coordinate conflicting operations.

And:

```text
MVCC
```

is another major mechanism for managing visibility and concurrency.

Think:

```text
Isolation
   ↓
Concurrency semantics

Locks + MVCC + other mechanisms
   ↓
Implementation
```

---

# 5️⃣2️⃣ Transaction State

Conceptually:

```text
START
  ↓
ACTIVE
  ↓
 ┌─────────────┐
 │             │
COMMIT       ROLLBACK
 │             │
 ↓             ↓
COMMITTED     ABORTED
```

An application should correctly handle all states.

Especially:

```text
Connection failure
Timeout
Deadlock
Serialization failure
Constraint violation
```

---

# 5️⃣3️⃣ Database Error Handling

Suppose:

```sql
UPDATE ...
```

fails because of:

```text
UNIQUE violation
```

The application should not blindly retry.

Classify errors:

```text
Permanent business/data error
        vs
Transient concurrency error
```

Examples:

```text
UNIQUE violation
→ often requires business handling

Deadlock
→ often retryable

Serialization failure
→ often retryable

Connection failure
→ depends on transaction outcome
```

---

# 👑 Senior Principle

> Retry only errors that are actually safe to retry.

---

# 5️⃣4️⃣ Unknown Transaction Outcome

This is a very advanced production problem.

Suppose:

```text
Application
   ↓
COMMIT request
   ↓
Database commits
   ↓
Network connection breaks
   ↓
Application receives timeout
```

The application sees:

```text
"Commit failed?"
```

But the database may actually have:

```text
COMMITTED
```

The client cannot always know from the timeout alone.

This is called an:

```text
Unknown outcome
```

problem.

---

# 🔥 Why Idempotency Matters

Suppose:

```text
Create payment
```

returns timeout.

Application retries.

Without idempotency:

```text
Payment A
Payment B
```

may be created.

With:

```text
idempotency_key UNIQUE
```

the retry can safely map back to the original logical operation.

This connects:

```text
Transactions
+
Distributed systems
+
Idempotency
```

---

# 5️⃣5️⃣ Transaction + Message Queue

Suppose:

```text
Order created
```

and you want:

```text
Kafka message
"ORDER_CREATED"
```

Naive:

```text
BEGIN
 ↓
INSERT order
 ↓
COMMIT
 ↓
Publish Kafka message
```

What if:

```text
COMMIT succeeds
```

but:

```text
Kafka publish fails
```

Now:

```text
Database says:
Order exists

Kafka says:
No event
```

This is a distributed consistency problem.

---

# 👑 Outbox Pattern

A common solution:

```text
BEGIN
   ↓
INSERT order
   ↓
INSERT outbox_event
   ↓
COMMIT
```

Then:

```text
Outbox
   ↓
Background Publisher
   ↓
Kafka
```

Now the database transaction atomically records:

```text
Business data
+
Event to publish
```

This is an important bridge between SQL transactions and distributed systems.

We'll study it deeply later.

---

# 5️⃣6️⃣ Transaction Design Pattern

For a local database workflow:

```text
Validate outside transaction
        ↓
BEGIN
        ↓
Read required rows
        ↓
Validate business conditions
        ↓
Modify data
        ↓
Write outbox event if needed
        ↓
COMMIT
        ↓
Perform asynchronous external work
```

The exact design depends on the application.

---

# 5️⃣7️⃣ Banking Example

Requirement:

```text
Transfer ₹5,000
```

Possible transaction:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 5000
WHERE id = 1
  AND balance >= 5000;

-- Check affected rows

UPDATE accounts
SET balance = balance + 5000
WHERE id = 2;

COMMIT;
```

Important:

```text
First update
+
Affected row check
+
Second update
+
Commit
```

If the first update affects:

```text
0 rows
```

the transfer should not proceed.

---

# 5️⃣8️⃣ Inventory Example

Stock:

```text
10
```

100 customers attempt purchase.

Atomic update:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 101
  AND quantity > 0;
```

Then:

```text
Rows affected = 1
```

means:

```text
Reservation succeeded
```

Repeat until:

```text
quantity = 0
```

Further requests:

```text
Rows affected = 0
```

This avoids a large class of application-side race conditions.

---

# 5️⃣9️⃣ Transaction Isolation Is a Business Decision

Do not choose isolation level because:

```text
"Senior engineers use SERIALIZABLE."
```

Instead ask:

```text
What anomalies can this business operation tolerate?

What invariants must always hold?

What is contention?

What is the workload?

What is the database engine?

What is the performance requirement?
```

Then choose the appropriate strategy.

---

# 👑 Senior Concurrency Framework

For every concurrent operation, ask:

```text
1. What data is shared?

2. What invariant must remain true?

3. Can two requests modify the same row?

4. Can they modify different rows that participate
   in the same business rule?

5. Can I solve it with an atomic UPDATE?

6. Do I need a UNIQUE constraint?

7. Do I need optimistic locking?

8. Do I need pessimistic locking?

9. What isolation level is appropriate?

10. Can deadlocks happen?

11. Can the operation be retried?

12. Is the operation idempotent?

13. How long will the transaction remain open?

14. What happens if the connection fails?

15. What happens if COMMIT outcome is unknown?
```

---

# 🧪 Hands-on Lab

# Lab 1 — Basic Transaction

Create two accounts.

Run:

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

Verify the balances.

---

# Lab 2 — Rollback

Run:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

ROLLBACK;
```

Verify that the change did not commit.

---

# Lab 3 — Autocommit

Execute an UPDATE without an explicit transaction.

Observe:

```text
Statement
   ↓
Commit
```

according to your client/database configuration.

---

# Lab 4 — Lost Update

Open two database sessions.

Start with:

```text
balance = 10000
```

Perform:

```text
Session A:
SELECT balance

Session B:
SELECT balance
```

Modify the value independently.

Observe the lost-update problem.

---

# Lab 5 — Atomic Update

Run:

```sql
UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;
```

from multiple sessions.

Observe how the database coordinates concurrent modifications.

---

# Lab 6 — Row Lock

Session A:

```sql
BEGIN;

SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

Keep the transaction open.

Session B:

attempt a conflicting update.

Observe:

```text
WAIT
```

Then:

```text
COMMIT
```

in Session A.

Observe Session B continue according to your database's behavior.

---

# Lab 7 — Optimistic Locking

Add:

```text
version
```

column.

Start with:

```text
version = 1
```

Run:

```sql
UPDATE accounts
SET
    balance = 9000,
    version = version + 1
WHERE id = 1
  AND version = 1;
```

Run a second update using:

```text
version = 1
```

Observe:

```text
Rows affected = 0
```

after the version has changed.

---

# Lab 8 — Dirty Read

If your database supports multiple isolation levels, experiment with:

```text
READ UNCOMMITTED
READ COMMITTED
```

Use two sessions.

Modify data in Session A without committing.

Attempt to read it from Session B.

Observe the database-specific behavior.

---

# Lab 9 — Non-Repeatable Read

Use:

```text
READ COMMITTED
```

Session A:

```text
BEGIN
SELECT row
```

Session B:

```text
UPDATE row
COMMIT
```

Session A:

```text
SELECT same row
```

Observe whether the value changed.

---

# Lab 10 — Repeatable Read

Repeat Lab 9 using:

```text
REPEATABLE READ
```

Compare the result.

---

# Lab 11 — Deadlock

Session A:

```text
BEGIN
Lock Row 1
Wait
Lock Row 2
```

Session B:

```text
BEGIN
Lock Row 2
Wait
Lock Row 1
```

Observe the database detecting the deadlock.

---

# Lab 12 — Inventory Race

Create:

```text
product_id = 101
quantity = 1
```

Attempt concurrent purchases using:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 101
  AND quantity > 0;
```

Check affected rows.

Only one transaction should successfully decrement the single available unit under appropriate transactional semantics.

---

# Lab 13 — Write Skew

Create:

```text
doctors
```

with:

```text
doctor_id
on_call
```

Start with:

```text
Doctor A = TRUE
Doctor B = TRUE
```

Run concurrent transactions where each doctor checks that another doctor is available before setting themselves off-call.

Observe what happens under different isolation levels.

---

# Lab 14 — Long Transaction

Start:

```sql
BEGIN;
```

Then leave the transaction open.

From another session:

```text
Inspect active transactions
Inspect locks
Inspect blocked queries
```

Then:

```sql
COMMIT;
```

Observe how the system changes.

---

# Lab 15 — Transaction Monitoring

For PostgreSQL, inspect:

```sql
SELECT *
FROM pg_stat_activity;
```

Look for:

```text
active
idle
idle in transaction
```

Then investigate long-running transactions and blocked sessions.

---

# 🎯 Interview Questions

# Beginner

### What is a transaction?

A transaction is a logical unit of database work governed by transactional guarantees.

---

### What is ACID?

```text
Atomicity
Consistency
Isolation
Durability
```

---

### What is COMMIT?

It completes a transaction successfully and makes its changes committed according to the database's transaction semantics.

---

### What is ROLLBACK?

It aborts the current transaction and undoes its uncommitted changes.

---

### What is a deadlock?

A cycle where transactions wait for resources held by one another.

---

### What is a lock?

A database concurrency-control mechanism used to coordinate access to resources.

---

# 🧠 Intermediate Questions

### What is a dirty read?

Reading data written by another transaction before that transaction has committed.

---

### What is a non-repeatable read?

A transaction reads the same logical row more than once and observes different committed values.

---

### What is a phantom read?

A repeated range query observes a different set of matching rows because concurrent transactions changed the qualifying set.

---

### What is MVCC?

Multi-Version Concurrency Control, a mechanism that maintains multiple row versions or visibility states so transactions can see appropriate snapshots.

---

### What is optimistic locking?

Detecting concurrent modifications using a version or similar condition.

---

### What is pessimistic locking?

Using database locks to protect data from conflicting concurrent operations.

---

### Why can long transactions be dangerous?

They can:

```text
Hold locks
Delay cleanup
Increase MVCC pressure
Consume connections
Increase blocking
Increase resource usage
```

---

# 👑 Senior Questions

### Is READ COMMITTED enough to prevent race conditions?

No.

It prevents certain anomalies such as dirty reads, but business-level race conditions can still occur.

---

### Is MVCC the same as locking?

No.

MVCC and locking are different concurrency mechanisms that can work together.

---

### Is SERIALIZABLE always the best isolation level?

No.

It provides strong correctness guarantees but may increase contention and serialization failures.

---

### Why can an atomic UPDATE be safer than SELECT + UPDATE?

Because the condition and modification can be evaluated together by the database rather than relying on a stale application-side read.

---

### Why are UNIQUE constraints important for concurrency?

They allow the database to enforce uniqueness even when concurrent requests race.

---

# 👑 20+ Year Experience Questions

## Question 1

100 customers attempt to buy the last item.

What would you avoid?

```text
SELECT stock
↓
Application check
↓
UPDATE
```

What might you use?

```text
Atomic conditional UPDATE
```

such as:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 101
  AND quantity > 0;
```

Then inspect:

```text
Rows affected
```

---

# Question 2

Two transactions update different rows but violate a business rule.

What problem could this be?

```text
Write Skew
```

Possible solutions include:

```text
Stronger isolation
Explicit locking
Database constraints
Redesigning the transaction
```

depending on the invariant.

---

# Question 3

A transaction is holding locks for 30 seconds.

What do you investigate?

```text
Transaction duration
SQL execution time
External API calls
Application pauses
Connection pool behavior
Lock waits
Execution plan
```

---

# Question 4

Why can a database be healthy but the application still be slow?

Because:

```text
Connection pool exhausted
```

or:

```text
Transactions waiting on locks
```

or:

```text
Long-running transactions
```

can create application latency even if database CPU is not fully saturated.

---

# Question 5

Two transactions deadlock every few minutes.

What do you investigate?

```text
Lock ordering
Transaction boundaries
Queries involved
Indexes
Access patterns
Transaction duration
Deadlock graphs/logs
```

Then standardize resource acquisition order where possible.

---

# Question 6

A payment request times out after COMMIT.

Did the transaction definitely fail?

No.

The database may have committed successfully and the response may have been lost.

This creates:

```text
Unknown transaction outcome
```

Use:

```text
Idempotency
Unique keys
Safe retry logic
```

to handle this class of failure.

---

# Question 7

Why shouldn't you hold a transaction open during:

```text
HTTP request
External API
Kafka call
User interaction
```

Because these operations have unpredictable latency and can cause:

```text
Lock retention
Connection exhaustion
MVCC pressure
Blocking
Reduced throughput
```

---

# Question 8

How would you design a high-contention inventory system?

Start with:

```text
Invariant:
quantity >= 0
```

Then consider:

```text
Atomic conditional UPDATE
+
Affected-row check
+
Database constraint where appropriate
+
Short transaction
+
Idempotency
+
Retry handling
```

For very high contention, the architecture may additionally require:

```text
Queueing
Reservation model
Partitioning
Sharding
Caching
```

depending on scale.

---

# 🔥 Production Scenario

Imagine a flash sale:

```text
Product:
RTX GPU

Stock:
10

Users:
100,000
```

At:

```text
10:00:00
```

100,000 requests arrive.

Naive architecture:

```text
100,000 requests
       ↓
SELECT stock
       ↓
Application checks
       ↓
UPDATE
```

This can create:

```text
Huge contention
Race conditions
Connection pressure
Database load
```

A better design begins with:

```text
Business invariant
       ↓
Stock cannot become negative
       ↓
Atomic reservation
       ↓
Database concurrency control
       ↓
Short transaction
       ↓
Idempotent request handling
```

For extreme traffic, you may additionally introduce:

```text
Queue
   ↓
Inventory workers
   ↓
Database
```

or other admission-control mechanisms.

---

# 🧠 Production Concurrency Architecture

A mature transactional workflow may look like:

```text
                  CLIENTS
                     │
                     ▼
                  API
                     │
                     ▼
              Idempotency Key
                     │
                     ▼
              Connection Pool
                     │
                     ▼
                 BEGIN
                     │
          ┌──────────┼───────────┐
          │          │           │
          ▼          ▼           ▼
       Read       Validate     Lock
          │          │           │
          └──────────┼───────────┘
                     ▼
              Atomic Updates
                     │
                     ▼
               Constraints
                     │
                     ▼
               Outbox Event
                     │
                     ▼
                  COMMIT
                     │
                     ▼
              Async Processing
                     │
                     ▼
              Message Broker
```

---

# 👑 Senior SQL Concurrency Mental Model

When you see:

```sql
BEGIN;
```

don't just think:

```text
"Start transaction."
```

Think:

```text
What is the atomic unit?

What data can be concurrently modified?

What invariant must remain true?

What isolation level is being used?

What locks can be acquired?

Can transactions block each other?

Can a deadlock occur?

Can a serialization failure occur?

How long will the transaction stay open?

Can the operation be retried?

Is retry safe?

What if COMMIT succeeds but the network fails?

What if the application crashes?

What if another request arrives simultaneously?
```

That is the difference between:

```text
SQL syntax knowledge
```

and:

```text
Production database engineering
```

---

# 🔥 Production Transaction Checklist

```text
TRANSACTION
-----------
□ What operations must be atomic?
□ Where does the transaction begin?
□ Where does it end?
□ Is the transaction unnecessarily long?
□ Are external calls inside it?


CONCURRENCY
-----------
□ Can two requests modify the same row?
□ Can different rows violate the same business rule?
□ Is there a race condition?
□ Could write skew occur?
□ Is optimistic locking appropriate?
□ Is pessimistic locking appropriate?


ISOLATION
---------
□ What isolation level is used?
□ Are dirty reads possible?
□ Can non-repeatable reads occur?
□ Can phantoms occur?
□ Is SERIALIZABLE necessary?


LOCKING
-------
□ What rows are locked?
□ What order are locks acquired?
□ Could blocking occur?
□ Could deadlocks occur?
□ How long are locks held?


FAILURE
-------
□ What happens on rollback?
□ What happens on timeout?
□ What happens on connection loss?
□ What happens if COMMIT outcome is unknown?
□ Is retry safe?
□ Is the operation idempotent?


PERFORMANCE
-----------
□ Transaction duration
□ Lock wait time
□ Connection pool usage
□ Commit rate
□ Rollback rate
□ Deadlocks
□ Serialization failures
□ Long-running transactions
```

---

# 📌 Chapter Summary

Transactions provide the foundation for safe database modifications.

Core concepts:

```text
BEGIN
COMMIT
ROLLBACK
SAVEPOINT
```

ACID:

```text
Atomicity
Consistency
Isolation
Durability
```

Concurrency problems:

```text
Dirty Read
Non-Repeatable Read
Phantom Read
Lost Update
Write Skew
Race Conditions
```

Concurrency mechanisms:

```text
Locks
MVCC
Optimistic Locking
Pessimistic Locking
Constraints
Atomic Updates
```

Isolation levels:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Production concerns:

```text
Deadlocks
Blocking
Long Transactions
Connection Pool Exhaustion
Unknown Commit Outcome
Retry Safety
Idempotency
Distributed Transactions
```

---

# 🔥 Critical Rules

```text
Rule 1
------
A transaction should represent a meaningful atomic business unit.


Rule 2
------
Keep transactions short, but never split operations
that must remain atomic.


Rule 3
------
Never assume READ COMMITTED prevents all race conditions.


Rule 4
------
Use atomic database operations whenever possible.


Rule 5
------
Use UNIQUE constraints to protect uniqueness under concurrency.


Rule 6
------
Use optimistic locking when conflicts should be detected,
not silently overwritten.


Rule 7
------
Use pessimistic locking when short-lived serialization
is required and contention justifies it.


Rule 8
------
Always acquire multiple resources in a consistent order
where possible.


Rule 9
------
Deadlocks are expected possibilities in concurrent systems.
Design for detection and safe retry.


Rule 10
------
Never blindly retry a failed transaction.


Rule 11
------
An unknown COMMIT result does not necessarily mean
the transaction failed.


Rule 12
------
Idempotency is critical when database operations interact
with unreliable networks and distributed systems.


Rule 13
------
Long transactions can damage database performance
even when they are not actively executing SQL.


Rule 14
------
MVCC does not mean databases have no locks.


Rule 15
------
Isolation level is a business correctness decision,
not simply a performance setting.


Rule 16
------
Understand your specific database engine's isolation
and locking semantics.


Rule 17
------
Always design concurrency around the business invariant,
not merely around individual SQL statements.
```

---

# 🎉 Module 1 — Database Fundamentals Continues

You now understand:

```text
SQL
 ↓
Filtering
 ↓
Aggregation
 ↓
GROUP BY
 ↓
JOINs
 ↓
Subqueries
 ↓
CTEs
 ↓
INSERT / UPDATE / DELETE
 ↓
UPSERT
 ↓
Transactions
 ↓
ACID
 ↓
Concurrency
 ↓
Locks
 ↓
MVCC
 ↓
Isolation Levels
```

The next step is to understand how databases **physically store rows and find them efficiently**.

Knowing SQL syntax is not enough.

A senior engineer must understand why:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

can take:

```text
milliseconds
```

while another query can take:

```text
minutes
```

even when both return only one row.

The answer leads us to:

```text
Indexes
B-Trees
Hash Indexes
Composite Indexes
Covering Indexes
Clustered Indexes
Index Selectivity
Cardinality
Query Plans
```

---

# 🚀 Next Chapter

# Chapter 15 — SQL Indexes Deep Dive: B-Tree, Composite, Covering, Selectivity, Cardinality & Query Performance

We will answer:

```text
Why does an index make queries faster?

How does a B-Tree actually work?

Why is searching an indexed column faster than scanning
the entire table?

What happens internally during an index lookup?

Why can too many indexes make INSERT/UPDATE slower?

What is a composite index?

Why does column order matter?

What is the leftmost-prefix rule?

What is a covering index?

What is a partial/filtered index?

What is a unique index?

What is index selectivity?

What is cardinality?

Why does the optimizer sometimes ignore an index?

Why can WHERE functions prevent index usage?

Why can LIKE '%abc%' be slow?

Why can OFFSET pagination become slow?

How do indexes behave with millions and billions of rows?

How do you design indexes for production APIs?

How do you find unused indexes?

How do you diagnose a slow query using EXPLAIN?

How do indexes interact with JOINs?

How do indexes affect ORDER BY?

How do indexes affect GROUP BY?

How do indexes affect UPDATE and DELETE?
```

The central question will be:

> **When you execute a SQL query, how does the database find the exact rows without scanning the entire table?**
