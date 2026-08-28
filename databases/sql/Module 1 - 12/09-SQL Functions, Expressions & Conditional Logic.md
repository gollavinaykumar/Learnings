# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 9 — SQL Functions, Expressions & Conditional Logic

> In the previous chapter, we learned how SQL filtering works:
>
> ```text
> WHERE
> AND
> OR
> NOT
> IN
> NOT IN
> BETWEEN
> LIKE
> EXISTS
> NOT EXISTS
> ```
>
> We also learned an important production principle:
>
> ```text
> SQL correctness
>        +
> Query performance
>        +
> Security
>        +
> Maintainability
> ```
>
> Now we move one level deeper.
>
> SQL is not only a language for retrieving rows.
>
> SQL can also **calculate, transform, classify, and derive data**.
>
> Imagine an e-commerce system storing:
>
> ```text
> price
> quantity
> discount
> tax
> created_at
> first_name
> last_name
> ```
>
> The application asks:
>
> ```text
> What is the final order amount?
> Which customers are VIP?
> How old is this customer?
> What month did this order happen?
> What should we display if a value is NULL?
> ```
>
> We could perform all these calculations in Java, Python, Node.js, or another application language.
>
> But SQL can perform many of them directly.
>
> This introduces an important architectural question:
>
> # Where should a calculation happen?
>
> ```text
> Database
>     ?
> Application
>     ?
> Cached / persisted data
> ```
>
> A developer with 1 year of experience asks:
>
> > "Can SQL do this?"
>
> A senior engineer asks:
>
> > "Where should this computation live, and what are the consequences?"

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* SQL expressions
* SQL functions
* Scalar functions
* Aggregate functions
* String functions
* Numeric functions
* Date/time functions
* `COALESCE`
* `NULLIF`
* `CASE`
* `CAST`
* Type conversion
* Calculated columns
* Conditional calculations
* Function composition
* NULL propagation
* Deterministic vs non-deterministic functions
* SQL-side calculations
* Application-side calculations
* Persisted calculations
* Real production problems
* Function-related performance issues

---

# 📖 What Is an SQL Expression?

An expression produces a value.

For example:

```sql
price * quantity
```

produces:

```text
total amount
```

Another:

```sql
first_name || ' ' || last_name
```

may produce:

```text
Vinay Kumar
```

Another:

```sql
price * 0.18
```

produces:

```text
tax
```

Think:

```text
Expression
    ↓
Input values
    ↓
Calculation
    ↓
Result value
```

---

# 🧠 Expression vs Statement

This distinction matters.

An SQL statement might be:

```sql
SELECT *
FROM products;
```

Inside it, an expression could be:

```sql
price * quantity
```

So:

```text
SQL Statement
      ↓
contains
      ↓
Expressions
```

---

# 1️⃣ Calculated Columns

Suppose:

```text
products

price | stock
------+------
100   | 10
500   | 20
```

We can calculate inventory value:

```sql
SELECT
    name,
    price,
    stock,
    price * stock AS inventory_value
FROM products;
```

Result:

```text
name      price   stock   inventory_value
--------- ------- ------- ----------------
Keyboard  100     10      1000
Monitor   500     20      10000
```

The column:

```text
inventory_value
```

does not necessarily exist physically in the table.

It is derived at query time.

---

# 🧠 Derived Data

Think:

```text
Stored Data
    ↓
price
stock
    ↓
Expression
    ↓
price * stock
    ↓
Derived Data
```

This is useful for:

```text
Reports
Analytics
API Responses
Dashboards
Calculations
```

---

# 2️⃣ Arithmetic Expressions

Basic operators:

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
    price * 1.18 AS price_with_tax
FROM products;
```

If:

```text
price = 1000
```

then:

```text
price_with_tax = 1180
```

---

# ⚠️ Numeric Data Types Matter

Suppose:

```text
price
```

is:

```text
NUMERIC(12,2)
```

Arithmetic behavior depends on the database's numeric type rules.

Do not assume every database handles:

```sql
10 / 3
```

the same way.

Some numeric types may produce:

```text
3
```

while others may produce:

```text
3.333...
```

depending on the operand types.

This is why senior engineers care about:

```text
Data Types
Precision
Scale
Casting
```

---

# 🔥 Money Calculation

Never casually use floating-point types for financial amounts.

Prefer appropriate exact numeric/decimal types supported by your database.

For example:

```sql
price NUMERIC(12,2)
```

rather than blindly using:

```text
FLOAT
```

for money.

Why?

Because financial calculations require predictable decimal precision.

---

# 3️⃣ String Functions

Different databases provide different string functions.

Common operations include:

```text
UPPER
LOWER
LENGTH
TRIM
SUBSTRING
REPLACE
CONCAT
```

Exact names and behavior can vary by SQL dialect.

---

# UPPER

```sql
SELECT
    UPPER(name) AS name_upper
