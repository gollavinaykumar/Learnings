# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 8 — SQL Filtering, Predicates & Expressions

> In the previous chapter, we went deep into:
>
> ```text
> NULL
> TRUE
> FALSE
> UNKNOWN
> ```
>
> We learned that SQL filtering is not simply:
>
> ```text
> if condition == true
> ```
>
> SQL has its own logical system.
>
> Now we will build on that foundation and understand how production applications actually construct filtering queries.
>
> Imagine an e-commerce API:
>
> ```text
> GET /products
> ```
>
> Users can search by:
>
> ```text
> category
> minimum price
> maximum price
> stock
> status
> product name
> created date
> ```
>
> The frontend may send:
>
> ```text
> category = Electronics
> minPrice = 50000
> maxPrice = 150000
> search = phone
> ```
>
> The backend must translate those requirements into SQL.
>
> This sounds simple.
>
> But production filtering introduces difficult problems:
>
> ```text
> NULL
> Optional filters
> Empty strings
> SQL injection
> LIKE patterns
> Date ranges
> Boolean logic
> IN lists
> EXISTS
> NOT EXISTS
> Dynamic SQL
> Parameter binding
> Index usage
> Pagination
> ```
>
> A senior SQL engineer must understand not only:
>
> ```text
> "How do I write this query?"
> ```
>
> but also:
>
> ```text
> "What exactly does this query mean?"
> ```
>
> and:
>
> ```text
> "What happens when the data reaches 100 million rows?"
> ```

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* SQL predicates
* Comparison predicates
* Logical predicates
* `IN`
* `NOT IN`
* `BETWEEN`
* `LIKE`
* `IS NULL`
* `EXISTS`
* `NOT EXISTS`
* Boolean expressions
* `CASE`
* Optional filters
* Dynamic filtering
* Parameterized queries
* SQL injection
* Date filtering
* Range filtering
* Search conditions
* Predicate precedence
* Predicate placement
* Sargability
* Basic filtering performance
* Real production query problems

---

# 🏗️ Our Application

Continue with our:

# E-Commerce Platform

We have:

```text
customers
products
orders
```

Products:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    sku TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price NUMERIC(12,2) NOT NULL,
    stock INTEGER NOT NULL DEFAULT 0,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL
);
```

Example:

```text
id | name       | category    | price  | stock | active
---+------------+-------------+--------+-------+--------
1  | iPhone     | Electronics | 80000  | 20    | true
2  | MacBook    | Electronics | 120000 | 10    | true
3  | Keyboard   | Accessories | 5000   | 100   | true
4  | Mouse      | Accessories | 2500   | 0     | false
5  | Monitor    | Electronics | 30000  | 25    | true
```

---

# 📖 What Is a Predicate?

A predicate is an expression that determines whether a condition is satisfied.

For example:

```sql
price > 50000
```

Conceptually:

```text
price > 50000
       ↓
TRUE / FALSE / UNKNOWN
```

Predicates are used by:

```text
WHERE
JOIN ... ON
HAVING
CASE
CHECK
```

---

# 🧠 Think of a Predicate as a Question

```sql
price > 50000
```

means:

> Is the product price greater than 50,000?

```sql
stock = 0
```

means:

> Is the stock exactly zero?

```sql
active = TRUE
```

means:

> Is this product active?

This mental model makes complex SQL easier to reason about.

---

# 1️⃣ Comparison Predicates

Basic operators:

```text
=
<>
!=
>
<
>=
<=
```

Example:

```sql
SELECT *
FROM products
WHERE price > 50000;
```

---

# Multiple Conditions

```sql
SELECT *
FROM products
WHERE price > 50000
  AND stock > 0;
```

This means:

```text
Price > 50000
        AND
Stock > 0
```

Both must be TRUE.

---

# 🔥 Real Software Requirement

Product team says:

> "Show expensive products that are currently available."

Translate:

```text
Expensive
+
Available
```

into:

```sql
SELECT *
FROM products
WHERE price > 50000
  AND stock > 0;
```

Notice how we first convert the business requirement into logical predicates.

---

# 2️⃣ AND

Example:

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
  AND price > 50000
  AND stock > 0;
```

Conceptually:

```text
                 Product
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
     Electronics   >50000    >0
          │         │         │
          └─────────┼─────────┘
                    ▼
                  TRUE
```

