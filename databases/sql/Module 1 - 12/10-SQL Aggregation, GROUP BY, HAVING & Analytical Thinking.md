# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 10 — SQL Aggregation, GROUP BY, HAVING & Analytical Thinking

> In the previous chapter, we learned how SQL works with individual values:
>
> ```text
> price * quantity
> COALESCE(...)
> CASE
> CAST(...)
> ROUND(...)
> ```
>
> Those operations work mainly at the **row level**.
>
> But real software systems rarely need only individual rows.
>
> A business usually asks questions like:
>
> ```text
> How many orders did we receive today?
>
> How much revenue did we generate?
>
> Which customer spent the most?
>
> What is the average order value?
>
> How many failed payments happened?
>
> How much revenue did each city generate?
> ```
>
> Now the problem changes.
>
> Instead of:
>
> ```text
> One Row
>     ↓
> One Result
> ```
>
> we need:
>
> ```text
> Millions of Rows
>        ↓
> Group Rows
>        ↓
> Calculate Metrics
>        ↓
> Business Result
> ```
>
> This is the world of:
>
> ```text
> COUNT
> SUM
> AVG
> MIN
> MAX
> GROUP BY
> HAVING
> ```
>
> Aggregation looks simple.
>
> But aggregation is one of the places where developers start writing SQL that is syntactically valid but **logically wrong**.
>
> A 20+ year SQL engineer doesn't ask only:
>
> > "Does this query run?"
>
> They ask:
>
> > "Does this metric actually represent the business question?"

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* Aggregate functions
* `COUNT`
* `COUNT(*)`
* `COUNT(column)`
* `COUNT(DISTINCT ...)`
* `SUM`
* `AVG`
* `MIN`
* `MAX`
* `GROUP BY`
* Multiple grouping columns
* `HAVING`
* `WHERE` vs `HAVING`
* NULL behavior in aggregates
* Conditional aggregation
* Aggregation over joins
* Duplicate-row problems
* Weighted vs simple averages
* Revenue calculations
* Business metrics
* Aggregation performance
* Correct metric design
* Real production reporting problems

---

# 🏗️ Our E-Commerce Database

Continue with:

```text
customers
products
orders
order_items
payments
```

Suppose:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status VARCHAR(30) NOT NULL,
    total_amount NUMERIC(12,2) NOT NULL,
    created_at TIMESTAMP NOT NULL
);
```

Example:

```text
id | customer_id | status    | total_amount
---+-------------+-----------+-------------
1  | 101         | PAID      | 5000
2  | 101         | PAID      | 3000
3  | 102         | PENDING   | 7000
4  | 103         | PAID      | 12000
5  | 101         | CANCELLED | 2000
```

Business asks:

```text
How many orders exist?
```

That is aggregation.

---

# 📖 What Is Aggregation?

Aggregation combines multiple rows into a summarized result.

Example:

```text
Orders

5000
3000
7000
12000
2000
```

Apply:

```text
SUM
```

Result:

```text
29000
```

Conceptually:

```text
Many Rows
    ↓
Aggregation Function
    ↓
One Result
```

---

# 1️⃣ COUNT

The most common aggregate:

```sql
SELECT COUNT(*)
FROM orders;
```

If there are:

```text
1000 orders
```

result:

```text
1000
```

---

# 🧠 What Does COUNT(*) Mean?

`COUNT(*)` counts rows.

It does **not** mean:

```text
Count a particular column
```

It means:

> Count the rows produced by the query.

This distinction becomes extremely important when NULL values appear.

---

# 2️⃣ COUNT(column)

Consider:

```text
customers

id | phone
---+-----------
1  | 9999999999
2  | NULL
3  | 8888888888
4  | NULL
```

Query:

```sql
SELECT COUNT(phone)
FROM customers;
```

Result:

```text
2
```

Why?

Because:

```text
COUNT(column)
```

counts non-NULL values.

---

# Compare

```sql
SELECT COUNT(*)
FROM customers;
```

returns:

```text
4
```

while:

```sql
SELECT COUNT(phone)
FROM customers;
```

returns:

```text
2
```

---

# 👑 Senior Rule

Remember:

```text
COUNT(*)
    ↓
Counts rows

COUNT(column)
    ↓
Counts non-NULL values
```

---

# 3️⃣ COUNT(DISTINCT)

Suppose orders contain:

```text
customer_id

