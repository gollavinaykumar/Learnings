# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 5 — SQL Language Fundamentals

> In the previous chapter, we opened the database engine and learned an important principle:
>
> ```text
> SQL
>  ↓
> WHAT?
>  ↓
> Database Optimizer
>  ↓
> HOW?
>  ↓
> Execution Plan
> ```
>
> SQL is not simply a collection of commands to memorize.
>
> SQL is a language for expressing operations over relational data.
>
> In this chapter, we will build SQL from the ground up using a realistic software system.
>
> We will start with:
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
> Aliases
> Operators
> Data Types
> ```
>
> But every concept will be connected to a **real software problem**.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What SQL actually represents
* `SELECT`
* `FROM`
* `WHERE`
* `ORDER BY`
* `LIMIT`
* `DISTINCT`
* Column aliases
* Expressions
* Arithmetic operators
* Comparison operators
* Logical operators
* `NULL`
* Three-valued logic
* Basic SQL data types
* SQL literals
* Type conversion
* SQL comments
* How SQL clauses logically work together
* Why SQL syntax order differs from logical execution order
* Basic query-writing discipline

---

# 🏗️ Our Real Application

Throughout this chapter, imagine we are building:

# E-Commerce Platform

Our system has:

```text
Customers
Products
Orders
Order Items
Payments
```

We'll begin with three tables.

---

# 👤 Customers

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    city TEXT,
    created_at TIMESTAMP NOT NULL
);
```

Example:

```text
id | name  | email              | city      | created_at
---+-------+--------------------+-----------+-------------------
1  | Vinay | vinay@example.com  | Hyderabad | 2026-01-10
2  | Rahul | rahul@example.com  | Chennai   | 2026-01-12
3  | Ravi  | ravi@example.com   | Bangalore | 2026-01-15
```

---

# 📦 Products

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price NUMERIC(12,2) NOT NULL,
    stock INTEGER NOT NULL
);
```

Example:

```text
id | name      | category    | price   | stock
---+-----------+-------------+---------+------
10 | iPhone    | Electronics | 80000   | 25
11 | MacBook   | Electronics | 120000  | 10
12 | Keyboard  | Accessories | 5000    | 100
13 | Mouse     | Accessories | 2500    | 200
```

---

# 🛒 Orders

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status TEXT NOT NULL,
    total_amount NUMERIC(12,2) NOT NULL,
    created_at TIMESTAMP NOT NULL,

    FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Example:

```text
id  | customer_id | status    | total_amount
----+-------------+-----------+-------------
101 | 1           | PAID      | 80000
102 | 1           | SHIPPED   | 125000
103 | 2           | PENDING   | 5000
104 | 3           | CANCELLED | 2500
```

Now let's learn SQL.

---

# 1️⃣ SELECT

The most fundamental SQL operation:

```sql
SELECT *
FROM customers;
```

This means:

```text
Return all columns
from customers
```

Result:

```text
id | name  | email             | city
---+-------+-------------------+----------
1  | Vinay | vinay@example.com | Hyderabad
2  | Rahul | rahul@example.com | Chennai
3  | Ravi  | ravi@example.com  | Bangalore
```

---

# 🧠 What Does `*` Mean?

This:

```sql
SELECT *
FROM customers;
```

means:

```text
Select all columns
```

But in production systems, avoid blindly using:

```sql
SELECT *
```

when you only need a few columns.

Prefer:

```sql
SELECT
    id,
    name,
    email
FROM customers;
```

Why?

Because the application may only need:

```text
id
name
email
```

instead of:

```text
Every column
```

---

# 🚨 Real Software Problem

Suppose the table eventually has:

```text
50 columns
```

and some columns contain:

```text
Large JSON
Images
Documents
Long text
Metadata
```

Your application runs:

```sql
SELECT *
FROM customers;
```

Now the database may need to return much more data than the application actually needs.

That can increase:

```text
Database Work
Network Transfer
Memory Usage
Application Processing
Serialization Cost
```

So a senior engineer asks:

> **What columns does the application actually need?**

---

# 2️⃣ FROM

The:

```sql
FROM
```

clause identifies the relation(s) from which data is retrieved.

Example:

```sql
SELECT name
FROM customers;
```

Think:

```text
FROM
 ↓