Every condition must be TRUE.

---

# 3️⃣ OR

Suppose users want:

```text
Electronics OR Accessories
```

Use:

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
   OR category = 'Accessories';
```

At least one condition must be TRUE.

---

# 🚨 OR Precedence Problem

Consider:

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
   OR category = 'Accessories'
  AND stock > 0;
```

SQL generally evaluates:

```text
AND
```

before:

```text
OR
```

So this means:

```text
Electronics
OR
(
Accessories AND stock > 0
)
```

It does NOT mean:

```text
(
Electronics OR Accessories
)
AND stock > 0
```

If the second meaning is intended:

```sql
SELECT *
FROM products
WHERE
    (
        category = 'Electronics'
        OR category = 'Accessories'
    )
    AND stock > 0;
```

---

# 👑 Senior Rule

When business logic contains multiple `AND` and `OR` conditions:

> **Use parentheses to make the intended logic explicit.**

Don't depend on readers remembering operator precedence.

---

# 4️⃣ IN

Suppose:

```text
Electronics
Accessories
Books
```

are allowed categories.

Instead of:

```sql
WHERE category = 'Electronics'
   OR category = 'Accessories'
   OR category = 'Books'
```

use:

```sql
WHERE category IN (
    'Electronics',
    'Accessories',
    'Books'
);
```

Cleaner.

---

# 🔥 Real API Problem

Frontend sends:

```text
categories:
[
    "Electronics",
    "Accessories",
    "Books"
]
```

Backend can construct a parameterized `IN` predicate.

Conceptually:

```sql
WHERE category IN (?, ?, ?)
```

with parameters:

```text
Electronics
Accessories
Books
```

Do not concatenate raw user input into SQL.

---

# 5️⃣ NOT IN

Example:

```sql
SELECT *
FROM products
WHERE category NOT IN (
    'Accessories',
    'Books'
);
```

This means:

```text
Category is not Accessories
AND
Category is not Books
```

But remember the previous chapter:

# ⚠️ NULL

If the compared value or list/subquery contains NULL, `NOT IN` can produce UNKNOWN and unexpected results.

This is one reason `NOT EXISTS` is often safer for exclusion based on another relation.

---

# 6️⃣ BETWEEN

Example:

```sql
SELECT *
FROM products
WHERE price BETWEEN 10000 AND 50000;
```

Conceptually:

```text
10000 <= price <= 50000
```

The endpoints are generally inclusive.

---

# ⚠️ Date BETWEEN Problem

Suppose:

```text
created_at
```

is a timestamp.

Developer writes:

```sql
SELECT *
FROM orders
WHERE created_at BETWEEN
    '2026-08-01'
    AND
    '2026-08-31';
```

They may think:

```text
Entire August
```

But:

```text
'2026-08-31'
```

can represent midnight at the start of that date depending on type conversion/database behavior.

Records later on August 31 may not be included.

---

# 👑 Better Date Range Pattern

For a full August period:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-01'
  AND created_at <  '2026-09-01';
```

Think:

```text
August 1
   │
   ├── Included
   │
   │
August 31
   │
   ├── Included
   │
September 1
   │
   └── Excluded
