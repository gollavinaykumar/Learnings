# Go Generics – Complete Interview Revision Guide

> Generics were introduced in **Go 1.18**.
>
> Before Generics, Go developers often had to:
>
> - Duplicate code
> - Use interface{}
> - Lose type safety
>
> Generics solve these problems by allowing functions, structs, and data structures to work with multiple types while maintaining compile-time type safety.

---

# 1. What are Generics?

Generics allow writing code once and using it with different data types.

Without Generics:

```go
func PrintInt(value int) {
    fmt.Println(value)
}

func PrintString(value string) {
    fmt.Println(value)
}
```

Problem:

```text
Duplicate code
```

---

# With Generics

```go
func Print[T any](value T) {
    fmt.Println(value)
}
```

Usage:

```go
Print(10)
Print("hello")
Print(true)
```

Output:

```text
10
hello
true
```

---

# Simple Definition

Generics allow functions and types to work with different data types while keeping type safety.

---

# Generic Syntax

```go
[T any]
```

---

## Example

```go
func Print[T any](value T) {
    fmt.Println(value)
}
```

---

# Breakdown

```go
T
```

Type Parameter

---

```go
any
```

Type Constraint

---

Meaning:

```text
T can be any type
```

---

# Visual

```go
Print(10)
```

Compiler sees:

```go
T = int
```

---

```go
Print("Go")
```

Compiler sees:

```go
T = string
```

---

# Why Not Use interface{}?

Before Generics:

```go
func Print(value interface{}) {
    fmt.Println(value)
}
```

Works.

But:

```text
No Type Safety
Type Assertions Needed
Less Performance
```

---

# Example

```go
func Add(a, b interface{}) {
}
```

Compiler cannot verify types.

---

# With Generics

```go
func Add[T int](a, b T) T {
    return a + b
}
```

Compiler checks types.

---

# Benefits of Generics

---

## Code Reuse

Write once.

Use everywhere.

---

## Type Safety

Compiler catches mistakes.

---

## Better Performance

Avoids many interface conversions.

---

## Cleaner APIs

Less duplicate code.

---

# 2. Type Parameters

Type parameters are placeholders for actual types.

---

## Example

```go
func Print[T any](value T)
```

Here:

```go
T
```

is a type parameter.

---

# Multiple Type Parameters

```go
func Print[T any, U any](
    a T,
    b U,
) {
    fmt.Println(a, b)
}
```

Usage:

```go
Print(10, "Go")
```

Output:

```text
10 Go
```

---

# Generic Function Example

---

## Without Generics

```go
func MaxInt(a, b int) int
```

---

```go
func MaxFloat(a, b float64) float64
```

Duplicate code.

---

## With Generics

```go
func Max[T int | float64](
    a, b T,
) T {

    if a > b {
        return a
    }

    return b
}
```

Usage:

```go
fmt.Println(Max(10, 20))
fmt.Println(Max(10.5, 5.5))
```

Output:

```text
20
10.5
```

---

# 3. Type Constraints

Most Important Generics Topic.

---

# What is a Constraint?

A constraint defines which types are allowed.

---

## Example

```go
[T any]
```

Means:

```text
Any type allowed
```

---

# Example

```go
func Print[T any](value T)
```

Can accept:

```go
int
string
bool
struct
slice
map
```

---

# Restricted Constraint

```go
func Add[T int](a, b T) T
```

Only:

```go
int
```

allowed.

---

Usage:

```go
Add(10, 20)
```

Works.

---

```go
Add("a", "b")
```

Compilation Error.

---

# Multiple Types

```go
func Add[T int | float64](
    a, b T,
) T {
    return a + b
}
```

Allowed:

```go
int
float64
```

---

# Visual

```text
Constraint
     ↓
Allowed Types
```

---

# Custom Constraint

Very Common Interview Topic.

---

## Example

```go
type Number interface {
    int | int32 | int64 |
    float32 | float64
}
```

---

Usage:

```go
func Sum[T Number](
    a, b T,
) T {

    return a + b
}
```

---

Works for:

```go
int
int64
float64
```

---

# Why Constraints Matter?

Without constraints:

```go
a + b
```

may not be valid.

---

Example:

```go
string
struct
slice
```

do not support all numeric operations.

---

Constraints tell compiler:

```text
Which operations are safe.
```

---

# Built-in Constraints

Common:

```go
any
```

Equivalent to:

```go
interface{}
```

---

# Comparable Constraint

Another Important Topic.

---

## Example

```go
func Equal[T comparable](
    a, b T,
) bool {

    return a == b
}
```

---

Usage:

```go
Equal(10, 10)
Equal("go", "go")
```

Output:

```text
true
true
```

---

Why?

Because:

```go
==
```

requires comparable types.

---

# 4. Generic Structs

Generics can be used with structs.

---

# Example

```go
type Box[T any] struct {
    Value T
}
```

---

Usage

```go
intBox := Box[int]{
    Value: 100,
}
```

---

```go
strBox := Box[string]{
    Value: "Go",
}
```

