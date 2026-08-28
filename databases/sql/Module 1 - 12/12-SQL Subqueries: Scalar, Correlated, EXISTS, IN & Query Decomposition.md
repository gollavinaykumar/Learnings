# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 12 — SQL Subqueries: Scalar, IN, EXISTS, Correlated Queries & CTEs

> In the previous chapter, we learned how JOINs combine data:
>
> ```text
> customers
>      ↓
> orders
>      ↓
> order_items
>      ↓
> products
> ```
>
> JOIN answers:
>
> > "How do I combine rows from different relations?"
>
> But many real-world questions are different.
>
> Consider:
>
> ```text
> Find customers who spent
> more than the average customer.
> ```
>
> We need to calculate:
>
> ```text
> Average customer spending
> ```
>
> and then compare every customer against it.
>
> That naturally creates:
>
> ```text
> Outer Query
>      ↓
> Compare against
>      ↓
> Inner Query
> ```
>
> This is the world of:
>
> ```text
> Subqueries
> ```
>
> Subqueries are not simply "queries inside queries."
>
> At senior level, you need to understand:
>
> ```text
> Query dependency
> Correlation
> Cardinality
> EXISTS semantics
> NULL behavior
> Optimization
> Materialization
> Query decorrelation
> CTEs
> ```
>
> A badly designed subquery can turn:
>
> ```text
> 1 SQL query
> ```
>
> into a logical:
>
> ```text
> N repeated operations
> ```
>
> while a well-designed subquery can make complicated business logic extremely clear.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What is a subquery?
* Scalar subqueries
* Single-row subqueries
* Multi-row subqueries
* `IN`
* `NOT IN`
* `EXISTS`
* `NOT EXISTS`
* Correlated subqueries
* Non-correlated subqueries
* Subqueries in `SELECT`
* Subqueries in `WHERE`
* Subqueries in `FROM`
* Derived tables
* CTEs
* Recursive CTEs
* Subquery vs JOIN
* Subquery vs EXISTS
* NULL problems with `NOT IN`
* Query decorrelation
* Materialization
* Subquery performance
* N+1 problems
* Production query design

---

# 📖 What Is a Subquery?

A subquery is a query nested inside another SQL expression or query.

Simple example:

```sql
SELECT *
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
);
```

The inner query:

```sql
SELECT customer_id
FROM orders;
```

produces customer IDs.

The outer query then uses them.

Conceptually:

```text
Inner Query
    ↓
Produces values
    ↓
Outer Query
    ↓
Produces final result
```

---

# 🧠 First Mental Model

Think of a subquery as a temporary question.

For example:

```text
Question 1:

Which customers have orders?

        ↓

Question 2:

Give me those customers.
```

SQL:

```sql
SELECT *
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
);
```

---

# 1️⃣ Non-Correlated Subquery

A non-correlated subquery does not depend on the current row of the outer query.

Example:

```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

Inner query:

```sql
SELECT AVG(price)
FROM products;
```

does not reference the outer query.

Conceptually:

```text
Calculate average price
        ↓
Get one value
        ↓
Compare every product
```

---

# 🔥 Real Business Problem

Product team asks:

> "Show products more expensive than the average product."

Query:

```sql
SELECT
    id,
    name,
    price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

Suppose:

```text
Average price = ₹2,500
```

Then products are filtered:

```text
₹1,000 → No
₹2,000 → No
₹3,000 → Yes
₹5,000 → Yes
```

---

# 🧠 Why This Is Useful

Without a subquery, you might calculate the average separately in application code.

That creates:

```text
Application
   ↓
Query 1 → Average
   ↓
Application
   ↓
Query 2 → Products
```

Instead:

```text
Database
   ↓
One SQL statement
   ↓
Result
```

This can improve consistency and reduce application-side orchestration.

---

# 2️⃣ Scalar Subquery

A scalar subquery returns one value.

Example:

```sql
SELECT
    name,
    price,
    (
        SELECT AVG(price)
        FROM products
    ) AS average_price
FROM products;
```

Result:

```text
name     | price | average_price
---------+-------+--------------
Laptop   | 50000 | 25000
Phone    | 20000 | 25000
Monitor  | 15000 | 25000
```

The subquery produces one value:

```text
25000
```

---

# 🧠 Scalar Means

```text
Scalar
   ↓
One value
```

Examples:

```text
100
25000
2026-08-28
TRUE
NULL
```

not:

```text
100
200
300
```

---

# ⚠️ Scalar Subquery Error

Suppose:

```sql
SELECT
    name
FROM products
WHERE price = (
    SELECT price
    FROM products
    WHERE category = 'Laptop'
);
```

If there are:

```text
100 laptop products
```

the inner query returns:

```text
100 rows
```

But the outer expression expects:

```text
One value
```

This creates a cardinality problem.

The database may report an error such as:

```text
Subquery returned more than one row
```

depending on the database.

---

# 👑 Senior Rule

Whenever you use:

```text
=
>
<
>=
<=
```

