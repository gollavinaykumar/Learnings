# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 7 — NULL, Three-Valued Logic & Missing Data

> In the previous chapter, we learned how database constraints protect data:
>
> ```text
> PRIMARY KEY
> FOREIGN KEY
> UNIQUE
> NOT NULL
> CHECK
> DEFAULT
> ```
>
> We also learned that:
>
> ```text
> Application Validation
>         +
> Database Constraints
>         ↓
> Reliable Data
> ```
>
> Now we are going to study one of the most important and most misunderstood concepts in SQL:
>
> # NULL
>
> Many developers initially think:
>
> ```text
> NULL = empty
> ```
>
> or:
>
> ```text
> NULL = 0
> ```
>
> or:
>
> ```text
> NULL = false
> ```
>
> None of these are correct.
>
> SQL treats NULL specially.
>
> Once NULL enters a condition, SQL's logic can produce:
>
> ```text
> TRUE
> FALSE
> UNKNOWN
> ```
>
> This is called:
>
> # Three-Valued Logic
>
> Understanding this is essential for writing correct SQL.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What NULL means
* Why NULL is not zero
* Why NULL is not an empty string
* Why `NULL = NULL` is not TRUE
* Three-valued logic
* TRUE, FALSE, UNKNOWN
* `IS NULL`
* `IS NOT NULL`
* NULL with `AND`
* NULL with `OR`
* NULL with `NOT`
* NULL with `IN`
* NULL with `NOT IN`
* NULL with JOINs
* NULL with aggregates
* `COUNT(*)`
* `COUNT(column)`
* `SUM`
* `AVG`
* `MIN`
* `MAX`
* `COALESCE`
* `NULLIF`
* `CASE`
* NULL sorting
* Real production bugs involving NULL
* How senior engineers reason about missing data

---

# 📖 The Real Software Problem

Imagine our e-commerce system has:

```text
customers

id | name  | city
---+-------+-----------
1  | Vinay | Hyderabad
2  | Rahul | Chennai
3  | Ravi  | NULL
```

What does:

```text
Ravi.city = NULL
```

mean?

It could mean:

```text
City is unknown
```

or:

```text
Customer didn't provide city
```

or:

```text
City doesn't apply
```

The exact business meaning depends on the application.

But technically:

```text
NULL
```

means the value is absent/missing/unknown according to the context.

---

# 🧠 NULL Is Not a Normal Value

This is the first concept to understand.

NULL is not:

```text
0
```

It is not:

```text
''
```

It is not:

```text
FALSE
```

It is not:

```text
'NULL'
```

These are all different.

```text
NULL
0
''
FALSE
'NULL'
```

are distinct concepts.

---

# 🚨 Common Beginner Mistake

Developer writes:

```sql
SELECT *
FROM customers
WHERE city = NULL;
```

They expect:

```text
Ravi
```

But this is not the correct way to test for NULL.

Use:

```sql
SELECT *
FROM customers
WHERE city IS NULL;
```

---

# 🧠 Why Doesn't `= NULL` Work?

Let's think about:

```text
5 = 5
```

Obviously:

```text
TRUE
```

Now:

```text
5 = 10
```

gives:

```text
FALSE
```

But:

```text
5 = NULL
```

What should the database answer?

We don't know what NULL represents.

So SQL produces:

```text
UNKNOWN
```

---

# 🔥 Three-Valued Logic

SQL conditions can produce:

```text
TRUE
FALSE
UNKNOWN
```

Think:

```text
5 = 5
 ↓
TRUE

5 = 10
 ↓
FALSE

5 = NULL
 ↓
UNKNOWN
```

This third state is the source of many SQL surprises.

---

# 🧠 NULL Is Not "Unknown Value = Some Hidden Value"

Be careful with the wording.

If:

```text
salary = NULL
```

you cannot simply think:

```text
salary = some secret number
```

NULL represents the absence of a value in the SQL data model.

The database therefore cannot establish ordinary equality or inequality with a concrete value.

---

# 1️⃣ IS NULL

Correct:

```sql
SELECT *
FROM customers
WHERE city IS NULL;
```

This asks:

```text
Is city NULL?
```

---

# 2️⃣ IS NOT NULL

```sql
SELECT *
FROM customers
WHERE city IS NOT NULL;
```

This asks:

```text
Is city present?
```

Result:

```text
Vinay
Rahul
```

while Ravi is excluded.

---

# 🧠 Why `IS` Instead of `=`?

Because:

```text
=
```

is used for value comparison.

While:

```text
IS NULL
```

tests for the special NULL state.

