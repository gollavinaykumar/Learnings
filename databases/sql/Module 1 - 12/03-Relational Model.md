# 🗄️ SQL Mastery — 20+ Years Experience

# Module 1 — Database Fundamentals

# Chapter 3 — The Relational Model

> In the previous chapter, we learned that a database is much more than a collection of tables.
>
> We saw the architecture:
>
> ```text
> Application
>      ↓
>     SQL
>      ↓
> Database Server
>      ↓
> Query Engine
>      ↓
> Optimizer
>      ↓
> Executor
>      ↓
> Storage Engine
>      ↓
> Disk
> ```
>
> Now we need to understand **why relational databases organize data into tables and relationships**.
>
> This chapter introduces the foundation underneath SQL:
>
> # The Relational Model
>
> Once you understand the relational model, concepts like:
>
> ```text
> JOIN
> PRIMARY KEY
> FOREIGN KEY
> UNIQUE
> NORMALIZATION
> CONSTRAINTS
> ```
>
> stop looking like isolated SQL features.
>
> They become consequences of how relational data is modeled.

---

# 🎯 Learning Objectives

By the end of this chapter, you'll understand:

* What the relational model is
* What a relation is
* What a tuple is
* What an attribute is
* What a domain is
* What a relation schema is
* What a relation instance is
* What a key is
* What a super key is
* What a candidate key is
* What a primary key is
* What a foreign key is
* What referential integrity means
* One-to-one relationships
* One-to-many relationships
* Many-to-many relationships
* Cardinality
* Why relationships matter
* Why relational databases use keys
* How relational thinking leads to JOINs

---

# 📖 The Problem: How Should We Organize Data?

Imagine we are building an e-commerce application.

We need to store:

```text
Customers
Products
Orders
Payments
```

A beginner might create one huge table:

```text
orders

order_id
customer_name
customer_email
customer_phone
product_name
product_price
payment_status
payment_date
```

Example:

```text
101 | Vinay | vinay@example.com | 9999999999 | iPhone | 80000 | PAID
102 | Vinay | vinay@example.com | 9999999999 | MacBook | 120000 | PAID
103 | Rahul | rahul@example.com | 8888888888 | iPad | 60000 | PENDING
```

At first:

```text
Looks simple.
```

But then we ask:

> What if Vinay places 10,000 orders?

We repeat:

```text
Vinay
vinay@example.com
9999999999
```

10,000 times.

Now imagine Vinay changes his phone number.

We need to update:

```text
10,000 rows
```

This is exactly the kind of problem relational modeling tries to solve.

---

# 🧠 Relational Thinking

Instead of putting everything into one giant structure, separate different concepts.

```text
customers

id | name  | email
---+-------+------------------
1  | Vinay | vinay@example.com
2  | Rahul | rahul@example.com
```

Products:

```text
products

id | name    | price
---+---------+-------
10 | iPhone  | 80000
11 | MacBook | 120000
12 | iPad    | 60000
```

Orders:

```text
orders

id  | customer_id
----+------------
101 | 1
102 | 1
103 | 2
```

Now:

```text
customers
     ↓
customer_id
     ↓
orders
```

This is relational thinking.

---

# 🔗 Why Relationships Exist

The application needs to answer:

> Which customer owns order 101?

Database:

```text
orders

id = 101
customer_id = 1
```

Then:

```text
customers

id = 1
name = Vinay
```

So:

```text
Order 101
    ↓
customer_id = 1
    ↓
Customer 1
    ↓
Vinay
```

The relationship is represented using keys.

---

# 🧠 What Is the Relational Model?

The relational model is a way of representing data using:

```text
Relations
```

A relation is commonly represented as:

```text
Table
```

A relation consists conceptually of:

```text
Attributes
+
Tuples
```

In practical SQL terminology:

```text
Relation
   ↓
Table

Tuple
   ↓
Row

Attribute
   ↓
Column
```

These mappings are useful, but the relational model has more precise mathematical definitions than everyday SQL terminology.

---

# 📋 What Is a Relation?

Consider:

```text
customers

id | name  | email
---+-------+------------------
1  | Vinay | vinay@example.com
2  | Rahul | rahul@example.com
3  | Ravi  | ravi@example.com
```

Conceptually, this is a relation.

Think:

```text
Relation
    ↓
Set of tuples
```

Each tuple represents one combination of attribute values.

---

# 🧱 What Is a Tuple?

A tuple is conceptually one row in a relation.

Example:

```text
(1, 'Vinay', 'vinay@example.com')
```

This represents one customer.

In SQL, we normally call it:

```text
Row
```

So:

```text
Relation
   ↓
Tuples
   ↓
Rows
```

---

# 🧱 What Is an Attribute?

An attribute describes one property of the relation.

Example:

```text
customers

id
name
email
```

These are attributes.

In SQL:

```text
Attribute
   ↓
Column
```

---

# 🧩 What Is a Domain?

A domain defines the valid set of values an attribute can take.

For example:

```text
salary
```

might have a domain such as:

```text
Positive numeric values
```

while:

```text
email
```

might have a domain based on an appropriate string representation and validation rules.

In SQL, data types provide part of this concept.

For example:

```sql
salary NUMERIC
```

and:

```sql
age INTEGER
```

But remember:

> A SQL data type alone does not necessarily enforce every business rule for a domain.

For example:

```sql
age INTEGER
```

does not automatically mean:

```text
age > 0
```

You may need:

```sql
CHECK (age > 0)
```

---

# 🧠 Relation Schema vs Relation Instance

This is an important distinction.

Suppose we define:

```sql
CREATE TABLE customers (
    id BIGINT,
    name TEXT,
    email TEXT
);
```

The structure is:

```text
customers
├── id
├── name
└── email
```

This describes the shape of the relation.

We can think of this as the:

```text
Relation Schema
```

Then the actual rows:

```text
1 | Vinay | vinay@example.com
2 | Rahul | rahul@example.com
```

represent the current:

```text
Relation Instance
```

So:

```text
Schema
=
Structure

Instance
=
Current Data
```

---

# 🧠 Think About a Class

In programming:

```java
class Customer {
    long id;
    String name;
    String email;
}
```

This describes structure.

Actual object:

```java
Customer c = new Customer(...);
```

is data based on that structure.

Similarly:

```text
Database

Schema
 ↓
Defines structure

Instance
 ↓
Contains current rows
```

The analogy is useful, but relational tables are not simply object-oriented classes.

---

# 🔑 The Problem of Identity

Now we have:

```text
customers

id | name
---+------
1  | Vinay
2  | Rahul
3  | Ravi
```

Question:

> How do we uniquely identify one customer?

We need:

# Key

For example:

```text
id
```

can identify each customer.

---

# 🔑 What Is a Super Key?

A super key is a set of one or more attributes that uniquely identifies a tuple within a relation.

Example:

```text
customers

id
email
name
```

Suppose:

```text
id
```

is unique.

Then:

```text
{id}
```

is a super key.

But:

```text
{id, name}
```

could also uniquely identify the row.

Why?

Because `id` already uniquely identifies it.

Therefore:

```text
{id}
```

and:

```text
{id, name}
```

can both be super keys.

---

# 🔑 What Is a Candidate Key?

A candidate key is a **minimal super key**.

Suppose:

```text
id
```

is unique.

Then:

```text
{id}
```

is a candidate key.

Suppose:

```text
email
```

is also guaranteed unique.

Then:

```text
{email}
```

can also be a candidate key.

So we may have:

```text
Candidate Keys

1. id
2. email
```

---

# 🥇 What Is a Primary Key?

The database designer chooses one candidate key as the:

# Primary Key

Example:

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT,
    email TEXT UNIQUE
);
```

Here:

```text
id
```

is the primary key.

Conceptually:

```text
Primary Key
     ↓
