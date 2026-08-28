# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 6 — Database Constraints: Making Invalid Data Impossible

> In the previous chapter, we learned SQL fundamentals:
>
> ```text
> SELECT
> FROM
> WHERE
> ORDER BY
> LIMIT
> DISTINCT
> NULL
> Expressions
> Operators
> ```
>
> But there is a serious problem.
>
> What stops an application from inserting **bad data**?
>
> Imagine our production database contains:
>
> ```text
> Customer ID = 1001
> Email = vinay@example.com
> ```
>
> Then another request inserts:
>
> ```text
> Customer ID = 1001
> ```
>
> Or:
>
> ```text
> Email = vinay@example.com
> ```
>
> twice.
>
> Or:
>
> ```text
> Product price = -5000
> ```
>
> Or:
>
> ```text
> Order customer_id = 999999
> ```
>
> when customer `999999` doesn't exist.
>
> These are not merely programming mistakes.
>
> They are **data integrity failures**.
>
> The database provides a powerful mechanism to prevent many of these invalid states:
>
> # Constraints
>
> Instead of saying:
>
> ```text
> "Please make sure the data is valid."
> ```
>
> we tell the database:
>
> ```text
> "This data MUST satisfy this rule."
> ```
>
> That difference is fundamental to professional database engineering.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What database constraints are
* Why constraints matter
* `PRIMARY KEY`
* `FOREIGN KEY`
* `UNIQUE`
* `NOT NULL`
* `CHECK`
* `DEFAULT`
* Referential integrity
* Entity integrity
* Domain integrity
* Composite constraints
* Constraint naming
* Constraint enforcement
* Constraint vs application validation
* `ON DELETE`
* `ON UPDATE`
* `CASCADE`
* `RESTRICT`
* `SET NULL`
* Real production constraint failures
* How constraints affect database design

---

# 📖 The Real Software Problem

Imagine an e-commerce application.

We have:

```text
Customer
Product
Order
Payment
```

The application has:

```text
Web API
Mobile API
Admin Dashboard
Background Workers
Data Import Jobs
```

All of them can potentially write to the database.

Now suppose the application developer writes:

```java
if (!emailExists(email)) {
    createUser(email);
}
```

Looks safe.

But what happens if two requests arrive simultaneously?

```text
Request A
    ↓
Check email
    ↓
Doesn't exist

Request B
    ↓
Check email
    ↓
Doesn't exist
```

Both now insert:

```text
vinay@example.com
```

Result:

```text
Duplicate Email
```

Application-level validation alone is not enough.

We need the database to enforce the invariant.

```text
email
  ↓
UNIQUE
```

---

# 🧠 What Is a Constraint?

A constraint is a rule enforced by the database that restricts which data states are valid.

Think:

```text
Application
     ↓
SQL
     ↓
Database
     ↓
Constraint
     ↓
Valid / Invalid
```

If the data violates the rule:

```text
Database
   ↓
Reject operation
```

This is powerful because the rule is enforced at the data boundary.

---

# 🔥 Core Constraints

The most important SQL constraints are:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
DEFAULT
```

Think of them as different types of protection.

```text
PRIMARY KEY
    ↓
Identity

FOREIGN KEY
    ↓
Relationship

UNIQUE
    ↓
No duplicates

NOT NULL
    ↓
Required value

CHECK
    ↓
Business condition

DEFAULT
    ↓
Automatic value
```

---

# 1️⃣ PRIMARY KEY

A primary key uniquely identifies a row.

Example:

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

Now:

```text
id
```

must uniquely identify each customer.

---

# 🚨 Test It

First:

```sql
INSERT INTO customers
    (id, name, email)
VALUES
    (1, 'Vinay', 'vinay@example.com');
```

Works.

Now:

```sql
INSERT INTO customers
    (id, name, email)
VALUES
    (1, 'Rahul', 'rahul@example.com');