Think:

```text
value comparison
      ↓
=

NULL test
      ↓
IS NULL
```

---

# 🔥 `NULL = NULL`

This is one of the most famous SQL questions.

What does:

```text
NULL = NULL
```

return?

Not:

```text
TRUE
```

It produces:

```text
UNKNOWN
```

Why?

Because SQL NULL does not represent an ordinary value that can be compared using normal equality.

Therefore:

```sql
SELECT
    CASE
        WHEN NULL = NULL THEN 'TRUE'
        ELSE 'NOT TRUE'
    END;
```

will not treat the condition as TRUE.

Use:

```sql
SELECT
    CASE
        WHEN NULL IS NULL THEN 'TRUE'
        ELSE 'FALSE'
    END;
```

Now it is TRUE.

---

# 🚨 Real Production Bug

Imagine:

```text
users

id | deleted_at
---+-------------------
1  | NULL
2  | 2026-08-20
3  | NULL
```

Business rule:

> Active users are users that have not been deleted.

Developer writes:

```sql
SELECT *
FROM users
WHERE deleted_at = NULL;
```

Result:

```text
ZERO rows
```

Correct:

```sql
SELECT *
FROM users
WHERE deleted_at IS NULL;
```

Now:

```text
User 1
User 3
```

are returned.

---

# 🧠 Soft Deletes

This pattern is common:

```text
deleted_at
```

where:

```text
NULL
```

means:

```text
Not deleted
```

and:

```text
2026-08-20 10:30:00
```

means:

```text
Deleted at this time
```

Then:

```sql
WHERE deleted_at IS NULL
```

can represent active records.

But be careful:

> A NULL-based soft-delete design is a business convention, not a universal database rule.

---

# 3️⃣ NULL With AND

Now things become interesting.

Consider:

```text
TRUE AND UNKNOWN
```

Result:

```text
UNKNOWN
```

Because we know one side is true, but the other side is unknown.

---

Consider:

```text
FALSE AND UNKNOWN
```

Result:

```text
FALSE
```

Because if one side is definitely false, the entire AND expression is false.

---

# 🧠 AND Truth Table

Conceptually:

```text
TRUE AND TRUE
    = TRUE

TRUE AND FALSE
    = FALSE

TRUE AND UNKNOWN
    = UNKNOWN

FALSE AND TRUE
    = FALSE

FALSE AND FALSE
    = FALSE

FALSE AND UNKNOWN
    = FALSE

UNKNOWN AND TRUE
    = UNKNOWN

UNKNOWN AND FALSE
    = FALSE

UNKNOWN AND UNKNOWN
    = UNKNOWN
```

This becomes important when filtering nullable columns.

---

# 4️⃣ NULL With OR

Now:

```text
TRUE OR UNKNOWN
```

is:

```text
TRUE
```

Because one side is definitely true.

But:

```text
FALSE OR UNKNOWN
```

is:

```text
UNKNOWN
```

---

# 🧠 OR Truth Table

```text
TRUE OR TRUE
    = TRUE

TRUE OR FALSE
    = TRUE

TRUE OR UNKNOWN
    = TRUE

FALSE OR TRUE
    = TRUE

FALSE OR FALSE
    = FALSE

FALSE OR UNKNOWN
    = UNKNOWN

UNKNOWN OR TRUE
    = TRUE

UNKNOWN OR FALSE
    = UNKNOWN

UNKNOWN OR UNKNOWN
    = UNKNOWN
```

---

# 5️⃣ NULL With NOT

Consider:

```text
NOT TRUE
```

gives:

```text
FALSE
```

and:

```text
NOT FALSE
```

gives:

```text
TRUE
```

But:

```text
NOT UNKNOWN
```

is:

```text
UNKNOWN
```

You cannot turn "unknown" into "known" simply by negating it.

---

# 🔥 Why This Matters

Suppose:

```sql
SELECT *
FROM customers
WHERE NOT city = 'Hyderabad';
```

What happens to:

```text
city = NULL
```

First:

```text
NULL = 'Hyderabad'
```

becomes:

```text
UNKNOWN
```

Then:

```text
NOT UNKNOWN
```

remains:

```text
UNKNOWN
```

Therefore the row is not returned by `WHERE`.

---

# 🚨 Real Production Bug

Requirement:

> "Show everyone who is not from Hyderabad."

Developer writes:

```sql
SELECT *
FROM customers
WHERE city <> 'Hyderabad';
```

They expect:

```text
Chennai
Bangalore
NULL
```

But NULL rows are excluded.

If the business requirement includes customers whose city is missing:

```sql
SELECT *
FROM customers
WHERE city <> 'Hyderabad'
   OR city IS NULL;
```

This explicitly expresses the intended behavior.

---

# 6️⃣ WHERE and UNKNOWN

This is critical.

A `WHERE` clause keeps rows when the condition evaluates to:

```text
TRUE
```

It does not return rows where the condition evaluates to:

```text
FALSE
```

or:

```text
UNKNOWN
```

Think:

```text
WHERE condition

TRUE
 ↓
KEEP

FALSE
 ↓
REMOVE

UNKNOWN
 ↓
REMOVE
```

---

# 🧠 This Explains Many SQL Bugs

Whenever a query involving NULL behaves strangely, ask:

```text
What does the expression evaluate to?

TRUE?
FALSE?
UNKNOWN?
```

This one question solves many problems.

---

# 7️⃣ NULL With IN

Suppose:

```text
city
```

can be NULL.

Query:

```sql
SELECT *
FROM customers
WHERE city IN (
    'Hyderabad',
    'Chennai'
);
```

For:

```text
city = NULL
```

the condition is not TRUE.

Therefore the row is excluded.

That's usually expected.

---

# 🔥 NULL + NOT IN — Dangerous

Now we reach one of SQL's most famous traps.

Suppose:

```text
customers
```

contains:

```text
1
2
3
```

and you run:

```sql
SELECT *
FROM customers
WHERE id NOT IN (
    2,
    NULL
);
```

Many developers expect:

```text
1
3
```

But that is not what SQL's three-valued logic gives you.

---

# 🧠 Why?

Conceptually:

```text
id NOT IN (2, NULL)
```

is related to:

```text
id <> 2
AND
id <> NULL
```

For:

```text
id = 1
```

we get:

```text
1 <> 2
    ↓
TRUE

1 <> NULL
    ↓
UNKNOWN
```

Therefore:

```text
TRUE AND UNKNOWN
    ↓
UNKNOWN
```

`WHERE` removes UNKNOWN.

So the row isn't returned.

Same problem for:

```text
id = 3
```

Therefore a NULL inside a `NOT IN` list can produce a result that surprises developers.

---

# 🚨 Real Production Incident

Imagine:

```sql
SELECT id
FROM blocked_users;
```

returns:

```text
10
20
NULL
```

Developer writes:

```sql
SELECT *
FROM users
WHERE id NOT IN (
    SELECT id
    FROM blocked_users
);
```

They expect:

```text
All users except 10 and 20
```

But because the subquery contains NULL, the `NOT IN` condition can evaluate to UNKNOWN for candidate rows.

Potential result:

```text
ZERO rows
```

or otherwise unexpected filtering depending on the exact query.

---

# 🧠 Safer Pattern: NOT EXISTS

Often, when expressing:

> "Give me users for whom no matching blocked user exists."

use:

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM blocked_users b
    WHERE b.id = u.id
);
```

This expresses the relational condition more directly.

It also avoids the classic `NOT IN` + NULL trap.

---

# 🔥 Senior Rule

Whenever you see:

```text
NOT IN
```

and the source can contain NULL:

```text
STOP
```

Ask:

> Can NULL exist in the compared values?

If yes, carefully evaluate whether `NOT EXISTS` better expresses the requirement.

---

# 8️⃣ NULL With JOIN

Suppose:

```text
customers

id | name
---+------
1  | Vinay
2  | Rahul
3  | Ravi
```

and:

```text
orders

id  | customer_id
----+------------
101 | 1
102 | 2
103 | NULL
```

Now:

```sql
SELECT
    c.name,
    o.id
FROM customers c
JOIN orders o
    ON c.id = o.customer_id;
```

What happens to:

```text
order 103
```

with:

```text
customer_id = NULL
```

The condition:

```text
c.id = NULL
```

does not become TRUE.

Therefore the inner join does not match that order.

---

# 9️⃣ LEFT JOIN Changes the Result

Now:

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id;
```

A `LEFT JOIN` preserves rows from the left side.

If there is no matching order:

```text
order columns
```

become NULL in the result.

Example:

```text
Customer | Order
---------+------
Vinay    | 101
Rahul    | 102
Ravi     | NULL
```

This is one of the most important uses of NULL in query results.

---

# 🚨 Real Software Problem

Requirement:

> "Show all customers, including customers who have never placed an order."

Wrong:

```sql
SELECT
    c.name,
    o.id
FROM customers c
JOIN orders o
    ON c.id = o.customer_id;
```

This removes customers without orders.