FROM customers;
```

Example:

```text
Vinay
```

becomes:

```text
VINAY
```

---

# LOWER

```sql
SELECT
    LOWER(email) AS normalized_email
FROM customers;
```

Example:

```text
VINAY@EXAMPLE.COM
```

becomes:

```text
vinay@example.com
```

---

# 🧠 Why Normalize Data?

Suppose users enter:

```text
Vinay@Example.com
VINAY@example.com
vinay@example.com
```

If your business treats these as the same logical email, you need a deliberate normalization strategy.

One option is to store normalized data.

Another is to use database features appropriate for case-insensitive comparison.

Do not assume:

```sql
LOWER(email)
```

is automatically the best production solution.

---

# 🔥 Real Production Problem

Suppose you have:

```text
100 million users
```

with:

```text
INDEX(email)
```

Then:

```sql
WHERE email = ?
```

may be able to use the index efficiently.

But:

```sql
WHERE LOWER(email) = ?
```

may not be able to use that ordinary index efficiently.

Possible solutions include:

```text
Functional / Expression Index
Normalized Column
Case-insensitive Data Type
Database-specific Index Strategy
```

The correct approach depends on the database.

---

# 4️⃣ TRIM

User input often contains unwanted whitespace.

Example:

```text
'   Vinay   '
```

Using:

```sql
TRIM(name)
```

can produce:

```text
'Vinay'
```

Useful for:

```text
Data Cleaning
Imports
User Input
Reports
ETL
```

---

# 🚨 But Don't Blindly Modify Stored Data

Suppose:

```text
address = 'Flat  101'
```

Whitespace may sometimes carry meaning.

Therefore:

```text
TRIM everything
```

is not automatically safe.

Understand the domain first.

---

# 5️⃣ CONCAT

Some databases support:

```sql
CONCAT(first_name, ' ', last_name)
```

Example:

```text
Vinay Kumar
```

This is useful for presentation:

```sql
SELECT
    CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;
```

---

# ⚠️ NULL and String Functions

Suppose:

```text
first_name = 'Vinay'
last_name = NULL
```

Different string concatenation operators/functions can have different NULL behavior.

For example, in some SQL dialects:

```sql
first_name || ' ' || last_name
```

may produce:

```text
NULL
```

while:

```sql
CONCAT(first_name, ' ', last_name)
```

may treat NULL differently.

Always check your database's semantics.

---

# 6️⃣ LENGTH

Example:

```sql
SELECT
    name,
    LENGTH(name) AS name_length