with a subquery, ask:

> **Can this subquery return more than one row?**

If yes, you probably need:

```text
IN
EXISTS
ANY
ALL
```

or a different query design.

---

# 3️⃣ IN Subquery

`IN` is used when the subquery can return multiple values.

Example:

```sql
SELECT
    id,
    name
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
);
```

Meaning:

```text
customer.id
    ↓
Does it exist in
    ↓
orders.customer_id?
```

---

# 🔥 Real Business Problem

> Find customers who have placed at least one order.

```sql
SELECT
    id,
    name
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
);
```

The inner query produces:

```text
101
101
102
103
103
```

The `IN` condition effectively tests membership.

The customer result contains:

```text
101
102
103
```

---

# 🧠 Duplicate Values

Notice:

```text
101
101
103
103
```

can appear in the subquery.

But membership doesn't require those values to be unique.

This is one reason `IN` can be convenient for existence-like questions.

---

# 4️⃣ NOT IN

Now:

> Find customers who don't have an order.

You might write:

```sql
SELECT
    id,
    name
FROM customers
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
);
```

Looks reasonable.

But there is a dangerous issue.

# NULL

---

# 💣 The NOT IN + NULL Problem

Suppose:

```text
orders.customer_id

101
102
NULL
```

Now:

```sql
customer_id NOT IN (
    101,
    102,
    NULL
)
```

creates SQL three-valued logic problems.

For example:

```text
103 NOT IN (...)
```

cannot be treated as simply TRUE because comparison against NULL introduces UNKNOWN.

The final `WHERE` condition keeps only rows where the predicate evaluates to TRUE.

Therefore `NOT IN` can unexpectedly return no rows or fewer rows than expected.

---

# 👑 Senior Rule

For anti-existence logic, prefer:

```sql
NOT EXISTS
```

when NULL semantics could make `NOT IN` dangerous.

Example:

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

This directly expresses:

> No matching order exists.

---

# 5️⃣ EXISTS

`EXISTS` asks a yes/no question:

> Does at least one matching row exist?

Example:

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

Conceptually:

```text
Customer 101
    ↓
Search orders
    ↓
Match found?
    ↓
YES
```

Then the customer is returned.

---

# 🧠 Why SELECT 1?

You may see:

```sql
SELECT 1
```

inside EXISTS.

The actual value isn't the point.

`EXISTS` cares whether at least one row satisfies the subquery.

These can often express the same existence test:

```sql
SELECT 1
```

or:

```sql
SELECT *
```

The conventional style is:

```sql
SELECT 1
```

because it communicates intent.

---

# 6️⃣ NOT EXISTS

Opposite:

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

```text
Return customer
IF
NO matching order exists
```

---

# 🔥 Real Production Problem

Requirement:

> "Send an offer to customers who have never purchased."

Query:

```sql
SELECT
    c.id,
    c.email
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

This is an anti-join pattern.

---

# 7️⃣ EXISTS vs IN

These queries can express similar business logic.

### IN

```sql
SELECT *
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
);
```

### EXISTS

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

Conceptually:

```text
IN
→ membership

EXISTS
→ existence
```

---

# 🧠 Which Is Faster?

There is no universal answer.

Modern optimizers can transform:

```text
IN
EXISTS
JOIN
```

into similar physical strategies.

Performance depends on:

```text
Database engine
Data size
Indexes
Statistics
Cardinality
NULL behavior
Query shape
```

Therefore:

> Don't choose SQL syntax based on folklore alone.

Use:

```sql
EXPLAIN
```

and benchmark representative data.

---

# 8️⃣ Correlated Subquery

Now we reach an important concept.

A correlated subquery references a column from the outer query.

Example:

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

Notice:

```text
c.id
```

comes from the outer query.

Therefore the inner query is correlated with the current customer.

---

# 🧠 Mental Model

Conceptually:

```text
For each customer:

    search orders
        ↓
    does matching order exist?
        ↓
    yes → return customer
```

This is a useful mental model.

But don't assume the database literally executes it as a separate full query for every row.

The optimizer may transform it into a more efficient plan.

---

# 👑 Important Distinction

Logical meaning:

```text
For each outer row,
evaluate a condition involving that row.
```

Physical execution:

```text
May be transformed by optimizer.
```

Never assume:

```text
correlated subquery = automatically N queries
```

The actual execution plan determines what happens.

---

# 9️⃣ Correlated Aggregate Subquery

Requirement:

> Find employees whose salary is greater than their department's average salary.

Tables:

```text
employees

id
name
department_id
salary
```

Query:

```sql
SELECT
    e.id,
    e.name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

The inner query references:

```text
e.department_id
```

from the outer query.

Therefore it is correlated.

---

# 🧠 What Is Happening?

For:

```text
Engineering
```

calculate:

```text
Engineering average
```

Then compare each engineer.

For:

```text
Sales
```

calculate:

```text
Sales average
```

Then compare sales employees.

Conceptually:

```text
Employee
   ↓
Find employee's department
   ↓
Calculate department average
   ↓
Compare salary
```