Where is the data coming from?
```

---

# 3️⃣ WHERE

Suppose we only want customers from Hyderabad.

```sql
SELECT *
FROM customers
WHERE city = 'Hyderabad';
```

Result:

```text
1 | Vinay | vinay@example.com | Hyderabad
```

`WHERE` filters rows based on a condition.

---

# 🧠 Think of WHERE as a Question

For every candidate row, conceptually ask:

```text
Is:

city = 'Hyderabad' ?
```

If true:

```text
Keep row
```

If false:

```text
Discard row
```

This is a simplified logical model; actual execution may use indexes or other optimizations rather than literally checking every row.

---

# 🔢 Comparison Operators

SQL provides operators such as:

```text
=
<>
!=
>
<
>=
<=
```

Examples:

```sql
SELECT *
FROM products
WHERE price > 50000;
```

---

```sql
SELECT *
FROM products
WHERE stock <= 20;
```

---

```sql
SELECT *
FROM orders
WHERE status <> 'CANCELLED';
```

---

# 🧠 `<>` vs `!=`

In many SQL systems:

```sql
<>
```

and:

```sql
!=
```

both represent "not equal".

However, SQL dialects differ in supported syntax and behavior details.

For portable SQL, `<>` is the standard SQL operator.

---

# 4️⃣ Logical Operators

We can combine conditions.

## AND

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
  AND price > 50000;
```

Both conditions must evaluate to true.

Conceptually:

```text
category = Electronics
        AND
price > 50000
```

---

# OR

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
   OR category = 'Accessories';
```

At least one condition must evaluate to true.

---

# NOT

```sql
SELECT *
FROM products
WHERE NOT category = 'Accessories';
```

This negates the condition.

Often clearer:

```sql
SELECT *
FROM products
WHERE category <> 'Accessories';
```

---

# 🧠 Operator Precedence

Consider:

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
   OR category = 'Accessories'
  AND price > 10000;
```

What does this mean?

Because `AND` generally has higher precedence than `OR`, it is interpreted conceptually as:

```text
category = Electronics

OR

(
category = Accessories
AND
price > 10000
)
```

If you mean something else, use parentheses:

```sql
SELECT *
FROM products
WHERE
    (
        category = 'Electronics'
        OR category = 'Accessories'
    )
    AND price > 10000;
```

Senior SQL engineers don't rely on readers remembering precedence.

They use parentheses when the business rule needs to be obvious.

---

# 5️⃣ ORDER BY

Suppose we want products from highest price to lowest.

```sql
SELECT
    id,
    name,
    price
FROM products
ORDER BY price DESC;
```

Result:

```text
MacBook   120000
iPhone     80000
Keyboard    5000
Mouse       2500
```

Ascending:

```sql
ORDER BY price ASC;
```

Descending:

```sql
ORDER BY price DESC;
```

---

# 🧠 Important: SQL Tables Do Not Have an Intrinsic Display Order

This is a critical beginner mistake.

If you run:

```sql
SELECT *
FROM customers;
```

and get:

```text
Vinay
Rahul
Ravi
```

do not assume the database guarantees that order forever.

Without:

```sql
ORDER BY
```

the result order is generally not guaranteed.

If your application needs:

```text
Newest first
```

write:

```sql
ORDER BY created_at DESC;
```

---

# 🚨 Real Software Problem

Imagine an API:

```text
GET /orders
```

Product owner says:

> "Show the newest orders first."

Bad query:

```sql
SELECT *
FROM orders;
```