101
101
102
103
103
103
```

Query:

```sql
SELECT COUNT(DISTINCT customer_id)
FROM orders;
```

Result:

```text
3
```

Because the unique customers are:

```text
101
102
103
```

---

# 🔥 Real Business Question

Product team asks:

> "How many customers purchased something?"

Do not answer:

```sql
SELECT COUNT(*)
FROM orders;
```

That gives:

```text
Number of orders
```

Instead:

```sql
SELECT COUNT(DISTINCT customer_id)
FROM orders;
```

This gives:

```text
Number of distinct customers represented by orders
```

assuming the order data is correctly modeled.

---

# 🧠 Metric Naming Matters

These are different:

```text
Total Orders
Unique Customers
Total Items
Total Revenue
Average Order Value
```

Never treat them as interchangeable.

---

# 4️⃣ SUM

Example:

```sql
SELECT SUM(total_amount)
FROM orders;
```

If:

```text
5000
3000
7000
12000
2000
```

result:

```text
29000
```

---

# 🔥 Real Revenue Query

```sql
SELECT
    SUM(total_amount) AS total_revenue
FROM orders
WHERE status = 'PAID';
```

Notice the sequence:

```text
Filter
  ↓
Aggregate
```

We don't want:

```text
Cancelled Revenue
Pending Revenue
```

if the business metric means:

```text
Actual Paid Revenue
```

---

# 5️⃣ AVG

```sql
SELECT AVG(total_amount)
FROM orders;
```

Suppose:

```text
100
200
300
```

result:

```text
200
```

Conceptually:

```text
SUM(values)
    /
COUNT(values)
```

But NULL behavior and data types matter.

---

# ⚠️ AVG + NULL

Suppose:

```text
100
200
NULL
```

`AVG(amount)` generally ignores NULL values.

Conceptually:

```text
300 / 2
=
150
```

not:

```text
300 / 3
=
100
```

This is another reason NULL semantics matter.

---

# 6️⃣ MIN

```sql
SELECT MIN(total_amount)
FROM orders;
```

Returns the minimum value.

Example:

```text
5000
3000
7000
12000
```

result:

```text
3000
```

---

# 7️⃣ MAX

```sql
SELECT MAX(total_amount)
FROM orders;
```

Returns:

```text
12000
```

Useful for:

```text
Largest Order
Highest Price
Earliest Date
Latest Date
Maximum Score
```

---

# 🧠 MIN/MAX Are Not Only for Numbers

Depending on the database and data type, aggregates can also operate on:

```text
Dates
Timestamps
Strings
Other comparable types
```

For example:

```sql
SELECT MIN(created_at)
FROM orders;
```

can find the earliest order timestamp.

---

# 8️⃣ GROUP BY

Now we move from:

```text
One Result
```

to:

```text
One Result Per Group
```

Requirement:

> "How much revenue did each customer generate?"

Query:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id;
```

Result:

```text
customer_id | revenue
------------+--------
101         | 8000
103         | 12000
```

---

# 🧠 What GROUP BY Does

Think:

```text
Orders
   ↓
Group by customer_id
   ↓
Customer 101
   ├── Order 1
   └── Order 2

Customer 102
   └── Order 3

Customer 103
   └── Order 4
```

Then:

```text
Each Group
    ↓
SUM()
    ↓
One Output Row
```

---

# 9️⃣ GROUP BY Multiple Columns

Suppose we want:

```text
Revenue by city and status
```

Query:

```sql
SELECT
    city,
    status,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY
    city,
    status;
```

Conceptually:

```text
Hyderabad + PAID
Hyderabad + PENDING
Bangalore + PAID
Bangalore + PENDING
```

Each unique combination becomes a group.

---

# 🧠 GROUP BY Is About Equivalence Classes

For:

```sql
GROUP BY city, status
```

rows are grouped according to:

```text
same city
AND
same status
```

Think:

```text
(city, status)
```

as a composite grouping key.

---

# 🔥 Real Analytics Query

```sql
SELECT
    city,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue,
    AVG(total_amount) AS avg_order_value
FROM orders
WHERE status = 'PAID'
GROUP BY city;
```

Output:

```text
city       | orders | revenue | avg_order_value
-----------+--------+---------+----------------
Hyderabad  | 100    | 500000  | 5000
Bangalore  | 80     | 600000  | 7500
Chennai    | 50     | 200000  | 4000
```

One query produces several business metrics.

---

# 🔟 WHERE vs GROUP BY

Important order of thought:

```text
WHERE
 ↓
Filter individual rows
 ↓
GROUP BY
 ↓
Create groups
 ↓
Aggregate
```

Example:

```sql
SELECT
    customer_id,
    SUM(total_amount)
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id;
```

Meaning:

```text
Take only PAID orders
        ↓
Group by customer
        ↓
Calculate revenue
```

---

# 1️⃣1️⃣ HAVING

Suppose the requirement is:

> "Find customers whose paid revenue is greater than ₹1,00,000."

You cannot normally write:

```sql
WHERE SUM(total_amount) > 100000
```

because `SUM()` is an aggregate over groups.