Correct:

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id;
```

Now every customer remains.

---

# 🔥 LEFT JOIN + WHERE Trap

Now suppose:

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id
WHERE o.status = 'PAID';
```

Many developers think:

> "I'm still using LEFT JOIN, so customers without orders remain."

Not necessarily.

For customers without matching orders:

```text
o.status = NULL
```

Then:

```text
NULL = 'PAID'
```

becomes:

```text
UNKNOWN
```

The `WHERE` clause removes the row.

The query effectively behaves like an inner join for this condition.

---

# 🧠 Correct Requirement Matters

If the requirement is:

> "Show all customers, and include paid order information when available."

You may need:

```sql
SELECT
    c.name,
    o.id
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id
   AND o.status = 'PAID';
```

Notice the condition moved into:

```text
ON
```

rather than:

```text
WHERE
```

This is a critical SQL concept.

---

# 🔥 ON vs WHERE

Think:

```text
ON
 ↓
Defines matching relationship
```

while:

```text
WHERE
 ↓
Filters final rows
```

With outer joins, moving a condition between `ON` and `WHERE` can change the meaning of the query.

---

# 1️⃣0️⃣ NULL and COUNT

Now we move to aggregates.

Suppose:

```text
employees

id | name  | bonus
---+-------+-------
1  | Vinay | 10000
2  | Rahul | NULL
3  | Ravi  | 5000
```

Run:

```sql
SELECT COUNT(*)
FROM employees;
```

Result:

```text
3
```

`COUNT(*)` counts rows.

---

# COUNT(column)

Now:

```sql
SELECT COUNT(bonus)
FROM employees;
```

Result:

```text
2
```

Why?

Because:

```text
bonus = NULL
```

is not counted by `COUNT(bonus)`.

---

# 🧠 Important Difference

```text
COUNT(*)
```

means:

```text
Count rows
```

while:

```text
COUNT(column)
```

means, conceptually:

```text
Count non-NULL values of column
```

This distinction is extremely important.

---

# 🚨 Real Reporting Bug

Dashboard requirement:

> "How many employees are in the company?"

Developer writes:

```sql
SELECT COUNT(bonus)
FROM employees;
```

Result:

```text
2
```

But there are:

```text
3 employees
```

Correct:

```sql
SELECT COUNT(*)
FROM employees;
```

---

# 1️⃣1️⃣ SUM and NULL

Suppose:

```text
payments

amount
------
1000
2000
NULL
```

Run:

```sql
SELECT SUM(amount)
FROM payments;
```

The NULL value does not contribute a numeric amount.

The result is:

```text
3000
```

assuming at least one non-NULL value exists.

---

# 🧠 What If Every Value Is NULL?

Suppose:

```text
amount
------
NULL
NULL
NULL
```

Then:

```sql
SELECT SUM(amount)
FROM payments;
```

returns:

```text
NULL
```

not:

```text
0
```

This distinction matters.

---

# 🔥 Why SUM(NULL) Isn't Automatically Zero

There is a difference between:

```text
No numeric values were present
```

and:

```text
The sum of numeric values is zero
```

These can have different business meanings.

If you explicitly want zero:

```sql
SELECT COALESCE(SUM(amount), 0)
FROM payments;
```

---

# 1️⃣2️⃣ AVG and NULL

Suppose:

```text
scores

score
-----
100
80
NULL
```

Then:

```sql
SELECT AVG(score)
FROM scores;
```

calculates the average using the non-NULL values.

Conceptually:

```text
(100 + 80) / 2
```

not:

```text
(100 + 80 + 0) / 3
```

NULL is not automatically converted to zero.

---

# 1️⃣3️⃣ MIN and MAX

For:

```text
100
200
NULL
300
```

these:

```sql
MIN(value)
MAX(value)
```

ignore NULL values for the aggregate calculation.

But if there are no non-NULL values, the aggregate result can be NULL.

---

# 🧠 Aggregate Mental Model

Think:

```text
COUNT(*)
    ↓
Rows

COUNT(column)
    ↓
Non-NULL column values

SUM(column)
    ↓
Non-NULL numeric values

AVG(column)
    ↓
Non-NULL numeric values

MIN(column)
MAX(column)
    ↓
Non-NULL values
```

---

# 1️⃣4️⃣ COALESCE

One of the most useful NULL functions is:

```text
COALESCE
```

It returns the first non-NULL expression.

Example:

```sql
SELECT
    name,
    COALESCE(city, 'Unknown') AS city
FROM customers;
```

If:

```text
city = Hyderabad
```

result:

```text
Hyderabad
```