Correct intent:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC;
```

The requirement is not:

```text
"Probably return newest first."
```

It is:

```text
"Guarantee newest first."
```

That means the ordering must be expressed in SQL.

---

# 6️⃣ LIMIT

Suppose we only want the latest five orders.

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 5;
```

Conceptually:

```text
Sort
 ↓
Take first 5
```

This is useful for:

```text
Top products
Latest orders
Recent customers
Leaderboard
Preview results
```

---

# ⚠️ LIMIT Without ORDER BY

Avoid:

```sql
SELECT *
FROM orders
LIMIT 5;
```

if the requirement is:

> "Give me the five latest orders."

`LIMIT` alone does not define which five rows should be returned.

Correct:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 5;
```

---

# 7️⃣ DISTINCT

Suppose we want to know which cities our customers are from.

```sql
SELECT city
FROM customers;
```

Result:

```text
Hyderabad
Chennai
Hyderabad
Bangalore
Chennai
```

Use:

```sql
SELECT DISTINCT city
FROM customers;
```

Result:

```text
Hyderabad
Chennai
Bangalore
```

`DISTINCT` removes duplicate result rows.

---

# 🚨 Real Software Problem

Suppose an admin dashboard needs:

> "Show all cities where we have customers."

You don't want:

```text
Hyderabad
Hyderabad
Hyderabad
Chennai
Chennai
Bangalore
```

You want:

```sql
SELECT DISTINCT city
FROM customers;
```

---

# 8️⃣ Column Aliases

Suppose:

```sql
SELECT
    name,
    total_amount
FROM orders;
```

Maybe the application wants:

```text
order_total
```

Use:

```sql
SELECT
    id AS order_id,
    total_amount AS order_total
FROM orders;
```

Result columns:

```text
order_id | order_total
---------+------------
101      | 80000
102      | 125000
```

Aliases are especially useful when:

```text
Joining tables
Using expressions
Creating reports
Returning API-friendly column names
```

---

# 9️⃣ Expressions

SQL can calculate values.

Example:

```sql
SELECT
    name,
    price,
    price * 1.18 AS price_with_tax
FROM products;
```

If:

```text
price = 100
```

then:

```text
price_with_tax = 118
```

The database evaluates the expression for the result.

---

# 🧮 Arithmetic Operators

Common operators:

```text
+
-
*
/
%
```

Example:

```sql
SELECT
    price,
    price * 0.90 AS discounted_price
FROM products;
```

This calculates a 10% discount.

---

# 🚨 Real Software Problem

Marketing says:

> "Display all products with a 10% promotional discount."

You could calculate it in the application:

```text
Database
 ↓
price
 ↓
Application
 ↓
price * 0.90
```

Or SQL can calculate:

```sql
SELECT
    name,
    price,
    price * 0.90 AS discounted_price
FROM products;
```

Which approach is better depends on:

```text
Business ownership
Reuse
Consistency
Performance
Precision
Architecture
```

Don't blindly put every calculation into SQL.

---

# 🔟 String Operations

Example:

```sql
SELECT
    name,
    UPPER(name) AS uppercase_name
FROM products;
```

Possible result:

```text
iPhone → IPHONE
MacBook → MACBOOK
```

Another example:

```sql
SELECT
    name,
    LENGTH(name) AS name_length
