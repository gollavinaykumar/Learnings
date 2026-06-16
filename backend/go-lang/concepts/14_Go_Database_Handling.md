# Go Database Handling – Complete Interview Revision Guide

> Database handling is one of the most important backend development topics.
>
> Almost every Go backend application interacts with:
>
> - PostgreSQL
> - MySQL
> - MongoDB
> - Redis
>
> Interviewers frequently ask about:
>
> - database/sql
> - Connection Pooling
> - Transactions
> - Prepared Statements
> - GORM
> - sqlx

---

# 1. Database Handling in Go

Most Go applications follow:

```text
HTTP Request
      ↓
Handler
      ↓
Service
      ↓
Repository
      ↓
Database
```

---

# Example

```text
Get User API
      ↓
Service Layer
      ↓
Repository Layer
      ↓
PostgreSQL
```

---

# Why Repository Layer?

Keeps database code separate from business logic.

Good:

```text
Service
Repository
Database
```

Bad:

```text
Service directly writes SQL
```

---

# 2. database/sql Package

Most Important Database Package.

---

# What is database/sql?

Go's standard package for SQL databases.

Import:

```go
import "database/sql"
```

---

# Important Point

`database/sql` is NOT a database driver.

It is an abstraction layer.

---

# Driver Example

PostgreSQL:

```go
import _ "github.com/lib/pq"
```

---

MySQL:

```go
import _ "github.com/go-sql-driver/mysql"
```

---

# Architecture

```text
Application
     ↓
database/sql
     ↓
Driver
     ↓
Database
```

---

# Open Database Connection

PostgreSQL:

```go
db, err := sql.Open(
    "postgres",
    connString,
)
```

---

MySQL:

```go
db, err := sql.Open(
    "mysql",
    connString,
)
```

---

# Check Connection

Always verify.

```go
err = db.Ping()
```

---

Example:

```go
if err != nil {
    log.Fatal(err)
}
```

---

# Query Single Row

```go
row := db.QueryRow(
    "SELECT id, name FROM users WHERE id=$1",
    id,
)
```

---

Read Result:

```go
var user User

err := row.Scan(
    &user.ID,
    &user.Name,
)
```

---

# Query Multiple Rows

```go
rows, err := db.Query(
    "SELECT id, name FROM users",
)
```

---

Loop:

```go
for rows.Next() {

    var user User

    rows.Scan(
        &user.ID,
        &user.Name,
    )
}
```

---

# Insert Data

```go
_, err := db.Exec(
    "INSERT INTO users(name) VALUES($1)",
    "Vinay",
)
```

---

# Update Data

```go
_, err := db.Exec(
    "UPDATE users SET name=$1 WHERE id=$2",
    "Alex",
    1,
)
```

---

# Delete Data

```go
_, err := db.Exec(
    "DELETE FROM users WHERE id=$1",
    1,
)
```

---

# database/sql Summary

Main Methods:

```go
Query()
QueryRow()
Exec()
Ping()
Begin()
```

---

# 3. Connection Pooling

Extremely Important Interview Topic.

---

# What is Connection Pooling?

Creating database connections is expensive.

Bad:

```text
Request
 ↓
Create Connection
 ↓
Query
 ↓
Close Connection
```

for every request.

---

# Solution

Reuse connections.

---

# Pool Architecture

```text
Application
     ↓
Connection Pool
     ↓
Database
```

---

# Good Example

```text
100 Requests
      ↓
10 Shared Connections
```

---

Instead of:

```text
100 Requests
100 Connections
```

---

# Important Fact

`sql.DB`

is NOT a single connection.

---

# Interview Question

### What is sql.DB?

Answer:

```text
sql.DB is a connection pool manager.
```

Not a single database connection.

---

# Configure Pool

---

## Max Open Connections

```go
db.SetMaxOpenConns(25)
```

---

Meaning:

```text
Maximum 25 active connections
```

---

## Max Idle Connections

```go
db.SetMaxIdleConns(5)
```

---

Meaning:

```text
Keep 5 idle connections ready
```

---

## Connection Lifetime

```go
db.SetConnMaxLifetime(
    time.Hour,
)
```

---

Why?

Avoid stale connections.

---

# Production Example

```go
db.SetMaxOpenConns(50)
db.SetMaxIdleConns(10)
db.SetConnMaxLifetime(time.Hour)
```

---

# Why Connection Pooling Matters?

Benefits:

```text
Better Performance
Lower Latency
Less DB Load
Resource Reuse
```

