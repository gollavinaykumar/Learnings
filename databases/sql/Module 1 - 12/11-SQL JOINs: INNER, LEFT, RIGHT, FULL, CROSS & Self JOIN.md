# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 11 — SQL JOINs: INNER, LEFT, RIGHT, FULL, CROSS & SELF JOIN

> In the previous chapter, we learned aggregation:
>
> ```text
> Many Rows
>     ↓
> GROUP BY
>     ↓
> COUNT / SUM / AVG
>     ↓
> Business Metrics
> ```
>
> But there is a problem.
>
> Real databases rarely keep everything in one table.
>
> An e-commerce system might look like:
>
> ```text
> customers
>     ↓
> orders
>     ↓
> order_items
>     ↓
> products
> ```
>
> A banking system:
>
> ```text
> customers
>     ↓
> accounts
>     ↓
> transactions
> ```
>
> A logistics system:
>
> ```text
> customers
>     ↓
> shipments
>     ↓
> tracking_events
> ```
>
> A company may have hundreds or thousands of tables.
>
> So the real question becomes:
>
> # How do we combine information stored in different tables?
>
> The answer is:
>
> ```text
> JOIN
> ```
>
> JOIN is one of the most important concepts in relational databases.
>
> But learning:
>
> ```sql
> SELECT *
> FROM orders
> JOIN customers ...
> ```
>
> is not enough.
>
> A 20+ year SQL engineer must understand:
>
> ```text
> Relationships
> Cardinality
> Row Multiplication
> NULL introduction
> Join predicates
> Data grain
> Indexes
> Query plans
> Optimizer behavior
> ```
>
> The biggest JOIN bugs are especially dangerous because:
>
> ```text
> Query runs successfully
>       ↓
> No SQL error
>       ↓
> Result looks reasonable
>       ↓
> Business numbers are wrong
> ```
>
> This chapter is about preventing those bugs.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* Why JOINs exist
* Primary key / foreign key relationships
* INNER JOIN
* LEFT JOIN
* RIGHT JOIN
* FULL OUTER JOIN
* CROSS JOIN
* SELF JOIN
* `ON` vs `WHERE`
* JOIN conditions
* Composite JOINs
* JOINing multiple tables
* One-to-one relationships
* One-to-many relationships
* Many-to-many relationships
* JOIN cardinality
* Row multiplication
* NULL behavior
* JOIN + aggregation
* JOIN + `GROUP BY`
* JOIN + `EXISTS`
* Semi-joins
* Anti-joins
* JOIN performance
* JOIN indexes
* Hash Join
* Nested Loop Join
* Merge Join
* Query execution plans
* Production JOIN debugging

---

# 📖 Why JOINs Exist

Imagine we have:

```text
customers

id | name
---+---------
101 | Vinay
102 | Ravi
103 | Kiran
```

And:

```text
orders

id | customer_id | total
---+-------------+------
1  | 101         | 5000
2  | 101         | 3000
3  | 102         | 7000
```

The order table doesn't contain:

```text
customer_name
```

It contains:

```text
customer_id
```

So if we need:

```text
Order ID
Customer Name
Order Amount
```

we need to combine:

```text
orders
```

with:

```text
customers
```

---

# 🧠 The Relationship

```text
customers
    │
    │ id
    │
    ▼
orders.customer_id
```

The relationship is:

```text
customers.id
       =
orders.customer_id
```

This becomes the JOIN condition.

---

# 1️⃣ INNER JOIN

The most common JOIN is:

```sql
SELECT
    o.id,
    c.name,
    o.total_amount
FROM orders o
INNER JOIN customers c
    ON c.id = o.customer_id;
```

Result:

```text
order_id | customer | total
---------+----------+------
1        | Vinay    | 5000
2        | Vinay    | 3000
3        | Ravi     | 7000
```

Only rows that match on both sides are returned.

---

# 🧠 INNER JOIN Mental Model

Think:

```text
Customers
    ∩
Orders
```

Only matching relationships survive.

---

# Visual

```text
CUSTOMERS                 ORDERS

101 Vinay  ─────────────── 101
102 Ravi   ─────────────── 102
103 Kiran
```

Kiran has no order.

Therefore:

```text
Kiran
```

doesn't appear in the INNER JOIN result.

---

# 2️⃣ LEFT JOIN

Now requirement changes:

> "Show every customer, even customers who have never placed an order."

Use:

```sql
SELECT
    c.id,
    c.name,
    o.id AS order_id,
    o.total_amount
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id;
```

Result:

```text
customer | order_id | total
---------+----------+------
Vinay    | 1        | 5000
Vinay    | 2        | 3000
Ravi     | 3        | 7000
Kiran    | NULL     | NULL
```

---

# 🧠 LEFT JOIN Mental Model

A LEFT JOIN means:

> Keep every row from the left table.

Then:

```text
If matching row exists
    ↓
combine it

If no match
    ↓
NULL for right-side columns
```

Visual:

```text
LEFT TABLE
    │
    ├── Match → Combined Row
    │
    ├── Match → Combined Row
    │
    └── No Match → NULL
```

---

# 🔥 Real Production Problem

Product asks:

> "Show all customers and their latest order."

If you use:

```sql
INNER JOIN
```

customers with no orders disappear.

That's a business bug.

Use:

```sql
LEFT JOIN
```

when the business requirement says:

> Keep the left-side population even when the relationship is missing.

---

# 3️⃣ RIGHT JOIN

RIGHT JOIN is the mirror image.

```sql
SELECT
    c.name,
    o.id
FROM customers c
RIGHT JOIN orders o
    ON o.customer_id = c.id;
```

It preserves all rows from the right table.

In practice, many teams prefer rewriting:

```text
RIGHT JOIN
```

as:

```text
LEFT JOIN
```

by reversing table order.

For readability, LEFT JOIN is often easier to reason about.

---

# 4️⃣ FULL OUTER JOIN

A FULL OUTER JOIN keeps:

```text
Matching rows
+
Unmatched left rows
+
Unmatched right rows
```

Example:

```sql
SELECT
    c.id AS customer_id,
    c.name,
    o.id AS order_id
FROM customers c
FULL OUTER JOIN orders o
    ON o.customer_id = c.id;
```

Conceptually:

```text
LEFT ONLY
+
MATCHED
+
RIGHT ONLY
```

Not every database supports FULL OUTER JOIN directly.

---

# 🧠 FULL JOIN Mental Model

```text
Customer only
      +
Customer + Order
      +
Order only
```

This can be useful for reconciliation problems.

---

# 🔥 Real Production Example — Data Reconciliation

Suppose:

```text
Application Database
```

contains:

```text
customer IDs
```

and an external system contains:

```text
customer IDs
```

You want:

```text
Customers in both systems
Customers only in application
Customers only in external system
```

A FULL OUTER JOIN can help identify mismatches.

Conceptually:

```text
System A
   ↕
FULL JOIN
   ↕
System B
```

Then classify:

```text
A only
Both
B only
```

This is a powerful data-quality pattern.

---

# 5️⃣ CROSS JOIN

A CROSS JOIN creates a Cartesian product.

Suppose:

```text
colors

Red
Blue
```

and:

```text
sizes

S
M
L
```

Then:

```sql
SELECT
    c.color,
    s.size
FROM colors c
CROSS JOIN sizes s;
```

Result:

```text
Red   S
Red   M
Red   L
Blue  S
Blue  M
Blue  L
```

Total:

```text
2 × 3 = 6 rows
```

---

# ⚠️ CROSS JOIN Can Explode

Suppose:

```text
customers = 1,000,000
products  = 100,000
```

A CROSS JOIN could conceptually produce:

```text
100,000,000,000
```

rows.

That's:

```text
100 billion rows
```

before further processing.

Therefore:

> Never accidentally create a Cartesian product in production.

---

# 🔥 Accidental CROSS JOIN

This query:

```sql
SELECT *
FROM customers c
JOIN orders o;
```

is invalid in many SQL dialects unless the syntax supports a specific form.

But a missing or incorrect join predicate can effectively create Cartesian behavior.

Always verify:

```text
How many rows are entering the next stage?
```

---

# 6️⃣ SELF JOIN

A table can JOIN to itself.

Example:

```text
employees

id | name   | manager_id
---+--------+-----------
1  | Vinay  | NULL
2  | Ravi   | 1
3  | Kiran  | 1
4  | Arun   | 2
```

Requirement:

```text
Employee
Manager
```

Query:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON m.id = e.manager_id;
```

Result:

```text
employee | manager
---------+--------
Vinay    | NULL
Ravi     | Vinay
Kiran    | Vinay
Arun     | Ravi
```

---

# 🧠 Why SELF JOIN?

The table contains two logical roles:

```text
Employee
Manager
```

Physically:

```text
One table
```

Logically:

```text
Two roles
```

So we use two aliases:

```text
e
m
```

---

# 7️⃣ One-to-One Relationship

Suppose:

```text
users
```

and:

```text
user_profiles
```

Each user has exactly one profile.

```text
users
  1
  │
  │
  1
  ▼
user_profiles
```

JOIN:

```sql
SELECT
    u.id,
    u.email,
    p.bio
FROM users u
JOIN user_profiles p
    ON p.user_id = u.id;
```

Expected relationship:

```text
1 user
 ↓
1 profile
```

---

# 8️⃣ One-to-Many Relationship

This is extremely common.

```text
Customer
   │
   │ 1
   │
   │
   │ many
   ▼
Orders
```

One customer:

```text
101
```

can have:

```text
Order 1
Order 2
Order 3
```

JOIN:

```sql
SELECT
    c.name,
    o.id
FROM customers c
JOIN orders o
    ON o.customer_id = c.id;
```

The customer appears multiple times.

That's expected.

---

# 🧠 Important

If:

```text
1 customer
3 orders
```

then after the JOIN:

```text
3 rows
```

not:

```text
1 row
```

This is not a JOIN bug.

This is the relationship.

---

# 9️⃣ Many-to-Many Relationship

Suppose:

```text
students
```

can enroll in multiple:

```text
courses
```

and each course has multiple students.

You cannot directly model this relationship with only:

```text
students
courses
```

Usually you need a junction table:

```text
student_courses
```

Architecture:

```text
students
    │
    │ 1
    ▼
student_courses
    ▲
    │ many
    │
courses
```

More precisely:

```text
Student
   │
   ├── Enrollment
   ├── Enrollment
   └── Enrollment
           │
           ▼
         Course
```

---

# 🔥 JOIN Many-to-Many

```sql
SELECT
    s.name AS student,
    c.name AS course
FROM students s
JOIN student_courses sc
    ON sc.student_id = s.id
JOIN courses c
    ON c.id = sc.course_id;
```

Result:

```text
Student | Course
--------+---------
Vinay   | SQL
Vinay   | Java
Ravi    | SQL
```

---

# 🔟 JOIN Cardinality

Now we reach one of the most important concepts.

# Cardinality

Cardinality tells us how many rows can match between two relations.

Examples:

```text
1 : 1
1 : N
N : 1
N : N
```

---

# 🧠 Why Cardinality Matters

Suppose:

```text
customers
```

contains:

```text
1 row
```

and:

```text
orders
```

contains:

```text
100 rows
```

for that customer.

JOIN produces:

```text
100 rows
```

If you then do:

```sql
SUM(...)
```

you are summing over:

```text
100 rows
```

not:

```text
1 customer row
```

---

# 🚨 The Most Dangerous JOIN Bug

Suppose:

```text
orders

id | total
---+------
1  | 1000
```

and:

```text
order_items

order_id | item
---------+------
1        | A
1        | B
1        | C
```

Now:

```sql
SELECT
    SUM(o.total)
FROM orders o
JOIN order_items oi
    ON oi.order_id = o.id;
```

Result may be:

```text
3000
```

instead of:

```text
1000
```

Why?

Because:

```text
Order 1
   ↓
3 matching items
   ↓