FROM products;
```

Function names and exact behavior can vary across SQL dialects.

---

# 🧠 SQL Functions

Functions can operate on:

```text
Strings
Numbers
Dates
JSON
Aggregates
```

Examples:

```text
UPPER()
LOWER()
LENGTH()
ROUND()
CURRENT_TIMESTAMP
```

Later we'll study functions deeply.

---

# 1️⃣1️⃣ SQL Data Types

A database needs to know:

> What kind of value is this?

Examples:

```text
INTEGER
BIGINT
NUMERIC
TEXT
BOOLEAN
DATE
TIMESTAMP
```

Example:

```sql
CREATE TABLE employees (
    id BIGINT,
    name TEXT,
    age INTEGER,
    salary NUMERIC(12,2),
    active BOOLEAN,
    joining_date DATE
);
```

---

# 🧠 Why Data Types Matter

Imagine storing salary as:

```text
TEXT
```

instead of:

```text
NUMERIC
```

Then:

```text
'100000'
'90000'
'8000'
```

are strings.

Numeric calculations and ordering become more complicated and error-prone.

Correct data types communicate:

```text
Meaning
```

to both:

```text
Database
+
Developers
```

---

# 💰 NUMERIC vs Floating Point

For financial values, be careful with floating-point types.

For example:

```text
₹10.10
```

and:

```text
₹20.20
```

may require exact decimal arithmetic.

A common relational choice is:

```sql
NUMERIC(12,2)
```

for values requiring exact decimal semantics.

Exact recommendations depend on the database and financial requirements.

---

# 🧠 `CHAR`, `VARCHAR`, and `TEXT`

Different database systems treat these types differently.

Conceptually:

```text
CHAR
```

is fixed-length.

```text
VARCHAR
```

is variable-length with an optional maximum in some systems.

```text
TEXT
```

is variable-length text in systems that support it.

Don't choose based purely on:

```text
"VARCHAR is always faster."
```

or:

```text
"TEXT is always better."
```

Understand the database's actual implementation and the application's requirements.

---

# 1️⃣2️⃣ NULL

Now we reach one of the most important and misunderstood SQL concepts.

Suppose a customer has no city.

We store:

```text
city = NULL
```

NULL does **not** necessarily mean:

```text
Empty string
```

and does not mean:

```text
0
```

It represents a missing/unknown/inapplicable value depending on the business meaning.

---

# 🚨 Common Mistake

Wrong:

```sql
SELECT *
FROM customers
WHERE city = NULL;
```

This does not correctly test for NULL.

Use:

```sql
SELECT *
FROM customers
WHERE city IS NULL;
```

And:

```sql
SELECT *
FROM customers
WHERE city IS NOT NULL;
```

---

# 🧠 Why Does `= NULL` Not Work?

SQL uses three-valued logic.

A comparison involving NULL generally produces:

```text
UNKNOWN
```

rather than:

```text
TRUE
```

or:

```text
FALSE
```

So:

```text
city = NULL
```

doesn't mean:

```text
"Is city missing?"
```

Instead, NULL requires special predicates:

```text
IS NULL
IS NOT NULL
```

---

# 🔥 Three-Valued Logic

Traditional programming often thinks:

```text
TRUE
FALSE
```

SQL introduces:

```text
TRUE
FALSE
UNKNOWN
```

For example:

```text
5 = 5
```

returns:

```text
TRUE
```

But:

```text
5 = NULL
```

returns:

```text
UNKNOWN
```

And:

```text
5 <> NULL
```

also results in:

```text
UNKNOWN
```

This is one reason SQL filtering can surprise beginners.

---

# 🧠 Why Does WHERE Remove UNKNOWN?

A `WHERE` clause keeps rows whose search condition evaluates to:

```text
TRUE
```

Rows evaluating to:

```text
FALSE
```

or:

```text
UNKNOWN
```

are not returned.

This becomes extremely important when writing conditions involving nullable columns.

---

# 🔥 Real Production Bug

Suppose:

```text
customers

id | name  | city
---+-------+-----------
1  | Vinay | Hyderabad
2  | Rahul | NULL
3  | Ravi  | Chennai
```

Developer writes:

```sql
SELECT *
FROM customers
WHERE city <> 'Hyderabad';
```

They may expect:

```text
Rahul
Ravi
```

But Rahul's:

```text
city = NULL
```

means:

```text
NULL <> 'Hyderabad'
```

is:

```text
UNKNOWN
```

Therefore Rahul is not returned.

If the requirement is:

> "Customers whose city is not Hyderabad, including customers with no city."

you need to express that explicitly:

```sql
SELECT *
FROM customers
WHERE city <> 'Hyderabad'
   OR city IS NULL;