```

The database rejects it because:

```text
id = 1
```

already exists.

---

# 🧠 What Does PRIMARY KEY Guarantee?

A primary key guarantees that the key identifies rows uniquely and does not contain NULL values.

For example:

```text
customers

id
---
1
2
3
```

Valid.

But:

```text
id
---
1
1
```

invalid.

And:

```text
id
---
1
NULL
```

invalid for a primary key.

---

# 🧩 Composite PRIMARY KEY

A primary key can contain multiple columns.

Example:

```sql
CREATE TABLE student_courses (
    student_id BIGINT,
    course_id BIGINT,

    PRIMARY KEY (student_id, course_id)
);
```

Now:

```text
student_id | course_id
-----------+----------
1          | 10
1          | 20
2          | 10
```

is valid.

But:

```text
1 | 10
1 | 10
```

is not.

The combination must be unique.

---

# 🧠 Important Principle

A primary key answers:

> **"Which exact row is this?"**

It is about identity.

---

# 2️⃣ UNIQUE

Now consider email.

We may have:

```text
id
name
email
```

The primary key is:

```text
id
```

But business rules may require:

```text
No two customers can have the same email.
```

Use:

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);
```

Now:

```text
id
```

identifies the customer.

While:

```text
email
```

must also be unique.

---

# 🧠 PRIMARY KEY vs UNIQUE

Think:

```text
PRIMARY KEY
     ↓
Main identity of the row
```

while:

```text
UNIQUE
     ↓
Another uniqueness rule
```

A table can have multiple unique constraints.

But it has one primary key constraint.

---

# 🚨 Real Production Problem

Suppose your authentication system searches:

```sql
SELECT *
FROM customers
WHERE email = 'vinay@example.com';
```

If email is supposed to identify one account, then:

```text
email
```

should have a uniqueness guarantee.

Otherwise the database could contain:

```text
id | email
---+------------------
1  | vinay@example.com
2  | vinay@example.com
```

Now what does login mean?

Which account should authenticate?

This is not just a query problem.

It is a **data model problem**.

---

# 🔥 Senior Principle

> If uniqueness is a business invariant, enforce it at the database level whenever the data ownership and architecture make that appropriate.

Don't rely only on:

```text
Application checks
```

---

# 3️⃣ NOT NULL

Suppose every customer must have a name.

Use:

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

Now this is invalid:

```sql
INSERT INTO customers
    (id, name, email)
VALUES
    (2, NULL, 'rahul@example.com');
```

The database rejects it.

---

# 🧠 Why NOT NULL Matters

Without `NOT NULL`:

```text
name
----
Vinay
Rahul
NULL
Ravi
```

Now every application querying the database must decide:

```text
What does NULL mean?
```

But if the business rule says:

> Every customer must have a name.

then represent that rule directly:

```sql
name TEXT NOT NULL
```

---

# 🚨 Real Software Problem

Suppose your application assumes:

```java
customer.getName().toUpperCase()
```

But database contains:

```text
name = NULL
```

Now the application may fail.

Instead of allowing invalid data and discovering the problem later:

```text
Database
   ↓
NOT NULL
   ↓
Invalid state prevented
```

---

# 🧠 NOT NULL Is About Required Data

Think:

```text
NOT NULL
    ↓
This value must exist.
```

But remember:

```text
NOT NULL
```

does not mean:

```text
non-empty string
```

For example:

```text
name = ''
```

may still be allowed unless another rule prevents it.

---

# 4️⃣ CHECK

Now suppose product prices must never be negative.

Bad:

```text
price = -5000
```

We can enforce:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC(12,2) NOT NULL,
    CHECK (price >= 0)
);
```

Now:

```sql
INSERT INTO products
    (id, name, price)
VALUES
    (10, 'Keyboard', -5000);