Order row appears 3 times
```

---

# 👑 Senior Rule

Before using:

```text
SUM
COUNT
AVG
```

after a JOIN, ask:

> **Did the JOIN change the grain of my data?**

---

# 1️⃣1️⃣ Grain

Grain means:

> What does one row represent?

Before JOIN:

```text
orders
```

may have:

```text
1 row = 1 order
```

After joining:

```text
order_items
```

you may have:

```text
1 row = 1 order item
```

That is a completely different grain.

---

# Visual

Before:

```text
ORDER TABLE

Order 1
Order 2
Order 3
```

Grain:

```text
1 row = 1 order
```

After:

```text
ORDER
 ↓
ORDER_ITEM
```

Result:

```text
Order 1 Item A
Order 1 Item B
Order 1 Item C
Order 2 Item A
Order 2 Item B
```

Grain:

```text
1 row = 1 order item
```

---

# 🔥 Always Identify Grain

Before every JOIN, ask:

```text
What does one row represent?
```

After every JOIN, ask again:

```text
What does one row represent now?
```

This habit alone prevents many SQL production bugs.

---

# 1️⃣2️⃣ JOIN Conditions

Typical JOIN:

```sql
SELECT *
FROM orders o
JOIN customers c
    ON c.id = o.customer_id;
```

The:

```text
ON
```

clause defines how rows match.

---

# Composite JOIN

Sometimes one column isn't enough.

Suppose:

```text
prices

product_id
region_id
price
```

and another table also identifies:

```text
product_id
region_id
```

Then:

```sql
SELECT *
FROM products p
JOIN prices pr
    ON pr.product_id = p.id
   AND pr.region_id = p.region_id;
```

Multiple columns define the relationship.

---

# 🧠 Why This Matters

If you JOIN only on:

```sql
ON pr.product_id = p.id
```

you may accidentally match:

```text
Product 10 + Region 1
Product 10 + Region 2
Product 10 + Region 3
```

and multiply rows.

The missing key component causes incorrect results.

---

# 1️⃣3️⃣ JOIN Using Business Keys

Sometimes developers JOIN:

```text
email
phone
name
```

instead of:

```text
primary key
```

This can be dangerous.

Names may not be unique.

Emails may change.

Phones may be shared or recycled.

Prefer stable identifiers where the data model provides them.

---

# 🔥 Bad JOIN

```sql
SELECT *
FROM orders o
JOIN customers c
    ON c.name = o.customer_name;
```

Suppose:

```text
Two customers:
Ravi
Ravi
```

Now one order can match multiple customers.

Result:

```text
Order
 ↓
Ravi #1
Ravi #2
```

The row count doubles.

---

# 👑 Senior Principle

> A JOIN condition should represent the real relationship, not merely produce a matching-looking result.

---

# 1️⃣4️⃣ ON vs WHERE

This is one of the most important SQL concepts.

Consider:

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

What happens?

Customers without orders have:

```text
o.status = NULL
```

Then:

```text
WHERE o.status = 'PAID'
```

is not TRUE.

So those customers disappear.

The LEFT JOIN effectively behaves like an INNER JOIN for this condition.

---

# Compare

## Query A

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
   AND o.status = 'PAID';
```

Here:

```text
All customers remain.
```

Only paid orders are matched.

---

## Query B

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

Here:

```text
Customers without paid orders disappear.
```

---

# 🧠 Difference

Query A:

```text
LEFT JOIN
   ↓
Match only PAID orders
   ↓
Keep all customers
```

Query B:

```text
LEFT JOIN
   ↓
Create result
   ↓
WHERE removes rows
   ↓
Customers without paid orders disappear
```

---

# 🔥 Real Production Problem

Requirement:

> "Show every customer and their paid orders, including customers with no paid orders."

Correct pattern:

```sql
SELECT
    c.id,
    c.name,
    o.id AS order_id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
   AND o.status = 'PAID';
```

Not:

```sql
WHERE o.status = 'PAID'
```

---

# 1️⃣5️⃣ LEFT JOIN + NULL Detection

Requirement:

> "Find customers who never placed an order."

Use:

```sql
SELECT
    c.id,
    c.name
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.id IS NULL;
```

Conceptually:

```text
Customers
   ↓
LEFT JOIN Orders
   ↓
No matching order
   ↓
Order columns = NULL
   ↓
WHERE order.id IS NULL
```

This is an anti-join pattern.

---

# 1️⃣6️⃣ NOT EXISTS

The same business question can often be written as:

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

Meaning:

> Find customers for whom no matching order exists.

---

# 🧠 JOIN vs EXISTS

If you only care whether a related row exists:

```text
EXISTS
```

is often more semantically direct than:

```text
JOIN
```

Why?

Because a JOIN produces matching rows.

EXISTS asks:

> Does at least one matching row exist?

---

# 🔥 Example

Requirement:

> "Find customers who have at least one paid order."

Using JOIN:

```sql
SELECT DISTINCT
    c.id,
    c.name
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

Potential issue:

```text
Customer
 ↓
10 paid orders
 ↓
10 rows
```

Therefore you need:

```text
DISTINCT
```

to get one customer.

Using EXISTS:

```sql
SELECT
    c.id,
    c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.status = 'PAID'
);
```

This directly expresses the requirement.

---

# 👑 Senior Rule

If your question is:

> "Does a related row exist?"

Think:

```text
EXISTS
```

If your question is:

> "Give me data from the related rows."

Think:

```text
JOIN
```

The optimizer may transform these internally, but semantic clarity should guide the SQL you write.

---

# 1️⃣7️⃣ JOIN + GROUP BY

Suppose requirement:

> "Revenue by customer."

```sql
SELECT
    c.id,
    c.name,
    SUM(o.total_amount) AS revenue
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID'
GROUP BY
    c.id,
    c.name;
```

Grain:

```text
1 row = 1 customer
```

after aggregation.

Before aggregation:

```text
1 row = 1 order
```

assuming the JOIN does not introduce additional multiplication.

---

# 🔥 Revenue Bug

Now suppose you additionally JOIN:

```text
order_items
```

```sql
SELECT
    c.id,
    c.name,
    SUM(o.total_amount) AS revenue
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
JOIN order_items oi
    ON oi.order_id = o.id