---

# 🔥 Alternative Using JOIN/CTE

The same problem can often be expressed by calculating department averages first.

```sql
WITH department_avg AS (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT
    e.id,
    e.name,
    e.salary
FROM employees e
JOIN department_avg d
    ON d.department_id = e.department_id
WHERE e.salary > d.avg_salary;
```

Now:

```text
employees
    ↓
GROUP BY department
    ↓
department_avg
    ↓
JOIN employees
    ↓
Compare salary
```

---

# 🧠 Which Is Better?

Neither is universally better.

Compare:

```text
Correlated Subquery
```

vs:

```text
CTE + JOIN
```

based on:

```text
Readability
Execution plan
Database optimizer
Data size
Indexes
Statistics
Reuse
Maintenance
```

---

# 🔟 Subquery in SELECT

You can put a subquery inside SELECT.

Example:

```sql
SELECT
    c.id,
    c.name,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customer_id = c.id
    ) AS order_count
FROM customers c;
```

Result:

```text
customer | order_count
---------+------------
Vinay    | 5
Ravi     | 10
Kiran    | 0
```

---

# ⚠️ Important

This is logically a correlated subquery.

The database may optimize it efficiently.

But if the optimizer cannot produce a good strategy and the dataset is large, this pattern can become expensive.

Always inspect the plan for important queries.

---

# 🔥 Alternative JOIN

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

This often provides a natural relational formulation.

Again:

```text
Don't blindly assume one is faster.
```

Benchmark.

---

# 1️⃣1️⃣ Subquery in FROM

A subquery can behave like a derived table.

Example:

```sql
SELECT
    customer_id,
    revenue
FROM (
    SELECT
        customer_id,
        SUM(total_amount) AS revenue
    FROM orders
    GROUP BY customer_id
) customer_revenue
WHERE revenue > 100000;
```

Inner query:

```text
orders
    ↓
GROUP BY customer
    ↓
customer_revenue
```

Outer query:

```text
Filter revenue > 100000
```

---

# 🧠 Derived Table

The result of:

```sql
(
    SELECT ...
)
```

in the FROM clause is often called a:

```text
Derived Table
```

It creates an intermediate relational result.

---

# 1️⃣2️⃣ Derived Table Grain

This is extremely important.

Inner query:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY customer_id;
```

has grain:

```text
1 row = 1 customer
```

The outer query must reason about that grain.

This makes derived tables useful for controlling intermediate data shape.

---

# 🔥 Senior SQL Thinking

Whenever you see:

```sql
FROM (
    SELECT ...
) x
```

ask:

```text
What is the grain of x?
```

For example:

```text
1 row = 1 customer
```

or:

```text
1 row = 1 order
```

or:

```text
1 row = 1 customer per month
```

This makes complex queries much easier to understand.

---

# 1️⃣3️⃣ CTE

CTE means:

```text
Common Table Expression
```

Syntax:

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(total_amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM customer_revenue
WHERE revenue > 100000;
```

It gives a name to an intermediate query.

---

# 🧠 Why CTEs Are Useful

Without CTE:

```text
Nested query
Nested query
Nested query
```

can become difficult to read.

With CTE:

```text
Step 1 → customer_revenue
Step 2 → filter
Step 3 → final result
```

The query becomes easier to reason about.

---

# 🔥 Complex Business Query

Suppose we need:

```text
Customers
whose paid revenue > ₹100,000
and who have at least 5 paid orders.
```

CTE:

```sql
WITH customer_stats AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS revenue
    FROM orders
    WHERE status = 'PAID'
    GROUP BY customer_id
)
SELECT
    customer_id,
    order_count,
    revenue
FROM customer_stats
WHERE order_count >= 5
  AND revenue > 100000;
```

This is much easier to understand.

---

# 1️⃣4️⃣ Multiple CTEs

You can build multiple logical stages.

```sql
WITH paid_orders AS (
    SELECT
        id,
        customer_id,
        total_amount
    FROM orders
    WHERE status = 'PAID'
),
customer_stats AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS revenue
    FROM paid_orders
    GROUP BY customer_id
)
SELECT
    *
FROM customer_stats
WHERE revenue > 100000;
```

Mental model:

```text
orders
   ↓
paid_orders
   ↓
customer_stats
   ↓
final filter
```

---

# 🧠 CTE Is Not Automatically a Temporary Table

This is a common misunderstanding.

A CTE is a query construct.

Whether the database:

```text
Inlines it
Materializes it
Spools it
Optimizes it differently
```

depends on the database and query.

Do not assume:

```text
CTE = physically stored temporary table
```

---

# 1️⃣5️⃣ CTE Materialization

Some database systems allow or choose materialization behavior under certain circumstances.

Conceptually:

```text
CTE
 ↓
Compute result
 ↓
Store intermediate result
 ↓
Reuse it
```

or:

```text
CTE
 ↓
Inline into larger query
```

The optimizer decides according to the database's rules and version.