If:

```text
city = NULL
```

result:

```text
Unknown
```

---

# 🧠 Think of COALESCE Like

```text
COALESCE(
    value1,
    value2,
    value3
)
```

means:

```text
Use value1
if not NULL

otherwise value2

otherwise value3
```

Example:

```sql
SELECT COALESCE(
    preferred_name,
    legal_name,
    'Unknown'
)
FROM users;
```

---

# 🔥 Real API Problem

Your API needs:

```json
{
  "city": "Unknown"
}
```

instead of:

```json
{
  "city": null
}
```

You could transform it in application code.

Or SQL could provide:

```sql
SELECT
    COALESCE(city, 'Unknown') AS city
FROM customers;
```

Which layer should own the transformation depends on your API and data architecture.

---

# ⚠️ COALESCE Is Not Always the Right Fix

Suppose:

```text
salary = NULL
```

You write:

```sql
COALESCE(salary, 0)
```

Now you have converted:

```text
Unknown salary
```

into:

```text
Salary = 0
```

Those may mean very different things.

So ask:

> Is NULL supposed to mean zero?

If not, don't blindly use COALESCE.

---

# 1️⃣5️⃣ NULLIF

Another useful function:

```text
NULLIF(a, b)
```

returns:

```text
NULL
```

if:

```text
a = b
```

otherwise it returns `a`.

Example:

```sql
SELECT NULLIF(0, 0);
```

Result:

```text
NULL
```

---

# 🔥 Division by Zero Example

Suppose:

```text
revenue
orders
```

You want:

```text
revenue / orders
```

If:

```text
orders = 0
```

division can fail.

Depending on the database, you may use:

```sql
SELECT
    revenue / NULLIF(orders, 0)
FROM sales;
```

Conceptually:

```text
orders = 0
       ↓
NULLIF(orders, 0)
       ↓
NULL
       ↓
revenue / NULL
       ↓
NULL
```

This can avoid a division-by-zero error in systems with standard NULL arithmetic behavior.

---

# 1️⃣6️⃣ CASE With NULL

You can explicitly handle NULL using:

```sql
SELECT
    name,
    CASE
        WHEN city IS NULL THEN 'Unknown'
        ELSE city
    END AS city
FROM customers;
```

This is useful when the transformation needs more logic than a simple COALESCE.

---

# 🧠 COALESCE vs CASE

Simple fallback:

```sql
COALESCE(city, 'Unknown')
```

Conditional logic:

```sql
CASE
    WHEN city IS NULL THEN 'Unknown'
    WHEN city = 'Hyderabad' THEN 'HYD'
    ELSE city
END
```

Use the simplest expression that clearly communicates the requirement.

---

# 1️⃣7️⃣ NULL Sorting

Suppose:

```text
created_at

2026-01-01
NULL
2026-02-01
```

What happens with:

```sql
ORDER BY created_at;
```

The placement of NULL values depends on the database and sort direction.

Do not assume all SQL databases order NULL identically.

Some databases allow explicit control such as:

```sql
ORDER BY created_at NULLS LAST;
```

or:

```sql
ORDER BY created_at NULLS FIRST;
```

Support and syntax vary by database.

---

# 🔥 Production Requirement

Suppose the requirement says:

> "Show customers with unknown signup dates at the bottom."

If your database supports it:

```sql
SELECT *
FROM customers
ORDER BY created_at NULLS LAST;
```

This makes the intended behavior explicit.

---

# 1️⃣8️⃣ NULL and UNIQUE

This is a subtle topic.

Suppose:

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email TEXT UNIQUE
);
```

What happens if:

```text
email = NULL
```

is inserted multiple times?

The exact behavior around NULL and unique constraints depends on the database's semantics and configuration.

Many relational databases allow multiple NULLs in a unique constraint because NULL is not treated as equal to another NULL for ordinary uniqueness comparison.

But:

> **Do not assume identical NULL uniqueness behavior across all database systems.**

Always verify the behavior of the specific database you're using.

---

# 🧠 Why This Matters

Suppose business says:

> Every user must have a unique email.

This:

```sql
email TEXT UNIQUE
```

may not be enough if email is optional.

You may instead need:

```text
email NOT NULL
+
UNIQUE(email)
```

if every user must have an email.

This is another example of:

```text
Business Rule
      ↓
Database Constraint Design
```

---

# 1️⃣9️⃣ NULL and Arithmetic

Consider:

```text
100 + NULL
```

The result is generally:

```text
NULL
```

Similarly:

```text
100 * NULL
```

becomes:

```text
NULL
```

and:

```text
NULL / 10
```

becomes:

```text
NULL
```

Conceptually:

```text
Unknown value
     +