```

This is why NULL semantics matter.

---

# 1️⃣3️⃣ IN

Suppose we want customers from:

```text
Hyderabad
Chennai
Bangalore
```

Instead of:

```sql
WHERE city = 'Hyderabad'
   OR city = 'Chennai'
   OR city = 'Bangalore'
```

we can write:

```sql
SELECT *
FROM customers
WHERE city IN (
    'Hyderabad',
    'Chennai',
    'Bangalore'
);
```

This is more readable.

---

# 1️⃣4️⃣ BETWEEN

Suppose we need products between ₹5,000 and ₹50,000.

```sql
SELECT *
FROM products
WHERE price BETWEEN 5000 AND 50000;
```

`BETWEEN` is generally inclusive of both endpoints.

Conceptually:

```text
5000 <= price <= 50000
```

---

# ⚠️ Be Careful With Dates

Consider:

```sql
WHERE created_at BETWEEN
    '2026-01-01'
    AND
    '2026-01-31'
```

If `created_at` includes time, this may not represent the entire month in the way you intended.

A safer half-open range is often:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2026-02-01'
```

This pattern is extremely useful in production systems.

---

# 🧠 Why Half-Open Time Ranges Are Powerful

Use:

```text
[start, end)
```

meaning:

```text
>= start
AND
< end
```

For example:

```text
January 1 inclusive
February 1 exclusive
```

This avoids problems involving:

```text
23:59:59
Milliseconds
Microseconds
Precision
```

and works cleanly for timestamp ranges.

---

# 1️⃣5️⃣ LIKE

Suppose we want customers whose names start with:

```text
Vin
```

Use:

```sql
SELECT *
FROM customers
WHERE name LIKE 'Vin%';
```

`%` means:

```text
Zero or more characters
```

Example:

```text
Vin
Vinay
Vineet
Vinod
```

could match.

---

# `_` Wildcard

The underscore generally represents:

```text
Exactly one character
```

Example:

```sql
WHERE name LIKE 'Ra_i';
```

could match names such as:

```text
Ravi
Rani
```

depending on the exact string.

---

# 🚨 Real Production Problem — Search

Product owner says:

> "Search products by name."

Developer writes:

```sql
WHERE name LIKE '%phone%'
```

This can have very different performance characteristics from:

```sql
WHERE name LIKE 'phone%'
```

depending on:

```text
Database
Collation
Index Type
Pattern
Query Planner
```

This is an early example of why:

> **SQL correctness and SQL performance are different problems.**

---

# 1️⃣6️⃣ Comments

SQL supports comments.

Single-line:

```sql
-- Find active customers
SELECT *
FROM customers
WHERE city = 'Hyderabad';
```

Multi-line:

```sql
/*
    Find expensive products
    for premium customers
*/
SELECT *
FROM products
WHERE price > 100000;
```

Comments are useful for explaining:

```text
Business logic
Non-obvious conditions
Performance decisions
Temporary debugging
```

But don't use comments to compensate for unreadable SQL.

---

# 🧠 SQL Clause Order vs Logical Query Processing

Now we reach a very important concept.

You write SQL like:

```sql
SELECT
    name
FROM customers
WHERE city = 'Hyderabad'
ORDER BY name;
```

The syntax order is:

```text
SELECT
FROM
WHERE
ORDER BY
```

But conceptually, relational query processing is often explained as:

```text
FROM
 ↓
WHERE
 ↓
SELECT
 ↓
ORDER BY
```

This is a **logical processing model**, not necessarily the physical execution order used by the database engine.

---

# 🔥 Why Does This Matter?

Consider:

```sql
SELECT
    price * 0.90 AS discounted_price
FROM products
WHERE discounted_price < 50000;
```

This generally fails because the `SELECT` alias is not available to the `WHERE` clause at that logical stage.