This matters for performance.

---

# 👑 Senior Rule

Never say:

> "CTEs are always faster."

And never say:

> "CTEs are always slower."

The right question is:

> **What execution plan does my database choose?**

---

# 1️⃣6️⃣ Recursive CTE

Some data is hierarchical.

Example:

```text
CEO
 ↓
VP
 ↓
Manager
 ↓
Employee
```

A normal JOIN handles known levels.

But what if the hierarchy has:

```text
2 levels
```

today and:

```text
20 levels
```

tomorrow?

Recursive CTEs can traverse hierarchical relationships in databases that support them.

---

# Example

```sql
WITH RECURSIVE employee_tree AS (
    SELECT
        id,
        name,
        manager_id,
        0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.id,
        e.name,
        e.manager_id,
        t.level + 1
    FROM employees e
    JOIN employee_tree t
        ON e.manager_id = t.id
)
SELECT *
FROM employee_tree;
```

---

# 🧠 Recursive Structure

```text
Anchor Query
     ↓
Initial Rows
     ↓
Recursive Query
     ↓
Find Children
     ↓
Find Grandchildren
     ↓
Find Great-Grandchildren
     ↓
...
```

---

# 🔥 Real Software Problems for Recursive CTEs

Useful for:

```text
Organization hierarchy
Category trees
Folder structures
Bill of materials
Dependency graphs
Parent-child relationships
Menu trees
Comment threads
```

---

# 1️⃣7️⃣ Subquery vs JOIN

Suppose:

> Find customers with orders.

Subquery:

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

JOIN:

```sql
SELECT DISTINCT
    c.*
FROM customers c
JOIN orders o
    ON o.customer_id = c.id;
```

Both can represent the same business result.

But conceptually they communicate different things.

---

# 🧠 EXISTS Says

```text
I care whether a relationship exists.
```

JOIN says:

```text
I want to combine relational rows.
```

If you don't need order columns, EXISTS often communicates intent more directly.

---

# 1️⃣8️⃣ Subquery vs JOIN for Aggregation

Requirement:

> Customer's order count.

Subquery:

```sql
SELECT
    c.id,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customer_id = c.id
    ) AS order_count
FROM customers c;
```

JOIN:

```sql
SELECT
    c.id,
    COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
GROUP BY c.id;
```

Both can be valid.

The choice depends on:

```text
Data model
Database optimizer
Required output
Readability
Execution plan
```

---

# 1️⃣9️⃣ N+1 Query Problem

This is one of the most important application-level problems related to subqueries.

Suppose application does:

```text
Query 1:
Get 100 customers

Then:

Query 2:
Get orders for customer 1

Query 3:
Get orders for customer 2

Query 4:
Get orders for customer 3

...

Query 101:
Get orders for customer 100
```

Total:

```text
101 queries
```

This is:

# N+1 Query Problem

---

# 💥 Why Is It Bad?

Every query has overhead:

```text
Application
   ↓
Connection
   ↓
Network
   ↓
Database
   ↓
Execution
   ↓
Network
   ↓
Application
```

Repeat 100 times:

```text
Latency × 100
```

This can destroy application performance.

---

# 🔥 Better Approach

Use one SQL statement:

```sql
SELECT
    c.id,
    c.name,
    o.id AS order_id
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE c.id IN (...);
```

Or use appropriate batching / eager loading / data-loader patterns.

The correct solution depends on the application's access pattern.

---

# 🧠 Database + Application Boundary

This teaches an important lesson:

```text
SQL performance
```

is not only about SQL syntax.

It also depends on:

```text
Application query patterns
Network latency
Connection pooling
Transaction scope
ORM behavior
Batching
Caching
```

---

# 2️⃣0️⃣ ORM N+1 Problem

ORMs can make this easy to accidentally create.

For example:

```text
customers = repository.findAll()

for customer in customers:
    customer.orders
```

Depending on ORM configuration, this may generate:

```text
1 query for customers
+
N queries for orders
```

A senior engineer checks generated SQL.

Never assume:

```text
One application operation
=
One SQL query
```

---

# 👑 Senior Rule

When using an ORM:

```text
Look at generated SQL.
```

Especially for:

```text
Loops
Relationships
Lazy Loading
Pagination
Reports
Dashboards
```

---

# 2️⃣1️⃣ Correlated Subquery Performance

Consider:

```sql
SELECT
    c.id,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customer_id = c.id
    )
FROM customers c;
```

If:

```text
customers = 10 million
orders = 1 billion
```

you need to understand the execution plan.

Potential questions:

```text
Is the subquery decorrelated?

Is an index available?

Is there a hash strategy?

Is aggregation being reused?

How many rows are scanned?

Are estimates accurate?
```

Do not assume:

```text
10 million customers
=
10 million complete scans of orders
```

Modern optimizers can transform the query.

---

# 2️⃣2️⃣ Query Decorrelation

A correlated query:

```text
Outer Row
   ↓
Inner Query
```

can sometimes be transformed into:

```text
Join
+
Aggregation
```

This is called:

```text
Decorrelation
```

Conceptually:

```text
Correlated Subquery

Customer
   ↓
Find Orders
   ↓
COUNT


may become:


Orders
   ↓
GROUP BY customer_id
   ↓
JOIN Customer
```

This can allow the optimizer to choose a more efficient execution strategy.

---

# 🧠 Why Senior Engineers Care

Because SQL text can look expensive:

```sql
SELECT ...
(
    SELECT COUNT(...)
)
```

while the actual plan can be efficient.

Conversely, a query that looks simple can generate a terrible plan.

Therefore:

```text
SQL text
≠
execution cost
```

---

# 2️⃣3️⃣ EXISTS and Early Match

Conceptually, EXISTS only needs to establish:

```text
At least one row exists
```

Once existence is established, additional matching rows aren't logically required for the EXISTS result.

For example:

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

The database can potentially use an efficient existence strategy.

But again:

> The actual plan is database-specific.

---

# 2️⃣4️⃣ EXISTS + Index

Suppose:

```text
orders.customer_id
```

has an appropriate index.

Query:

```sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

An index can allow efficient matching in suitable plans.

Conceptually:

```text
Customer 101
    ↓
Index lookup
    ↓
orders.customer_id = 101
    ↓
Match found
    ↓
EXISTS = TRUE
```

---

# 2️⃣5️⃣ NOT EXISTS + NULL

Consider:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
)
```

If:

```text
orders.customer_id = NULL
```

that NULL does not match:

```text
c.id
```

through ordinary equality.

But importantly, it does not poison the entire NOT EXISTS predicate the way a NULL inside a `NOT IN` list can.

This is one reason:

```text
NOT EXISTS
```

is often safer for anti-existence logic.

---

# 2️⃣6️⃣ Subquery in UPDATE

Subqueries aren't limited to SELECT.

Example:

> Increase prices for products whose category average price is above ₹10,000.

A database-specific implementation could use a subquery in an UPDATE.

For example:

```sql
UPDATE products p
SET price = price * 1.10
WHERE category_id IN (
    SELECT category_id
    FROM products
    GROUP BY category_id
    HAVING AVG(price) > 10000
);
```

Now the inner query identifies qualifying categories.

The UPDATE modifies products belonging to those categories.

---

# ⚠️ Production Warning

For UPDATE/DELETE with subqueries:

```text
Always test the SELECT equivalent first.
```

For example:

```sql
SELECT *
FROM products
WHERE category_id IN (
    ...
);
```

Verify:

```text
Which rows will change?
How many?
Why?
```

Then perform the mutation inside an appropriate transaction and deployment process.

---

# 2️⃣7️⃣ Subquery in DELETE

Requirement:

> Delete temporary records that have no associated customer.

Conceptually:

```sql
DELETE FROM temporary_orders t
WHERE NOT EXISTS (
    SELECT 1
    FROM customers c
    WHERE c.id = t.customer_id
);
```

Again:

```text
First validate:
SELECT equivalent
```

Then:

```text
DELETE
```

---

# 👑 Senior Production Rule

For destructive queries:

```text
SELECT
   ↓
Verify result
   ↓
Transaction
   ↓
UPDATE / DELETE
```

Never test production logic directly with:

```text
DELETE
```

without validating the target set.

---

# 2️⃣8️⃣ Subquery + NULL

Consider:

```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

If there are no non-NULL prices:

```text
AVG(price)
=
NULL
```

Then:

```text
price > NULL
```

is UNKNOWN.

Therefore no rows satisfy the WHERE condition.

This is another example of why:

```text
NULL
```

must be part of your query reasoning.

---

# 2️⃣9️⃣ ANY and ALL

SQL also supports comparison against sets using constructs such as:

```text
ANY
ALL
```

Example:

```sql
SELECT *
FROM products
WHERE price > ALL (
    SELECT price
    FROM products
    WHERE category = 'Budget'
);
```

Conceptually:

> Product price is greater than every price in the subquery.

`ANY` means the comparison succeeds for at least one value, subject to SQL's NULL semantics.

These constructs are less common in everyday application SQL, but understanding them helps with advanced relational reasoning.

---

# 3️⃣0️⃣ Subquery as a Set

A powerful mental model is:

```text
Subquery
   ↓
Produces a relation/set of rows
```

Then the outer query decides how to use it:

```text
Scalar
IN
EXISTS
JOIN
FROM
```

Different contexts impose different cardinality expectations.

---

# 🧠 Cardinality Contract

Think:

```text
Scalar subquery
→ exactly one value

IN
→ zero or more values

EXISTS
→ zero or more matching rows,
   but result is Boolean

FROM subquery
→ relation/table-like result
```

This mental model prevents many SQL errors.

---

# 🔥 Production Scenario

Suppose product manager asks:

> Find users whose spending is greater than the average spending of all users.

First define:

```text
What counts as spending?