FROM customers;
```

Useful for validation and reporting.

But remember:

```text
characters
```

and:

```text
bytes
```

are not always the same thing.

Unicode makes this especially important.

---

# 🌍 Real Production Problem

Suppose a system accepts:

```text
Name
```

with:

```text
Indian languages
Chinese
Japanese
Arabic
Emoji
```

Then:

```text
character length
```

may differ from:

```text
byte length
```

Database encoding and string functions matter.

---

# 7️⃣ Numeric Functions

Common numeric functions include:

```text
ABS
ROUND
CEIL / CEILING
FLOOR
MOD
POWER
```

Exact function names vary by database.

---

# ABS

```sql
SELECT ABS(-500);
```

Result:

```text
500
```

Useful when you need magnitude rather than sign.

---

# ROUND

```sql
SELECT ROUND(1234.5678, 2);
```

Conceptually:

```text
1234.57
```

Useful for presentation or controlled calculations.

But:

> Do not round financial data at arbitrary stages.

Rounding rules are business rules.

---

# 🔥 Financial Example

Suppose:

```text
Item price = ₹999.99
Tax = 18%
```

You calculate:

```text
999.99 × 0.18
```

which produces:

```text
179.9982
```

Should the tax be:

```text
₹180.00
```

?

Probably.

But the exact answer depends on:

```text
Tax rules
Currency
Rounding policy
Invoice requirements
```

SQL can calculate the number.

The business defines what the number should mean.

---

# 8️⃣ FLOOR

```sql
SELECT FLOOR(19.8);
```

Result:

```text
19
```

---

# 9️⃣ CEILING

```sql
SELECT CEILING(19.2);
```

Result:

```text
20
```

Useful for:

```text
Pagination
Capacity calculations
Packaging
Resource allocation
```

---

# 🔥 Real Pagination Calculation

Suppose:

```text
total_items = 101
page_size = 10
```

Number of pages:

```text
CEILING(101 / 10)
```

But remember:

```text
101 / 10
```

can behave differently depending on operand types.

You may need explicit numeric conversion to ensure the intended division semantics.

This is why:

```text
Types
```

matter even in simple-looking calculations.

---

# 🔟 Date and Time Functions

Date/time operations are among the most important SQL functions.

Common operations include:

```text
CURRENT_DATE
CURRENT_TIMESTAMP
EXTRACT
DATE_TRUNC
DATE_ADD / INTERVAL
DATE_DIFF
```

Exact syntax varies significantly between databases.

---

# CURRENT_TIMESTAMP

Example:

```sql
SELECT CURRENT_TIMESTAMP;
```

Returns the current timestamp according to the database/session semantics.

---

# ⚠️ Important

Do not assume:

```text
Database clock
=
Application server clock
=
User's local clock
```

They may be different.

Modern distributed systems may have:

```text
Application servers
Database servers
Containers
Cloud services
Users
```

running in different time zones.

---

# 🧠 Store Time Carefully

For distributed applications, you need a deliberate policy for:

```text
Timezone
UTC
Timestamp precision
Display timezone
Daylight saving
```

A common architecture is:

```text
Database
    ↓
Store canonical time
    ↓
Application
    ↓
Convert for user
```

The exact implementation depends on your database and application architecture.

---

# 1️⃣1️⃣ EXTRACT

Suppose:

```text
created_at
```

contains:

```text
2026-08-28 14:35:00
```

You may want:

```text
year
month
day
hour
```

Some databases support syntax such as:

```sql
SELECT
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at)
FROM orders;
```

Result conceptually:

```text
2026
8
```

---

# 🔥 Reporting Problem

Requirement:

> "How many orders were created in each month?"

One approach:

```sql
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    EXTRACT(YEAR FROM created_at),
    EXTRACT(MONTH FROM created_at);
```

This works.

But later we'll study an important performance issue:

```text
Function on indexed column
```

and why range-based filtering is often preferable.

---

# 1️⃣2️⃣ DATE_TRUNC

Some databases provide:

```sql
DATE_TRUNC(...)
```

which can truncate timestamps to:

```text
day
week
month
year
```

For example:

```sql
DATE_TRUNC('month', created_at)
```

can conceptually convert:

```text
2026-08-28 14:35
```

to:

```text
2026-08-01 00:00
```

depending on database semantics and timezone context.

This is useful for grouping:

```text
Monthly Revenue
Weekly Orders
Daily Users
```

---

# 1️⃣3️⃣ COALESCE

We studied this in the previous chapter.

```sql
COALESCE(value, fallback)
```

Example:

```sql
SELECT
    name,
    COALESCE(phone, 'Not Provided') AS phone
FROM customers;
```

If:

```text
phone = NULL
```

result:

```text
Not Provided
```

---

# 🧠 Multiple COALESCE Values

```sql
COALESCE(
    preferred_name,
    first_name,
    username,
    'Unknown'
)
```

SQL chooses the first non-NULL value.

Think:

```text
preferred_name?
   ↓
YES → use it
NO
 ↓
first_name?
 ↓
YES → use it
NO
 ↓
username?
 ↓
YES → use it
NO
 ↓
Unknown
```

---

# 1️⃣4️⃣ NULLIF

Recall:

```sql
NULLIF(a, b)
```

If:

```text
a = b
```

then:

```text
NULL
```

otherwise:

```text
a
```

Example:

```sql
NULLIF(quantity, 0)
```

Useful for preventing:

```text
division by zero
```

Example:

```sql
SELECT
    revenue / NULLIF(order_count, 0)
FROM daily_sales;
```

---

# 1️⃣5️⃣ CASE

`CASE` is one of the most powerful SQL expressions.

Example:

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 100000 THEN 'PREMIUM'
        WHEN price >= 50000 THEN 'HIGH'
        WHEN price >= 10000 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS price_category
FROM products;
```

---

# 🧠 CASE Works Like Conditional Logic

Conceptually:

```text
IF price >= 100000
    PREMIUM

ELSE IF price >= 50000
    HIGH

ELSE IF price >= 10000
    MEDIUM

ELSE
    LOW
```

---

# 🔥 Order Matters

Consider:

```sql
CASE
    WHEN price >= 10000 THEN 'MEDIUM'
    WHEN price >= 100000 THEN 'PREMIUM'
    ELSE 'LOW'
END
```

A price of:

```text
150000
```

matches:

```text
price >= 10000
```

first.

So it becomes:

```text
MEDIUM
```

not:

```text
PREMIUM
```

Therefore:

> **CASE conditions are evaluated in order.**

Put more specific conditions before broader conditions when required.

---

# 1️⃣6️⃣ CASE With NULL

Example:

```sql
SELECT
    name,
    CASE
        WHEN city IS NULL THEN 'Unknown'
        ELSE city
    END AS city
FROM customers;
```

---

# 1️⃣7️⃣ Simple CASE

You may also see:

```sql
CASE status
    WHEN 'PENDING' THEN 'Waiting'
    WHEN 'PAID' THEN 'Completed'
    WHEN 'FAILED' THEN 'Error'
    ELSE 'Unknown'
END
```

This is useful when comparing one expression against multiple values.

---

# 🔥 Real Order Dashboard

Database stores:

```text
PENDING
PAID
SHIPPED
DELIVERED
CANCELLED
```

Dashboard wants:

```text
PENDING   → Waiting
PAID      → Payment Complete
SHIPPED   → In Transit
DELIVERED  → Complete
CANCELLED → Cancelled
```

SQL:

```sql
SELECT
    id,
    CASE status
        WHEN 'PENDING' THEN 'Waiting'
        WHEN 'PAID' THEN 'Payment Complete'
        WHEN 'SHIPPED' THEN 'In Transit'
        WHEN 'DELIVERED' THEN 'Complete'
        WHEN 'CANCELLED' THEN 'Cancelled'
        ELSE 'Unknown'
    END AS display_status
FROM orders;
```

---

# 1️⃣8️⃣ CAST

Sometimes you need to convert one data type into another.

Example:

```sql
CAST(price AS INTEGER)
```

Conceptually:

```text
NUMERIC
   ↓
INTEGER
```

Exact conversion rules vary by database.

---

# 🧠 Why Type Conversion Matters

Suppose:

```text
price = 99.99
```

Casting to integer may produce:

```text
99
```

depending on the database's conversion rules.

You need to understand:

```text
Rounding
Truncation
Overflow
Precision
Scale
```

before using casts in financial calculations.

---

# 🔥 String → Number

Suppose an import system receives:

```text
"5000"
```

as text.

You may need:

```sql
CAST('5000' AS INTEGER)
```

But if input contains:

```text
"five thousand"
```

the conversion may fail.

This is why data validation belongs before critical transformations.

---

# 1️⃣9️⃣ Implicit vs Explicit Conversion

Suppose:

```text
price
```

is numeric.

Application sends:

```text
"5000"
```

Some database systems may implicitly convert types.

That can be convenient.

But relying heavily on implicit conversion can create:

```text
Performance surprises
Index issues
Unexpected comparison behavior
Data bugs
```

Senior engineers prefer explicit and predictable types at system boundaries.

---

# 🧠 Type System Mental Model

```text
Application
     ↓
Parameter Type
     ↓
Database Column Type
     ↓
Expression Type
     ↓
Result Type
```

Every step matters.

---

# 2️⃣0️⃣ Function Composition

Functions can be combined.

Example:

```sql
COALESCE(
    TRIM(name),
    'Unknown'
)
```

Or:

```sql
UPPER(
    TRIM(name)
)
```

Or:

```sql
ROUND(
    price * tax_rate,
    2
)
```

Think:

```text
Input
 ↓
TRIM
 ↓
UPPER
 ↓
Result
```

---

# 🚨 Don't Create Impossible-to-Understand SQL

You can technically write:

```sql
COALESCE(
    UPPER(
        TRIM(
            REPLACE(
                CONCAT(...)
            )
        )
    ),
    ...
)
```

But if nobody can understand it:

```text
Maintainability
    ↓
BAD
```

Senior SQL is not about maximizing the number of functions in one statement.

It's about making the logic:

```text
Correct
Readable
Testable
Maintainable
Performant
```

---

# 2️⃣1️⃣ SQL Function vs Application Function

Suppose you need:

```text
Customer full name
```

Should SQL calculate:

```sql
CONCAT(first_name, ' ', last_name)
```

or should Java calculate it?

There is no universal answer.

Ask:

```text
Who owns the transformation?

Is it needed by many consumers?

Is it presentation-only?

Does the database need the result for filtering?

Does it need indexing?

Is it expensive?

Does it need localization?
```

---

# 🧠 Example

If:

```text
full_name
```

is only for UI display:

```text
Application
```

may be a better place.

But if:

```text
normalized_search_name
```

is part of a database search strategy:

```text
Database design
```

may need to account for it.

---

# 2️⃣2️⃣ Derived Data vs Stored Data

Suppose:

```text
price
quantity
```

exist.

You can calculate:

```text
total = price * quantity
```

Should you store:

```text
total
```

as another column?

Maybe.

---

# Option A — Calculate

```sql
SELECT
    price * quantity AS total
FROM order_items;
```

Advantages:

```text
Always derived from current source values
No duplicated value
No synchronization problem
```

---

# Option B — Store

```text
total
```

Advantages can include:

```text
Faster reads
Historical snapshot
Stable value
Expensive computation avoided
```

But now:

```text
price
quantity
total
```

can become inconsistent.

---

# 🔥 The Duplication Problem

Suppose:

```text
price = 100
quantity = 5
total = 500
```

Then application updates:

```text
quantity = 10
```

but forgets:

```text
total
```

Now:

```text
price = 100
quantity = 10
total = 500
```

Invalid state.

This is:

# Data Duplication Risk

---

# 🧠 But Sometimes Storing Derived Values Is Correct

Imagine an invoice:

```text
Product Price
Tax
Discount
Final Amount
```

The business may require the invoice to preserve:

```text
The exact amount charged at that historical moment.
```

If today's product price changes:

```text
Product price today ≠ historical invoice price
```

Therefore storing historical values is correct.

This is not merely a performance optimization.

It is a domain requirement.

---

# 👑 Senior Principle

> **Don't ask only "Can this value be calculated?" Ask "Is this value supposed to be derived or historically recorded?"**

---

# 2️⃣3️⃣ Deterministic Functions

A deterministic function produces the same result for the same inputs.

Example conceptually:

```text
ABS(-10)
```

always gives:

```text
10
```

But functions involving current time:

```sql
CURRENT_TIMESTAMP
```

depend on execution context/time.

Other functions may depend on:

```text
Randomness
Session state
Locale
Timezone
Database configuration
```

This matters for:

```text
Indexes
Generated columns
Caching
Replication
Materialization
Query optimization
```

Exact restrictions vary by database.

---

# 🔥 Why Determinism Matters

Suppose you want to store:

```text
generated column
```

based on:

```text
price * quantity
```

That is deterministic.

But:

```text
CURRENT_TIMESTAMP
```

changes over time.

Therefore database systems may impose different restrictions on what expressions can be used in:

```text
Generated Columns
Indexes
Constraints
Materialized Structures
```

Always check your database's rules.

---

# 2️⃣4️⃣ Function in WHERE

Consider:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'vinay@example.com';
```

The function is applied to:

```text
email
```

This can affect index usage.

Compare:

```sql
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

The second form is usually easier for a normal index on `email`.

But the correct optimization depends on your database and indexing strategy.

---

# 2️⃣5️⃣ Function on Date Column

Consider:

```sql
SELECT *
FROM orders
WHERE DATE(created_at) = '2026-08-28';
```

This may be less index-friendly than:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-08-28'
  AND created_at < '2026-08-29';
```

The second expresses a direct range.

---

# 🧠 Important Senior Concept

When you write:

```sql
WHERE function(column) = value
```

ask:

```text
Can the database still use an efficient access path?
```

Don't automatically assume:

```text
Function = Slow
```

or:

```text
Function = No Index
```

The real answer depends on:

```text
Database
Index Type
Function
Optimizer
Statistics
Query
Data Distribution
```

---

# 2️⃣6️⃣ Aggregate Functions

Functions such as:

```text
COUNT
SUM
AVG
MIN
MAX
```

operate over sets of rows.

Example:

```sql
SELECT
    AVG(price)
FROM products;
```

This differs from:

```sql
SELECT
    price * 1.18
FROM products;
```

The first is:

```text
Aggregate
```

The second is:

```text
Row-level expression
```

---

# 🧠 Row-Level vs Aggregate

```text
Each row
   ↓