GROUP BY
    c.id,
    c.name;
```

Potential result:

```text
Revenue × Number of Items
```

because the order row was multiplied.

---

# 🧠 Correct Approach

If you need item information, determine the correct grain first.

Possible approaches:

```text
Aggregate order_items first
        ↓
One row per order
        ↓
JOIN orders
        ↓
Aggregate customers
```

or:

```text
Aggregate orders first
        ↓
One row per customer/order
        ↓
Join other information
```

The correct strategy depends on the metric.

---

# 1️⃣8️⃣ Join Order and SQL Optimizer

You write:

```sql
FROM A
JOIN B
JOIN C
```

Do not assume the database physically executes:

```text
A → B → C
```

The optimizer may reorder joins.

It evaluates possible plans based on things like:

```text
Statistics
Indexes
Cardinality estimates
Join algorithms
Filters
Costs
```

The SQL describes the result.

The optimizer chooses a physical strategy.

---

# 🧠 Logical vs Physical Query

Logical:

```text
A
JOIN
B
JOIN
C
```

Physical execution might be:

```text
B
 ↓
Filter
 ↓
Hash
 ↓
Join C
 ↓
Join A
```

The exact plan depends on the database.

---

# 1️⃣9️⃣ Join Algorithms

Modern relational databases commonly use algorithms such as:

```text
Nested Loop Join
Hash Join
Merge Join
```

Understanding them is essential for senior SQL work.

---

# 🔹 Nested Loop Join

Conceptually:

```text
For each row in A:

    find matching rows in B
```

Example:

```text
A = 10 rows
B = indexed lookup
```

Can be very efficient.

---

# Visual

```text
A1 → search B
A2 → search B
A3 → search B
A4 → search B
```

If B has an appropriate index and A is small, this can be excellent.

---

# 🔥 Example

```text
customers
= 10 rows

orders
= 100 million rows

orders.customer_id
= indexed
```

A nested-loop strategy can potentially be efficient when only a small number of customers are being searched.

But if:

```text
A = 100 million rows
```

then repeatedly probing B may become expensive.

---

# 🔹 Hash Join

Conceptually:

```text
Build hash table from one side
        ↓
Scan/probe other side
        ↓
Match keys
```

Visual:

```text
Table A
  ↓
Hash Table
  ↑
Probe
  ↑
Table B
```

Hash joins are often effective for large equality joins, depending on the database and available resources.

---

# ⚠️ Hash Join Memory

Hash joins may need significant memory.

If memory is insufficient, the database may spill intermediate data to disk.

This can make a query much slower.

Therefore:

```text
JOIN
+
Memory
+
Cardinality
```

can interact heavily.

---

# 🔹 Merge Join

Merge join works efficiently when both inputs are ordered by the join key.

Conceptually:

```text
A sorted
1
3
5
8

B sorted
1
2
5
8
```

Walk through both:

```text
1 = 1 → match

3 < 5 → advance A

5 = 5 → match

8 = 8 → match
```

This can be efficient when inputs are already suitably ordered.

---

# 🧠 Senior Skill

You don't need to manually choose:

```text
Hash Join
Nested Loop
Merge Join
```

in most SQL.

But you should understand:

> Why did the optimizer choose this algorithm?

Use:

```sql
EXPLAIN
```

or the database's equivalent execution-plan tool.

---

# 2️⃣0️⃣ JOIN Indexes

Suppose:

```text
orders.customer_id
```

is used frequently:

```sql
SELECT *
FROM orders o
JOIN customers c
    ON c.id = o.customer_id;
```

An index on:

```text
orders(customer_id)
```

may help certain access paths.

But don't blindly assume:

```text
Every JOIN column needs an index.
```

The optimizer considers:

```text
Table size
Selectivity
Join algorithm
Query predicates
Data distribution
Existing indexes
```

---

# 👑 Important

A primary key index on:

```text
customers.id
```

is common.

But the foreign-key side:

```text
orders.customer_id
```

often deserves careful indexing too.

Especially for queries such as:

```text
Find orders for customer
JOIN orders to customers
Delete/update parent relationships
```

depending on the database and schema behavior.

---

# 2️⃣1️⃣ Composite Indexes and JOINs

Suppose query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND status = 'PAID';
```

A composite index might be useful:

```text
(customer_id, status)
```

But index design depends on workload.

Don't create indexes solely because columns appear in SQL.

Think about:

```text
Filtering
Joining
Ordering
Grouping
Selectivity
Write cost
Storage
Query frequency
```

---

# 2️⃣2️⃣ JOIN + NULL

Consider:

```text
orders.customer_id = NULL
```

Then:

```sql
ON c.id = o.customer_id
```

does not match NULL using ordinary equality.

Remember:

```text
NULL = NULL
```

is not TRUE.

It is UNKNOWN.

Therefore:

```text
NULL foreign key
```

can result in an unmatched relationship.

---

# 🔥 LEFT JOIN Example

```text
orders

id | customer_id
---+------------
1  | 101
2  | NULL
```

Query:

```sql
SELECT
    o.id,
    c.name
FROM orders o
LEFT JOIN customers c
    ON c.id = o.customer_id;
```

Result:

```text
order | customer
------+---------
1     | Vinay
2     | NULL
```

The second order remains because the LEFT side is preserved.

---

# 2️⃣3️⃣ JOIN on NULL-Safe Equality

Sometimes business logic says:

> NULL on one side should match NULL on the other.

Normal:

```sql
a.value = b.value
```

does not do this.

Some databases provide NULL-safe comparison operators, while others require explicit logic.

For example, conceptually:

```text
(a = b)
OR
(both are NULL)
```

But don't add this casually.

It changes the relationship semantics.

---

# 2️⃣4️⃣ Duplicate Rows

Suppose:

```text
customers

101 Vinay
```

and:

```text
orders

1  101
2  101
3  101
```

Query:

```sql
SELECT
    c.name,
    o.id
FROM customers c
JOIN orders o
    ON o.customer_id = c.id;
```

returns:

```text
Vinay | 1
Vinay | 2
Vinay | 3
```

