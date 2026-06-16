# Go Context Package – Complete Interview Revision Guide

> The `context` package is one of the most important packages in modern Go backend development.
>
> It is used everywhere:
>
> - HTTP APIs
> - Microservices
> - Database Queries
> - gRPC
> - Background Jobs
> - Distributed Systems
> - Graceful Shutdown
>
> Almost every production Go application uses Context.

---

# 1. What is Context?

Context is a mechanism used to:

```text
Carry request information
Control cancellation
Handle timeouts
Share request-scoped values
```

between goroutines and function calls.

---

# Why Was Context Created?

Imagine:

```text
HTTP Request
      ↓
Service
      ↓
Database
      ↓
External API
```

If the user closes the browser:

```text
Request is gone
```

Should database and API calls continue?

```text
No
```

We should stop everything.

Context helps propagate that signal.

---

# Simple Definition

Context allows you to:

```text
Cancel work
Set deadlines
Pass request-specific data
```

across function boundaries.

---

# Context Package

```go
import "context"
```

---

# Create Context

Most common:

```go
ctx := context.Background()
```

---

# What is Background()?

Returns an empty root context.

Think:

```text
Starting point
```

for all contexts.

---

# Example

```go
ctx := context.Background()

fmt.Println(ctx)
```

Output:

```text
context.Background
```

---

# Context Hierarchy

Very Important Concept.

---

```text
Background()
      |
      +---- Child Context
      |
      +---- Child Context
```

All child contexts inherit from parent.

---

# Example

```go
parent := context.Background()

child, cancel :=
    context.WithCancel(parent)
```

---

# Why Hierarchy?

If parent is cancelled:

```text
All children are cancelled
```

automatically.

---

# Context Rules

Interview Favorite.

---

## Rule 1

Pass context as first parameter.

Good:

```go
func GetUser(
    ctx context.Context,
)
```

---

Bad:

```go
func GetUser(
    id int,
    ctx context.Context,
)
```

---

## Rule 2

Never store Context in structs.

Bad:

```go
type Service struct {
    ctx context.Context
}
```

---

Good:

```go
func (s Service) GetUser(
    ctx context.Context,
)
```

---

## Rule 3

Always pass context down.

Example:

```text
HTTP
 ↓
Service
 ↓
Repository
 ↓
Database
```

Same context should travel through all layers.

---

# 2. Cancellation

Most Important Context Topic.

---

# What is Cancellation?

Allows stopping ongoing work.

---

# Create Cancelable Context

```go
ctx, cancel :=
    context.WithCancel(
        context.Background(),
    )
```

---

# Example

```go
ctx, cancel :=
    context.WithCancel(
        context.Background(),
    )

go func() {

    <-ctx.Done()

    fmt.Println("Cancelled")
}()
```

---

Cancel:

```go
cancel()
```

Output:

```
Cancelled
```

---

# What is Done()?

Returns a channel.

---

```go
ctx.Done()
```

closes when:

```text
Cancelled
Timeout reached
Deadline exceeded
```

---

# Example

```go
select {

case <-ctx.Done():
    fmt.Println("Stopped")
}
```

---

# Real Backend Example

```go
func ProcessOrder(
    ctx context.Context,
) {

    select {

    case <-ctx.Done():
        return

    default:
        // Continue work
    }
}
```

---

# Why Cancellation Matters?

Suppose:

```text
User disconnects
```

No reason to continue:

```text
Database Query
API Call
Background Work
```

Cancellation saves:

```text
CPU
Memory
Network
```

---

# Always Call cancel()

Interview Question.

---

Bad:

```go
ctx, cancel :=
    context.WithCancel(...)
```

Never used.

---

Good:

```go
ctx, cancel :=
    context.WithCancel(...)

defer cancel()
```

---

Why?

Prevents resource leaks.

---

# 3. Timeout

Most Asked Context Interview Topic.

---

# Problem

What if an API takes:

```text
30 seconds
```

but we only want:

```text
5 seconds
```

---

# Solution

Use timeout.

---

# Create Timeout Context

```go
ctx, cancel :=
    context.WithTimeout(
        context.Background(),
        5*time.Second,
    )

defer cancel()
```

---

# Example

```go
ctx, cancel :=
    context.WithTimeout(
        context.Background(),
        2*time.Second,
    )

defer cancel()

select {

case <-time.After(5 * time.Second):
    fmt.Println("Completed")

case <-ctx.Done():
    fmt.Println("Timeout")
}
```

Output:

```
Timeout
```

---

# Why?

Context automatically cancels after:

```text
2 seconds
```

---

# Check Error

```go
ctx.Err()
```

---

Example:

```go
fmt.Println(ctx.Err())
```

Output:

```text
context deadline exceeded
```

---

# Common Production Usage

Database Query:

```go
ctx, cancel :=
    context.WithTimeout(
        ctx,
        3*time.Second,
    )

defer cancel()

db.QueryContext(ctx, query)
```

---

# Timeout Visualization

```text
Request Starts
      ↓
2 Seconds Pass
      ↓
Context Cancels
      ↓
Database Stops
```

---

# 4. Request-Scoped Values

Context can carry data throughout request lifecycle.

---

# Example

```text
User ID
Request ID
Trace ID
Correlation ID
```

---

# Create Value Context

```go
ctx :=
    context.WithValue(
        context.Background(),
        "userID",
        101,
    )
```

---

# Retrieve Value