Use:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(total_amount) > 100000;
```

---

# 🧠 WHERE vs HAVING

Remember:

```text
WHERE
    ↓
Filters rows

GROUP BY
    ↓
Creates groups

HAVING
    ↓
Filters groups
```

Visual:

```text
Raw Rows
   ↓
WHERE
   ↓
Filtered Rows
   ↓
GROUP BY
   ↓
Groups
   ↓
HAVING
   ↓
Remaining Groups
   ↓
Result
```

---

# 🔥 Real Example

Requirement:

> "Find cities with more than 100 paid orders."

```sql
SELECT
    city,
    COUNT(*) AS paid_orders
FROM orders
WHERE status = 'PAID'
GROUP BY city
HAVING COUNT(*) > 100;
```

Notice:

```text
status = 'PAID'
```

is a row-level filter.

Therefore:

```text
WHERE
```

And:

```text
COUNT(*) > 100
```

is a group-level filter.

Therefore:

```text
HAVING
```

---

# 1️⃣2️⃣ Logical Query Processing

A useful conceptual model is:

```text
FROM
  ↓
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
SELECT
  ↓
ORDER BY
  ↓
LIMIT
```

This is a **logical processing model**, not necessarily the physical execution order inside the database engine.

The optimizer is free to transform the execution plan while preserving semantics.

---

# 🧠 Why This Matters

Consider:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(total_amount) > 100000;
```

Think:

```text
1. Find orders
2. Remove non-PAID rows
3. Group remaining rows
4. Calculate SUM per customer
5. Remove groups <= 100000
6. Return result
```

This mental model makes aggregation much easier.

---

# 1️⃣3️⃣ The Famous GROUP BY Error

Consider:

```sql
SELECT
    customer_id,
    status,
    SUM(total_amount)
FROM orders
GROUP BY customer_id;
```

Problem:

```text
customer_id
```

is grouped.

But:

```text
status
```

is neither:

```text
GROUP BY
```

nor:

```text
Aggregate
```

What status should SQL return if one customer has:

```text
PAID
PENDING
CANCELLED
```

?

There is no single correct answer.

Many databases reject this query.

---

# 👑 Senior Principle

Every selected expression must have a well-defined relationship to the grouping.

Typically it is either:

```text
Grouping Key
```

or:

```text
Aggregate Expression
```

unless the database can prove functional dependency under its rules.

---

# 1️⃣4️⃣ Conditional Aggregation

Suppose dashboard requires:

```text
Total Orders
Paid Orders
Pending Orders
Cancelled Orders
```

We can calculate everything together.

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE
            WHEN status = 'PAID'
            THEN 1
            ELSE 0
        END
    ) AS paid_orders,

    SUM(
        CASE
            WHEN status = 'PENDING'
            THEN 1
            ELSE 0
        END
    ) AS pending_orders,

    SUM(
        CASE
            WHEN status = 'CANCELLED'
            THEN 1
            ELSE 0
        END
    ) AS cancelled_orders

FROM orders;
```

---

# 🧠 How Conditional Aggregation Works

For:

```sql
SUM(
    CASE
        WHEN status = 'PAID'
        THEN 1
        ELSE 0
    END
)
```

each row becomes:

```text
PAID
 ↓
1

Anything else
 ↓
0
```

Then:

```text
SUM
 ↓
Number of PAID orders
```

---

# 🔥 More Powerful Example

Revenue by status:

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'PAID'
            THEN total_amount
            ELSE 0
        END
    ) AS paid_revenue,

    SUM(
        CASE
            WHEN status = 'CANCELLED'
            THEN total_amount
            ELSE 0
        END
    ) AS cancelled_value

FROM orders;
```

One scan may provide multiple metrics, subject to the optimizer and database execution strategy.

---

# 1️⃣5️⃣ COUNT + CASE

Instead of:

```sql
SUM(
    CASE
        WHEN status = 'PAID'
        THEN 1
        ELSE 0
    END
)
```

some databases support patterns such as:

```sql
COUNT(*) FILTER (
    WHERE status = 'PAID'
)
```

This syntax is database-specific.

Conceptually both express:

```text
Count rows satisfying condition
```

Use the idiom that is appropriate and readable for your database.

---

# 1️⃣6️⃣ NULL + SUM

Suppose:

```text
amount

100
200
NULL
```

Then:

```sql
SELECT SUM(amount)
FROM payments;
```

typically ignores NULL values.

Result:

```text
300
```

But what if **every row is NULL**?

The result of `SUM` is generally:

```text
NULL
```

not:

```text
0
```

This matters.

---

# 🔥 Production Reporting Problem

Dashboard runs:

```sql
SELECT SUM(amount)
FROM payments
WHERE status = 'REFUNDED';
```

There are no refunded payments.