Uniquely identifies a row
```

---

# 🧠 Primary Key Is More Than "ID"

Many beginners think:

```text
Primary Key
=
id column
```

That's not correct.

A primary key is a **key constraint**, not necessarily a column named `id`.

It can be composite.

Example:

```sql
PRIMARY KEY (student_id, course_id)
```

This is useful when the combination identifies the record.

---

# 🔢 Composite Key

Imagine:

```text
student_courses

student_id | course_id
-----------+----------
1          | 10
1          | 20
2          | 10
```

A student can take many courses.

A course can have many students.

The combination:

```text
student_id
+
course_id
```

uniquely identifies an enrollment.

So:

```sql
PRIMARY KEY (student_id, course_id)
```

is a composite primary key.

---

# 🔗 Foreign Keys

Now we reach one of the most important relational concepts.

We have:

```text
customers

id | name
---+------
1  | Vinay
2  | Rahul
```

and:

```text
orders

id  | customer_id
----+------------
101 | 1
102 | 1
103 | 2
```

What does:

```text
customer_id
```

mean?

It refers to:

```text
customers.id
```

This is a:

# Foreign Key

Example:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,

    customer_id BIGINT,

    FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

---

# 🛡️ Referential Integrity

Now imagine someone tries:

```sql
INSERT INTO orders
(id, customer_id)
VALUES
(104, 999999);
```

But:

```text
customer 999999
```

does not exist.

Without a foreign key, the database might allow an orphaned reference.

With a foreign key:

```text
orders.customer_id
        ↓
must reference
        ↓
customers.id
```

The database can reject the invalid relationship.

This is:

# Referential Integrity

---

# 🚨 Real Software Problem

Imagine an e-commerce system.

```text
customers

id
1
2
3
```

Orders:

```text
orders

id | customer_id
---+------------
101| 1
102| 2
103| 999
```

Customer `999` doesn't exist.

Now the application asks:

```sql
SELECT *
FROM orders o
JOIN customers c
    ON c.id = o.customer_id;
```

Order 103 cannot be associated with a real customer.

You have broken data integrity.

A foreign key can prevent this class of invalid relationship.

---

# 🧠 Why Not Validate This Only in Java?

Suppose:

```text
Web Application
```

checks:

```text
Does customer exist?
```

before inserting an order.

Seems safe.

But now:

```text
Web Application
Mobile Application
Admin Tool
Background Worker
Data Import Script
```

all write to the database.

Each one must correctly implement the same validation.

One service makes a mistake:

```text
Invalid customer_id
```

Database-level constraints provide a centralized enforcement mechanism.

---

# 🔥 Real Production Principle

> **The database should protect critical data invariants.**

Application validation is still useful.

But critical rules should not necessarily depend entirely on application code.

Examples:

```text
Email must be unique
        ↓
UNIQUE

Order must reference valid customer
        ↓
FOREIGN KEY

Balance cannot be negative
        ↓
CHECK

Order ID must be unique
        ↓
PRIMARY KEY
```

---

# 🔗 Relationships

Now let's understand how entities relate.

There are three common relationship types.

```text
One-to-One
One-to-Many
Many-to-Many
```

---

# 1️⃣ One-to-One

Example:

```text
User
 ↓
Passport
```

One user has one passport.

Conceptually:

```text
User 1
   ↓
Passport 1
```

Possible design:

```text
users
```

and:

```text
passports
```

with a unique foreign key.

Example:

```sql
CREATE TABLE passports (
    id BIGINT PRIMARY KEY,
    user_id BIGINT UNIQUE REFERENCES users(id)
);
```

The:

```text
UNIQUE(user_id)
```

helps enforce one passport per user.

---

# 2️⃣ One-to-Many

This is extremely common.

Example:

```text
Customer
   ↓
Orders
```

One customer:

```text
1
```

can have:

```text
Many orders
```

Example:

```text
Vinay
 ├── Order 101
 ├── Order 102
 ├── Order 103
 └── Order 104
```

Database:

```text
customers

id | name
---+------
1  | Vinay
```

```text
orders

id  | customer_id
----+------------
101 | 1
102 | 1
103 | 1
104 | 1
```

---

# 3️⃣ Many-to-Many

Example:

```text
Students
     ↕