Paid orders only?
Refunds?
Cancelled orders?
Taxes?
Shipping?
Discounts?
```

Then:

```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(total_amount) AS spending
    FROM orders
    WHERE status = 'PAID'
    GROUP BY customer_id
)
SELECT
    customer_id,
    spending
FROM customer_spending
WHERE spending > (
    SELECT AVG(spending)
    FROM customer_spending
);
```

Now the metric has explicit grain:

```text
1 row = 1 customer
```

and the comparison is:

```text
Customer spending
        >
Average customer spending
```

This is far safer than averaging raw orders if the business question is about customers.

---

# 👑 The Grain Trap Again

Suppose:

```text
Customer A
1 order = ₹10,000

Customer B
100 orders = ₹100 each
```

Average order value:

```text
(10000 + 10000) / 101
≈ ₹198
```

But average customer spending:

```text
(10000 + 10000) / 2
=
₹10,000
```

Completely different.

Therefore:

```text
Average Order
≠
Average Customer
```

Subqueries do not solve metric-definition problems automatically.

---

# 🧠 Senior SQL Thinking

Before writing a subquery, ask:

```text
1. What is the outer query's grain?

2. What is the inner query's grain?

3. What cardinality does the inner query return?

4. Is the subquery correlated?

5. Does NULL affect the logic?

6. Would EXISTS express the requirement better?

7. Would JOIN be clearer?

8. Would a CTE make the stages easier to understand?

9. Can the optimizer decorrelate it?

10. What does EXPLAIN show?
```

---

# 🧪 Hands-on Lab

## Lab 1 — Above Average Product

```sql
SELECT
    id,
    name,
    price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

---

# Lab 2 — Customers With Orders

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

# Lab 3 — Customers Without Orders

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

# Lab 4 — Compare IN vs EXISTS

Write both:

```sql
WHERE id IN (...)
```

and:

```sql
WHERE EXISTS (...)
```

Compare:

```text
Results
Execution Plan
Indexes
Estimated Rows
Actual Rows
```

---

# Lab 5 — Test NOT IN + NULL

Create test data:

```text
101
102
NULL
```

Run:

```sql
SELECT *
FROM customers
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
);
```

Then:

```sql
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

Compare the results.

Understand the NULL difference.

---

# Lab 6 — Correlated Subquery

Find employees earning more than their department average:

```sql
SELECT
    e.id,
    e.name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

---

# Lab 7 — Rewrite Using CTE

Rewrite Lab 6 using:

```text
CTE
+
GROUP BY
+
JOIN
```

Compare readability and execution plan.

---

# Lab 8 — Customer Order Count

Using a correlated subquery:

```sql
SELECT
    c.id,
    c.name,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customer_id = c.id
    ) AS order_count
FROM customers c;
```

Then rewrite using:

```text
LEFT JOIN
+
GROUP BY
```

Compare.

---

# Lab 9 — Derived Table

Calculate customer revenue:

```sql
SELECT *
FROM (
    SELECT
        customer_id,
        SUM(total_amount) AS revenue
    FROM orders
    GROUP BY customer_id
) x
WHERE revenue > 100000;
```

Identify the grain of `x`.

---

# Lab 10 — Multiple CTEs

Build:

```text
paid_orders
    ↓
customer_stats
    ↓
high_value_customers
```

Use:

```text
COUNT
SUM
HAVING
```

and return:

```text
customer_id
order_count
revenue
```

---

# Lab 11 — Recursive CTE

Using:

```text
employees
```

build an organization hierarchy.

Return:

```text
employee
manager
level
```

---

# Lab 12 — EXPLAIN

Run:

```sql
EXPLAIN
SELECT
    c.id,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customer_id = c.id
    )
FROM customers c;
```

Ask:

```text
Did the database use an index?

Is the subquery executed literally per row?

Was it transformed?

What is the estimated cost?

How many rows are expected?
```

---

# 🎯 Interview Questions

## Beginner

### What is a subquery?

A query nested inside another SQL expression or query.

---

### What is a scalar subquery?

A subquery that returns a single value.

---

### What is IN?

It tests whether a value belongs to a set of values produced by an expression or subquery.

---

### What is EXISTS?

It tests whether the subquery produces at least one row.

---

### What is a correlated subquery?

A subquery that references values from the outer query.

---

### What is a CTE?

A named query expression introduced using `WITH`, used to structure a larger SQL statement.

---

# 🧠 Intermediate Questions

### IN vs EXISTS?

Conceptually:

```text
IN
→ membership

EXISTS
→ existence
```

The optimizer may implement them similarly.

---

### Why can NOT IN be dangerous?

Because NULL in the subquery result interacts with SQL's three-valued logic and can make the predicate UNKNOWN.

---

### Why is NOT EXISTS often preferred for anti-existence?

It directly expresses:

```text
No matching row exists
```

and avoids the classic NULL poisoning behavior of `NOT IN`.

---

### What is a derived table?

A subquery used in the FROM clause that produces a table-like intermediate result.

---

### Is a CTE always materialized?