price * quantity
   ↓
One result per row
```

while:

```text
Many rows
   ↓
SUM(price)
   ↓
One result for the group
```

We'll study aggregates and grouping deeply in the next module.

---

# 2️⃣7️⃣ CASE + Aggregate

A powerful pattern:

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'PAID' THEN amount
            ELSE 0
        END
    ) AS paid_amount
FROM orders;
```

Conceptually:

```text
For each order:

PAID
 ↓
amount

Anything else
 ↓
0

Then SUM everything
```

This is sometimes called:

```text
Conditional Aggregation
```

It is extremely useful in reporting.

---

# 🔥 Real Dashboard

Requirement:

```text
Total Orders
Paid Orders
Pending Orders
Cancelled Orders
```

You can calculate them together:

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

One query can produce a dashboard summary.

---

# 2️⃣8️⃣ NULL + CASE

Consider:

```sql
CASE
    WHEN discount > 100 THEN 'HIGH'
    ELSE 'NORMAL'
END
```

If:

```text
discount = NULL
```

then:

```text
discount > 100
```

is:

```text
UNKNOWN
```

The condition isn't TRUE.

Therefore the `ELSE` branch is used.

This connects our previous chapter directly to today's topic.

---

# 🧠 SQL Concepts Build on Each Other

We have now learned:

```text
NULL
   ↓
Three-Valued Logic
   ↓
Predicates
   ↓
Expressions
   ↓
Functions
   ↓
CASE
```

SQL is not a collection of isolated commands.

The concepts interact.

---

# 🏗️ Real Production Example

Suppose an order has:

```text
quantity = 3
unit_price = 2500
discount = NULL
tax_rate = 18%
```

Business rule:

```text
NULL discount means no discount.
```

Final amount:

```sql
SELECT
    (
        quantity * unit_price
        - COALESCE(discount, 0)
    )
    *
    (1 + tax_rate / 100.0)
    AS final_amount
FROM order_items;
```

Now ask:

```text
Is NULL really zero?
What numeric type is tax_rate?
What precision is required?
When should rounding happen?
Should tax be calculated before or after discount?
Should the final amount be stored?
What happens if quantity is negative?
What happens if tax_rate is NULL?
```

This is where SQL becomes software engineering.

---

# 👑 20+ Year Experience Thinking

A junior developer sees:

```sql
price * quantity
```

A senior developer sees:

```text
Data Type
Precision
NULL
Business Meaning
Rounding
Currency
Concurrency
Historical State
Performance
Indexing
Auditability
```

The expression is easy.

The **semantics** are hard.

---

# 🧪 Hands-on Lab

## Lab 1 — Arithmetic

```sql
SELECT
    name,
    price,
    stock,
    price * stock AS inventory_value
FROM products;
```

---

# Lab 2 — String Functions

```sql
SELECT
    name,
    UPPER(name) AS uppercase_name,
    LOWER(name) AS lowercase_name,
    TRIM(name) AS trimmed_name
FROM products;
```

---

# Lab 3 — COALESCE

```sql
SELECT
    name,
    COALESCE(description, 'No Description') AS description
FROM products;
```

---

# Lab 4 — CASE

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 100000 THEN 'PREMIUM'
        WHEN price >= 50000 THEN 'HIGH'
        WHEN price >= 10000 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS category
FROM products;
```

---

# Lab 5 — NULLIF

```sql
SELECT
    revenue,
    order_count,
    revenue / NULLIF(order_count, 0) AS average_order_value
FROM daily_sales;
```

---

# Lab 6 — Date Extraction

Use the date/time syntax appropriate to your database.

Conceptually:

```sql
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month
FROM orders;
```

---

# Lab 7 — Type Conversion

```sql
SELECT
    CAST(price AS INTEGER)
FROM products;
```

Observe exactly how your database handles:

```text
99.99
```

---

# Lab 8 — Conditional Aggregation

```sql
SELECT
    COUNT(*) AS total,
    SUM(
        CASE
            WHEN status = 'PAID'
            THEN 1
            ELSE 0
        END
    ) AS paid
FROM orders;
```

---

# Lab 9 — Function and Index

If you have:

```text
INDEX(email)
```

compare:

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'vinay@example.com';
```

with:

```sql
EXPLAIN
SELECT *
FROM users
WHERE LOWER(email) = 'vinay@example.com';
```

Ask:

```text
Did the access path change?
Why?
```

---

# Lab 10 — Timestamp Filtering