```

This is called a:

```text
Half-Open Interval
```

or:

```text
[start, end)
```

---

# 🧠 Why Senior Engineers Like Half-Open Ranges

They avoid worrying about:

```text
23:59:59
milliseconds
microseconds
timestamp precision
```

Instead:

```text
>= start
<
end
```

This is especially useful for:

```text
Reports
Billing periods
Daily jobs
Monthly analytics
Time-series queries
Pagination by timestamp
```

---

# 7️⃣ LIKE

Search:

```sql
SELECT *
FROM products
WHERE name LIKE 'Mac%';
```

`%` represents:

```text
Zero or more characters
```

Possible matches:

```text
Mac
MacBook
MacBook Pro
```

---

# `%` at the Beginning

```sql
WHERE name LIKE '%phone%'
```

means:

```text
Anything
+
phone
+
Anything
```

Potential matches:

```text
iPhone
Smartphone
Phone Case
```

depending on case sensitivity and database collation.

---

# `_` Wildcard

```sql
WHERE name LIKE 'M_use';
```

The `_` generally represents one character.

Possible:

```text
Mouse
Mause
```

depending on actual data.

---

# 🔥 Real Search API

User enters:

```text
phone
```

Application executes:

```sql
WHERE name LIKE '%phone%'
```

This may be fine for small datasets.

But suppose:

```text
products = 100 million
```

Now ask:

```text
Can a normal B-tree index efficiently support this pattern?
```

Often:

```text
LIKE 'phone%'
```

is much easier for a standard B-tree index to optimize than:

```text
LIKE '%phone%'
```

But exact behavior depends on:

```text
Database
Collation
Operator class
Index type
Query pattern
```

For large-scale search, you may need:

```text
Full-text search
Search engine
Specialized indexes
```

---

# 🧠 Correctness vs Performance

Both queries may be logically correct:

```sql
WHERE name LIKE 'phone%'
```

and:

```sql
WHERE name LIKE '%phone%'
```

But they have different search semantics and potentially very different performance.

Senior engineers ask both:

```text
Does it return the right rows?
```

and:

```text
Can it scale?
```

---

# 8️⃣ Boolean Predicates

Suppose:

```sql
active BOOLEAN NOT NULL
```

You can write:

```sql
SELECT *
FROM products
WHERE active = TRUE;
```

In databases that support boolean predicates directly, you may also write:

```sql
SELECT *
FROM products
WHERE active;
```

For false:

```sql
WHERE NOT active
```

or:

```sql
WHERE active = FALSE
```

Exact boolean syntax varies across SQL dialects.

---

# 9️⃣ IS NULL

From the previous chapter:

```sql
SELECT *
FROM products
WHERE discontinued_at IS NULL;
```

Correct.

Not:

```sql
WHERE discontinued_at = NULL;
```

---

# 🔟 EXISTS

Now we move into an important relational predicate.

Suppose:

```text
customers
```

and:

```text
orders
```

Requirement:

> "Find customers who have at least one order."

One approach:

```sql
SELECT DISTINCT c.id, c.name
FROM customers c
JOIN orders o
    ON o.customer_id = c.id;
```

Another:

```sql
SELECT
    c.id,
    c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

The second query expresses:

> Does at least one matching order exist?

---

# 🧠 EXISTS Is About Existence

Think:

```text
Customer
   │
   ▼
Search Orders
   │
   ├── Found one
   │      ↓
   │    TRUE
   │
   └── Found none
          ↓
        FALSE
```

The database does not need to return the order rows as part of the `EXISTS` predicate.

---

# 🔥 Real Business Problem

Requirement:

> "Send marketing emails only to customers who have purchased something."

Query:

```sql
SELECT
    c.id,
    c.email
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

This directly represents the business condition.

---

# 1️⃣1️⃣ NOT EXISTS

Requirement:

> "Find customers who have never placed an order."

Use:

```sql
SELECT
    c.id,
    c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

This means:

```text
For each customer:

Does a matching order exist?

YES
 ↓
Exclude

NO
 ↓
Keep
```

---

# 🔥 NOT EXISTS vs NOT IN

Suppose:

```text
orders.customer_id
```

can contain NULL.

This can be dangerous:

```sql
SELECT *
FROM customers
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
);
```

because a NULL in the subquery can affect the `NOT IN` predicate.

Instead:

```sql
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

The latter directly asks:

> Is there no matching order?

---

# 1️⃣2️⃣ EXISTS and SELECT 1

You often see:

```sql
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
)
```

Why:

```text
SELECT 1
```

?

Because the actual selected value is irrelevant.

`EXISTS` is testing whether the subquery produces at least one row.

You could see other expressions used, but:

```sql
SELECT 1
```

clearly communicates the intent.

---

# 1️⃣3️⃣ CASE

SQL can make conditional expressions.

Example:

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 100000 THEN 'PREMIUM'
        WHEN price >= 50000 THEN 'HIGH'
        ELSE 'STANDARD'
    END AS price_category
FROM products;
```

Result:

```text
MacBook   120000   PREMIUM
iPhone     80000   HIGH
Keyboard    5000   STANDARD
```

---

# 🧠 CASE Is an Expression

`CASE` can be used to calculate a result.

Conceptually:

```text
IF condition
    THEN value
ELSE
    value
```

But SQL's `CASE` supports multiple branches and is part of SQL's expression system.

---

# 🔥 Real Reporting Problem

Business wants:

```text
Revenue > 100000 → VIP
Revenue > 50000  → GOLD
Otherwise        → STANDARD
```