Instead:

```sql
SELECT
    price * 0.90 AS discounted_price
FROM products
WHERE price * 0.90 < 50000;
```

Or use a subquery/CTE:

```sql
SELECT *
FROM (
    SELECT
        name,
        price * 0.90 AS discounted_price
    FROM products
) p
WHERE discounted_price < 50000;
```

This is an important consequence of SQL's logical query model.

---

# 🧠 Logical Query Processing

A simplified model is:

```text
FROM
 ↓
JOIN
 ↓
WHERE
 ↓
GROUP BY
 ↓
HAVING
 ↓
SELECT
 ↓
DISTINCT
 ↓
ORDER BY
 ↓
LIMIT
```

This is not the same thing as the physical execution plan.

The optimizer is free to rearrange operations when it can preserve the query's semantics.

This distinction is fundamental.

---

# 🔥 SQL Is Declarative Again

You write:

```sql
SELECT name
FROM customers
WHERE city = 'Hyderabad';
```

You don't say:

```text
Open customers
Read page 1
Read page 2
Compare city
Return row
```

Instead:

```text
WHAT?
```

The optimizer decides:

```text
HOW?
```

Maybe:

```text
Index Scan
```

Maybe:

```text
Sequential Scan
```

You don't normally control the physical algorithm directly through basic SQL syntax.

---

# 🧠 Query Correctness vs Query Performance

Suppose these both return the same result:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

and some equivalent rewritten form.

Correctness asks:

```text
Do they return the right data?
```

Performance asks:

```text
Which one requires less work?
```

These are different questions.

A senior SQL engineer always separates them.

---

# 🔥 Real Software Problem

Imagine:

```text
Orders
=
500 million rows
```

Requirement:

```text
Find customer 1001's orders.
```

Correct SQL:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

But the production question is:

```text
How fast?
```

Now we need to investigate:

```text
Index
Statistics
Execution Plan
Data Distribution
Storage
Cache
Concurrency
```

That is where the next levels of SQL mastery begin.

---

# 🧪 Hands-on Lab

## Lab 1 — Create Customers

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    city TEXT,
    created_at TIMESTAMP NOT NULL
);
```

Insert:

```sql
INSERT INTO customers
    (id, name, email, city, created_at)
VALUES
    (1, 'Vinay', 'vinay@example.com', 'Hyderabad', '2026-01-10'),
    (2, 'Rahul', 'rahul@example.com', 'Chennai', '2026-01-12'),
    (3, 'Ravi', 'ravi@example.com', 'Bangalore', '2026-01-15'),
    (4, 'Arun', 'arun@example.com', NULL, '2026-01-20');
```

---

# Lab 2 — Basic SELECT

```sql
SELECT *
FROM customers;
```

Then:

```sql
SELECT
    id,
    name,
    email
FROM customers;
```

Compare the result sets.

---

# Lab 3 — WHERE

```sql
SELECT *
FROM customers
WHERE city = 'Hyderabad';
```

---

# Lab 4 — ORDER BY

```sql
SELECT *
FROM customers
ORDER BY created_at DESC;
```

---

# Lab 5 — LIMIT

```sql
SELECT *
FROM customers
ORDER BY created_at DESC
LIMIT 2;
```

---

# Lab 6 — DISTINCT

```sql
SELECT DISTINCT city
FROM customers;
```

Observe the NULL value.

---

# Lab 7 — NULL

Run:

```sql
SELECT *
FROM customers
WHERE city = NULL;
```

Then:

```sql
SELECT *
FROM customers
WHERE city IS NULL;
```

Compare the results.

---

# Lab 8 — IN

```sql
SELECT *
FROM customers
WHERE city IN (
    'Hyderabad',
    'Chennai'
);
```

---

# Lab 9 — Expressions

```sql
SELECT
    name,
    price
FROM products;
```

Then:

```sql
SELECT
    name,
    price,
    price * 0.90 AS discounted_price