Depending on SQL semantics, the aggregate result may be:

```text
NULL
```

But UI expects:

```text
₹0
```

Use:

```sql
SELECT COALESCE(
    SUM(amount),
    0
) AS refunded_amount
FROM payments
WHERE status = 'REFUNDED';
```

Now:

```text
No matching values
        ↓
SUM = NULL
        ↓
COALESCE
        ↓
0
```

---

# 1️⃣7️⃣ COUNT Has Different NULL Behavior

Consider:

```sql
COUNT(amount)
```

NULL values are ignored.

But:

```sql
COUNT(*)
```

counts rows.

This difference is extremely important.

---

# 1️⃣8️⃣ COUNT(DISTINCT) + NULL

Consider:

```text
customer_id

101
101
102
NULL
```

Then:

```sql
COUNT(DISTINCT customer_id)
```

typically counts:

```text
101
102
```

and excludes NULL.

Result:

```text
2
```

Again:

```text
NULL semantics
```

matter.

---

# 1️⃣9️⃣ The Average Trap

Suppose two customers have:

```text
Customer A:
Orders = 1
Revenue = 1000

Customer B:
Orders = 100
Revenue = 100000
```

You calculate:

```text
Customer A average = 1000

Customer B average = 1000
```

Fine.

But consider:

```text
Customer A:
Orders = 1
Revenue = 1000

Customer B:
Orders = 100
Revenue = 10000
```

Customer-level averages:

```text
A = 1000
B = 100
```

Simple average of customer averages:

```text
(1000 + 100) / 2
=
550
```

But overall order average:

```text
11000 / 101
≈ 108.91
```

Very different.

---

# 👑 Senior Lesson

> **An average of averages is not necessarily the overall average.**

Always understand the denominator.

---

# 🔥 Weighted Average

Suppose:

```text
Group A:
average = 1000
count = 1

Group B:
average = 100
count = 100
```

The correct weighted average is:

```text
(
    1000 × 1
    +
    100 × 100
)
/
101
```

not:

```text
(1000 + 100) / 2
```

This is extremely important in:

```text
Analytics
Finance
Performance metrics
Machine learning metrics
Business dashboards
```

---

# 2️⃣0️⃣ Aggregation After JOIN

Now a dangerous production problem.

Suppose:

```text
customers
```

has:

```text
1 customer
```

and:

```text
orders
```

has:

```text
3 orders
```

and:

```text
order_items
```

has:

```text
10 items
```

If you join:

```text
customers
    ↓
orders
    ↓
order_items
```

one order can appear multiple times.

---

# 💥 Duplicate Aggregation Problem

Suppose:

```text
Order 1
total_amount = 1000
```

has:

```text
5 order_items
```

Query:

```sql
SELECT
    SUM(o.total_amount)
FROM orders o
JOIN order_items oi
    ON oi.order_id = o.id;
```

You may effectively calculate:

```text
1000
+
1000
+
1000
+
1000
+
1000
=
5000
```

instead of:

```text
1000
```

The join multiplied the order row.

---

# 🚨 This Is One of the Most Important SQL Bugs

The query may:

```text
Run successfully
Return a number
Look reasonable
```

and still be completely wrong.

This is why senior SQL engineers think about:

```text
Cardinality
```

before aggregating.

---

# 🧠 Cardinality

Cardinality describes how many rows are associated across relationships.

For:

```text
orders
```

to:

```text
order_items
```

we may have:

```text
1 Order
   ↓
Many Items
```

Therefore:

```text
JOIN
```

can multiply rows.

---

# 👑 Rule

Before writing:

```sql
SUM(...)
COUNT(...)
AVG(...)
```

after joins, ask:

> **What does one output row represent at this point in the query?**

This question prevents many production analytics bugs.

---

# 2️⃣1️⃣ Correct Aggregation Strategy

Suppose we need:

```text
Order revenue
```

but also need item information.

One approach is to aggregate items first:

```sql
SELECT
    o.id,
    o.total_amount,
    item_summary.item_count
FROM orders o
JOIN (
    SELECT
        order_id,
        COUNT(*) AS item_count
    FROM order_items
    GROUP BY order_id
) item_summary
    ON item_summary.order_id = o.id;
```

Now:

```text
order_items
    ↓
GROUP BY order_id
    ↓
One row per order
    ↓
JOIN orders
```

The grain is controlled.

---

# 🧠 Grain

A very important analytics word:

# Grain

Grain means:

> What does one row represent?

Examples:

```text
One row = One Customer

One row = One Order

One row = One Order Item

One row = One Customer per Month

One row = One City per Day
```

If you don't know the grain of your intermediate result, aggregation becomes dangerous.

---

# 🔥 Senior SQL Question

Before this query:

```sql
SELECT
    customer_id,
    SUM(total_amount)
FROM ...
```

ask:

```text
What is the grain of the rows entering SUM?
```

If the same order appears five times because of a join:

```text
SUM
```

will sum it five times.

---

# 2️⃣2️⃣ WHERE Before Aggregation

Suppose:

```text
1 million orders
```

Only:

```text
100,000
```

are paid.

Query:

```sql
SELECT
    SUM(total_amount)
FROM orders
WHERE status = 'PAID';
```

Conceptually:

```text
1,000,000 rows
       ↓
WHERE
       ↓
100,000 rows
       ↓
SUM
```

This can reduce the amount of data that needs to participate in aggregation.

The optimizer may further transform execution.

---

# 2️⃣3️⃣ HAVING After Aggregation

Example:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(total_amount) > 100000;
```

Think:

```text
Orders
  ↓
PAID only
  ↓
Group by customer
  ↓
Calculate revenue
  ↓
Keep customers > 100000
```

---

# 🔥 Why Not Use HAVING for Everything?

You could write:

```sql
SELECT
    customer_id,
    SUM(total_amount)
FROM orders
GROUP BY customer_id
HAVING status = 'PAID';
```

But this is not generally valid because:

```text
status
```

is neither grouped nor aggregated.

And conceptually, if your intention is:

> Filter individual rows before grouping.

Then:

```text
WHERE
```

is the correct semantic tool.

---

# 2️⃣4️⃣ Aggregation and Indexes

Suppose:

```text
orders = 500 million
```

Query:

```sql
SELECT COUNT(*)
FROM orders
WHERE customer_id = 1001;
```

An index on:

```text
customer_id
```

may allow the database to find matching rows efficiently.

But:

```sql
SELECT COUNT(*)
FROM orders;
```

has no filter.

The optimizer may choose:

```text
Sequential Scan
Index Scan
Index-only strategy
Other database-specific strategy
```

depending on the database, table structure, statistics, visibility rules, storage, and query.

---

# 🧠 Don't Assume

Never say:

> "COUNT always scans the whole table."

And don't say:

> "COUNT always uses the index."

The database optimizer decides.

Use:

```sql
EXPLAIN
```

or the database's execution-plan tooling.

---

# 2️⃣5️⃣ Aggregation on Large Tables

Suppose:

```text
orders = 2 billion
```

Query:

```sql
SELECT
    customer_id,
    SUM(total_amount)
FROM orders
GROUP BY customer_id;
```

Now the database must perform significant work.

Potential concerns:

```text
CPU
Memory
Sorting
Hashing
Disk I/O
Parallelism
Network transfer
Spilling
Partitioning
```

The exact execution strategy depends on the database engine.

---

# 🔥 Production Architecture Question

Should the application run this query on every dashboard request?

Maybe not.

If:

```text
2 billion orders
```

and:

```text
10,000 users
```

request the same metric every minute:

```text
Huge repeated computation
```

A better architecture might involve:

```text
Pre-aggregation
Materialized Views
Summary Tables
Data Warehouse
OLAP System
Caching
Streaming Aggregation
```

depending on the workload.

---

# 👑 20+ Year Thinking

Don't optimize only the SQL statement.

Ask:

```text
Should this metric be calculated live?

How fresh must it be?

Can it be cached?

Can it be precomputed?

Is the database OLTP or OLAP?

How many users request it?

How much historical data is needed?

Can we aggregate incrementally?
```

This is database architecture.

---

# 2️⃣6️⃣ OLTP vs Analytics

An application database might be optimized for:

```text
INSERT
UPDATE
DELETE
Small SELECTs
Transactions
```

This is generally:

# OLTP

Online Transaction Processing.

A reporting workload may ask:

```text
Scan 5 years of orders
Group by customer
Group by month
Calculate revenue
Calculate averages
```

This is closer to:

# OLAP

Online Analytical Processing.

---

# 🧠 Don't Make One Database Do Everything

A mature architecture may look like:

```text
Application
    ↓
OLTP Database
    ↓
CDC / ETL / Streaming
    ↓
Analytics Store / Warehouse
    ↓
Dashboards
```

The exact architecture depends on scale and business requirements.

---

# 2️⃣7️⃣ Conditional Revenue

Requirement:

```text
Paid revenue
Pending value
Cancelled value
```

Query:

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'PAID'
            THEN total_amount
            ELSE 0
        END
    ) AS paid_revenue,

    SUM(
        CASE
            WHEN status = 'PENDING'
            THEN total_amount
            ELSE 0
        END
    ) AS pending_value,

    SUM(
        CASE
            WHEN status = 'CANCELLED'
            THEN total_amount
            ELSE 0
        END
    ) AS cancelled_value
FROM orders;
```