Instead of implementing every presentation rule in application code, SQL can derive the category:

```sql
CASE
    WHEN revenue > 100000 THEN 'VIP'
    WHEN revenue > 50000 THEN 'GOLD'
    ELSE 'STANDARD'
END
```

Whether this transformation belongs in SQL or application code depends on the architecture and ownership of the business logic.

---

# 1️⃣4️⃣ Optional Filters

This is one of the most common production query problems.

API:

```text
GET /products
```

Possible parameters:

```text
category
minPrice
maxPrice
active
```

But every parameter is optional.

For example:

```text
GET /products?category=Electronics
```

or:

```text
GET /products?minPrice=50000
```

or:

```text
GET /products?category=Electronics&minPrice=50000&maxPrice=100000
```

How do we write SQL?

---

# ❌ Bad Approach — String Concatenation

Developer builds:

```text
"SELECT * FROM products WHERE category = '" + userInput + "'"
```

This is dangerous.

---

# 💥 SQL Injection

Suppose attacker enters:

```text
' OR 1=1 --
```

The generated SQL could become logically equivalent to:

```sql
SELECT *
FROM products
WHERE category = ''
   OR 1 = 1
   -- ...
```

Now the attacker may bypass the intended filter.

This is:

# SQL Injection

One of the most important application security problems involving databases.

---

# 🛡️ Correct Approach — Parameterized Queries

Use:

```sql
SELECT *
FROM products
WHERE category = ?;
```

Then bind:

```text
Electronics
```

through the database driver's parameter-binding mechanism.

The application should not construct SQL by directly inserting untrusted values into the SQL string.

---

# 👑 Senior Rule

> **Never treat user input as SQL syntax. Treat it as data.**

Use:

```text
Parameterized Queries
Prepared Statements
Bound Parameters
```

according to your database driver/framework.

---

# 🧠 Dynamic Filters

Suppose the API sends:

```text
category = Electronics
minPrice = 50000
maxPrice = 100000
```

The application can construct a query with only the requested predicates:

```sql
SELECT *
FROM products
WHERE category = ?
  AND price >= ?
  AND price <= ?;
```

Parameters:

```text
Electronics
50000
100000
```

This is usually clearer than trying to write one giant query handling every possible optional parameter.

---

# ⚠️ The "OR Parameter IS NULL" Pattern

You may see:

```sql
SELECT *
FROM products
WHERE
    (? IS NULL OR category = ?);
```

The idea is:

```text
Parameter NULL
    ↓
Ignore filter

Parameter provided
    ↓
Apply filter
```

This can be convenient.

But at scale, such patterns can complicate optimization and index usage depending on the database, parameter values, and plan selection.

Don't assume:

> "One query is always better than dynamically constructing the appropriate predicates."

Measure.

---

# 🔥 Real Production Problem

Imagine:

```text
Products
=
200 million rows
```

API supports:

```text
category
price
stock
active
created_at
```

A developer creates one huge query:

```sql
WHERE
    (? IS NULL OR category = ?)
AND
    (? IS NULL OR price >= ?)
AND
    (? IS NULL OR price <= ?)
AND
    (? IS NULL OR active = ?)
```

It may work correctly.

But now ask:

```text
Will the optimizer choose good plans?

Will prepared-plan behavior matter?

Will indexes be used effectively?

Will different parameter combinations need different plans?
```

This is where senior SQL engineering begins.

---

# 1️⃣5️⃣ Predicate Pushdown

Suppose:

```sql
SELECT *
FROM orders o
JOIN customers c
    ON c.id = o.customer_id
WHERE c.city = 'Hyderabad';
```

Conceptually, the database may be able to apply the filter earlier:

```text
Customers
    ↓
city = Hyderabad
    ↓
Smaller set
    ↓
Join
```

rather than joining everything first.

This is broadly related to:

# Predicate Pushdown

The optimizer may move filtering closer to the data source when it can do so without changing query semantics.

---

# 🧠 Important

You write:

```sql
WHERE c.city = 'Hyderabad'
```

You don't normally manually tell the optimizer:

```text
"Filter customers before joining."
```

The optimizer determines an efficient execution strategy.

---

# 1️⃣6️⃣ SARGability

Now an important performance concept.

Consider:

```sql
WHERE customer_id = 1001
```

This is usually friendly to an index on:

```text
customer_id
```

Now consider:

```sql
WHERE LOWER(email) = 'vinay@example.com'
```

If the only index is:

```text
INDEX(email)
```

the database may not be able to use that ordinary index as efficiently because the predicate applies a function to the indexed column.

This is related to:

# SARGability

A predicate is commonly called sargable when the database can efficiently use an available access path such as an index to restrict the search.

Exact behavior depends heavily on the database and index design.

---

# 🚨 Real Performance Problem

Suppose:

```text
users = 100 million
```

and:

```text
email
```

has an index.

Query:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

can potentially use the index directly.

Now:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'vinay@example.com';
```

The database may need a different strategy.

Possible solutions include:

```text
Functional / Expression Index
Normalized Stored Value
Case-insensitive Data Type
Database-specific Indexing
```

The correct solution depends on the database.

---

# 1️⃣7️⃣ Functions on Indexed Columns

Another example:

```sql
WHERE DATE(created_at) = '2026-08-28'
```

This transforms:

```text
created_at
```

before comparison.

A range predicate is often more index-friendly:

```sql
WHERE created_at >= '2026-08-28'
  AND created_at <  '2026-08-29'
```

The second query expresses the time range directly.

---

# 🧠 Why This Matters

These two queries can be logically equivalent:

```sql
DATE(created_at) = '2026-08-28'
```

and:

```sql
created_at >= '2026-08-28'
AND created_at < '2026-08-29'
```

But their performance can be dramatically different on large tables.

Again:

```text
Correctness
    ≠
Performance
```

---

# 1️⃣8️⃣ Filtering NULL Correctly

Suppose:

```text
discount
```

can be NULL.

This:

```sql
WHERE discount > 10
```

does not include:

```text
discount = NULL
```

because:

```text
NULL > 10
```

is:

```text
UNKNOWN
```

If requirement is:

> Discount greater than 10 OR no discount information.

Use:

```sql
WHERE discount > 10
   OR discount IS NULL;
```

---

# 1️⃣9️⃣ Filtering With COALESCE

You may see:

```sql
WHERE COALESCE(discount, 0) > 10;
```

This treats:

```text
NULL
```

as:

```text
0
```

This may be correct if the business rule is:

> Missing discount means zero discount.

But it can affect index usage because the expression transforms the column.

Alternative:

```sql
WHERE discount > 10;
```

already excludes NULL naturally.

If you want:

```text
NULL treated as 0
```

you must explicitly decide whether the semantic and performance consequences are acceptable.

---

# 🔥 2️⃣0️⃣ Complex Predicate Example

Requirement:

> Find active Electronics products priced between ₹50,000 and ₹150,000, with stock available, whose name contains "phone".

SQL:

```sql
SELECT
    id,
    name,
    price,
    stock
FROM products
WHERE active = TRUE
  AND category = 'Electronics'
  AND price >= 50000
  AND price < 150000
  AND stock > 0
  AND name LIKE '%phone%';
```

Now don't stop at:

> "The query works."

Ask:

```text
What indexes exist?

Which predicate is selective?

How many rows match category?

How many match price?

How expensive is '%phone%'?

Can the database use indexes?

What does EXPLAIN show?
```

That is the senior mindset.

---

# 🧠 Predicate Selectivity

Suppose:

```text
products = 100 million
```

Predicate:

```sql
active = TRUE
```

matches:

```text
95 million
```

Very low selectivity.

But:

```sql
sku = 'IPHONE-15-PRO-256'
```

matches:

```text
1 row
```

Very high selectivity.

Conceptually:

```text
High Selectivity
      ↓
Small fraction of rows match

Low Selectivity
      ↓