Courses
```

One student can attend:

```text
Java
SQL
Docker
```

One course can have:

```text
Student A
Student B
Student C
```

We need a bridge/junction table:

```text
student_courses

student_id | course_id
-----------+----------
1          | 10
1          | 20
2          | 10
3          | 20
```

Architecture:

```text
Students
    ↓
student_courses
    ↑
Courses
```

This is how many-to-many relationships are commonly represented in relational databases.

---

# 📊 Cardinality

Cardinality describes how many entities can participate in a relationship.

Examples:

```text
1 : 1
```

One-to-one.

```text
1 : N
```

One-to-many.

```text
N : M
```

Many-to-many.

---

# 🧠 Optionality

Another important concept is whether the relationship is required.

Example:

```text
Customer
   ↓
Order
```

Could an order exist without a customer?

If the answer is:

```text
No
```

then:

```sql
customer_id BIGINT NOT NULL
```

may be appropriate.

If the relationship is optional:

```text
customer_id BIGINT
```

may allow:

```text
NULL
```

So relationship modeling includes:

```text
Cardinality
+
Optionality
```

---

# 🔥 Real Software Example

Consider a food delivery system.

Entities:

```text
Customer
Restaurant
Driver
Order
Payment
Delivery
```

Relationships:

```text
Customer
   ↓
Orders
```

```text
Restaurant
   ↓
Orders
```

```text
Order
   ↓
Payment
```

```text
Order
   ↓
Delivery
```

```text
Driver
   ↓
Deliveries
```

Conceptually:

```text
Customer
    │
    │ 1:N
    ▼
  Orders
    │
    ├──────────► Payment
    │
    └──────────► Delivery
                    ▲
                    │
                    │ N:1
                  Driver
```

Now relational design gives us a way to represent these relationships explicitly.

---

# 🔥 Why JOIN Exists

Once data is separated into tables:

```text
customers
```

and:

```text
orders
```

we need a way to combine them.

That's where:

```sql
JOIN
```

comes in.

Example:

```sql
SELECT
    c.name,
    o.id,
    o.amount
FROM customers c
JOIN orders o
    ON c.id = o.customer_id;
```

Conceptually:

```text
customers
     +
orders
     ↓
Matching relationship
     ↓
Combined result
```

So:

> JOIN is not an arbitrary SQL feature.

It exists because relational systems allow related information to be stored separately.

---

# 🧠 Relational Algebra Connection

The relational model has mathematical operations that correspond conceptually to operations we perform in SQL.

Important ideas include:

```text
Selection
Projection
Join
Union
Intersection
Difference
Cartesian Product
```

For example:

```sql
SELECT *
FROM customers
WHERE id = 10;
```

conceptually involves:

```text
Selection
```

Selecting rows satisfying a predicate.

While:

```sql
SELECT name, email
FROM customers;
```

conceptually involves:

```text
Projection
```

Selecting attributes/columns.

And:

```sql
SELECT *
FROM customers
JOIN orders
    ON customers.id = orders.customer_id;
```

uses:

```text
Join
```

We'll study relational algebra and JOIN algorithms much more deeply later.

---

# 🧩 NULL and Relationships

Suppose:

```text
orders

id | customer_id
---+------------
101| 1
102| NULL
```

What does:

```text
customer_id = NULL
```

mean?

Potentially:

```text
No customer associated
```

But NULL does not mean:

```text
0
```

or:

```text
Unknown numeric ID = 0
```

NULL has special semantics in SQL.

This becomes very important when using:

```text
LEFT JOIN
```

and:

```text
WHERE
```

We'll study NULL and three-valued logic in detail later.

---

# 🏗️ Complete Relational Example

Let's design a small e-commerce model.

## Customers

```text
customers

id | name  | email
---+-------+------------------
1  | Vinay | vinay@example.com
2  | Rahul | rahul@example.com
```

## Products

```text
products

id | name    | price
---+---------+-------
10 | iPhone  | 80000
11 | MacBook | 120000
```

## Orders

```text
orders