```

should be rejected.

---

# 🧠 CHECK Constraints

A `CHECK` constraint expresses a condition that values must satisfy.

Examples:

```sql
CHECK (price >= 0)
```

```sql
CHECK (stock >= 0)
```

```sql
CHECK (age >= 18)
```

```sql
CHECK (quantity > 0)
```

---

# 🔥 Real Inventory Problem

Imagine:

```text
stock = 100
```

A bug executes:

```sql
UPDATE products
SET stock = -500
WHERE id = 10;
```

If the database has:

```sql
CHECK (stock >= 0)
```

the invalid state is rejected.

This is extremely valuable.

---

# ⚠️ CHECK and Business Logic

Be careful.

A simple rule:

```sql
CHECK (price >= 0)
```

is easy.

But a complex business rule such as:

```text
Customer cannot have more than
5 active loans across all branches
```

may require:

```text
Queries
Transactions
Triggers
Application logic
Stored procedures
Architecture
```

depending on the database and system design.

Don't try to force every business rule into a simple `CHECK`.

---

# 5️⃣ DEFAULT

Suppose every new order should initially be:

```text
PENDING
```

We can define:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status TEXT NOT NULL DEFAULT 'PENDING',
    total_amount NUMERIC(12,2) NOT NULL
);
```

Now:

```sql
INSERT INTO orders
    (id, customer_id, total_amount)
VALUES
    (101, 1, 5000);
```

The database can automatically assign:

```text
status = PENDING
```

---

# 🧠 DEFAULT Does Not Mean "Always This Value"

This is important.

`DEFAULT` is used when the column is omitted from an insert, subject to the database's exact semantics.

It does not mean:

```text
"The column can never contain another value."
```

For example:

```sql
status TEXT DEFAULT 'PENDING'
```

does not prevent:

```sql
status = 'PAID'
```

It only supplies a value when one isn't provided.

---

# 🔥 DEFAULT + NOT NULL

Often used together:

```sql
status TEXT NOT NULL DEFAULT 'PENDING'
```

This means:

```text
A value must exist.
+
If caller doesn't provide one,
use PENDING.
```

Very common production pattern.

---

# 6️⃣ FOREIGN KEY

Now we protect relationships.

Orders:

```text
orders.customer_id
```

must reference:

```text
customers.id
```

Use:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,

    FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Now:

```text
customers
   │
   │
   ▼
customer_id
   │
   ▼
orders
```

---

# 🚨 Invalid Foreign Key

Suppose:

```text
customers

id
---
1
2
3
```

Now:

```sql
INSERT INTO orders
    (id, customer_id)
VALUES
    (101, 999);
```

Customer `999` does not exist.

The database rejects the insert.

---

# 🧠 Foreign Key Answers

> **"Does this referenced entity actually exist?"**

It protects relationships.

---

# 🔥 Entity Integrity vs Referential Integrity

Two important concepts:

## Entity Integrity

A row must have a valid identity.

Typically enforced through:

```text
PRIMARY KEY
```

---

## Referential Integrity

Relationships must reference valid rows.

Typically enforced through:

```text
FOREIGN KEY
```

Think:

```text
Entity Integrity
      ↓
"Who are you?"

Referential Integrity
      ↓
"Who do you refer to?"
```

---

# 🔗 ON DELETE

Now imagine:

```text
Customer 1
   ↓
Order 101
Order 102
Order 103
```

What happens if we delete Customer 1?

```sql
DELETE FROM customers
WHERE id = 1;
```

The database needs a defined relationship policy.

Possible behavior includes:

```text
RESTRICT
NO ACTION
CASCADE
SET NULL
SET DEFAULT
```

Support and exact behavior vary by database.

---

# 1️⃣ RESTRICT

Conceptually:

```text
Parent has children
        ↓
Don't allow deletion
```

Example:

```text
Customer
   ↓
Orders
```

If orders exist:

```text
DELETE Customer
```

can be rejected.

This is often desirable for important historical records.

---

# 2️⃣ CASCADE

Conceptually:

```text
Delete Parent
      ↓
Delete Children
```

Example:

```text
User
 ↓
User Sessions
```