This is correct.

The mistake is expecting:

```text
Vinay
```

to appear only once.

If you want one row per customer, you need a different query design.

---

# 2️⃣5️⃣ DISTINCT Is Not a Universal Fix

Developers often see duplicate results and write:

```sql
SELECT DISTINCT ...
```

This can hide the symptom.

But ask:

```text
Why are rows duplicated?
```

Maybe:

```text
1-to-many relationship
Wrong JOIN condition
Missing JOIN condition
Incorrect business key
Many-to-many relationship
```

`DISTINCT` may remove duplicate-looking output without fixing the underlying logic.

---

# 👑 Senior Rule

> **Never use DISTINCT as a band-aid until you understand why the JOIN multiplied rows.**

---

# 2️⃣6️⃣ JOIN + COUNT

Suppose:

```text
customers
```

has:

```text
101 Vinay
```

and:

```text
orders
```

has:

```text
1 101
2 101
3 101
```

Query:

```sql
SELECT
    c.id,
    COUNT(*) AS count
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
GROUP BY c.id;
```

Result:

```text
101 | 3
```

Correct.

---

# 2️⃣7️⃣ COUNT(*) vs COUNT(joined_column)

With LEFT JOIN:

```sql
SELECT
    c.id,
    COUNT(*) AS rows,
    COUNT(o.id) AS orders
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY c.id;
```

For a customer with no orders:

```text
COUNT(*)
```

may return:

```text
1
```

because the LEFT JOIN still produces one result row.

But:

```text
COUNT(o.id)
```

returns:

```text
0
```

because:

```text
o.id = NULL
```

This is an extremely important pattern.

---

# 🧠 Example

Customer:

```text
Kiran
```

No orders.

LEFT JOIN produces:

```text
Kiran | NULL
```

Therefore:

```text
COUNT(*)   = 1
COUNT(o.id) = 0
```

If the requirement is:

> Number of orders

use:

```text
COUNT(o.id)
```

not blindly:

```text
COUNT(*)
```

---

# 2️⃣8️⃣ LEFT JOIN + SUM

Suppose customer has no orders.

```sql
SELECT
    c.id,
    SUM(o.total_amount) AS revenue
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY c.id;
```

For customers with no matching non-NULL order amounts:

```text
SUM(...)
```

may return:

```text
NULL
```

If business wants:

```text
0
```

use:

```sql
COALESCE(
    SUM(o.total_amount),
    0
)
```

---

# 🔥 Production Dashboard

```sql
SELECT
    c.id,
    c.name,
    COUNT(o.id) AS order_count,
    COALESCE(
        SUM(o.total_amount),
        0
    ) AS revenue
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY
    c.id,
    c.name;
```

This gives:

```text
Customer
Order Count
Revenue
```

including customers with no orders.

---

# 2️⃣9️⃣ Three-Table JOIN

Real applications commonly need:

```text
customers
    ↓
orders
    ↓
order_items
```

Query:

```sql
SELECT
    c.name,
    o.id AS order_id,
    oi.product_id,
    oi.quantity
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
JOIN order_items oi
    ON oi.order_id = o.id;
```

Logical relationship:

```text
Customer
   ↓
Order
   ↓
Order Item
```

---

# 🔥 Four-Table JOIN

Add:

```text
products
```

```sql
SELECT
    c.name,
    o.id AS order_id,
    p.name AS product,
    oi.quantity
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
JOIN order_items oi
    ON oi.order_id = o.id
JOIN products p
    ON p.id = oi.product_id;
```

Now:

```text
Customer
    ↓
Order
    ↓
Order Item
    ↓
Product
```

---

# 🧠 But What Is the Grain?

The final result is:

```text
1 row = 1 order item
```

not:

```text
1 row = 1 order
```

This distinction is critical.

---

# 3️⃣0️⃣ JOIN + Aggregation Correctly

Suppose requirement:

> "Show customer revenue."

And order total is already authoritative.

Do:

```sql
SELECT
    c.id,
    c.name,
    SUM(o.total_amount) AS revenue
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID'
GROUP BY
    c.id,
    c.name;
```

Do not unnecessarily JOIN `order_items` unless required.

Why?

Every additional JOIN can change:

```text
Cardinality
Grain
Performance
```

---

# 👑 Senior Principle

> **Join only the tables required to answer the question.**

More JOINs are not automatically better.

---

# 3️⃣1️⃣ JOIN Elimination

Modern optimizers can sometimes eliminate unnecessary joins if they can prove the JOIN doesn't affect the result.

But don't depend on this blindly.

Write logically clean SQL first.

---

# 3️⃣2️⃣ Join Filtering

Suppose:

```text
orders = 100 million
```

but:

```text
status = PAID
```

is only:

```text
10 million
```

A query:

```sql
SELECT ...
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

allows the optimizer to consider filtering orders early.

The optimizer may push predicates through the plan when safe.

This is called:

```text
Predicate Pushdown
```

---

# 🧠 Important

Don't assume SQL text order equals physical execution order.

The optimizer may transform:

```text
JOIN
+
WHERE
```

into an efficient execution strategy.

Use:

```sql
EXPLAIN
```

to see what actually happened.

---

# 3️⃣3️⃣ Query Execution Example

Suppose:

```text
customers = 10 million
orders = 500 million
```

Query:

```sql
SELECT
    c.name,
    o.total_amount
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE c.id = 101;
```

The optimizer may choose a strategy like:

```text
Find customer 101
       ↓
Find matching orders
       ↓
Return rows
```

rather than:

```text
Scan all customers
       ↓
Scan all orders
       ↓
Join everything
       ↓