```go
id := ctx.Value("userID")
```

Output:

```
101
```

---

# Example

```go
ctx :=
    context.WithValue(
        context.Background(),
        "userID",
        101,
    )

fmt.Println(
    ctx.Value("userID"),
)
```

Output:

```
101
```

---

# Real Backend Example

Request:

```text
User = 101
```

Store:

```go
ctx = context.WithValue(
    ctx,
    "userID",
    101,
)
```

---

Pass through:

```text
Handler
 ↓
Service
 ↓
Repository
```

Any layer can access:

```go
ctx.Value("userID")
```

---

# Best Practice

Avoid using plain strings.

Bad:

```go
ctx.Value("userID")
```

---

Good:

```go
type contextKey string

const UserIDKey contextKey = "userID"
```

---

Usage:

```go
ctx =
    context.WithValue(
        ctx,
        UserIDKey,
        101,
    )
```

---

# Important Warning

Context values are for:

```text
Request Metadata
```

NOT:

```text
Business Data
```

---

Good:

```text
Request ID
User ID
Trace ID
```

---

Bad:

```text
Product List
User Object
Database Connection
```

---

# 5. Graceful Shutdown

Extremely Important Production Topic.

---

# What is Graceful Shutdown?

When application stops:

```text
Finish active work
Close resources
Exit safely
```

instead of:

```text
Immediate crash
```

---

# Why Needed?

Suppose:

```text
Server receives SIGTERM
```

while:

```text
100 requests running
```

Should we kill them?

```text
No
```

Let them finish.

---

# Context Helps Here

Create cancellation signal.

---

# Example

```go
ctx, cancel :=
    context.WithCancel(
        context.Background(),
    )
```

---

When shutdown signal arrives:

```go
cancel()
```

---

All goroutines receive:

```go
<-ctx.Done()
```

and stop gracefully.

---

# Example Worker

```go
func worker(
    ctx context.Context,
) {

    for {

        select {

        case <-ctx.Done():
            fmt.Println("Stopping Worker")
            return

        default:
            fmt.Println("Working")
            time.Sleep(time.Second)
        }
    }
}
```

---

# Main

```go
ctx, cancel :=
    context.WithCancel(
        context.Background(),
    )

go worker(ctx)

time.Sleep(3*time.Second)

cancel()
```

Output:

```text
Working
Working
Working
Stopping Worker
```

---

# Real HTTP Server Shutdown

Very Common Interview Question.

---

```go
ctx, cancel :=
    context.WithTimeout(
        context.Background(),
        5*time.Second,
    )

defer cancel()

server.Shutdown(ctx)
```

---

Meaning:

```text
Give active requests
5 seconds to finish
```

Then stop server.

---

# Context Functions Summary

---

## Background

```go
ctx := context.Background()
```

Root context.

---

## WithCancel

```go
ctx, cancel :=
    context.WithCancel(parent)
```

Manual cancellation.

---

## WithTimeout

```go
ctx, cancel :=
    context.WithTimeout(
        parent,
        5*time.Second,
    )
```

Automatic timeout.

---

## WithValue

```go
ctx :=
    context.WithValue(
        parent,
        key,
        value,
    )
```

Store request metadata.

---

# Most Asked Interview Questions

---

## What is Context?

A mechanism for cancellation, timeouts, and request-scoped values.

---

## Why Is Context Important?

Allows requests to propagate cancellation and deadlines across services.

---

## What is context.Background()?

Root context used to create child contexts.

---

## What is context.WithCancel()?

Creates a context that can be manually cancelled.

---

## What is context.WithTimeout()?

Creates a context that automatically cancels after a specified duration.

---

## What Does ctx.Done() Do?

Returns a channel that closes when context is cancelled.

---

## Why Must cancel() Be Called?

To release resources and avoid leaks.

---

## What Are Context Values Used For?

Request metadata.

Examples:

```text
User ID
Request ID
Trace ID
```

---

## Should Context Store Business Data?

No.

Only request-scoped metadata.

---

## What is Graceful Shutdown?

Stopping application safely while allowing active work to finish.

---

# Real Backend Flow

```text
HTTP Request
      ↓
Context Created
      ↓
Handler
      ↓
Service
      ↓
Repository
      ↓
Database

Same Context Passed Everywhere
```

---

# Quick Revision Cheat Sheet

## Root Context

```go
ctx := context.Background()
```

---

## Cancel Context

```go
ctx, cancel :=
    context.WithCancel(ctx)

defer cancel()
```

---

## Timeout Context

```go
ctx, cancel :=
    context.WithTimeout(
        ctx,
        5*time.Second,
    )

defer cancel()
```

---

## Done Channel

```go
<-ctx.Done()
```

---

## Error

```go
ctx.Err()
```

---

## Store Value

```go
ctx =
    context.WithValue(
        ctx,
        key,
        value,
    )
```

---

## Read Value

```go
ctx.Value(key)
```

---

## Graceful Shutdown

```go
server.Shutdown(ctx)
```

---

# Golden Interview Answer

### Why Is Context So Important In Production Go Applications?

Context provides a standard way to manage:

- Cancellation
- Timeouts
- Request-scoped values
- Graceful shutdown

It allows every layer of an application:

```text
HTTP
 ↓
Service
 ↓
Repository
 ↓
Database
```

to respond consistently to request cancellation and deadlines.

That's why context is passed through almost every production Go codebase.

---

# End of Context Package Revision