---

Output:

```text
100
Go
```

---

# Visual

```text
Box[int]
Box[string]
Box[User]
```

Same struct.

Different types.

---

# Generic Methods

---

## Example

```go
type Box[T any] struct {
    Value T
}

func (b Box[T]) Get() T {
    return b.Value
}
```

---

Usage

```go
box := Box[int]{
    Value: 100,
}

fmt.Println(box.Get())
```

Output:

```text
100
```

---

# 5. Generic Data Structures

Most Common Real-World Usage.

---

Before Generics:

```go
IntStack
StringStack
FloatStack
```

Many versions.

---

With Generics:

```go
Stack[T]
```

Single implementation.

---

# Generic Stack Example

```go
type Stack[T any] struct {
    items []T
}
```

---

# Push

```go
func (s *Stack[T]) Push(
    item T,
) {

    s.items =
        append(s.items, item)
}
```

---

# Pop

```go
func (s *Stack[T]) Pop() T {

    n := len(s.items)

    item := s.items[n-1]

    s.items = s.items[:n-1]

    return item
}
```

---

# Usage

```go
var stack Stack[int]

stack.Push(10)
stack.Push(20)

fmt.Println(stack.Pop())
```

Output:

```
20
```

---

# String Stack

Same code.

```go
var stack Stack[string]

stack.Push("Go")
stack.Push("Lang")
```

---

No duplicate implementation needed.

---

# Generic Queue Example

```go
type Queue[T any] struct {
    items []T
}
```

Works for:

```go
Queue[int]
Queue[string]
Queue[User]
```

---

# Generic Map Wrapper

```go
type Store[K comparable, V any] struct {
    data map[K]V
}
```

Usage:

```go
Store[string, int]
Store[int, User]
```

---

# Real Backend Examples

Generics are commonly used for:

---

## Repository Helpers

```go
func Save[T any](obj T)
```

---

## API Responses

```go
type Response[T any] struct {
    Data T
}
```

---

## Caches

```go
type Cache[K comparable, V any]
```

---

## Queues

```go
Queue[T]
```

---

## Stacks

```go
Stack[T]
```

---

## Collections

```go
List[T]
```

---

# Generics vs Interface{}

Most Asked Interview Question.

---

## Interface{}

```go
func Print(v interface{})
```

Pros:

```text
Flexible
```

Cons:

```text
No Type Safety
Requires Assertions
```

---

## Generics

```go
func Print[T any](v T)
```

Pros:

```text
Type Safe
Reusable
Cleaner
```

Cons:

```text
Slightly More Complex
```

---

# Interface vs Generics

Another Important Question.

---

# Interfaces

Used when behavior matters.

Example:

```go
type Animal interface {
    Speak()
}
```

---

Goal:

```text
Polymorphism
```

---

# Generics

Used when type varies but logic remains same.

Example:

```go
Stack[int]
Stack[string]
```

---

Goal:

```text
Code Reuse
```

---

# Rule of Thumb

Use:

```go
Interface
```

when behavior changes.

---

Use:

```go
Generics
```

when only data type changes.

---

# Most Asked Interview Questions

---

## What are Generics?

Generics allow writing reusable, type-safe code that works with multiple data types.

---

## When Were Generics Introduced?

```text
Go 1.18
```

---

## What is a Type Parameter?

Placeholder for a type.

Example:

```go
[T any]
```

---

## What is a Type Constraint?

Defines which types are allowed.

Example:

```go
[T int | float64]
```

---

## What is any?

Alias for:

```go
interface{}
```

---

## What is comparable?

Constraint for types supporting:

```go
==
!=
```

---

## Why Use Generics?

- Reusability
- Type Safety
- Less Duplication
- Better Performance

---

## When Should Interfaces Be Used Instead?

When behavior varies.

Example:

```go
Animal
Reader
Writer
```

---

## Common Generic Data Structures?

```text
Stack
Queue
Cache
List
Repository
Response Wrapper
```

---

# Quick Revision Cheat Sheet

## Generic Function

```go
func Print[T any](v T)
```

---

## Generic Struct

```go
type Box[T any] struct {
    Value T
}
```

---

## Multiple Types

```go
[T int | float64]
```

---

## Custom Constraint

```go
type Number interface {
    int | float64
}
```

---

## Comparable

```go
[T comparable]
```

---

## Generic Stack

```go
type Stack[T any]
```

---

## Generic Queue

```go
type Queue[T any]
```

---

## Generic Map Wrapper

```go
type Store[K comparable, V any]
```

---

## any

```go
any
```

Same as:

```go
interface{}
```

---

# Golden Interview Answer

### Why Were Generics Added To Go?

Generics were added to eliminate code duplication while preserving type safety.

Before Generics, developers often had to:

```text
Write multiple versions
of the same code
```

or use:

```go
interface{}
```

which lost type safety.

Generics allow Go developers to build reusable libraries, generic data structures, and cleaner APIs while keeping compile-time type checking.

---

# End of Go Generics Revision