Filter customer 101
```

Indexes and statistics can make an enormous difference.

---

# 3️⃣4️⃣ EXPLAIN

For production JOIN debugging:

```sql
EXPLAIN
SELECT
    c.name,
    SUM(o.total_amount)
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID'
GROUP BY c.id, c.name;
```

Look for:

```text
Estimated rows
Actual rows
Join algorithm
Index usage
Sequential scans
Sort
Hash
Memory
Spill
Parallelism
```

Exact output depends on the database.

---

# 🔥 Estimated vs Actual Rows

One of the most valuable things to inspect.

Suppose optimizer estimates:

```text
10,000 rows
```

but actual execution produces:

```text
10,000,000 rows
```

That's a huge mismatch.

Potential causes:

```text
Outdated statistics
Data skew
Correlated columns
Poor cardinality estimates
Complex predicates
```

Bad cardinality estimates can lead to bad JOIN strategies.

---

# 👑 Senior SQL Debugging

When a JOIN query is slow:

```text
Don't immediately add an index.
```

First ask:

```text
What is the row count?

What is the cardinality?

What is the grain?

Which JOIN is expanding the data?

What algorithm was selected?

Are estimates accurate?

Where is the time spent?

Is there a sort?

Is there a hash spill?

Is the query returning too many rows?
```

Then optimize.

---

# 3️⃣5️⃣ Real Production Problem — Slow JOIN

Suppose:

```text
orders = 1 billion
customers = 50 million
```

Query:

```sql
SELECT
    c.name,
    o.total_amount
FROM customers c
JOIN orders o
    ON o.customer_id = c.id;
```

Output may contain:

```text
Hundreds of millions of rows
```

Even if the JOIN itself is efficient, returning that much data is expensive.

The problem may not be:

```text
JOIN algorithm
```

It may be:

```text
Query asks for too much data.
```

---

# 👑 Senior Principle

> **The fastest query is often the query that doesn't need to process or return unnecessary data.**

---

# 3️⃣6️⃣ SELECT *

Avoid:

```sql
SELECT *
```

in large JOINs.

If you only need:

```text
customer name
order amount
```

write:

```sql
SELECT
    c.name,
    o.total_amount
```

instead of:

```sql
SELECT *
```

Benefits include:

```text
Less data
Less network traffic
Less memory
Potentially better plans
Clearer intent
```

Exact optimizer effects vary.

---

# 3️⃣7️⃣ JOIN and Network Cost

Suppose:

```text
Database
    ↓
Application
```

Query returns:

```text
100 million rows
```

Even if database processing is fast:

```text
Network transfer
+
Serialization
+
Application memory
+
JSON conversion
```

can become the real bottleneck.

Therefore:

```text
SQL performance
```

is not only:

```text
CPU inside database
```

It includes the entire data path.

---

# 🔥 Production Architecture

Instead of:

```text
Database
   ↓
100M rows
   ↓
Application
   ↓
Filter
```

prefer:

```text
Database
   ↓
Filter
   ↓
Aggregate
   ↓
Small result
   ↓
Application
```

Push appropriate data processing closer to the data.

But don't blindly move every business rule into SQL.

---

# 3️⃣8️⃣ JOIN vs Application-Side Combination

Suppose:

```text
customers
```

and:

```text
orders
```

are in the same database.

Usually:

```text
Database JOIN
```

is appropriate.

But suppose data lives in:

```text
Database A
```

and:

```text
External API B
```

You cannot simply SQL JOIN them unless your architecture provides a mechanism such as:

```text
Federation
Foreign Data Wrapper
External Table
ETL
Data Warehouse
```

Otherwise you may have to combine data in an application or data pipeline.

---

# 🧠 Data Locality Matters

Think:

```text
Same database
   ↓
JOIN is natural

Different systems
   ↓
Distributed data problem
```

This becomes much more important in microservices.

---

# 3️⃣9️⃣ JOINs in Microservices

Suppose:

```text
Customer Service
```

owns:

```text
customers
```

and:

```text
Order Service
```

owns:

```text
orders
```

You may not want:

```text
Customer Service DB
        ↓
Cross-service SQL JOIN
        ↓
Order Service DB
```

because service boundaries usually imply ownership boundaries.

Instead:

```text
API calls
Events
Read Models
CQRS
Data Replication
Analytics Store
```

may be used.

---

# 👑 Senior Architecture Principle

> A relational JOIN is easy inside one database. Cross-service JOINs are an architectural problem.

This is why distributed systems and database design are deeply connected.

---

# 4️⃣0️⃣ JOIN + Security

Be careful when JOINing sensitive tables.

Suppose:

```text
users
payments
```

A query may accidentally expose:

```text
email
phone
payment information
```

just because the tables are easy to JOIN.

Production SQL must respect:

```text
Authorization
Row-level security
Column-level security
Data minimization
PII policies
```

---

# 🧪 Hands-on Lab

## Lab 1 — INNER JOIN

```sql
SELECT
    o.id,
    c.name,
    o.total_amount
FROM orders o
JOIN customers c
    ON c.id = o.customer_id;
```

---

# Lab 2 — LEFT JOIN

```sql
SELECT
    c.id,
    c.name,
    o.id AS order_id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id;
```

---

# Lab 3 — Find Customers Without Orders

```sql
SELECT
    c.id,
    c.name
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.id IS NULL;
```

---

# Lab 4 — NOT EXISTS

Write the same requirement using:

```sql
WHERE NOT EXISTS (...)
```

Compare the execution plans where appropriate.

---

# Lab 5 — Customer Revenue

```sql
SELECT
    c.id,
    c.name,
    SUM(o.total_amount) AS revenue
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID'
GROUP BY
    c.id,
    c.name;
```

---

# Lab 6 — Customer Order Count

```sql
SELECT
    c.id,
    c.name,
    COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY
    c.id,
    c.name;
```

---

# Lab 7 — Compare COUNT(*)

Run:

```sql
SELECT
    c.id,
    COUNT(*) AS rows
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY c.id;
```

Then:

```sql
SELECT
    c.id,
    COUNT(o.id) AS orders
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY c.id;
```

Find customers with zero orders.

Understand why the results differ.

---

# Lab 8 — ON vs WHERE

Run:

```sql
SELECT
    c.id,
    o.id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
   AND o.status = 'PAID';
```

Then:

```sql
SELECT
    c.id,
    o.id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

Compare the results.

---

# Lab 9 — JOIN Multiplication

Create:

```text
orders
order_items
```

with:

```text
1 order
5 items
```

