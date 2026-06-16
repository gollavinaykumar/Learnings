# Go Testing – Complete Interview Revision Guide

> Testing is a critical skill for backend developers.
>
> Production Go applications rely heavily on tests to ensure:
>
> - Correct functionality
> - Safe refactoring
> - Fewer bugs
> - Better maintainability
>
> Common interview topics:
>
> - Unit Testing
> - Table-Driven Tests
> - Mocking
> - Benchmark Testing

---

# 1. What is Testing?

Testing means verifying that code behaves as expected.

Example:

```go
Add(2, 3)
```

Expected:

```go
5
```

A test automatically checks this.

---

# Why Testing Matters?

Without tests:

```text
Code Changes
     ↓
Unexpected Bugs
     ↓
Production Issues
```

---

With tests:

```text
Code Changes
     ↓
Run Tests
     ↓
Confidence
```

---

# Types of Testing

---

## Unit Testing

Tests a small piece of code.

Example:

```go
Function
Method
```

---

## Integration Testing

Tests interaction between components.

Example:

```text
API + Database
```

---

## End-to-End Testing

Tests complete system.

Example:

```text
User Login Flow
```

---

# Interview Focus

Most Go interviews focus on:

```text
Unit Testing
```

---

# 2. Unit Testing

Most Important Testing Topic.

---

# Test File Naming

Suppose:

```go
user.go
```

---

Test file:

```go
user_test.go
```

---

# Function Naming

Test functions must start with:

```go
Test
```

---

Example:

```go
func TestUser(
    t *testing.T,
)
```

---

# Import Package

```go
import "testing"
```

---

# Example Function

```go
func Add(
    a int,
    b int,
) int {

    return a + b
}
```

---

# Test

```go
func TestAdd(
    t *testing.T,
) {

    result := Add(2, 3)

    if result != 5 {

        t.Errorf(
            "expected 5, got %d",
            result,
        )
    }
}
```

---

# Run Tests

```bash
go test
```

---

Output:

```text
PASS
```

---

# If Test Fails

Output:

```text
FAIL
```

with details.

---

# Common Test Functions

---

## Errorf

```go
t.Errorf()
```

Logs error and continues.

---

## Fatalf

```go
t.Fatalf()
```

Stops test immediately.

---

Example:

```go
if err != nil {

    t.Fatalf(
        "unexpected error: %v",
        err,
    )
}
```

---

# Test Structure

Common pattern:

```go
Arrange
Act
Assert
```

---

Example:

```go
result := Add(2, 3)

if result != 5 {
    t.Errorf(...)
}
```

---

# 3. Table-Driven Tests

Most Asked Go Testing Interview Topic.

---

# What is Table-Driven Testing?

Testing multiple cases using a single test function.

---

# Problem

Bad approach:

```go
TestAdd1()
TestAdd2()
TestAdd3()
```

Lots of duplication.

---

# Solution

Store test cases in a table.

---

# Example Function

```go
func Add(
    a int,
    b int,
) int {

    return a + b
}
```

---

# Table Test

```go
func TestAdd(
    t *testing.T,
) {

    tests := []struct {
        a        int
        b        int
        expected int
    }{
        {1, 2, 3},
        {5, 5, 10},
        {10, 20, 30},
    }

    for _, test := range tests {

        result :=
            Add(
                test.a,
                test.b,
            )

        if result != test.expected {

            t.Errorf(
                "expected %d got %d",
                test.expected,
                result,
            )
        }
    }
}
```

---

# Why Table-Driven Tests?

Benefits:

```text
Less Duplication
Easy To Extend
Cleaner Tests
```

---

# Real Production Example

Testing:

```text
Validation
Business Rules
Permissions
Calculations
```

Usually done with table-driven tests.

---

# Subtests

Common Interview Topic.

---

# Example

```go
for _, tc := range tests {

    t.Run(
        tc.name,
        func(t *testing.T) {

        },
    )
}
```

---

# Benefits

Output clearly shows:

```text
Which test failed
```

---

# Example Output

```text
PASS: Positive Numbers
FAIL: Negative Numbers
```

---

# 4. Mocking

Extremely Important Backend Testing Topic.

---

# What is Mocking?

Replacing real dependencies with fake ones during testing.

---

# Why?

Suppose:

```text
User Service
      ↓
Database
```

Testing database every time is:

```text
Slow
Unreliable
Hard
```

---

# Solution

Mock database.

---

# Real Dependency

```go
type UserRepository interface {

    GetUser(
        id int,
    ) (*User, error)
}
```

---

# Service

```go
type UserService struct {
    repo UserRepository
}
```

---

# Production Repository

```go
PostgresRepository
```

---

# Test Repository

```go
MockRepository
```

---

# Mock Example

```go
type MockRepository struct {
}
```

---

Implement interface:

```go
func (
    m MockRepository,
) GetUser(
    id int,
) (*User, error) {

    return &User{
        Name: "Vinay",
    }, nil
}
```

---

# Test

```go
repo := MockRepository{}

service :=
    UserService{
        repo: repo,
    }
```

---

Now test:

```text
Without Database
```

---

# Why Interfaces Matter?