id  | customer_id
----+------------
101 | 1
102 | 1
103 | 2
```

## Order Items

```text
order_items

order_id | product_id | quantity
---------+------------+---------
101      | 10         | 1
101      | 11         | 1
102      | 10         | 2
```

Relationships:

```text
Customer
   │
   │ 1:N
   ▼
 Orders
   │
   │ 1:N
   ▼
Order Items
   │
   │ N:1
   ▼
Product
```

This is relational design.

---

# 🔎 Querying the Model

Suppose we want:

> Show Vinay's orders and products.

We can write:

```sql
SELECT
    c.name,
    o.id AS order_id,
    p.name AS product,
    oi.quantity
FROM customers c
JOIN orders o
    ON c.id = o.customer_id
JOIN order_items oi
    ON o.id = oi.order_id
JOIN products p
    ON p.id = oi.product_id
WHERE c.id = 1;
```

The query travels through relationships:

```text
Customer
   ↓
Orders
   ↓
Order Items
   ↓
Products
```

This is why understanding relationships is essential before mastering JOINs.

---

# 🚨 Real Production Problem — Broken Relationships

Imagine:

```text
orders = 500 million rows
```

and:

```text
customer_id
```

has no useful index.

The database may have to perform expensive work for certain queries.

So relational design eventually leads to:

```text
Relationships
       ↓
Foreign Keys
       ↓
JOINs
       ↓
Indexes
       ↓
Query Optimization
```

Everything connects.

---

# 🧠 Another Production Problem — Deleting Customers

Suppose:

```text
Customer 1
   ↓
10,000 Orders
```

Now someone runs:

```sql
DELETE FROM customers
WHERE id = 1;
```

What should happen?

Possible business rules:

```text
Reject deletion
```

or:

```text
Delete orders too
```

or:

```text
Set customer_id to NULL
```

This is where foreign-key actions matter.

Examples:

```text
CASCADE
RESTRICT
NO ACTION
SET NULL
```

The correct choice depends on business requirements and database behavior.

---

# 🧠 Why Referential Actions Matter

Suppose:

```text
Customer
   ↓
Orders
```

If the customer is deleted, should historical financial orders disappear?

Usually:

```text
No
```

So blindly using:

```text
ON DELETE CASCADE
```

could be dangerous.

Senior engineers don't ask:

> "Which option is easiest?"

They ask:

> "What does the business meaning of this relationship require?"

---

# 🧠 Relational Model vs Object Model

Applications often think in terms of:

```text
Objects
```

For example:

```text
Customer
 ├── name
 ├── email
 └── orders[]
```

Relational databases think more in terms of:

```text
Relations
```

such as:

```text
customers
orders
```

The application might see:

```text
Customer Object
```

while the database stores:

```text
customers row
+
orders rows
```

The application and database therefore need a mapping.

This leads to concepts such as:

```text
ORM
Object-Relational Mapping
```

Examples include:

```text
Hibernate
JPA
Entity Framework
SQLAlchemy
Prisma
```

But remember:

> ORM is not a replacement for understanding the relational model.

---

# 🔥 Common Beginner Mistake

Developers sometimes design:

```text
customers

id
name
orders_json
```

where:

```text
orders_json
```

contains:

```json
[
  {"id":101,"amount":500},
  {"id":102,"amount":900}
]
```

This can be appropriate for some document-oriented designs, but in a relational model it often creates difficulties for:

```text
Relationships
Constraints
Queries
Indexes
Updates
Transactions
```

The right design depends on access patterns and requirements.

The lesson is:

> **Choose the data model based on the problem, not convenience alone.**

---

# 🧪 Hands-on Lab

## Lab 1 — Create Customers

```sql
CREATE TABLE customers (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);
```

Insert:

```sql
INSERT INTO customers
    (id, name, email)
VALUES
    (1, 'Vinay', 'vinay@example.com'),
    (2, 'Rahul', 'rahul@example.com');
```

---

# Lab 2 — Create Orders

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,

    FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Insert:

```sql
INSERT INTO orders
    (id, customer_id)