Deleting a user might reasonably delete their sessions.

Example:

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE
```

---

# 🚨 CASCADE Can Be Dangerous

Imagine:

```text
Customer
   ↓
Orders
   ↓
Order Items
   ↓
Payments
```

If you configure cascading deletes carelessly:

```text
DELETE Customer
```

could trigger:

```text
Customer
 ↓
Orders
 ↓
Order Items
 ↓
Payments
```

potentially deleting large amounts of data.

For financial or historical data, this may be completely unacceptable.

---

# 🔥 Senior Rule

Never add:

```text
ON DELETE CASCADE
```

just because:

> "It makes deletes easier."

Ask:

```text
What does deleting this entity mean
in the business domain?
```

---

# 3️⃣ SET NULL

Suppose an order can survive without an optional reference.

Then:

```sql
FOREIGN KEY (sales_rep_id)
REFERENCES employees(id)
ON DELETE SET NULL
```

could mean:

```text
Employee deleted
      ↓
Order remains
      ↓
sales_rep_id = NULL
```

But the column must be nullable for this design.

---

# 🧠 Choosing Referential Actions

Think:

```text
Should child disappear?
        ↓
CASCADE

Should parent deletion be blocked?
        ↓
RESTRICT / NO ACTION

Should relationship disappear
but child remain?
        ↓
SET NULL
```

Exact semantics depend on the database.

---

# 🧩 Constraint Naming

You can define explicit names:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,

    customer_id BIGINT NOT NULL,

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Why name constraints?

Because errors become easier to understand.

Instead of:

```text
some generated constraint name
```

you get:

```text
fk_orders_customer
```

This helps during:

```text
Debugging
Migrations
Incident Investigation
Schema Maintenance
```

---

# 🔥 Real Production Migration

Imagine production contains:

```text
500 tables
```

and:

```text
thousands of constraints
```

A migration fails:

```text
ALTER TABLE ...
```

If constraints have meaningful names:

```text
fk_orders_customer
uq_users_email
chk_products_price
```

you immediately understand what failed.

Good schema naming is an operational tool.

---

# 🧠 Multiple Constraints Can Work Together

Consider:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,

    sku TEXT NOT NULL UNIQUE,

    name TEXT NOT NULL,

    price NUMERIC(12,2) NOT NULL,

    stock INTEGER NOT NULL DEFAULT 0,

    CHECK (price >= 0),

    CHECK (stock >= 0)
);
```

Now the table guarantees:

```text
id
 ↓
Unique identity

sku
 ↓
Unique product identifier

name
 ↓
Required

price
 ↓
Required + non-negative

stock
 ↓
Required + non-negative + defaults to 0
```

This is much stronger than simply defining columns.

---

# 🏗️ Constraints Form a Data Contract

Think of a database schema as a contract.

```text
Database Schema
      ↓
Data Contract
      ↓
Allowed States
```

For example:

```text
Product
```

may allow:

```text
price >= 0
stock >= 0
sku unique
name required
```

Everything else is invalid.

---

# 🔥 The Database as a Boundary

Imagine:

```text
Web API
Mobile API
Admin
Worker
Import Script
       │
       ▼
   DATABASE
       │
       ▼
 CONSTRAINTS
       │
       ▼
 Valid State
```

Every writer must pass through the same database rules.

This is one of the strongest reasons constraints are valuable.

---

# 🚨 Race Conditions

Now let's understand why application validation can fail.

Suppose we want unique emails.

Application:

```text
Request A
   ↓
SELECT email
   ↓
Not found

Request B
   ↓
SELECT email
   ↓
Not found
```

Then:

```text
A → INSERT
B → INSERT
```

Without a unique constraint:

```text
Duplicate!
```

With:

```sql
UNIQUE(email)
```

the database provides the final enforcement point.

One transaction succeeds.

The conflicting operation is rejected according to the database's concurrency and constraint semantics.

---

# 🧠 Validation vs Constraint

Application validation:

```text
"Is this input acceptable?"
```

Database constraint:

```text
"Is this data state allowed?"
```

These are related but different.

Use application validation for:

```text
User-friendly errors
Input formatting
API contracts
Business workflows
Early rejection
```

Use database constraints for:

```text
Data integrity
Uniqueness
Required fields
Relationships
Simple invariants
```

Good systems often use both.

---

# 🔥 Real Architecture

Consider:

```text
Frontend
   ↓
API
   ↓
Service
   ↓
Repository
   ↓
Database
   ↓
Constraints
```

Validation might happen at:

```text
Frontend
API
Service
```

But the final persistent state is protected by:

```text
Database Constraints
```

---

# 🧠 What Happens If Application and Database Rules Conflict?

Suppose application says:

```text
price can be negative
```

but database says:

```sql
CHECK (price >= 0)
```

The database wins at persistence time.

The application must handle the resulting error appropriately.

This is why database constraints should be treated as part of the system's contract.

---

# 🔥 Real Production Problem — Inventory

Suppose:

```text
stock = 5
```

Two requests arrive:

```text
Request A → buy 5
Request B → buy 5
```

Application reads:

```text
stock = 5
```

Both requests think they can proceed.

Without proper concurrency control, the system can produce an invalid result.

A `CHECK (stock >= 0)` can prevent the final stored value from becoming negative, but:

> A CHECK constraint alone does not solve the concurrency problem.

You may also need:

```text
Transactions
Row locking
Atomic UPDATE
Optimistic concurrency
Isolation
```

This is a very important senior-level distinction.

---

# 🧠 Constraints Are Not Transactions

A constraint answers:

```text
"Is this resulting state valid?"
```

A transaction answers:

```text
"How should a group of operations behave atomically and consistently?"
```

These concepts work together.

Later we will study:

```text
Transactions
Isolation
Locks
MVCC
Concurrency
```

in depth.

---

# 🧪 Hands-on Lab