One result row:

```text
paid_revenue
pending_value
cancelled_value
```

---

# 2️⃣8️⃣ Conditional Distinct Count

Requirement:

> "How many unique customers have paid orders?"

Conceptually:

```sql
SELECT
    COUNT(
        DISTINCT CASE
            WHEN status = 'PAID'
            THEN customer_id
        END
    ) AS paying_customers
FROM orders;
```

The `CASE` produces:

```text
customer_id
```

for paid rows and:

```text
NULL
```

for other rows.

`COUNT(DISTINCT ...)` ignores NULL.

Again, several SQL concepts combine:

```text
CASE
+
DISTINCT
+
COUNT
+
NULL semantics
```

---

# 2️⃣9️⃣ Multiple Metrics in One Query

A dashboard might need:

```text
Total Orders
Paid Orders
Unique Customers
Paid Revenue
Average Paid Order
```

One query can produce:

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE
            WHEN status = 'PAID'
            THEN 1
            ELSE 0
        END
    ) AS paid_orders,

    COUNT(
        DISTINCT CASE
            WHEN status = 'PAID'
            THEN customer_id
        END
    ) AS paying_customers,

    SUM(
        CASE
            WHEN status = 'PAID'
            THEN total_amount
            ELSE 0
        END
    ) AS paid_revenue,

    AVG(
        CASE
            WHEN status = 'PAID'
            THEN total_amount
        END
    ) AS avg_paid_order

FROM orders;
```

---

# 🧠 Why Does AVG Work Here?

For:

```sql
AVG(
    CASE
        WHEN status = 'PAID'
        THEN total_amount
    END
)
```

non-paid rows produce:

```text
NULL
```

`AVG` ignores those NULL values.

Therefore the average is calculated over paid orders.

---

# 3️⃣0️⃣ The Metric Definition Problem

Suppose dashboard says:

```text
Conversion Rate = 20%
```

What exactly is that?

Possibilities:

```text
Paid Orders / All Orders
```

or:

```text
Customers Who Purchased
/
Customers Who Visited
```

or:

```text
Successful Payments
/
Payment Attempts
```

These are completely different metrics.

---

# 👑 Senior Principle

> **SQL can calculate a metric perfectly while the metric itself is conceptually wrong.**

Always define:

```text
Numerator
Denominator
Population
Time Window
Filters
NULL Policy
Duplicate Policy
Grain
```

before writing the SQL.

---

# 🔥 Example: Average Order Value

AOV usually means:

```text
Total Revenue
----------------
Number of Orders
```

But which orders?

```text
All orders?
Paid orders?
Completed orders?
Delivered orders?
```

If cancelled orders are included in the denominator but excluded from revenue:

```text
AOV
```

becomes distorted.

The SQL may still be syntactically perfect.

---

# 🧠 Metric Contract

For important metrics, document:

```text
Metric:
Average Paid Order Value

Numerator:
SUM(total_amount)
for PAID orders

Denominator:
COUNT(orders)
for PAID orders

Time:
UTC calendar day

Grain:
One order

NULL:
Amounts cannot be NULL

Cancelled:
Excluded
```

Now the SQL has a precise definition.

---

# 🧪 Hands-on Lab

## Lab 1 — COUNT

```sql
SELECT COUNT(*)
FROM orders;
```

---

# Lab 2 — COUNT(column)

```sql
SELECT COUNT(customer_id)
FROM orders;
```

Compare with:

```sql
SELECT COUNT(*)
FROM orders;
```

---

# Lab 3 — COUNT(DISTINCT)

```sql
SELECT COUNT(DISTINCT customer_id)
FROM orders;
```

---

# Lab 4 — SUM

```sql
SELECT
    SUM(total_amount) AS revenue
FROM orders;
```

---

# Lab 5 — AVG

```sql
SELECT
    AVG(total_amount) AS average_order
FROM orders;
```

---

# Lab 6 — GROUP BY

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;
```

---

# Lab 7 — Revenue by Customer

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id;
```

---

# Lab 8 — HAVING

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(total_amount) > 100000;
```

---

# Lab 9 — Conditional Aggregation

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE
            WHEN status = 'PAID'
            THEN 1
            ELSE 0
        END
    ) AS paid_orders
FROM orders;
```

---

# Lab 10 — Revenue by Status

```sql
SELECT
    status,
    COUNT(*) AS order_count,
    SUM(total_amount) AS revenue,
    AVG(total_amount) AS average_order
FROM orders
GROUP BY status;
```

---

# Lab 11 — Revenue by City

```sql
SELECT
    city,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY city
ORDER BY revenue DESC;
```

---

# Lab 12 — Find High-Value Customers

```sql
SELECT
    customer_id,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(total_amount) > 100000