VALUES
    (101, 1),
    (102, 1),
    (103, 2);
```

---

# Lab 3 — Test Referential Integrity

Try:

```sql
INSERT INTO orders
    (id, customer_id)
VALUES
    (104, 999);
```

What happens?

The database should reject the operation because customer `999` doesn't exist, assuming the foreign key is enforced.

---

# Lab 4 — Test the Relationship

Run:

```sql
SELECT
    c.name,
    o.id AS order_id
FROM customers c
JOIN orders o
    ON c.id = o.customer_id;
```

Observe:

```text
Vinay → 101
Vinay → 102
Rahul → 103
```

---

# Lab 5 — Many-to-Many

Create:

```sql
CREATE TABLE students (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL
);
```

Create:

```sql
CREATE TABLE courses (
    id BIGINT PRIMARY KEY,
    name TEXT NOT NULL
);
```

Then:

```sql
CREATE TABLE student_courses (
    student_id BIGINT REFERENCES students(id),
    course_id BIGINT REFERENCES courses(id),

    PRIMARY KEY (student_id, course_id)
);
```

You have now created a many-to-many relationship.

---

# 🧪 Lab 6 — Draw the Database

Before writing SQL, draw:

```text
Customer
   │
   │ 1:N
   ▼
 Order
   │
   │ 1:N
   ▼
Order Item
   │
   │ N:1
   ▼
Product
```

Then identify:

```text
Primary Keys
Foreign Keys
Nullable Relationships
Unique Constraints
```

This is an important database-design habit.

---

# 🎯 Interview Questions

## Beginner

### What is a relational database?

A database system based on the relational model, where data is represented using relations commonly implemented as tables.

---

### What is a relation?

A mathematical structure consisting of tuples over defined attributes. In SQL practice, a relation is commonly represented as a table.

---

### What is a tuple?

A member of a relation; commonly represented as a row in SQL.

---

### What is an attribute?

A named property of a relation; commonly represented as a column.

---

### What is a primary key?

A selected candidate key used to uniquely identify rows in a table.

---

### What is a foreign key?

A constraint that requires values in one table to reference matching key values in another table or, depending on the schema, another relation.

---

# 🧠 Intermediate Questions

### What is the difference between a candidate key and a primary key?

A relation can have multiple candidate keys.

One of them is selected as the primary key.

Example:

```text
Candidate Keys:

id
email
```

Choose:

```text
Primary Key = id
```

---

### What is a composite key?

A key consisting of multiple attributes.

Example:

```sql
PRIMARY KEY (student_id, course_id)
```

---

### What is referential integrity?

The property that references represented by foreign keys remain valid according to the defined constraints.

---

### Why are foreign keys important?

They prevent invalid relationships and help keep related data consistent.

---

# 👑 Senior Questions

### Why not store all information in one giant table?

Because it can cause:

```text
Data Duplication
Update Anomalies
Insert Anomalies
Delete Anomalies
Large Rows
Difficult Maintenance
```

Normalization provides a systematic way to reduce such problems.

---

### Why not create a separate database for every table?

Because relationships and queries often require combining related data.

Keeping logically related data within a database system allows the database to enforce relationships and execute relational queries.

---

### Why does JOIN exist?

Because relational design allows related data to be stored in separate relations.

JOIN provides a mechanism to combine related rows based on a relationship.

---

# 👑 20+ Year Experience Questions

### Question 1

Why is:

```text
Primary Key
```

a logical concept while:

```text
Index
```

is an implementation/performance mechanism?

Don't confuse:

```text
Data Identity
```

with:

```text
Data Access Path
```

---

### Question 2

Can a table have:

```text
Multiple candidate keys?
```

Yes.

Can it have:

```text
Multiple primary keys?
```

No.

It has one primary key constraint, although that primary key can contain multiple columns.

---

### Question 3

Should every foreign key have an index?

Don't answer:

```text
Always
```

or:

```text
Never
```

Think about:

```text
Query Patterns
Join Performance
Delete / Update Parent Operations
Write Cost
Database Implementation
```

---

### Question 4

Should every relationship be enforced with a foreign key?

Think about:

```text
Data Ownership
Cross-Service Databases
Distributed Systems
Performance
Operational Constraints
```

In a single relational database, foreign keys can provide strong integrity. In distributed architectures, enforcing relationships across independent databases is a different problem.

---

### Question 5

Should every table have an `id` column?

No.

A natural or composite key may sometimes be appropriate.

The important question is:

> What uniquely identifies the entity?

---

### Question 6

Why can normalization improve correctness but sometimes hurt read performance?

Because normalization reduces duplication but can require more joins.

This creates a trade-off:

```text
Normalization
      ↓
