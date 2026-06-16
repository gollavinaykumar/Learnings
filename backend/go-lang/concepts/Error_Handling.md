# Go Error Handling – Complete Interview Revision Guide

> Error Handling is one of the most important Go concepts.
>
> Unlike Java, Python, or C#, Go does NOT use exceptions for normal errors.
>
> Go follows:
>
> ```text
> Explicit Error Handling
> ```
>
> This is heavily used in:
>
> - APIs
> - Databases
> - Microservices
> - File Operations
> - Production Backend Systems

---

# 1. What is Error Handling?

Errors represent situations where something went wrong.

Examples:

- Database connection failed
- File not found
- API request failed
- Invalid input

Instead of throwing exceptions, Go returns errors as values.

---

# Basic Error Handling

```go
result, err := doSomething()
```

If:

```go
err == nil
```

Operation succeeded.

If:

```go
err != nil
```

Something went wrong.

---

# Example

```go
package main

import (
    "fmt"
)

func divide(a, b int) (int, error) {

    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }

    return a / b, nil
}

func main() {

    result, err := divide(10, 2)

    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Println(result)
}
```

Output:

```
5
```

---

# Why Go Uses Errors Instead of Exceptions?

Go philosophy:

```text
Errors are values.
```

Benefits:

- Easy to understand
- Explicit control flow
- No hidden exceptions
- Easier debugging

---

# Common Pattern

You will see this everywhere:

```go
value, err := someFunction()

if err != nil {
    return err
}
```

This is the standard Go style.

---

# Error Type

Built-in interface:

```go
type error interface {
    Error() string
}
```

Every error implements:

```go
Error() string
```

method.

---

# Creating Errors

Use:

```go
errors.New()
```

---

## Example

```go
import "errors"

func getUser() error {
    return errors.New("user not found")
}
```

Output:

```
user not found
```

---

# fmt.Errorf()

Another common way.

---

## Example

```go
err := fmt.Errorf("user %d not found", 101)
```

Output:

```
user 101 not found
```

---

# Custom Errors

Most Asked Interview Topic.

---

# Why Custom Errors?

Sometimes a simple string isn't enough.

We may need additional information.

---

## Example

```go
type ValidationError struct {
    Field string
}
```

---

# Implement Error Interface

```go
func (v ValidationError) Error() string {
    return v.Field + " is invalid"
}
```

Now:

```go
ValidationError
```

becomes an error.

---

# Usage

```go
func validate() error {
    return ValidationError{
        Field: "email",
    }
}
```

Output:

```
email is invalid
```

---

# Complete Example

```go
type ValidationError struct {
    Field string
}

func (v ValidationError) Error() string {
    return v.Field + " is invalid"
}

func validateEmail() error {
    return ValidationError{
        Field: "email",
    }
}
```

---

# Why Use Custom Errors?

Useful when:

- Returning structured information
- Validation failures
- Business rules
- Domain-specific errors

---

# Error Propagation

Very Important Interview Topic.

---

# What is Error Propagation?

Passing errors up the call stack.

---

## Example

```go
func readFile() error {
    return errors.New("file not found")
}
```

---

## Service Layer

```go
func loadConfig() error {

    err := readFile()

    if err != nil {
        return err
    }

    return nil
}
```

---

## Main

```go
err := loadConfig()

if err != nil {
    fmt.Println(err)
}
```

Output:

```
file not found
```

---

# Visual

```text
readFile()
     ↓
loadConfig()
     ↓
main()
```

Error travels upward.

---

# Why?

Each layer can decide:

```text
Handle error
OR
Pass error upward
```

---

# Wrapping Errors

Most Important Modern Go Topic.

Introduced in:

```text
Go 1.13
```

---

# Problem

Without wrapping:

```go
return err
```

We lose context.

---

## Example

```go
file not found
```

Question:

```text
Which file?
Which operation?
```

Not clear.

---

# Solution: Error Wrapping

Use:

```go
%w
```

inside:

```go
fmt.Errorf()
```

---

## Example

```go
return fmt.Errorf(
    "failed to load config: %w",
    err,
)
```

Output:

```text
failed to load config:
file not found
```

---

# Complete Example

```go
func readFile() error {
    return errors.New("file not found")
}

func loadConfig() error {

    err := readFile()

    if err != nil {
        return fmt.Errorf(
            "loadConfig failed: %w",
            err,
        )
    }

    return nil
}
```

Output:

```text
loadConfig failed:
file not found
```

---

# Why Wrapping Matters?

Provides context while preserving original error.

---

# Checking Wrapped Errors

Use:

```go
errors.Is()
```

---

## Example

```go
var ErrNotFound = errors.New("not found")
```

---

```go
err := fmt.Errorf(
    "db error: %w",
    ErrNotFound,
)
```

---

Check:

```go
if errors.Is(err, ErrNotFound) {
    fmt.Println("Not Found")
}
```

Output:

```
Not Found
```

---

# Sentinel Errors

Very Common Interview Question.

---

# What is a Sentinel Error?

A predefined shared error value.

---

## Example

```go
var ErrNotFound = errors.New("not found")
```

---