Large fraction of rows match
```

Selectivity can strongly influence the optimizer's access-path choices.

---

# 🔥 Why Selectivity Matters

Suppose:

```text
100 million rows
```

Query:

```sql
WHERE active = TRUE
```

If:

```text
95 million
```

rows match, an index may not necessarily be the best strategy.

A sequential scan may be cheaper.

But:

```sql
WHERE sku = 'XYZ'
```

matching one row can make an index extremely attractive.

---

# 🧠 Never Say

> "Indexes are always faster."

Instead say:

> "An index can be beneficial when its access path is cheaper than the alternatives for the specific data distribution and query."

This distinction separates beginner thinking from senior database thinking.

---

# 2️⃣1️⃣ Query Parameters and Types

Suppose:

```sql
SELECT *
FROM products
WHERE price >= ?;
```

Parameter:

```text
50000
```

The application should bind it using an appropriate numeric type.

Avoid sending:

```text
"50000"
```

as arbitrary text when the database/application driver can bind the correct numeric type.

Correct parameter typing helps maintain:

```text
Correctness
Predictability
Performance
```

The exact behavior depends on the driver and database.

---

# 2️⃣2️⃣ Dynamic IN Lists

Suppose API receives:

```text
ids = [10, 20, 30, 40]
```

Don't generate:

```sql
WHERE id IN (10,20,30,40)
```

by blindly concatenating raw input.

Use parameter placeholders appropriate for your driver:

```sql
WHERE id IN (?, ?, ?, ?)
```

and bind:

```text
10
20
30
40
```

Some database drivers/frameworks provide higher-level helpers for expanding parameter lists.

---

# 🧠 Security Rule

SQL syntax:

```text
SELECT ...
WHERE id IN (?, ?, ?)
```

Data:

```text
10
20
30
```

Keep:

```text
SQL syntax
```

separate from:

```text
User data
```

---

# 🧪 Hands-on Lab

## Lab 1 — Comparison

```sql
SELECT *
FROM products
WHERE price > 50000;
```

---

# Lab 2 — Multiple Predicates

```sql
SELECT *
FROM products
WHERE price > 50000
  AND stock > 0;
```

---

# Lab 3 — OR + Parentheses

Run:

```sql
SELECT *
FROM products
WHERE category = 'Electronics'
   OR category = 'Accessories'
  AND stock > 0;
```

Then:

```sql
SELECT *
FROM products
WHERE
    (
        category = 'Electronics'
        OR category = 'Accessories'
    )
    AND stock > 0;
```

Compare.

---

# Lab 4 — IN

```sql
SELECT *
FROM products
WHERE category IN (
    'Electronics',
    'Accessories'
);
```

---

# Lab 5 — BETWEEN

```sql
SELECT *
FROM products
WHERE price BETWEEN 5000 AND 50000;
```

---

# Lab 6 — Date Range

Create orders with timestamps around midnight:

```text
2026-08-31 00:00:00
2026-08-31 12:00:00
2026-08-31 23:59:59
2026-09-01 00:00:00
```

Compare:

```sql
SELECT *
FROM orders
WHERE created_at BETWEEN
    '2026-08-01'
    AND
    '2026-08-31';
```

with:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-01'
  AND created_at <  '2026-09-01';
```

Understand why the second pattern is usually safer for timestamp periods.

---

# Lab 7 — LIKE

```sql
SELECT *
FROM products
WHERE name LIKE 'Mac%';
```

Then:

```sql
SELECT *
FROM products
WHERE name LIKE '%phone%';
```

---

# Lab 8 — EXISTS

```sql
SELECT
    c.id,
    c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

---

# Lab 9 — NOT EXISTS

```sql
SELECT
    c.id,
    c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

---

# Lab 10 — NULL

```sql
SELECT *
FROM products
WHERE discontinued_at IS NULL;
```

Then deliberately try:

```sql
SELECT *
FROM products
WHERE discontinued_at = NULL;
```

Understand the difference.

---

# Lab 11 — EXPLAIN

Run:

```sql
EXPLAIN
SELECT *
FROM products
WHERE sku = 'IPHONE-001';
```

Then:

```sql
EXPLAIN
SELECT *
FROM products
WHERE LOWER(sku) = 'iphone-001';
```

If your database supports expression indexes, later experiment with one.

The goal is not to memorize a particular plan.

The goal is to ask:

```text
What access path did the optimizer choose?
Why?
```

---

# 🎯 Interview Questions

## Beginner

### What is a predicate?

An expression that evaluates a condition and can produce TRUE, FALSE, or UNKNOWN.

---

### What does IN do?

It tests whether a value matches one of a specified set of values.

---

### What does BETWEEN do?

It tests whether a value falls within a range, generally including both endpoints.

---

### What does LIKE do?

It performs pattern matching using wildcard characters such as `%` and `_`.

---

### What does EXISTS do?

It tests whether a subquery produces at least one row.

---

### What does NOT EXISTS do?

It tests whether a subquery produces no matching rows.

---