Less Duplication
Better Integrity
      +
Potentially More Joins
```

and:

```text
Denormalization
      ↓
Fewer Joins
Potentially Faster Reads
      +
More Duplication
More Maintenance Complexity
```

---

# 🧠 Database Design Mental Model

When designing a relational database, think in this order:

```text
Business Entities
        ↓
Relationships
        ↓
Attributes
        ↓
Keys
        ↓
Constraints
        ↓
Normalization
        ↓
Indexes
        ↓
Query Patterns
        ↓
Performance
```

Don't start with:

```text
"What columns should I create?"
```

Start with:

```text
"What are the business entities
and how are they related?"
```

---

# 🔥 The Most Important Lesson

A relational database is not simply:

```text
Tables
```

It is a model of:

```text
Entities
+
Attributes
+
Relationships
+
Constraints
```

For example:

```text
Customer
   │
   │ 1:N
   ▼
Orders
   │
   │ 1:N
   ▼
Order Items
   │
   │ N:1
   ▼
Products
```

Then the database uses:

```text
Primary Keys
Foreign Keys
Unique Constraints
Check Constraints
```

to protect that model.

SQL allows us to query it:

```text
SELECT
JOIN
GROUP BY
WHERE
```

And later:

```text
Indexes
Optimizer
Transactions
Storage Engine
```

make those operations efficient and reliable.

---

# 📌 Chapter Summary

The relational model gives us a structured way to represent data.

The key concepts are:

```text
Relation
 ↓
Table

Tuple
 ↓
Row

Attribute
 ↓
Column

Domain
 ↓
Allowed Value Space

Candidate Key
 ↓
Minimal Unique Identifier

Primary Key
 ↓
Chosen Candidate Key

Foreign Key
 ↓
Relationship Reference
```

Relationships can be:

```text
1 : 1
1 : N
N : M
```

Many-to-many relationships are commonly represented using a junction table.

Foreign keys provide:

```text
Referential Integrity
```

And relational design leads naturally to:

```text
JOINs
```

---

# 🏗️ Final Mental Model

Keep this architecture in your mind:

```text
                    BUSINESS DOMAIN
                          │
                          ▼
                       ENTITIES
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
           Customer     Order       Product
              │           │           │
              └──────┬────┴─────┬─────┘
                     ▼          ▼
                  RELATIONSHIPS
                       │
                       ▼
                     TABLES
                       │
              ┌────────┴────────┐
              ▼                 ▼
          PRIMARY KEY       FOREIGN KEY
              │                 │
              └────────┬────────┘
                       ▼
                REFERENTIAL
                  INTEGRITY
                       │
                       ▼
                     SQL
                       │
                       ▼
                    JOIN
                       │
                       ▼
                QUERY ENGINE
```

---

# 🚀 Next Chapter

# Chapter 4 — SQL vs Database Engine

We will answer a critical question:

> **When you write SQL, who actually executes it?**

We'll trace:

```text
SQL Query
   ↓
Client
   ↓
Network
   ↓
Database Server
   ↓
Parser
   ↓
Analyzer
   ↓
Optimizer
   ↓
Execution Plan
   ↓
Executor
   ↓
Storage
   ↓
Result
```

And we'll use a real production problem:

```text
Application
   ↓
SQL Query
   ↓
API takes 8 seconds
```

to understand why **SQL is a declarative language and the database decides HOW the query is executed**.