Then:

```sql
SELECT
    o.id,
    o.total_amount
FROM orders o
JOIN order_items oi
    ON oi.order_id = o.id;
```

Count the rows.

Then test:

```sql
SELECT
    SUM(o.total_amount)
FROM orders o
JOIN order_items oi
    ON oi.order_id = o.id;
```

Compare with:

```sql
SELECT
    SUM(total_amount)
FROM orders;
```

Understand the multiplication.

---

# Lab 10 — EXPLAIN JOIN

Run:

```sql
EXPLAIN
SELECT
    c.name,
    o.total_amount
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE c.id = 101;
```

Identify:

```text
Join algorithm
Estimated rows
Index usage
Scan type
```

---

# Lab 11 — Many-to-Many

Create:

```text
students
courses
student_courses
```

Then retrieve:

```text
Student
Course
```

using two JOINs.

---

# Lab 12 — SELF JOIN

Use:

```text
employees
```

to retrieve:

```text
Employee
Manager
```

---

# 🎯 Interview Questions

## Beginner

### What is a JOIN?

A JOIN combines rows from tables based on a relationship or matching condition.

---

### What is INNER JOIN?

Returns rows where the JOIN condition matches on both sides.

---

### What is LEFT JOIN?

Returns all rows from the left side and matching rows from the right side. Missing right-side values become NULL.

---

### What is RIGHT JOIN?

Returns all rows from the right side and matching rows from the left side.

---

### What is FULL OUTER JOIN?

Returns matched rows plus unmatched rows from both sides.

---

### What is CROSS JOIN?

Produces the Cartesian product of two input relations.

---

### What is SELF JOIN?

Joining a table to itself using aliases to represent different logical roles.

---

# 🧠 Intermediate Questions

### What is cardinality?

The number of matching rows between relations or, more generally, the size of a relation/result.

---

### What is a one-to-many relationship?

One row in one table can relate to many rows in another table.

Example:

```text
Customer
   ↓
Many Orders
```

---

### Why can JOINs create duplicate rows?

Because the relationship may legitimately contain multiple matching rows.

A one-to-many JOIN changes the number of result rows.

---

### Why can SUM become incorrect after a JOIN?

Because the JOIN may multiply rows containing the value being summed.

---

### What is the difference between ON and WHERE?

`ON` defines JOIN matching.

`WHERE` filters rows from the resulting relation.

With outer JOINs, moving conditions between them can change which unmatched rows survive.

---

### Why is COUNT(o.id) useful with LEFT JOIN?

Because unmatched rows have:

```text
o.id = NULL
```

and `COUNT(o.id)` ignores NULL, allowing you to count actual matching orders.

---

# 👑 Senior Questions

### JOIN vs EXISTS?

Use JOIN when you need data from related rows.

Use EXISTS when the question is primarily whether a related row exists.

---

### Why is DISTINCT not always the solution to duplicates?

Because duplicates may indicate:

```text
Wrong relationship
Unexpected cardinality
Incorrect JOIN condition
Many-to-many relationship
```

DISTINCT can hide the underlying issue.

---

### Why can moving a condition from ON to WHERE change a LEFT JOIN?

Because WHERE executes as a filter on the resulting rows. NULL-extended rows from the outer join may fail the WHERE predicate and disappear.

---

### What is grain?

The semantic meaning of one row in a dataset.

For example:

```text
1 row = 1 order
```

or:

```text
1 row = 1 order item
```

---

### Why is grain important?

Because aggregation depends on how many times each business entity appears.

---

# 👑 20+ Year Experience Questions

## Question 1

You have:

```text
orders
order_items
```

and this query:

```sql
SELECT
    SUM(o.total_amount)
FROM orders o
JOIN order_items oi
    ON oi.order_id = o.id;
```

returns 10× the expected revenue.

What do you investigate?

```text
1. Relationship cardinality

2. Number of order items per order

3. Grain before JOIN

4. Grain after JOIN

5. Whether order total is being repeated

6. Whether the metric should aggregate orders or items

7. Whether order_items needs pre-aggregation

8. Whether another JOIN is also multiplying rows
```

---

# Question 2

Why can this:

```sql
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID'
```

behave differently from:

```sql
LEFT JOIN orders o
    ON o.customer_id = c.id
   AND o.status = 'PAID'
```

Because the first filters after the outer JOIN.

The second restricts which right-side rows qualify for the JOIN while preserving unmatched left rows.

---

# Question 3

You need:

> Customers who have at least one paid order.

Which is clearer?

```sql
JOIN + DISTINCT
```

or:

```sql
EXISTS
```

Often:

```sql
EXISTS
```

because the business question is existence rather than retrieval of order rows.

---

# Question 4

Why can a JOIN be fast but the overall query still be slow?

Because the query may produce or transfer enormous amounts of data.

Consider:

```text
Database CPU
+
Memory
+
Disk I/O
+
Network
+
Serialization
+
Application processing
```

Performance is end-to-end.

---

# Question 5

What is the first thing you check when a JOIN query produces unexpected totals?

Not the index.

First check:

```text
Grain
+
Cardinality
+
Row multiplication
```

Then verify:

```text
JOIN condition
Filters
NULL behavior
Aggregation
```

---

# Question 6

Why can a missing foreign-key-side index matter?

A JOIN often needs to locate related rows efficiently.

For example:

```text
customers.id
```

may be indexed by the primary key, but looking up:

```text
orders.customer_id
```

can also benefit from an appropriate index depending on the query pattern and database.

---

# Question 7

Why shouldn't you assume the written JOIN order is the execution order?

Because the optimizer can reorder operations and choose different physical algorithms while preserving the logical result.

---

# Question 8

What causes bad JOIN plans?

Potential causes include:

```text
Poor cardinality estimates
Outdated statistics
Data skew
Missing indexes
Bad predicates
Large intermediate results
Incorrect schema assumptions
```

---

# 🔥 Real Production Scenario

You have:

```text
customers
= 50 million

orders
= 1 billion

order_items
= 5 billion
```

Business asks:

> "Show total paid revenue by customer."

A junior developer might write:

```sql
SELECT
    c.id,
    SUM(o.total_amount)
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
JOIN order_items oi
    ON oi.order_id = o.id
WHERE o.status = 'PAID'
GROUP BY c.id;
```

It may produce incorrect revenue because:

```text
Order
 ↓
Many Items
 ↓
Order total repeated
```

And even if corrected, processing:

```text
5 billion order items
```

may be unnecessary.

If revenue already exists at:

```text
orders.total_amount
```

then:

```text
order_items
```

is irrelevant to the metric.

A better query may be:

```sql
SELECT
    o.customer_id,
    SUM(o.total_amount) AS revenue
FROM orders o
WHERE o.status = 'PAID'
GROUP BY o.customer_id;
```

Now the grain is:

```text
1 row = 1 order
```

before aggregation.

This is both:

```text
More correct
```

and potentially:

```text
Much cheaper
```

---

# 👑 The Most Important JOIN Question

Whenever you see:

```sql
FROM A
JOIN B
```

immediately ask:

```text
What does one row in A represent?

What does one row in B represent?

How many B rows can match one A row?

How many A rows can match one B row?

What will the result grain be?
```

Then ask:

```text
What happens when I aggregate?
```

This is the foundation of advanced SQL.

---

# 🧠 JOIN Mental Model

```text
                    TABLE A
                       │
                       │
                  JOIN CONDITION
                       │
                       ▼
                    TABLE B
                       │
                       ▼
                 CARDINALITY
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
            1:1       1:N       N:N
             │         │         │
             └─────────┼─────────┘
                       ▼
                    NEW GRAIN
                       │
                       ▼
                   FILTERING
                       │
                       ▼
                  AGGREGATION
                       │
                       ▼
                 BUSINESS METRIC
```

---

# 🔥 Production JOIN Checklist

Before deploying a JOIN query:

```text
1. What business question am I answering?

2. What does one row represent in each table?

3. What is the JOIN key?

4. Is the key actually unique?

5. What is the cardinality?

6. Can the JOIN multiply rows?

7. What is the resulting grain?

8. Do I need INNER or OUTER JOIN?

9. Should the condition be in ON or WHERE?

10. How should NULL behave?

11. Am I using JOIN when EXISTS is more appropriate?

12. Do I really need every joined table?

13. Could DISTINCT be hiding a modeling problem?

14. Will aggregation happen before or after multiplication?

15. Are the JOIN columns indexed appropriately?

16. What does EXPLAIN show?

17. Are estimated and actual rows close?

18. How much data will be transferred?

19. What happens at 100 million / 1 billion rows?

20. Does this query belong on the OLTP database?
```

---

# 📌 Chapter Summary

JOINs allow relational databases to combine related data.

Core JOIN types:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
CROSS JOIN
SELF JOIN
```

Important relationship types:

```text
1:1
1:N
N:1
N:N
```

Critical concepts:

```text
Cardinality
Grain
Join Key
NULL
Row Multiplication
```

Important patterns:

```text
JOIN + GROUP BY
JOIN + COUNT
JOIN + SUM
LEFT JOIN + IS NULL
EXISTS
NOT EXISTS
```

Performance concepts:

```text
Indexes
Nested Loop Join
Hash Join
Merge Join
Cardinality Estimates
Statistics
EXPLAIN
Predicate Pushdown
```

---

# 🔥 Critical Rules

```text
Rule 1
------
Always understand the relationship before JOINing.


Rule 2
------
Always know the grain of your data.


Rule 3
------
One-to-many JOINs naturally multiply rows.


Rule 4
------
JOIN multiplication can silently corrupt SUM and COUNT.


Rule 5
------
Never use DISTINCT blindly to hide duplicates.


Rule 6
------
WHERE and ON are not interchangeable with OUTER JOINs.


Rule 7
------
Use EXISTS when the business question is existence.


Rule 8
------
COUNT(*) and COUNT(joined_column) behave differently
with LEFT JOINs.


Rule 9
------
JOIN only the tables required for the business question.


Rule 10
------
Understand cardinality before aggregation.


Rule 11
------
Don't assume SQL text order equals physical execution order.


Rule 12
------
Use EXPLAIN to understand the actual execution plan.


Rule 13
------
Indexes help JOINs, but index design must consider
the complete workload.


Rule 14
------
A fast JOIN can still produce too much data.


Rule 15
------
Correctness comes before optimization.


Rule 16
------
The fastest query is often the query that processes
less data in the first place.
```

---

# 🎉 Module 1 — Database Fundamentals Continues

You now understand:

```text
SQL Filtering
      ↓
Expressions & Functions
      ↓
Aggregation
      ↓
GROUP BY / HAVING
      ↓
JOINs
      ↓
Relationships
      ↓
Cardinality
      ↓
Grain
```

We can now move from basic relational operations into one of the most important SQL concepts for real-world applications:

```text
Subqueries
```

---

# 🚀 Next Chapter

# Chapter 12 — SQL Subqueries: Scalar, Correlated, EXISTS, IN & Query Decomposition

We will answer:

> **When should a query contain another query, and how does the database execute it?**

We'll cover:

```text
Scalar Subqueries
IN Subqueries
EXISTS
NOT EXISTS
Correlated Subqueries
Non-Correlated Subqueries
Subqueries in SELECT
Subqueries in WHERE
Subqueries in FROM
Derived Tables
Common Table Expressions
Subquery vs JOIN
Subquery vs EXISTS
Correlated Subquery Performance
Nested Query Optimization
```

And solve real production problems:

```text
Find customers above average spending.

Find customers who never ordered.

Find products more expensive than the average.

Find the latest order for every customer.

Find employees earning more than their department average.

Find records that exist in another table.

Find records that don't exist in another table.

Avoid N+1 query patterns.

Understand when a correlated subquery becomes expensive.

Understand when EXISTS is better than JOIN.

Understand when a subquery should become a CTE.
```

Most importantly, we'll move from:

```text
"How do I write a nested SELECT?"
```

to:

```text
"How does query decomposition affect
correctness, readability, cardinality,
optimization, and production performance?"
```