Compare:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE DATE(created_at) = '2026-08-28';
```

with:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE created_at >= '2026-08-28'
  AND created_at < '2026-08-29';
```

Understand how expressions can influence index access.

---

# 🎯 Interview Questions

## Beginner

### What is an SQL expression?

An expression is a combination of values, operators, columns, and/or functions that produces a value.

---

### What is a scalar function?

A function that operates on individual input values and produces a result for each row/context.

Examples include:

```text
UPPER
LOWER
ABS
ROUND
COALESCE
```

---

### What is CASE?

A conditional expression that chooses a result based on conditions.

---

### What does COALESCE do?

Returns the first non-NULL expression.

---

### What does NULLIF do?

Returns NULL when its two arguments are equal; otherwise returns the first argument.

---

### What does CAST do?

Converts an expression from one data type to another according to the database's conversion rules.

---

# 🧠 Intermediate Questions

### What is the difference between a row-level expression and an aggregate?

Row-level expressions produce values for individual rows.

Aggregates operate across sets of rows and produce summary values.

---

### Why can functions on indexed columns hurt performance?

Because transforming the indexed column can prevent the optimizer from using a normal index access path efficiently.

---

### Why use:

```sql
created_at >= start
AND created_at < end
```

instead of:

```sql
DATE(created_at) = date
```

For timestamp filtering, the range predicate can preserve direct access to the original timestamp column and avoid applying a function to every candidate value.

---

### Why is `LOWER(email)` potentially problematic?

Because a normal index on `email` may not directly support the transformed expression.

Possible solutions include:

```text
Expression Index
Normalized Column
Case-insensitive Index/Type
```

depending on the database.

---

# 👑 Senior Questions

### Should every calculation happen in SQL?

No.

Decide based on:

```text
Data ownership
Reuse
Performance
Indexing
Business semantics
Historical requirements
API design
Maintainability
```

---

### When should a calculated value be stored?

When the value represents something that must be:

```text
Historically preserved
Expensive to calculate
Frequently queried
Stable at a specific point in time
Required for domain semantics
```

But storing derived values introduces synchronization concerns.

---

### What is the biggest problem with duplicated calculated data?

It can become inconsistent with its source values.

---

### Is rounding a technical decision?

Not always.

For financial systems, rounding can be a business and regulatory requirement.

---

# 👑 20+ Year Experience Questions

## Question 1

You have:

```text
price
quantity
total
```

Should `total` be stored?

Answer:

It depends.

If:

```text
total = current price × current quantity
```

and can safely be derived:

```sql
price * quantity
```

may be preferable.

But if:

```text
total
```

represents:

> The exact amount charged historically.

then storing it may be necessary.

---

# Question 2

Why shouldn't you blindly use:

```sql
ROUND(...)
```

everywhere?

Because rounding changes values.

You must define:

```text
When to round
How to round
Number of decimal places
Currency rules
Accumulation rules
```

before implementing it.

---

# Question 3

Why is:

```sql
COALESCE(discount, 0)
```

not merely a technical NULL fix?

Because it changes the semantic interpretation from:

```text
discount is unknown/missing
```

to:

```text
discount is zero
```

Only do this if the business meaning supports it.

---

# Question 4

Why can:

```sql
LOWER(email)
```

cause a performance problem?

Because the database may have an index on:

```text
email
```

but the query is searching:

```text
LOWER(email)
```

which is a different expression.

The solution may require an expression index or normalized data strategy.

---

# Question 5

Why is:

```sql
CURRENT_TIMESTAMP
```

different from:

```sql
price * quantity
```

from a determinism perspective?

Because:

```text
price * quantity
```

depends only on its input values.

Current-time functions depend on execution time/session/database semantics.

This matters for:

```text
Caching
Generated Columns
Indexes
Materialization
Replication
```

depending on the database.

---

# 🔥 Real Production Scenario

Suppose an order contains:

```text
unit_price = 999.99
quantity = 3
discount = NULL
tax_rate = 18
```

Business says:

```text
NULL discount = zero
Tax applies after discount
Final amount rounded to 2 decimals
```

A possible calculation is:

```sql
SELECT
    ROUND(
        (
            quantity * unit_price
            - COALESCE(discount, 0)
        )
        *
        (1 + tax_rate / 100.0),
        2
    ) AS final_amount
FROM order_items;
```

But a senior engineer still asks:

```text
Is tax_rate nullable?

What is the exact numeric type?

What happens for negative quantity?

What happens for refunds?

Should discount be percentage or fixed amount?

Should tax be rounded per item or on the invoice total?

Should final_amount be persisted?

What happens if the product price changes later?

Does currency have 2 decimal places?

What does the legal invoice require?
```

The SQL expression is only the beginning.

---

# 🧠 The Calculation Architecture Decision

Use this mental model:

```text
                    CALCULATION
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
        Database     Application   Persisted
            │            │            │
            ▼            ▼            ▼
       Query-time     Runtime       Stored
        Derived       Derived       Snapshot
```

Ask:

```text
Is it needed for filtering?
        ↓
Database may be appropriate

Is it presentation-only?
        ↓
Application may be appropriate

Is it historical?
        ↓
Persisted value may be appropriate

Is it expensive and reused heavily?
        ↓
Consider materialization/cache/precomputation
```

---

# 🔥 Senior SQL Mental Model

When you see:

```sql
SELECT ...
```

don't only see syntax.

See:

```text
Data
 ↓
Types
 ↓
NULL semantics
 ↓
Expression
 ↓
Function
 ↓
Result
 ↓
Business meaning
 ↓
Performance
 ↓
Storage decision
```

---

# 📌 Chapter Summary

SQL expressions allow you to:

```text
Calculate
Transform
Normalize
Classify
Convert
Aggregate
Format
```

Important functions/concepts:

```text
UPPER
LOWER
TRIM
CONCAT
LENGTH

ABS
ROUND
FLOOR
CEILING

CURRENT_TIMESTAMP
EXTRACT
DATE_TRUNC

COALESCE
NULLIF
CASE
CAST
```

---

# 🔥 Critical Rules

```text
Rule 1
------
Understand the data type before doing arithmetic.


Rule 2
------
Don't blindly use FLOAT for money.


Rule 3
------
NULL handling is part of calculation semantics.


Rule 4
------
COALESCE changes meaning if NULL does not mean zero.


Rule 5
------
CASE conditions are evaluated in order.


Rule 6
------
Don't blindly apply functions to indexed columns.


Rule 7
------
Use explicit type conversions when predictability matters.


Rule 8
------
Rounding is often a business rule.


Rule 9
------
Not every calculation belongs in SQL.


Rule 10
------
Not every calculated value should be stored.


Rule 11
------
Historical values may need to be persisted even
when they are mathematically derivable.


Rule 12
------
Always separate:
"Can SQL calculate this?"
from:
"Should SQL calculate this?"
```

---

# 🧠 Final Mental Model

```text
                  RAW DATA
                     │
                     ▼
                  DATA TYPES
                     │
                     ▼
               NULL SEMANTICS
                     │
                     ▼
                EXPRESSIONS
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       String     Numeric      Date
       Functions  Functions    Functions
          │          │          │
          └──────────┼──────────┘
                     ▼
              CONDITIONAL LOGIC
                     │
                   CASE
                     │
                     ▼
                TYPE CASTING
                     │
                     ▼
               DERIVED VALUE
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Database   Application  Stored
        Query      Runtime     Snapshot
          │          │          │
          └──────────┼──────────┘
                     ▼
              BUSINESS RESULT
```

The real skill is not memorizing:

```text
UPPER()
ROUND()
COALESCE()
CASE()
CAST()
```

The real skill is understanding:

> **What does this expression mean, what happens with NULL and data types, where should the computation live, and will the design remain correct and efficient when the system grows?**

That is the mindset required to move from writing SQL to **engineering database systems**.

---

# 🚀 Next Chapter

# Chapter 10 — SQL Aggregation, GROUP BY, HAVING & Analytical Thinking

We will move from:

```text
One Row
   ↓
Expression
   ↓
Result
```

to:

```text
Millions of Rows
       ↓
GROUP BY
       ↓
Aggregation
       ↓
Business Metrics
```

We will deeply understand:

```text
COUNT
SUM
AVG
MIN
MAX
GROUP BY
HAVING
WHERE vs HAVING
GROUPING
Conditional Aggregation
DISTINCT Aggregates
NULL + Aggregates
```

And solve real production problems:

```text
Daily Revenue
Monthly Sales
Top Customers
Order Counts
Average Order Value
Conversion Metrics
Failed Transactions
Revenue by Region
Customer Segmentation
```

including the senior-level question:

> **Why can a query that returns the correct aggregation still be fundamentally wrong for a production analytics system?**