ORDER BY revenue DESC;
```

---

# Lab 13 — Find the Join Multiplication Bug

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

Then run:

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

Understand why the values can differ.

---

# Lab 14 — Explain

Run:

```sql
EXPLAIN
SELECT
    customer_id,
    SUM(total_amount)
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id;
```

Ask:

```text
Is the database sorting?

Hashing?

Using an index?

Parallelizing?

Spilling?

Scanning the table?
```

Your database's actual plan is the answer.

---

# 🎯 Interview Questions

## Beginner

### What is an aggregate function?

A function that combines values from multiple rows into a summarized result.

Examples:

```text
COUNT
SUM
AVG
MIN
MAX
```

---

### What is GROUP BY?

It divides rows into groups based on one or more expressions so aggregates can be calculated per group.

---

### What is HAVING?

It filters groups after grouping/aggregation.

---

### Difference between WHERE and HAVING?

```text
WHERE
→ filters rows

HAVING
→ filters groups
```

---

### Difference between COUNT(*) and COUNT(column)?

```text
COUNT(*)
→ counts rows

COUNT(column)
→ counts non-NULL values
```

---

# 🧠 Intermediate Questions

### What does COUNT(DISTINCT customer_id) calculate?

The number of distinct non-NULL customer IDs in the input rows.

---

### Does SUM(NULL) return zero?

No.

Aggregate behavior depends on the set of input rows.

For example:

```text
SUM(NULL, NULL)
```

generally results in:

```text
NULL
```

Use:

```sql
COALESCE(SUM(amount), 0)
```

if the business meaning requires zero for no non-NULL values/no rows.

---

### Why can JOIN + SUM produce incorrect totals?

Because a one-to-many join can duplicate the parent row before aggregation.

---

### What is grain?

The meaning of one row in a result or intermediate dataset.

Examples:

```text
One row per customer
One row per order
One row per customer per month
```

---

### Why is grain important?

Because aggregation assumes a particular row population.

If joins change the grain unexpectedly, metrics can be multiplied or otherwise distorted.

---

# 👑 Senior Questions

### Why is `AVG(AVG(...))` dangerous?

Because groups may contain different numbers of observations.

A simple average of group averages gives every group equal weight.

The overall average gives every underlying observation appropriate weight.

---

### Why can a correct SQL query produce a wrong business metric?

Because SQL only executes the specified logic.

If:

```text
Numerator
Denominator
Population
Time window
Grain
Filters
```

are incorrectly defined, SQL can calculate the wrong metric perfectly.

---

### Why might a huge aggregation query not belong on the OLTP database?

Because large analytical scans can compete with transactional workloads for:

```text
CPU
Memory
I/O
Connections
Locks/resources
```

A mature architecture may separate:

```text
OLTP
```

from:

```text
Analytics
```

using replication, CDC, ETL, a warehouse, or an analytical database.

---

# 👑 20+ Year Experience Questions

## Question 1

You have:

```text
2 billion orders
```

and dashboard asks:

```text
Revenue by customer
```

every 30 seconds.

Would you run:

```sql
GROUP BY customer_id
```

against the transactional table every time?

Not necessarily.

Consider:

```text
Materialized Views
Summary Tables
Incremental Aggregation
Data Warehouse
OLAP Database
Cache
Streaming Aggregation
```

The right solution depends on:

```text
Freshness Requirements
Query Frequency
Data Volume
Update Rate
Cost
Consistency Requirements
```

---

# Question 2

Why can this query be wrong?

```sql
SELECT
    SUM(o.total_amount)
FROM orders o
JOIN order_items oi
    ON oi.order_id = o.id;
```

Because:

```text
1 Order
   ↓
Many Items
```

can produce:

```text
Order Row
Order Row
Order Row
Order Row
```

before:

```text
SUM
```

Therefore:

```text
Order Amount
```

may be counted multiple times.

---

# Question 3

How would you debug an incorrect revenue metric?

Start with:

```text
1. Define the metric.

2. Define the grain.

3. Count source rows.

4. Check join cardinality.

5. Identify duplicate business keys.

6. Compare before/after each JOIN.

7. Validate filters.

8. Validate NULL behavior.

9. Validate date boundaries.

10. Compare against known sample data.

11. Check execution plan.

12. Test edge cases.
```

---

# Question 4

What is more important for an analytics query:

```text
Fast
```

or:

```text
Correct
```

Correct first.

A dashboard returning:

```text
₹10 million
```

in 100 ms is worse than:

```text
₹10 million
```

in 5 seconds if the correct answer is actually:

```text
₹8 million
```

But production engineering ultimately requires:

```text
Correctness
+
Performance
```

---

# Question 5

What is the most important question before aggregation?

> **What does one row represent at this point in the query?**

If the answer is unclear:

```text
STOP
```

before writing:

```text
SUM
COUNT
AVG
```

---

# 🔥 Production Aggregation Checklist

Before deploying a reporting query:

```text
1. What business metric am I calculating?