Known value
     ↓
Unknown result
```

---

# 🚨 Real Calculation Bug

Suppose:

```text
price = 100
discount = NULL
```

Developer writes:

```sql
SELECT
    price - discount
FROM products;
```

Result:

```text
NULL
```

If the business meaning of NULL discount is:

```text
No discount
```

then you may need:

```sql
SELECT
    price - COALESCE(discount, 0)
FROM products;
```

But only if:

```text
NULL discount = 0 discount
```

is actually the business rule.

---

# 🧠 Never Automatically Convert NULL

This is one of the most important senior lessons.

Don't write:

```sql
COALESCE(column, 0)
```

just because:

> "NULL is causing problems."

Ask:

```text
What does NULL mean?
```

Maybe:

```text
Unknown
Not provided
Not applicable
Not yet calculated
Not available
Not loaded
```

These are different business meanings.

---

# 🔥 Real Production Incident

Imagine an analytics dashboard.

The database contains:

```text
customer_age

25
30
NULL
```

The analyst writes:

```sql
SELECT AVG(COALESCE(customer_age, 0))
FROM customers;
```

This treats missing age as:

```text
0
```

and produces a misleading average.

Correct:

```sql
SELECT AVG(customer_age)
FROM customers;
```

if the requirement is:

> Average age among customers whose age is known.

---

# 🧠 NULL Is a Modeling Decision

Before allowing NULL, ask:

```text
Can this value genuinely be missing?

Does NULL have a clear meaning?

Should the value be mandatory?

Can "unknown" be different from "not applicable"?

Should we use a separate status instead?
```

For example:

```text
phone_number = NULL
```

might mean:

```text
Not provided
```

But:

```text
verification_status = NULL
```

might be a poor design if the application really needs explicit states:

```text
PENDING
VERIFIED
FAILED
```

Sometimes an explicit state model is better than NULL.

---

# 🏗️ NULL vs Explicit State

Instead of:

```text
verified_at = NULL
```

you might have:

```text
verification_status
```

with:

```text
PENDING
VERIFIED
FAILED
```

Both can be valid designs.

The choice depends on:

```text
Business Semantics
Queries
Constraints
Lifecycle
Audit Requirements
```

---

# 🔥 Senior Principle

> **NULL should have a deliberate meaning, not be used as a generic garbage bucket.**

---

# 🧪 Hands-on Lab

## Lab 1 — Create Test Data

```sql
CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    city TEXT,
    salary NUMERIC(12,2),
    bonus NUMERIC(12,2)
);
```

Insert:

```sql
INSERT INTO employees
    (id, name, city, salary, bonus)
VALUES
    (1, 'Vinay', 'Hyderabad', 50000, 10000),
    (2, 'Rahul', 'Chennai', 60000, NULL),
    (3, 'Ravi', NULL, 70000, 5000),
    (4, 'Arun', NULL, NULL, NULL);
```

---

# Lab 2 — Find NULL

```sql
SELECT *
FROM employees
WHERE city IS NULL;
```

---

# Lab 3 — Incorrect NULL Comparison

```sql
SELECT *
FROM employees
WHERE city = NULL;
```

Observe the difference.

---

# Lab 4 — NOT NULL

```sql
SELECT *
FROM employees
WHERE city IS NOT NULL;
```

---

# Lab 5 — COUNT

Run:

```sql
SELECT COUNT(*)
FROM employees;
```

Then:

```sql
SELECT COUNT(bonus)
FROM employees;
```

Compare the results.

---

# Lab 6 — AVG

Run:

```sql
SELECT AVG(salary)
FROM employees;
```

Then:

```sql
SELECT AVG(COALESCE(salary, 0))
FROM employees;
```

Compare the results.

Ask:

> Which one represents the average salary of employees whose salary is known?

---

# Lab 7 — SUM

Run:

```sql
SELECT SUM(bonus)
FROM employees;
```

Then:

```sql
SELECT COALESCE(SUM(bonus), 0)
FROM employees;
```

Understand why the second query may be useful for reporting.

---

# Lab 8 — NOT IN

Create:

```sql
CREATE TABLE blocked_users (
    user_id BIGINT
);
```

Insert:

```sql
INSERT INTO blocked_users
VALUES
    (2),
    (3),
    (NULL);