---

# 4. Transactions

Most Asked Database Interview Topic.

---

# What is a Transaction?

A transaction is a group of operations treated as a single unit.

---

# Example

Bank Transfer:

```text
Deduct Money
Add Money
```

Both must succeed.

---

Bad:

```text
Money Deducted
Add Failed
```

Data inconsistency.

---

# Solution

Transaction.

---

# Transaction Flow

```text
BEGIN
  ↓
Operations
  ↓
COMMIT
```

---

If error:

```text
BEGIN
  ↓
Operations
  ↓
ROLLBACK
```

---

# Start Transaction

```go
tx, err := db.Begin()
```

---

# Execute Queries

```go
_, err = tx.Exec(...)
```

---

# Commit

```go
err = tx.Commit()
```

---

# Rollback

```go
tx.Rollback()
```

---

# Complete Example

```go
tx, err := db.Begin()

if err != nil {
    return err
}

defer tx.Rollback()

_, err = tx.Exec(
    "UPDATE accounts SET balance=balance-100 WHERE id=1",
)

if err != nil {
    return err
}

_, err = tx.Exec(
    "UPDATE accounts SET balance=balance+100 WHERE id=2",
)

if err != nil {
    return err
}

return tx.Commit()
```

---

# Interview Question

### Why Use Transactions?

To ensure:

```text
Consistency
Atomicity
Reliability
```

of database operations.

---

# ACID Properties

Very Common Interview Topic.

---

## Atomicity

All or nothing.

---

## Consistency

Data remains valid.

---

## Isolation

Transactions don't interfere.

---

## Durability

Committed data survives crashes.

---

# 5. Prepared Statements

Very Important Security Topic.

---

# What is a Prepared Statement?

A precompiled SQL statement.

---

# Why Use It?

Benefits:

```text
Better Performance
Prevents SQL Injection
```

---

# Bad Example

```go
query :=
    "SELECT * FROM users WHERE id=" +
    userInput
```

Danger:

```text
SQL Injection
```

---

# Good Example

```go
db.QueryRow(
    "SELECT * FROM users WHERE id=$1",
    id,
)
```

---

# Explicit Prepared Statement

```go
stmt, err := db.Prepare(
    "SELECT * FROM users WHERE id=$1",
)
```

---

Execute:

```go
row := stmt.QueryRow(id)
```

---

Close:

```go
defer stmt.Close()
```

---

# Why Prepared Statements Matter?

```text
Security
Performance
Cleaner Queries
```

---

# SQL Injection Example

Bad Input:

```text
1 OR 1=1
```

---

Unsafe Query:

```sql
SELECT * FROM users
WHERE id=1 OR 1=1
```

Returns everything.

---

Prepared statements prevent this.

---

# PostgreSQL

Most Popular Go Database.

---

# Why PostgreSQL?

Features:

```text
Open Source
Reliable
ACID Compliant
Rich Features
JSON Support
```

---

# Driver

```go
github.com/lib/pq
```

or

```go
github.com/jackc/pgx
```

---

# Placeholder

```sql
$1
$2
```

Example:

```go
db.QueryRow(
    "SELECT * FROM users WHERE id=$1",
    id,
)
```

---

# MySQL

Another popular relational database.

---

# Driver

```go
github.com/go-sql-driver/mysql
```

---

# Placeholder

```sql
?
```

Example:

```go
db.QueryRow(
    "SELECT * FROM users WHERE id=?",
    id,
)
```

---

# PostgreSQL vs MySQL

| Feature | PostgreSQL | MySQL |
|----------|-----------|--------|
| JSON Support | Better | Good |
| Advanced Queries | Better | Good |
| Popularity | High | High |
| Performance | Excellent | Excellent |

---

# MongoDB

NoSQL Database.

---

# Stores

```text
Documents
```

instead of tables.

---

Example:

```json
{
  "name":"Vinay",
  "age":25
}
```

---

# Official Driver

```go
go.mongodb.org/mongo-driver
```

---

# Common Usage

```go
collection.Find()
collection.InsertOne()
collection.UpdateOne()
```

---

# When Use MongoDB?

Good for:

```text
Flexible Schemas
Document Data
Rapid Development
```

---

# Redis

Very Common Interview Topic.

---

# What is Redis?

In-memory key-value database.

---

# Use Cases

```text
Caching
Sessions
Rate Limiting
Queues
Leaderboards
```

---

# Example