Usage:

```go
return ErrNotFound
```

---

Comparison:

```go
if errors.Is(err, ErrNotFound) {
}
```

---

# Why Sentinel Errors?

Allows reliable error checking.

---

# Example

```go
var ErrUserNotFound =
    errors.New("user not found")
```

---

```go
func getUser(id int) error {
    return ErrUserNotFound
}
```

---

Check:

```go
if errors.Is(err, ErrUserNotFound) {
    fmt.Println("Handle User Missing")
}
```

---

# Real World Examples

Standard Library uses:

```go
io.EOF
```

Example:

```go
if err == io.EOF {
}
```

---

# Panic

Very Important Interview Topic.

---

# What is Panic?

Panic immediately stops normal execution.

Think:

```text
Something impossible happened.
```

---

## Example

```go
panic("something went wrong")
```

Output:

```text
panic:
something went wrong
```

Program crashes.

---

# Example

```go
func main() {
    panic("database corrupted")
}
```

Output:

```text
panic: database corrupted
```

---

# When to Use Panic?

Rarely.

Use only for:

- Programmer mistakes
- Impossible states
- Startup failures
- Critical corruption

---

# When NOT to Use Panic?

Bad:

```go
user not found
```

Bad:

```go
validation failed
```

Bad:

```go
API returned 404
```

These should return errors.

---

# Error vs Panic

| Error | Panic |
|---------|---------|
| Expected failure | Unexpected failure |
| Can recover | Program crashes |
| Normal flow | Exceptional situation |

---

# Recover

Used to catch a panic.

---

# Example

```go
func main() {

    defer func() {

        if r := recover(); r != nil {
            fmt.Println("Recovered:", r)
        }

    }()

    panic("something bad happened")
}
```

Output:

```
Recovered:
something bad happened
```

---

# Why Use Recover?

Prevent application crash.

Common in:

- HTTP middleware
- Web frameworks
- Background workers

---

# Visual

Without Recover:

```text
panic
  ↓
program exits
```

---

With Recover:

```text
panic
  ↓
recover
  ↓
continue running
```

---

# Real Backend Example

HTTP Server:

```go
func middleware(next Handler) {

    defer func() {

        if r := recover(); r != nil {
            log.Println(r)
        }

    }()

}
```

Prevents one request from crashing server.

---

# errors.As()

Advanced Interview Topic.

Used for custom errors.

---

## Example

```go
var validationErr ValidationError

if errors.As(err, &validationErr) {
    fmt.Println(validationErr.Field)
}
```

Used when checking error types.

---

# Most Asked Interview Questions

---

## What is Error Handling in Go?

Go returns errors as values instead of throwing exceptions.

---

## What is the Standard Error Pattern?

```go
value, err := someFunction()

if err != nil {
    return err
}
```

---

## What is a Custom Error?

A struct implementing:

```go
Error() string
```

---

## What is Error Propagation?

Passing errors upward through function calls.

---

## What is Error Wrapping?

Adding context while preserving original error.

```go
fmt.Errorf("%w", err)
```

---

## What is a Sentinel Error?

A predefined shared error value.

Example:

```go
var ErrNotFound =
    errors.New("not found")
```

---

## How Do You Check Wrapped Errors?

```go
errors.Is(err, ErrNotFound)
```

---

## What is Panic?

An unrecoverable runtime error that stops execution.

---

## What is Recover?

A mechanism to catch and handle panics.

---

## When Should Panic Be Used?

Only for truly exceptional situations.

Not for normal business errors.

---

# Real Backend Error Handling Pattern

```go
func GetUser(id int) (*User, error) {

    user, err := repository.Find(id)

    if err != nil {
        return nil,
            fmt.Errorf(
                "failed to fetch user: %w",
                err,
            )
    }

    return user, nil
}
```

Benefits:

- Clear errors
- Preserves root cause
- Easy debugging

---

# Quick Revision Cheat Sheet

## Basic Error

```go
result, err := doSomething()

if err != nil {
    return err
}
```

---

## Create Error

```go
errors.New("something failed")
```

---

## Formatted Error

```go
fmt.Errorf("user %d not found", id)
```

---

## Custom Error

```go
type ValidationError struct{}

func (v ValidationError) Error() string
```

---

## Error Propagation

```go
return err
```

---

## Wrap Error

```go
fmt.Errorf("service failed: %w", err)
```

---

## Sentinel Error

```go
var ErrNotFound =
    errors.New("not found")
```

---

## Check Sentinel Error

```go
errors.Is(err, ErrNotFound)
```

---

## Check Error Type

```go
errors.As(err, &target)
```

---

## Panic

```go
panic("something bad happened")
```

---

## Recover

```go
recover()
```

---

# Golden Interview Answer

### Why Does Go Prefer Explicit Error Handling Instead of Exceptions?

Go treats errors as normal values.

This makes error handling:

- Explicit
- Predictable
- Easy to follow
- Easy to debug

Combined with:

```go
custom errors
error wrapping
sentinel errors
errors.Is()
errors.As()
```

Go provides powerful and maintainable error handling without relying on exceptions.

---

# End of Error Handling Revision