FROM products;
```

---

# Lab 10 — EXPLAIN

Run:

```sql
EXPLAIN
SELECT *
FROM customers
WHERE id = 1;
```

Then:

```sql
EXPLAIN
SELECT *
FROM customers
WHERE city = 'Hyderabad';
```

Ask:

```text
Why are the plans different?

Does an index exist?

How many rows are expected?

What happens as the table grows?
```

You don't need all the answers yet.

The goal is to start asking the right questions.

---

# 🎯 Interview Questions

## Beginner

### What does SELECT do?

It specifies the expressions/columns to return in the query result.

---

### What does FROM do?

It identifies the source relations from which the query retrieves data.

---

### What does WHERE do?

It filters rows based on a search condition.

---

### What does ORDER BY do?

It specifies the ordering of the query result.

---

### What does LIMIT do?

It restricts the number of rows returned by the query in systems that support it.

---

### What does DISTINCT do?

It removes duplicate result rows according to the selected expressions.

---

# 🧠 Intermediate Questions

### Is row order guaranteed without ORDER BY?

No.

If order matters, explicitly specify `ORDER BY`.

---

### What is NULL?

NULL represents the absence of a value, with its exact business interpretation depending on context.

It is not equivalent to:

```text
0
''
FALSE
```

---

### Why is `= NULL` incorrect?

Because comparisons involving NULL use SQL's three-valued logic and normally produce UNKNOWN.

Use:

```sql
IS NULL
```

instead.

---

### What is three-valued logic?

SQL conditions can evaluate to:

```text
TRUE
FALSE
UNKNOWN
```

instead of only TRUE/FALSE.

---

# 👑 Senior Questions

### Why should you avoid SELECT * in production queries?

Because it can:

```text
Return unnecessary data
Increase network traffic
Increase serialization cost
Increase memory usage
Reduce clarity
Create coupling to schema changes
```

The exact performance impact depends on the workload.

---

### Why can `LIMIT` without `ORDER BY` be dangerous?

Because the database is not required to return a deterministic subset unless ordering is specified.

---

### Why are half-open time ranges useful?

Instead of:

```sql
BETWEEN start AND end
```

use:

```sql
timestamp >= start
AND timestamp < end
```

when modeling time intervals.

This avoids ambiguity around the final instant of a period.

---

# 👑 20+ Year Experience Questions

## Question 1

A developer says:

> "The database returned the rows in insertion order yesterday."

Would you accept that as a guarantee?

**No.**

Unless the query specifies:

```sql
ORDER BY
```

the result order should not be relied upon.

---

# Question 2

A developer says:

> "We don't need a database constraint because our Java code validates the data."

Is that enough?

Think about:

```text
Multiple Services
Background Jobs
Admin Scripts
Data Imports
Concurrency
Race Conditions
```

Application validation and database constraints solve different parts of the integrity problem.

---

# Question 3

Why can this:

```sql
WHERE city <> 'Hyderabad'
```

exclude rows with:

```text
city = NULL
```

Because:

```text
NULL <> 'Hyderabad'
```

evaluates to:

```text
UNKNOWN
```

and `WHERE` retains only rows for which the condition evaluates to TRUE.

---

# Question 4

Why can the database execute:

```sql
SELECT *
FROM orders
WHERE customer_id = 1001;
```

without reading every row?

Potentially because it can use:

```text
Index
```

or another access strategy.

But SQL does not guarantee a particular access method.

---

# Question 5

Why might adding an index not improve:

```sql
SELECT *
FROM orders
WHERE status = 'PAID';
```

if:

```text
95%
```

of rows are:

```text
PAID
```

Because the predicate has low selectivity.

Reading most of the table through an index may not be cheaper than another access path.

---

# Question 6

Why can:

```sql
SELECT *
FROM orders
LIMIT 10;
```

return different rows at different times?

Because without:

```sql
ORDER BY
```

the database does not promise a deterministic order.

---

# Question 7

Why is SQL clause order:

```text
SELECT
FROM
WHERE
```

different from the common logical processing explanation:

```text
FROM
WHERE
SELECT
```

Because SQL syntax is designed for human expression, while the logical query model describes how relational operations conceptually build the result.

The physical optimizer may then transform the query further.

---

# 🔥 Senior SQL Mental Model

Don't memorize:

```text
SELECT
FROM
WHERE
ORDER BY
LIMIT
```

as syntax alone.

Think:

```text
FROM
 ↓