```text
User Cache
```

Instead of:

```text
Database Query
```

Use:

```text
Redis Lookup
```

Much faster.

---

# Popular Library

```go
github.com/redis/go-redis/v9
```

---

# Example

Set:

```go
rdb.Set(
    ctx,
    "user:1",
    "Vinay",
    time.Hour,
)
```

---

Get:

```go
rdb.Get(
    ctx,
    "user:1",
)
```

---

# GORM

Most Popular Go ORM.

---

# What is ORM?

Object Relational Mapping.

Converts:

```go
Struct
```

into:

```sql
Database Table
```

---

# Install

```bash
go get gorm.io/gorm
```

---

# Model

```go
type User struct {
    ID   uint
    Name string
}
```

---

# Create

```go
db.Create(&user)
```

---

# Find

```go
db.First(&user, 1)
```

---

# Update

```go
db.Save(&user)
```

---

# Delete

```go
db.Delete(&user)
```

---

# Advantages

```text
Easy
Fast Development
Less SQL
```

---

# Disadvantages

```text
Less Control
Can Generate Complex Queries
```

---

# sqlx

Very Popular Interview Topic.

---

# What is sqlx?

Enhanced version of:

```go
database/sql
```

---

# Benefits

```text
Less Boilerplate
Struct Mapping
Still Uses SQL
```

---

# Example

```go
var users []User

db.Select(
    &users,
    "SELECT * FROM users",
)
```

---

No manual Scan loop needed.

---

# Why Developers Love sqlx?

Keeps:

```text
Raw SQL Control
```

while reducing boilerplate.

---

# database/sql vs GORM vs sqlx

| Feature | database/sql | sqlx | GORM |
|-----------|------------|------|------|
| Raw SQL | Yes | Yes | Optional |
| Control | High | High | Medium |
| Boilerplate | High | Low | Very Low |
| Learning Curve | Medium | Easy | Easy |
| Performance | Best | Very Good | Good |

---

# Interview Recommendation

For production systems:

```text
database/sql
or
sqlx
```

are commonly preferred.

For rapid development:

```text
GORM
```

is popular.

---

# Most Asked Interview Questions

---

## What is database/sql?

Go's standard database abstraction package.

---

## Is sql.DB a Database Connection?

No.

It is a connection pool manager.

---

## Why Connection Pooling?

To reuse database connections and improve performance.

---

## What is a Transaction?

A group of operations executed as a single unit.

---

## What is Commit?

Permanently save transaction changes.

---

## What is Rollback?

Undo transaction changes.

---

## Why Use Prepared Statements?

- Prevent SQL Injection
- Improve Performance

---

## PostgreSQL Placeholder?

```sql
$1
$2
```

---

## MySQL Placeholder?

```sql
?
```

---

## What is Redis?

An in-memory key-value store used for caching and fast lookups.

---

## What is GORM?

Popular ORM for Go.

---

## What is sqlx?

Improved version of database/sql with struct mapping support.

---

# Quick Revision Cheat Sheet

## Open DB

```go
db, err := sql.Open(...)
```

---

## Ping

```go
db.Ping()
```

---

## Query Row

```go
db.QueryRow(...)
```

---

## Query Multiple Rows

```go
db.Query(...)
```

---

## Execute Query

```go
db.Exec(...)
```

---

## Start Transaction

```go
tx, err := db.Begin()
```

---

## Commit

```go
tx.Commit()
```

---

## Rollback

```go
tx.Rollback()
```

---

## Prepared Statement

```go
stmt, err := db.Prepare(...)
```

---

## Pool Settings

```go
db.SetMaxOpenConns()
db.SetMaxIdleConns()
```

---

## PostgreSQL

```text
$1
$2
```

---

## MySQL

```text
?
```

---

## Redis

```text
Caching
Sessions
Rate Limiting
```

---

## ORM

```go
GORM
```

---

## SQL Helper

```go
sqlx
```

---

# Golden Interview Answer

### How Does Go Handle Databases Efficiently?

Go uses the `database/sql` package, which provides a connection pool manager and database abstraction layer.

Key production concepts include:

- Connection Pooling
- Transactions
- Prepared Statements
- Context-aware Queries
- Efficient Resource Management

For relational databases, PostgreSQL and MySQL are the most common choices, while Redis is widely used for caching and MongoDB for document-based storage.

Libraries such as `sqlx` and `GORM` help reduce boilerplate while building scalable backend applications.

---

# End of Database Handling Revision