Interfaces make mocking easy.

---

Example:

```go
type Repository interface {
    Save()
}
```

---

Production:

```go
PostgresRepository
```

---

Test:

```go
MockRepository
```

---

# Popular Mock Libraries

---

## testify/mock

```go
github.com/stretchr/testify
```

---

## gomock

```go
go.uber.org/mock
```

---

Interview Tip:

Even if you don't use a library, understand manual mocking.

---

# 5. Benchmark Testing

Very Common Performance Topic.

---

# What is Benchmarking?

Measures performance.

Example:

```text
How fast is this function?
```

---

# Benchmark Naming

Must start with:

```go
Benchmark
```

---

# Example

```go
func BenchmarkAdd(
    b *testing.B,
) {

    for i := 0; i < b.N; i++ {

        Add(1, 2)
    }
}
```

---

# Run Benchmark

```bash
go test -bench=.
```

---

Example Output

```text
BenchmarkAdd
1000000000
0.3 ns/op
```

---

# Meaning

```text
ns/op
```

=

```text
Nanoseconds Per Operation
```

---

# Benchmark Structure

```go
for i := 0; i < b.N; i++ {

}
```

---

Why?

Go automatically determines:

```text
How many iterations
```

are needed.

---

# Compare Implementations

Example:

```go
String Concatenation
```

vs

```go
strings.Builder
```

---

Benchmark shows:

```text
Which is faster
```

---

# Memory Benchmark

Run:

```bash
go test -bench=. -benchmem
```

---

Output:

```text
allocs/op
B/op
```

---

# Meaning

---

## B/op

Bytes allocated per operation.

---

## allocs/op

Allocations per operation.

---

# Example

```text
10 allocs/op
```

means:

```text
10 memory allocations
per execution
```

---

# Benchmark Example

Bad:

```go
func Join() string {

    s := ""

    for i := 0; i < 100; i++ {
        s += "Go"
    }

    return s
}
```

---

Better:

```go
func JoinBuilder() string {

    var b strings.Builder

    for i := 0; i < 100; i++ {
        b.WriteString("Go")
    }

    return b.String()
}
```

---

Benchmark proves which performs better.

---

# Test Coverage

Common Interview Topic.

---

# Generate Coverage

```bash
go test -cover
```

---

Output:

```text
coverage: 85%
```

---

# Detailed Coverage

```bash
go test -coverprofile=cover.out
```

---

View:

```bash
go tool cover -html=cover.out
```

---

# Should Coverage Be 100%?

Interview Answer:

```text
Not necessarily.
```

Goal:

```text
Meaningful Coverage
```

not:

```text
Artificial 100%
```

---

# Best Practices

---

## Test Public Behavior

Good:

```go
Test output
```

---

Bad:

```go
Test internal implementation
```

---

## Use Table Tests

Preferred in Go.

---

## Keep Tests Independent

One test should not depend on another.

---

## Mock External Systems

Don't hit:

```text
Database
Redis
Third-party APIs
```

during unit tests.

---

## Test Edge Cases

Examples:

```text
Empty Input
Nil Values
Negative Numbers
Errors
```

---

# Most Asked Interview Questions

---

## What is Unit Testing?

Testing a small unit of code independently.

---

## Why Use _test.go Files?

Go automatically discovers test files ending with:

```go
_test.go
```

---

## How Do You Run Tests?

```bash
go test
```

---

## What Are Table-Driven Tests?

Using a slice of test cases and looping through them.

---

## Why Are Table-Driven Tests Popular?

Less duplication and easier maintenance.

---

## What is Mocking?

Replacing real dependencies with fake implementations.

---

## Why Use Interfaces For Mocking?

Allows swapping real and mock implementations.

---

## What is Benchmark Testing?

Measuring execution speed and resource usage.

---

## How Do You Run Benchmarks?

```bash
go test -bench=.
```

---

## How Do You Measure Memory Usage?

```bash
go test -bench=. -benchmem
```

---

# Quick Revision Cheat Sheet

## Test Function

```go
func TestAdd(
    t *testing.T,
)
```

---

## Run Tests

```bash
go test
```

---

## Table Test

```go
tests := []struct{}
```

---

## Subtest

```go
t.Run(...)
```

---

## Mocking

```go
type MockRepo struct{}
```

---

## Interface

```go
type Repository interface{}
```

---

## Benchmark

```go
func BenchmarkAdd(
    b *testing.B,
)
```

---

## Run Benchmark

```bash
go test -bench=.
```

---

## Memory Benchmark

```bash
go test -bench=. -benchmem
```

---

## Coverage

```bash
go test -cover
```

---

# Golden Interview Answer

### How Is Testing Typically Done In Go?

Go provides a built-in testing framework through the `testing` package.

Production Go applications commonly use:

- Unit Tests for business logic
- Table-Driven Tests for multiple scenarios
- Mocking through interfaces
- Benchmarks for performance measurement
- Coverage reports for quality tracking

The most idiomatic Go testing style is:

```text
Table-Driven Tests
+
Interface-Based Mocking
```

because it keeps tests simple, maintainable, and scalable.

---

# End of Go Testing Revision