# 🧠 Intermediate Questions

### Why is NOT IN dangerous when NULL exists?

Because NULL can cause the predicate to evaluate to UNKNOWN.

---

### Why use NOT EXISTS?

It directly expresses the absence of a matching row and avoids the classic `NOT IN` + NULL problem.

---

### Why use parentheses with AND and OR?

To make logical grouping explicit and prevent incorrect interpretation.

---

### Why are timestamp ranges often written as:

```sql
>= start
AND < end
```

Because half-open intervals avoid ambiguity around the final instant and timestamp precision.

---

### What is SQL injection?

An attack where untrusted input changes the intended SQL syntax/logic because the application constructs SQL unsafely.

---

### How do you prevent SQL injection?

Use:

```text
Parameterized Queries
Prepared Statements
Bound Parameters
```

rather than concatenating untrusted input into SQL.

---

# 👑 Senior Questions

### What is sargability?

It refers to whether a predicate can be transformed into an efficient search condition that allows the database to use an access path such as an index.

---

### Why can this be problematic?

```sql
WHERE LOWER(email) = 'x@example.com'
```

because applying a function to the column may prevent use of a normal index on `email`.

Depending on the database, an expression/functional index or another design may help.

---

### Why can:

```sql
WHERE DATE(created_at) = '2026-08-28'
```

be slower than:

```sql
WHERE created_at >= '2026-08-28'
  AND created_at <  '2026-08-29'
```

Because the range predicate can allow the database to search the timestamp index directly, whereas transforming every candidate timestamp can interfere with normal index access.

---

### Are indexes always beneficial?

No.

Index usefulness depends on:

```text
Data Distribution
Selectivity
Query Pattern
Table Size
Write Cost
Storage
Optimizer
Access Path
```

---

# 👑 20+ Year Experience Questions

## Question 1

Your API has:

```text
category
minPrice
maxPrice
active
search
```

all optional.

Would you write one enormous SQL query with:

```sql
(? IS NULL OR ...)
```

for everything?

Not automatically.

Possible approaches include:

```text
Dynamic predicate construction
Static query
Query builder
Prepared statements
Multiple specialized queries
```

Choose based on:

```text
Correctness
Maintainability
Plan behavior
Performance
Security
```

and measure production-like workloads.

---

# Question 2

Why is:

```sql
WHERE active = TRUE
```

not necessarily a good candidate for an index?

Suppose:

```text
100 million rows
```

and:

```text
95 million active
```

The predicate matches almost the entire table.

An index may not provide enough benefit.

The optimizer may prefer another access path.

---

# Question 3

Why is:

```sql
WHERE sku = ?
```

usually much more index-friendly?

Because SKU may be highly selective:

```text
100 million rows
      ↓
1 matching row
```

An index can dramatically reduce the amount of data that needs to be examined.

---

# Question 4

Why is:

```sql
WHERE name LIKE '%phone%'
```

different from:

```sql
WHERE name LIKE 'phone%'
```

Because the leading wildcard makes it harder for many conventional B-tree indexes to navigate directly to the matching range.

The exact optimization depends on database/index configuration.

---

# Question 5

Why should you not automatically replace:

```sql
WHERE discount > 10
```

with:

```sql
WHERE COALESCE(discount, 0) > 10
```

Because:

```text
NULL
```

may have a different business meaning from:

```text
0
```

and applying a function can also change index-access opportunities.

---

# Question 6

Why can this query:

```sql
SELECT *
FROM customers
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
);
```

return unexpected results?

Because if the subquery returns NULL, the `NOT IN` predicate can evaluate to UNKNOWN.

A clearer alternative is often:

```sql
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

---

# Question 7

Which is more important?

```text
Query works
```

or:

```text
Query is fast
```

Neither alone.

Production SQL must satisfy:

```text
Correctness
+
Performance
+
Security
+
Maintainability
```

---

# 🔥 Senior Filtering Mental Model

When writing a WHERE clause, think:

```text
Business Requirement
        ↓
Logical Conditions
        ↓
Predicates
        ↓
NULL Semantics
        ↓
Boolean Logic
        ↓
SQL
        ↓
Optimizer
        ↓
Execution Plan
        ↓
Actual Performance
```

Don't jump directly from:

```text
"User wants search"
```

to:

```sql
WHERE ...
```

First understand the semantics.

---

# 🧠 Production Query Checklist

Before deploying a filtering query, ask:

```text
1. What exactly should match?