No. Materialization behavior depends on the database engine, version, query, and optimizer.

---

# 👑 Senior Questions

### Is a correlated subquery always slow?

No.

The optimizer may transform it into an efficient plan.

Always inspect the actual execution plan.

---

### Why can two logically equivalent queries have different performance?

Because the optimizer may choose different:

```text
Join algorithms
Indexes
Access paths
Aggregation strategies
Materialization strategies
Parallelism
```

---

### What is query decorrelation?

Transforming a correlated subquery into an equivalent relational operation such as a JOIN or aggregation when the optimizer can safely do so.

---

### Why is grain important in subqueries?

Because the inner query may produce:

```text
One row per customer
```

while the outer query expects:

```text
One value
```

or:

```text
One row per order
```

A mismatch can produce incorrect results or cardinality errors.

---

# 👑 20+ Year Experience Questions

## Question 1

You see:

```sql
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
)
```

and `orders.customer_id` is nullable.

What do you check first?

```text
NULL behavior
```

Then consider:

```sql
NOT EXISTS
```

instead.

---

# Question 2

A developer says:

> "Correlated subqueries are always N+1 queries."

Is that correct?

No.

That's a logical mental model, not necessarily the physical execution.

The optimizer may transform the query into:

```text
JOIN
+
Aggregation
```

or another efficient strategy.

---

# Question 3

When should you consider EXISTS?

When the business requirement is:

```text
Does a related row exist?
```

rather than:

```text
Give me all matching related rows.
```

---

# Question 4

When would you use a CTE?

Useful when you need:

```text
Clear query stages
Reusable intermediate logic
Complex transformations
Recursive processing
Improved maintainability
```

But don't assume it automatically improves performance.

---

# Question 5

How do you debug a slow correlated subquery?

Start with:

```text
1. EXPLAIN / execution plan

2. Outer row count

3. Inner table size

4. Correlation predicate

5. Indexes

6. Cardinality estimates

7. Actual vs estimated rows

8. Whether the optimizer decorrelated it

9. Whether a JOIN + aggregation is clearer

10. Whether pre-aggregation is possible
```

---

# Question 6

How do you debug a scalar subquery returning multiple rows?

Check the inner query's cardinality.

For example:

```sql
SELECT price
FROM products
WHERE category = 'Laptop';
```

If it returns:

```text
500 rows
```

but the outer expression expects:

```text
1 value
```

the query is logically invalid.

Possible solutions:

```text
MIN
MAX
AVG
IN
EXISTS
Additional filter
Different query design
```

depending on the business requirement.

---

# Question 7

Can a JOIN and subquery produce different results?

Absolutely.

Differences can arise from:

```text
Duplicates
NULL behavior
Join cardinality
Aggregation grain
Outer JOIN semantics
```

Equivalent-looking SQL does not guarantee equivalent semantics.

---

# 🔥 Real Production Scenario

You have:

```text
customers
= 20 million

orders
= 2 billion
```

Requirement:

> "Find customers who have at least one paid order."

Developer writes:

```sql
SELECT DISTINCT
    c.id
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

This may produce a large intermediate result:

```text
Customer
   ↓
Many paid orders
   ↓
Many rows
   ↓
DISTINCT
   ↓
Customers
```

Alternative:

```sql
SELECT
    c.id
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.status = 'PAID'
);
```

Conceptually:

```text
Customer
   ↓
Does paid order exist?
   ↓
YES
   ↓
Return customer
```

Which is faster?

You cannot know from syntax alone.

You must inspect:

```text
Indexes
Statistics
Data distribution
Execution plan
Database engine
```

But the second query often communicates the business intent more directly.

---

# 🔥 Another Production Scenario

Requirement:

> "Find customers with no paid orders."

Potential query:

```sql
SELECT
    c.id
FROM customers c
WHERE c.id NOT IN (
    SELECT customer_id
    FROM orders
    WHERE status = 'PAID'
);
```

If:

```text
customer_id
```

can contain NULL:

```text
Potential NULL problem
```

Safer expression:

```sql
SELECT
    c.id
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.status = 'PAID'
);
```

The second directly represents:

```text
No paid order exists for this customer.
```

---

# 🔥 Advanced Production Scenario

Requirement:

> "Find customers whose total paid revenue is greater than the average customer paid revenue."

Incorrect approach:

```text
Average of order amounts
```

because the business metric is:

```text
Average customer revenue
```

Correct reasoning:

```text
Orders
   ↓
Filter PAID
   ↓
GROUP BY customer
   ↓
Customer Revenue
   ↓
AVG(Customer Revenue)
   ↓
Compare each Customer Revenue
```

SQL:

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(total_amount) AS revenue
    FROM orders
    WHERE status = 'PAID'
    GROUP BY customer_id
)
SELECT
    customer_id,
    revenue
FROM customer_revenue
WHERE revenue > (
    SELECT AVG(revenue)
    FROM customer_revenue
);
```

Notice the critical detail:

```text
1 row = 1 customer
```

before calculating:

```text
AVG(revenue)
```

This is senior-level SQL thinking.

---

# 👑 Subquery Mental Model

```text
                         BUSINESS QUESTION
                                │
                                ▼
                         DEFINE THE GRAIN
                                │
                                ▼
                         OUTER QUERY
                                │
              ┌─────────────────┼──────────────────┐
              │                 │                  │
              ▼                 ▼                  ▼
           SCALAR              IN               EXISTS
              │                 │                  │
              ▼                 ▼                  ▼
          One Value        Many Values        Existence
              │                 │                  │
              └─────────────────┼──────────────────┘
                                ▼
                         CORRELATION?
                                │
                         ┌──────┴──────┐
                         ▼             ▼
                        NO            YES
                         │             │
                         ▼             ▼
                   Independent     Correlated
                         │             │
                         └──────┬──────┘
                                ▼
                          CHECK NULLS
                                │
                                ▼
                       CHECK CARDINALITY
                                │
                                ▼
                     JOIN / CTE ALTERNATIVE?
                                │
                                ▼
                         EXPLAIN / TEST
                                │
                                ▼
                          PRODUCTION
```

---

# 🔥 Production Subquery Checklist

Before deploying a subquery:

```text
1. What does the outer query represent?

2. What does the inner query represent?

3. What is the inner query's cardinality?

4. Is it scalar?

5. Can it return multiple rows?

6. Is it correlated?

7. Does NULL affect the result?

8. Would EXISTS communicate the requirement better?

9. Would JOIN be clearer?

10. Would a CTE improve readability?

11. Is aggregation happening at the correct grain?

12. Could the query produce duplicate rows?

13. Could it become an N+1 pattern at the application layer?

14. What indexes support the correlation predicate?

15. What does EXPLAIN show?

16. Are estimated rows accurate?

17. Can the optimizer decorrelate the query?

18. Is materialization involved?

19. How does it behave with millions/billions of rows?

20. Is the metric mathematically correct?
```

---

# 📌 Chapter Summary

Subqueries allow SQL statements to reason about intermediate results.

Core forms:

```text
Scalar Subquery
IN
NOT IN
EXISTS
NOT EXISTS
Correlated Subquery
Derived Table
CTE
Recursive CTE
```

Important concepts:

```text
Cardinality
Correlation
NULL semantics
Grain
Query decorrelation
Materialization
```

Important patterns:

```text
EXISTS
→ Does a related row exist?

NOT EXISTS
→ Does no related row exist?

IN
→ Is the value a member of a set?

Scalar
→ Return one value

CTE
→ Structure complex query stages

Recursive CTE
→ Traverse hierarchies
```

---

# 🔥 Critical Rules

```text
Rule 1
------
Always know the cardinality expected from a subquery.


Rule 2
------
Scalar subqueries must produce one value.


Rule 3
------
Use IN when reasoning about membership.


Rule 4
------
Use EXISTS when reasoning about existence.


Rule 5
------
Be extremely careful with NOT IN + NULL.


Rule 6
------
NOT EXISTS is often the safer anti-existence pattern.


Rule 7
------
Correlated does not automatically mean slow.


Rule 8
------
SQL text does not tell you the physical execution cost.


Rule 9
------
CTEs improve structure, but are not automatically faster.


Rule 10
------
Always understand the grain of intermediate results.


Rule 11
------
Don't confuse average order value with average customer value.


Rule 12
------
Watch for N+1 behavior in application/ORM code.


Rule 13
------
Use EXPLAIN for performance decisions.


Rule 14
------
A query can be syntactically correct and semantically wrong.


Rule 15
------
Define the business metric before designing the subquery.
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
Subqueries
      ↓
EXISTS / NOT EXISTS
      ↓
CTEs
```

The next step is to understand how SQL controls **which rows and values can be modified**, and how databases maintain correctness when multiple operations happen simultaneously.

---

# 🚀 Next Chapter

# Chapter 13 — SQL INSERT, UPDATE, DELETE & UPSERT: Data Modification at Production Scale

We will move from:

```text
SELECT
```

to:

```text
INSERT
UPDATE
DELETE
UPSERT
MERGE
```

and solve real production problems:

```text
How does INSERT actually work?

What happens when 1 million rows are inserted?

How do transactions protect updates?

How do you safely UPDATE millions of rows?

Why can an UPDATE accidentally modify every row?

How do DELETE and TRUNCATE differ?

How do soft deletes work?

What happens when two requests update the same row?

What is UPSERT?

How do PostgreSQL ON CONFLICT patterns work?

How does MySQL INSERT ... ON DUPLICATE KEY work?

What is MERGE?

How do unique constraints prevent duplicate data?

How do you perform bulk inserts efficiently?

How do you update data without locking an entire production table?

How do you safely migrate billions of rows?
```

The chapter will move from:

```text
"How do I write UPDATE?"
```

to:

```text
"How do I safely modify production data
under concurrency, constraints, failures,
large datasets, and zero-downtime deployments?"
```