## Lab 1 — Create a Strong Products Table

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,

    sku TEXT NOT NULL UNIQUE,

    name TEXT NOT NULL,

    price NUMERIC(12,2) NOT NULL,

    stock INTEGER NOT NULL DEFAULT 0,

    CHECK (price >= 0),

    CHECK (stock >= 0)
);
```

---

# Lab 2 — Valid Insert

```sql
INSERT INTO products (
    id,
    sku,
    name,
    price,
    stock
)
VALUES (
    1,
    'KB-001',
    'Mechanical Keyboard',
    5000,
    100
);
```

Should succeed.

---

# Lab 3 — Duplicate SKU

Try:

```sql
INSERT INTO products (
    id,
    sku,
    name,
    price,
    stock
)
VALUES (
    2,
    'KB-001',
    'Another Keyboard',
    6000,
    50
);
```

Expected:

```text
UNIQUE constraint violation
```

---

# Lab 4 — Negative Price

Try:

```sql
INSERT INTO products (
    id,
    sku,
    name,
    price,
    stock
)
VALUES (
    3,
    'KB-002',
    'Keyboard',
    -5000,
    20
);
```

Expected:

```text
CHECK constraint violation
```

---

# Lab 5 — Negative Stock

```sql
INSERT INTO products (
    id,
    sku,
    name,
    price,
    stock
)
VALUES (
    4,
    'KB-003',
    'Keyboard',
    5000,
    -10
);
```

Should be rejected.

---

# Lab 6 — DEFAULT

Run:

```sql
INSERT INTO products (
    id,
    sku,
    name,
    price
)
VALUES (
    5,
    'MS-001',
    'Wireless Mouse',
    2500
);
```

Then:

```sql
SELECT *
FROM products
WHERE id = 5;
```

Observe:

```text
stock = 0
```

because of:

```sql
DEFAULT 0
```

---

# Lab 7 — Foreign Key

Create:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,

    customer_id BIGINT NOT NULL,

    total_amount NUMERIC(12,2) NOT NULL,

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Now try:

```sql
INSERT INTO orders (
    id,
    customer_id,
    total_amount
)
VALUES (
    1001,
    999999,
    5000
);
```

If customer `999999` does not exist, the foreign key should reject the insert.

---

# 🎯 Interview Questions

## Beginner

### What is a constraint?

A database rule that restricts data to valid states.

---

### What does PRIMARY KEY do?

It uniquely identifies rows and does not allow NULL values in the key.

---

### What does UNIQUE do?

It enforces uniqueness according to the database's unique-constraint semantics.

---

### What does NOT NULL do?

It prevents a column from containing NULL.

---

### What does CHECK do?

It enforces a condition on row values according to the database's CHECK semantics.

---

### What does DEFAULT do?

It supplies a value when a column is omitted from an insert, according to the database's semantics.

---

### What does FOREIGN KEY do?

It enforces a relationship between values in one table and a referenced key in another table.

---

# 🧠 Intermediate Questions

### Can a table have multiple UNIQUE constraints?

Yes.

Example:

```text
email UNIQUE
phone UNIQUE
username UNIQUE
```

---

### Can a table have multiple primary keys?

No.

But the one primary key can contain multiple columns.

---

### Can a foreign key reference a UNIQUE key?

Yes, where the database permits referencing the relevant unique constraint/key and the referenced columns satisfy the required uniqueness rules.

---

### Does NOT NULL prevent empty strings?

No.

For text:

```text
NULL
```

and:

```text
''
```

are different values.

---

### Does CHECK automatically solve every business rule?

No.

Complex rules involving multiple rows, aggregates, or external systems may require other mechanisms.

---

# 👑 Senior Questions

### Why should constraints exist if application validation already exists?

Because multiple applications and processes may write to the database, and concurrent requests can bypass application-level checks.

Constraints provide centralized data integrity enforcement.

---

### Can UNIQUE prevent race conditions?

It provides the uniqueness guarantee at the database level, including under concurrent writes according to the database's transaction/constraint semantics.

But the application still needs to correctly handle uniqueness violations.

---

### Can CHECK solve inventory overselling?

Not by itself.

It can prevent:

```text
stock < 0
```

but does not determine which concurrent transaction gets the inventory.

You also need appropriate concurrency control.

---

### Should all foreign keys use CASCADE?

No.

The correct action depends on the business meaning and lifecycle of the data.

---

# 👑 20+ Year Experience Questions

## Question 1

You have:

```text
Application
   ↓
SELECT email
   ↓
If not exists
   ↓
INSERT
```

Is that sufficient for uniqueness?

No.

Two concurrent transactions can both observe that the email does not exist.

Use a database-level uniqueness constraint:

```sql
UNIQUE(email)
```

and handle the possible conflict correctly.

---

# Question 2

You have:

```text
CHECK (stock >= 0)
```

Can two users still buy the last product simultaneously?

Yes.

The constraint only restricts invalid resulting states.

It does not by itself provide the concurrency semantics required to safely allocate the same inventory to competing transactions.

---

# Question 3

When should you use:

```text
NOT NULL
```

instead of handling NULL in Java?

When the database invariant is:

> This value must exist for every row.

Then enforce it at the data layer.

---

# Question 4

Why is:

```text
UNIQUE(email)
```

usually better than:

```text
Application checks email
```

Because the database sees all writes that reach it and can enforce the invariant atomically with the write operation.

---

# Question 5

Why can:

```text
ON DELETE CASCADE
```

be dangerous?

Because deleting one parent row can delete many dependent rows.

The impact can be much larger than the original operation appears to be.

---

# Question 6

Should every possible business rule be a database constraint?

No.

Ask:

```text
Is it a row-level invariant?

Is it multi-row?

Does it involve external systems?

Does it require complex workflow?

Does it need temporal reasoning?