2. What should NOT match?

3. Can any involved column be NULL?

4. Are AND/OR conditions grouped correctly?

5. Are date boundaries correct?

6. Is BETWEEN appropriate?

7. Could NOT IN encounter NULL?

8. Would EXISTS / NOT EXISTS express the requirement better?

9. Is user input parameterized?

10. Are functions applied to indexed columns?

11. How selective are the predicates?

12. What indexes exist?

13. What happens with 100 million rows?

14. What does EXPLAIN show?

15. Does the query remain correct for edge cases?
```

---

# 🔥 Real Production Scenario

Requirement:

> "Search active electronics products priced from ₹50,000 to ₹1,50,000, with stock available, created during August 2026, and whose name contains 'phone'."

A reasonable query:

```sql
SELECT
    id,
    name,
    price,
    stock
FROM products
WHERE active = TRUE
  AND category = 'Electronics'
  AND stock > 0
  AND price >= 50000
  AND price < 150000
  AND created_at >= '2026-08-01'
  AND created_at <  '2026-09-01'
  AND name LIKE '%phone%';
```

Now the senior engineer asks:

```text
Is category selective?

Is active selective?

Is stock selective?

Is price indexed?

Is created_at indexed?

Can name search use an appropriate search index?

How many products exist?

What does EXPLAIN say?

What happens with 500 million products?
```

That is the transition from:

```text
SQL Developer
```

to:

```text
Database Engineer
```

---

# 📌 Chapter Summary

SQL filtering is based on predicates.

Core predicates:

```text
=
<>
>
<
>=
<=
IN
NOT IN
BETWEEN
LIKE
IS NULL
IS NOT NULL
EXISTS
NOT EXISTS
```

Logical operators:

```text
AND
OR
NOT
```

Important concepts:

```text
Three-Valued Logic
Predicate Precedence
Date Ranges
SARGability
Selectivity
SQL Injection
Parameterized Queries
```

---

# 🔥 Critical Rules

```text
Rule 1
------
Use parentheses when mixing AND and OR.


Rule 2
------
Be careful with NOT IN + NULL.


Rule 3
------
Prefer NOT EXISTS when expressing
"no matching row exists."


Rule 4
------
Use half-open timestamp ranges:

>= start
< end


Rule 5
------
Never concatenate untrusted input into SQL.


Rule 6
------
Use parameterized queries.


Rule 7
------
Don't blindly apply functions to indexed columns.


Rule 8
------
Indexes are not automatically beneficial.


Rule 9
------
Query correctness comes before optimization.


Rule 10
------
Measure performance using execution plans
and realistic data.
```

---

# 🧠 Final Mental Model

```text
                  USER REQUEST
                       │
                       ▼
              "Find matching data"
                       │
                       ▼
                BUSINESS RULE
                       │
                       ▼
                  PREDICATES
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      WHERE          EXISTS         NOT EXISTS
        │
        ▼
   AND / OR / NOT
        │
        ▼
   NULL Semantics
        │
        ▼
      SQL Query
        │
        ▼
     Optimizer
        │
        ▼
  Execution Plan
        │
        ├── Index
        ├── Scan
        ├── Join
        └── Other Operations
        │
        ▼
     Result
```

The important progression is:

```text
Beginner
↓
"I know WHERE."

Intermediate
↓
"I know predicates."

Advanced
↓
"I understand NULL and Boolean logic."

Senior
↓
"I understand how predicates affect execution."

20+ Years
↓
"I design queries and schemas so that
correctness, security, maintainability,
and performance remain predictable as
the system and data scale."
```

---

# 🚀 Next Chapter

# Chapter 9 — SQL Functions, Expressions & Conditional Logic

We will go deeper into SQL's expression system:

```text
String Functions
Numeric Functions
Date/Time Functions
COALESCE
NULLIF
CASE
CAST
CONVERT
Type Conversion
Conditional Expressions
Computed Columns
```

Then we'll solve real software problems such as:

```text
Normalize customer names
Format phone numbers
Calculate order totals
Calculate discounts
Handle missing values
Calculate age
Build reports
Convert data types
Format timestamps
Create business classifications
```

And we'll answer an important senior-level question:

> **When should a calculation happen inside SQL, inside the application, or be persisted as data?**