2. What is the numerator?

3. What is the denominator?

4. What is the row grain?

5. Which rows are included?

6. Which rows are excluded?

7. How does NULL behave?

8. Can JOINs multiply rows?

9. Is DISTINCT actually required?

10. Is AVG weighted correctly?

11. Are date boundaries correct?

12. Is the metric reproducible?

13. Is the data fresh enough?

14. Can the OLTP database handle this workload?

15. What does EXPLAIN show?

16. What happens with 100 million rows?

17. What happens with 2 billion rows?

18. Should the result be precomputed?
```

---

# 🧠 Senior Aggregation Mental Model

```text
                    BUSINESS QUESTION
                           │
                           ▼
                    DEFINE METRIC
                           │
                           ▼
                         GRAIN
                           │
                           ▼
                    SOURCE DATA
                           │
                           ▼
                         JOINs
                           │
                           ▼
                  CHECK CARDINALITY
                           │
                           ▼
                         WHERE
                           │
                           ▼
                       GROUP BY
                           │
                           ▼
                    AGGREGATE
                           │
               ┌───────────┼───────────┐
               ▼           ▼           ▼
             COUNT        SUM         AVG
               │           │           │
               └───────────┼───────────┘
                           ▼
                         HAVING
                           │
                           ▼
                       VALIDATE
                           │
                           ▼
                    EXECUTION PLAN
                           │
                           ▼
                    PRODUCTION SCALE
```

---

# 📌 Chapter Summary

Aggregation transforms:

```text
Many Rows
    ↓
Business Metrics
```

Core functions:

```text
COUNT
SUM
AVG
MIN
MAX
```

Core clauses:

```text
GROUP BY
HAVING
```

Important distinctions:

```text
WHERE
→ row filtering

GROUP BY
→ grouping

HAVING
→ group filtering
```

Critical production concepts:

```text
NULL behavior
COUNT(*) vs COUNT(column)
COUNT(DISTINCT)
Conditional aggregation
Join cardinality
Grain
Weighted averages
Metric definitions
OLTP vs OLAP
Execution plans
Pre-aggregation
```

---

# 🔥 Critical Rules

```text
Rule 1
------
COUNT(*) counts rows.


Rule 2
------
COUNT(column) ignores NULL.


Rule 3
------
COUNT(DISTINCT column) counts distinct non-NULL values.


Rule 4
------
WHERE filters rows.


Rule 5
------
HAVING filters groups.


Rule 6
------
Always understand the grain before aggregating.


Rule 7
------
JOINs can multiply rows and therefore multiply SUM/COUNT results.


Rule 8
------
Never blindly average averages.


Rule 9
------
NULL behavior must be explicitly understood.


Rule 10
------
A mathematically correct query can represent
a completely wrong business metric.


Rule 11
------
Large analytical queries may not belong on the OLTP database.


Rule 12
------
At scale, consider pre-aggregation,
materialized views, warehouses, or OLAP systems.
```

---

# 🎉 Module 1 — Database Fundamentals Continues

You now understand:

```text
SQL Filtering
      ↓
SQL Expressions
      ↓
SQL Functions
      ↓
Conditional Logic
      ↓
Aggregation
      ↓
GROUP BY
      ↓
HAVING
```

The next major step is where SQL becomes much more powerful.

---

# 🚀 Next Chapter

# Chapter 11 — SQL JOINs: INNER, LEFT, RIGHT, FULL, CROSS & Self JOIN

We will answer one of the most important questions in relational databases:

> **How does SQL combine rows from different tables?**

We will build from:

```text
Customer
    ↓
Order
    ↓
Order Item
    ↓
Product
    ↓
Payment
```

and understand:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
CROSS JOIN
SELF JOIN
JOIN conditions
Composite JOINs
NULLs in JOINs
JOIN cardinality
One-to-One
One-to-Many
Many-to-Many
JOIN + GROUP BY
JOIN + EXISTS
JOIN duplication problems
```

Most importantly, we will solve real production problems such as:

```text
Why did my COUNT suddenly double?

Why did SUM return 10x revenue?

Why did a LEFT JOIN behave like an INNER JOIN?

Why does moving a condition from ON to WHERE change the result?

How does the database actually execute a JOIN?

How do indexes affect JOIN performance?

When should I use JOIN vs EXISTS?

How do I join a 1-billion-row table efficiently?
```

The chapter will move from:

```text
"How to write JOIN syntax"
```

to:

```text
"How to reason about relational data,
cardinality, correctness, and JOIN performance
at production scale."
```