What data sources?
 ↓
WHERE
 ↓
Which rows?
 ↓
GROUP BY
 ↓
How should rows be grouped?
 ↓
HAVING
 ↓
Which groups?
 ↓
SELECT
 ↓
What should the result contain?
 ↓
ORDER BY
 ↓
How should the result be ordered?
 ↓
LIMIT
 ↓
How many rows should be returned?
```

Then remember:

```text
Logical Query Model
        ↓
Optimizer
        ↓
Physical Execution Plan
```

---

# 🧠 The 10 Questions You Should Ask While Writing SQL

Before submitting a production query, ask:

```text
1. What data do I actually need?

2. Which tables contain it?

3. Which rows should be returned?

4. Can NULL affect the condition?

5. Does ordering matter?

6. Do I really need every column?

7. Could the result contain duplicates?

8. How much data might this query process?

9. What happens when the table reaches 100 million rows?

10. What execution plan will the database choose?
```

This mindset is more valuable than memorizing hundreds of SQL functions.

---

# 📌 Chapter Summary

We learned the foundation of SQL.

## Basic Query

```sql
SELECT
    columns
FROM
    table
WHERE
    condition
ORDER BY
    column
LIMIT
    number;
```

---

## Filtering

```sql
WHERE
AND
OR
NOT
IN
BETWEEN
LIKE
```

---

## Result Formatting

```text
Aliases
Expressions
DISTINCT
```

---

## NULL

Remember:

```text
NULL ≠ 0
NULL ≠ ''
NULL ≠ FALSE
```

Use:

```sql
IS NULL
IS NOT NULL
```

and understand:

```text
TRUE
FALSE
UNKNOWN
```

---

## Data Types

Common categories include:

```text
Integer
Decimal
Text
Boolean
Date
Timestamp
```

Use data types that reflect the actual meaning and required semantics of the data.

---

# 🔥 Final Mental Model

```text
                    SQL
                     │
                     ▼
                  FROM
                     │
                     ▼
              Identify Sources
                     │
                     ▼
                  WHERE
                     │
                     ▼
               Filter Rows
                     │
                     ▼
                 SELECT
                     │
                     ▼
             Build Result
                     │
                     ▼
                DISTINCT
                     │
                     ▼
               Remove Duplicates
                     │
                     ▼
                ORDER BY
                     │
                     ▼
                Sort Result
                     │
                     ▼
                  LIMIT
                     │
                     ▼
              Return Rows
```

But remember:

```text
This is a logical model.
```

The actual database may execute the query very differently:

```text
SQL
 ↓
Parser
 ↓
Analyzer
 ↓
Optimizer
 ↓
Execution Plan
 ↓
Index / Scan / Join
 ↓
Buffer Cache
 ↓
Storage
```

The optimizer may reorder operations when it can preserve the semantics of the query.

---

# 🚀 Next Chapter

# Chapter 6 — Database Constraints: Making Invalid Data Impossible

We will solve real production problems such as:

```text
Two users register with the same email.

An order references a customer that doesn't exist.

A product gets a negative price.

Inventory becomes -500.

A required field becomes NULL.

Two payments use the same transaction ID.
```

We'll learn:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
DEFAULT
```

and, more importantly:

> **Why senior database engineers try to make invalid states impossible instead of relying entirely on application code to prevent them.**