```

Now test:

```sql
SELECT *
FROM employees
WHERE id NOT IN (
    SELECT user_id
    FROM blocked_users
);
```

Then compare with:

```sql
SELECT *
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM blocked_users b
    WHERE b.user_id = e.id
);
```

Understand the difference.

---

# Lab 9 — LEFT JOIN

Create:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT,
    status TEXT
);
```

Insert:

```sql
INSERT INTO orders
    (id, employee_id, status)
VALUES
    (101, 1, 'PAID'),
    (102, 2, 'PENDING');
```

Run:

```sql
SELECT
    e.name,
    o.id,
    o.status
FROM employees e
LEFT JOIN orders o
    ON e.id = o.employee_id;
```

Observe employees without orders.

---

# Lab 10 — LEFT JOIN + WHERE

Run:

```sql
SELECT
    e.name,
    o.id
FROM employees e
LEFT JOIN orders o
    ON e.id = o.employee_id
WHERE o.status = 'PAID';
```

Then compare:

```sql
SELECT
    e.name,
    o.id
FROM employees e
LEFT JOIN orders o
    ON e.id = o.employee_id
   AND o.status = 'PAID';
```

This is a critical exercise.

---

# 🎯 Interview Questions

## Beginner

### What is NULL?

NULL represents the absence of a value in SQL, with the precise business meaning determined by the application's data model.

---

### Is NULL equal to zero?

No.

---

### Is NULL equal to an empty string?

No.

---

### How do you check for NULL?

```sql
IS NULL
```

or:

```sql
IS NOT NULL
```

---

### Why doesn't `column = NULL` work?

Because comparisons involving NULL produce UNKNOWN rather than ordinary TRUE/FALSE equality.

---

# 🧠 Intermediate Questions

### What are the three SQL logical states?

```text
TRUE
FALSE
UNKNOWN
```

---

### Why does WHERE ignore UNKNOWN?

A WHERE filter retains rows only when the condition evaluates to TRUE.

---

### What is the difference between COUNT(*) and COUNT(column)?

```text
COUNT(*)
```

counts rows.

```text
COUNT(column)
```

counts non-NULL values of that column.

---

### What does COALESCE do?

It returns the first non-NULL expression.

Example:

```sql
COALESCE(city, 'Unknown')
```

---

### What does NULLIF do?

It returns NULL when its two arguments are equal; otherwise it returns the first argument.

---

# 👑 Senior Questions

### Why is `NOT IN` dangerous with NULL?

Because NULL can cause the comparison to evaluate to UNKNOWN, causing rows to be filtered out.

---

### Why can LEFT JOIN behave like INNER JOIN?

If a condition on the right-side table is placed in the `WHERE` clause, rows with no match have NULL values and may be filtered out.

---

### When should a condition go in ON instead of WHERE?

When the condition is part of determining which rows from the right side should match while preserving unmatched left-side rows in an outer join.

---

### Should NULL always be replaced using COALESCE?

No.

First determine what NULL means.

---

# 👑 20+ Year Experience Questions

## Question 1

Why can this query unexpectedly return zero rows?

```sql
SELECT *
FROM users
WHERE id NOT IN (
    SELECT user_id
    FROM blocked_users
);
```

Answer:

If `blocked_users.user_id` contains NULL, the `NOT IN` predicate can become UNKNOWN for candidate rows.

Investigate whether:

```text
NOT EXISTS
```

better represents the intended relationship.

---

# Question 2

Why is:

```sql
AVG(COALESCE(salary, 0))
```

potentially dangerous?

Because it treats missing salary as zero.

That changes the mathematical meaning of the data.

If NULL means:

```text
salary unknown
```

then zero is incorrect.

---

# Question 3

Why does:

```sql
COUNT(*)
```

return 100 while:

```sql
COUNT(email)
```

returns 95?

Five rows have:

```text
email = NULL
```

---

# Question 4

Why does:

```sql
SUM(amount)
```

sometimes return NULL instead of zero?

If there are no non-NULL values to aggregate, the result can be NULL.

Use:

```sql
COALESCE(SUM(amount), 0)
```

when the business/reporting requirement explicitly defines that case as zero.

---

# Question 5

Why does this:

```sql
LEFT JOIN orders o
    ON c.id = o.customer_id
WHERE o.status = 'PAID'
```

exclude customers with no orders?

Because unmatched rows have:

```text
o.status = NULL
```

and:

```text
NULL = 'PAID'
```

is UNKNOWN.

The WHERE clause removes UNKNOWN.

---

# Question 6

Should you avoid NULL entirely?

No.

NULL is useful when the absence of a value is meaningful.

The real question is:

> **Does NULL have a clearly defined semantic meaning in this column?**

---