Does it depend on distributed state?
```

Simple invariants are excellent candidates for constraints.

Complex domain behavior may require other architectural mechanisms.

---

# 🔥 Senior Database Design Mental Model

When designing a table, ask:

```text
What identifies this row?
        ↓
PRIMARY KEY

What values must be unique?
        ↓
UNIQUE

What values are mandatory?
        ↓
NOT NULL

What values must satisfy simple rules?
        ↓
CHECK

What relationships must exist?
        ↓
FOREIGN KEY

What values should be automatically supplied?
        ↓
DEFAULT
```

This creates a powerful schema.

---

# 🏗️ Example — Production-Style Table

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,

    customer_id BIGINT NOT NULL,

    status TEXT NOT NULL DEFAULT 'PENDING',

    total_amount NUMERIC(12,2) NOT NULL,

    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CHECK (total_amount >= 0),

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Think about what the database now guarantees:

```text
id
 ↓
Unique identity

customer_id
 ↓
Required relationship

status
 ↓
Required + default

total_amount
 ↓
Required + non-negative

created_at
 ↓
Automatically populated if omitted
```

This is much stronger than:

```text
CREATE TABLE orders (...)
```

with no constraints.

---

# 🧠 Constraints and Schema Evolution

Now imagine the table already contains:

```text
100 million rows
```

and you want to add:

```sql
CHECK (price >= 0)
```

What if existing data contains:

```text
price = -10
```

The migration may fail or require a strategy specific to the database.

This introduces a major production concern:

```text
Schema Change
      ↓
Existing Data
      ↓
Constraint Validation
      ↓
Locking / Performance
      ↓
Deployment Strategy
```

Later, we'll study safe production migrations in detail.

---

# 🔥 Important Principle

> **A constraint is not merely documentation. It is executable data integrity.**

Compare:

```text
Comment:

"Price should never be negative."
```

with:

```sql
CHECK (price >= 0)
```

The comment says:

```text
Please follow this rule.
```

The constraint says:

```text
The database will enforce this rule.
```

That is a huge difference.

---

# 📌 Chapter Summary

Database constraints protect the integrity of your data.

The major constraints are:

```text
PRIMARY KEY
     ↓
Unique row identity

FOREIGN KEY
     ↓
Valid relationships

UNIQUE
     ↓
No duplicate values according to the constraint

NOT NULL
     ↓
Required values

CHECK
     ↓
Allowed conditions

DEFAULT
     ↓
Automatic values when omitted
```

Constraints can prevent:

```text
Duplicate Users
Invalid References
Negative Prices
Negative Inventory
Missing Required Data
Invalid Relationships
```

---

# 🔥 Final Mental Model

```text
                 APPLICATIONS
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
         Web        Mobile      Workers
          │           │           │
          └───────────┼───────────┘
                      ▼
                     SQL
                      │
                      ▼
                  DATABASE
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       PRIMARY      FOREIGN      UNIQUE
        KEY          KEY
          │           │           │
          └───────────┼───────────┘
                      ▼
                 NOT NULL
                      │
                      ▼
                    CHECK
                      │
                      ▼
                   DEFAULT
                      │
                      ▼
                VALID DATA
```

The database is not just storing data.

It is protecting the **allowed states of your system**.

That is one of the biggest differences between:

```text
A database
```

and:

```text
A reliable database system.
```

---

# 🚀 Next Chapter

# Chapter 7 — NULL, Three-Valued Logic & Missing Data

We will go much deeper into one of SQL's most dangerous concepts:

```text
NULL
```

We'll solve real bugs involving:

```text
NULL = NULL

NULL <> value

NOT IN + NULL

LEFT JOIN + NULL

WHERE + NULL

COUNT + NULL

SUM + NULL

COALESCE

CASE

NULL ordering
```

And we'll investigate a real production bug where:

```sql
WHERE customer_id NOT IN (...)
```

unexpectedly returns **zero rows** because of a single `NULL`.

> **If you truly master NULL and three-valued logic, a large class of SQL bugs becomes much easier to reason about.**