# 🔥 Senior NULL Mental Model

Whenever you see NULL, ask:

```text
1. What does NULL mean here?

2. Is NULL allowed?

3. What happens during comparison?

4. What happens during AND/OR?

5. What happens during JOIN?

6. What happens during aggregation?

7. What happens during sorting?

8. What happens during UNIQUE constraints?

9. Does the application interpret NULL correctly?

10. Would an explicit state be better?
```

---

# 🧠 The NULL Decision Tree

When designing a column:

```text
Can the value be absent?
        │
        ├── NO
        │
        ▼
    NOT NULL
        │
        └── YES
             │
             ▼
       What does NULL mean?
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
    Unknown Missing N/A
       │     │     │
       └─────┴─────┘
             │
             ▼
     Is NULL the best model?
             │
       ┌─────┴─────┐
       ▼           ▼
      YES          NO
       │            │
       ▼            ▼
    Allow NULL   Explicit State
```

This is a database-design question, not merely a SQL-syntax question.

---

# 🔥 Real Production Debugging Checklist

If a SQL query involving NULL returns unexpected results:

```text
Step 1
↓
Find every nullable column.

Step 2
↓
Identify which expressions can produce NULL.

Step 3
↓
Evaluate each expression as:
TRUE / FALSE / UNKNOWN

Step 4
↓
Check WHERE conditions.

Step 5
↓
Check JOIN conditions.

Step 6
↓
Check NOT IN.

Step 7
↓
Check aggregate behavior.

Step 8
↓
Check COALESCE / CASE transformations.

Step 9
↓
Verify business meaning of NULL.

Step 10
↓
Test with actual NULL rows.
```

---

# 📌 Chapter Summary

NULL is not:

```text
0
''
FALSE
'NULL'
```

SQL uses:

```text
TRUE
FALSE
UNKNOWN
```

because NULL represents missing/absent information.

Use:

```sql
IS NULL
IS NOT NULL
```

instead of:

```sql
= NULL
<> NULL
```

Remember:

```text
NULL = NULL
       ↓
   UNKNOWN
```

and:

```text
NOT UNKNOWN
       ↓
   UNKNOWN
```

`WHERE` retains only:

```text
TRUE
```

---

# 🧠 Critical Rules

```text
Rule 1
------
Never use = NULL.
Use IS NULL.


Rule 2
------
Be extremely careful with NOT IN
when NULL can appear.


Rule 3
------
COUNT(*) ≠ COUNT(column).


Rule 4
------
SUM/AVG/MIN/MAX generally ignore NULL
values in their aggregate calculations.


Rule 5
------
Don't blindly COALESCE NULL to zero.


Rule 6
------
LEFT JOIN + WHERE on the right table
can eliminate unmatched rows.


Rule 7
------
NULL should have a deliberate business meaning.
```

---

# 🔥 Final Mental Model

```text
                       NULL
                        │
                        ▼
              Missing / Absent Value
                        │
                        ▼
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
        Comparison            Logic
              │                   │
              ▼                   ▼
          UNKNOWN          TRUE/FALSE/UNKNOWN
              │                   │
              └─────────┬─────────┘
                        ▼
                      WHERE
                        │
                ┌───────┴───────┐
                ▼               ▼
              TRUE         FALSE/UNKNOWN
                │               │
                ▼               ▼
              KEEP            REMOVE
```

And across SQL:

```text
NULL
 │
 ├── WHERE
 │
 ├── JOIN
 │
 ├── NOT IN
 │
 ├── COUNT
 │
 ├── SUM
 │
 ├── AVG
 │
 ├── ORDER BY
 │
 ├── UNIQUE
 │
 └── COALESCE / CASE
```

NULL is not a small SQL feature.

It affects:

```text
Query Correctness
Data Modeling
Reporting
JOINs
Aggregations
Constraints
Application Behavior
```

Once you understand NULL properly, a large class of SQL bugs becomes much easier to diagnose.

---

# 🚀 Next Chapter

# Chapter 8 — SQL Filtering, Predicates & Expressions

We will go deeper into how databases evaluate conditions:

```text
=
<>
>
<
>=
<=
AND
OR
NOT
IN
EXISTS
BETWEEN
LIKE
IS NULL
CASE
COALESCE
```

Then we'll solve real production problems involving:

```text
Dynamic Search APIs
Optional Filters
Date Ranges
User Search
Pagination
Status Filters
Multi-condition Queries
NULL-safe Filtering
```

and begin understanding:

> **Why a query that looks logically correct can still produce the wrong result or perform terribly at scale.**
