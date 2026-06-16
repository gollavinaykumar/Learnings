# Go Interfaces – Complete Interview Revision Guide

> One of the most important Go topics for backend interviews.
>
> Interfaces are heavily used in:
>
> - Clean Architecture
> - Dependency Injection
> - Microservices
> - Testing & Mocking
> - Database Layers
> - API Development

---

# 1. What is an Interface?

An interface defines behavior.

It tells:

```text
"What a type can do"
```

instead of

```text
"What a type is"
```

---

## Real Life Example

Think about a Remote Control.

A remote doesn't care whether it's controlling:

- Samsung TV
- LG TV
- Sony TV

It only cares that the device can:

```go
TurnOn()
TurnOff()
```

That's exactly how interfaces work.

---

# Basic Interface

```go
type Animal interface {
    Speak()
}
```

This means:

```text
Any type that has Speak()
can be treated as Animal.
```

---

# Example

```go
type Dog struct{}

func (d Dog) Speak() {
    fmt.Println("Woof")
}
```

Dog has:

```go
Speak()
```

Therefore:

```text
Dog implements Animal
```

---

## Usage

```go
var a Animal

a = Dog{}

a.Speak()
```

Output:

```
Woof
```

---

# Key Interview Point

An interface defines behavior, not data.

Example:

```go
type Animal interface {
    Speak()
}
```

No fields.

Only methods.

---

# Implicit Implementation

Most Important Interview Topic.

---

# How Other Languages Work

Languages like Java require:

```java
class Dog implements Animal
```

---

# How Go Works

Go does NOT require:

```go
implements
```

keyword.

---

## Example

```go
type Animal interface {
    Speak()
}

type Dog struct{}

func (d Dog) Speak() {
    fmt.Println("Woof")
}
```

That's enough.

Go automatically understands:

```text
Dog implements Animal
```

---

# Why?

Because Dog has all methods required by Animal.

---

# Example

```go
type Animal interface {
    Speak()
}

type Cat struct{}

func (c Cat) Speak() {
    fmt.Println("Meow")
}
```

Cat also implements Animal automatically.

---

# Multiple Implementations

```go
type Dog struct{}
type Cat struct{}
type Cow struct{}
```

All can implement:

```go
Speak()
```

---

## Example

```go
type Animal interface {
    Speak()
}

type Dog struct{}

func (d Dog) Speak() {
    fmt.Println("Woof")
}

type Cat struct{}

func (c Cat) Speak() {
    fmt.Println("Meow")
}

func makeAnimalSpeak(a Animal) {
    a.Speak()
}

func main() {
    makeAnimalSpeak(Dog{})
    makeAnimalSpeak(Cat{})
}
```

Output:

```
Woof
Meow
```

---

# Why Interfaces Are Powerful

Without Interface:

```go
func makeDogSpeak(d Dog)
func makeCatSpeak(c Cat)
func makeCowSpeak(c Cow)
```

Many functions.

---

With Interface:

```go
func makeAnimalSpeak(a Animal)
```

Single function.

---

# Interface Value Internally

An interface contains:

```go
Type
Value
```

Visual:

```text
Animal
   |
   +---- Dog
   +---- Cat
```

Example:

```go
var a Animal

a = Dog{}
```

Internally:

```text
Type  = Dog
Value = Dog{}
```

---

# Empty Interface

Another Very Important Topic.

---

## Definition

```go
interface{}
```

or

```go
any
```

(Go 1.18+)

---

## Example

```go
var data interface{}
```

This variable can hold anything.

---

## Store Different Types

```go
var value interface{}

value = 10
value = "hello"
value = true
```

All valid.

---

# Why?

Every type implements:

```go
interface{}
```

because it has:

```go
0 methods
```

to satisfy.

---

# Example

```go
func printValue(v interface{}) {
    fmt.Println(v)
}

printValue(10)
printValue("hello")
printValue(true)
```

Output:

```
10
hello
true
```

---

# any Keyword

Modern Go equivalent:

```go
func printValue(v any)
```

Same as:

```go
func printValue(v interface{})
```

---

# When Is Empty Interface Used?

Common examples:

```go
fmt.Println()
json.Unmarshal()
database/sql
```

Because they handle many data types.

---

# Type Assertion

Most Asked Interview Question.

---

# Problem

When using interface:

```go
var value interface{} = "hello"
```

Go only knows:

```text
value is interface{}
```

We need actual type.

---

# Type Assertion Syntax

```go
value.(Type)
```

---

## Example

```go
var value interface{} = "hello"

str := value.(string)

fmt.Println(str)
```

Output:

```
hello
```

---

# Safe Type Assertion

Recommended.

---

## Example

```go
var value interface{} = "hello"

str, ok := value.(string)

fmt.Println(str)
fmt.Println(ok)
```

Output:

```
hello
true
```

---

# Wrong Type

```go
num, ok := value.(int)
```

Output:

```
0
false
```

---

# Unsafe Type Assertion

```go
num := value.(int)
```

Output:

```text
panic:
interface conversion
```

---

# Interview Rule

Always prefer:

```go
value, ok := x.(Type)
```

instead of:

```go
value := x.(Type)
```

---

# Type Switch

Used to check multiple possible types.

---

## Syntax

```go
switch v := value.(type) {
case string:
case int:
}
```

---

## Example

```go
func checkType(v interface{}) {

    switch value := v.(type) {

    case string:
        fmt.Println("String:", value)

    case int:
        fmt.Println("Int:", value)

    case bool:
        fmt.Println("Bool:", value)

    default:
        fmt.Println("Unknown")
    }
}
```

Usage:

```go
checkType("hello")
checkType(100)
checkType(true)
```

Output:

```
String: hello
Int: 100
Bool: true
```

---

# Why Type Switch?

Without it:

```go
if _, ok := v.(string); ok {}

if _, ok := v.(int); ok {}

if _, ok := v.(bool); ok {}
```

Messy.

---

Type switch is cleaner.

---

# Interface Segregation Principle (ISP)

Very Important Backend Design Topic.

---

# What Is ISP?

A client should not be forced to implement methods it doesn't need.

---

# Bad Design

```go
type Animal interface {
    Eat()
    Sleep()
    Fly()
}
```

Now:

```go
type Dog struct{}
```

Dog must implement:

```go
Fly()
```

which makes no sense.

---

# Better Design

Split interfaces.

```go
type Eater interface {
    Eat()
}

type Sleeper interface {
    Sleep()
}

type Flyer interface {
    Fly()
}
```

Now:

```go
type Dog struct{}
```

implements:

```go
Eater
Sleeper
```

Only.

---

# Why Go Encourages Small Interfaces

Go philosophy:

```text
Small Interfaces
Large Implementations
```

---

# Example from Standard Library

Famous interface:

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Only one method.

Yet used everywhere.

---

# Another Example

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Simple and powerful.

---

# Interface Composition

Interfaces can be combined.

---

## Example

```go
type Reader interface {
    Read([]byte)
}

type Writer interface {
    Write([]byte)
}
```

Combine:

```go
type ReadWriter interface {
    Reader
    Writer
}
```

Now:

```go
ReadWriter
```

requires:

```go
Read()
Write()
```

---

# Real Backend Example

Database layer:

```go
type UserRepository interface {
    GetUser(id int) (*User, error)
    CreateUser(user User) error
}
```

---

Service layer:

```go
type UserService struct {
    repo UserRepository
}
```

Now Service depends on:

```go
Interface
```

not

```go
Concrete Database
```

---

# Why Is This Useful?

Easy testing.

---

## Production

```go
MySQLRepository
```

---

## Testing

```go
MockRepository
```

Both satisfy:

```go
UserRepository
```

---

# Dependency Injection Using Interfaces

Very common interview question.

---

## Interface

```go
type PaymentGateway interface {
    Pay(amount float64)
}
```

---

## Razorpay

```go
type Razorpay struct{}
```

---

## Stripe

```go
type Stripe struct{}
```

Both implement:

```go
Pay()
```

Service doesn't care which implementation is used.

---

# Interface Nil Trap

Advanced Interview Topic.

---

## Example

```go
var d *Dog = nil

var a Animal = d
```

Now:

```go
a != nil
```

Why?

Because interface contains:

```text
Type  = *Dog
Value = nil
```

Interface itself is not nil.

---

# Most Asked Interview Questions

---

## What Is an Interface?

A collection of method signatures that defines behavior.

---

## Does Go Use Explicit Implementation?

No.

Go uses implicit implementation.

---

## What Is Implicit Implementation?

If a type has all required methods, it automatically implements the interface.

---

## What Is Empty Interface?

```go
interface{}
```

or

```go
any
```

Can hold values of any type.

---

## What Is Type Assertion?

Used to extract concrete type from interface.

```go
value, ok := x.(string)
```

---

## What Is Type Switch?

Used to check multiple possible types.

```go
switch v := x.(type)
```

---

## What Is Interface Segregation?

Create small focused interfaces.

Avoid large interfaces with unrelated methods.

---

## Why Are Interfaces Important?

- Decoupling
- Dependency Injection
- Testing
- Mocking
- Clean Architecture
- Backend Scalability

---

# Quick Revision Cheat Sheet

## Interface

```go
type Animal interface {
    Speak()
}
```

---

## Implementation

```go
type Dog struct{}

func (d Dog) Speak() {}
```

No keyword required.

---

## Empty Interface

```go
interface{}
```

or

```go
any
```

---

## Type Assertion

```go
value, ok := x.(string)
```

---

## Type Switch

```go
switch v := x.(type)
```

---

## Interface Composition

```go
type ReadWriter interface {
    Reader
    Writer
}
```

---

## Interface Segregation

Bad:

```go
type Animal interface {
    Eat()
    Sleep()
    Fly()
}
```

Good:

```go
type Eater interface {
    Eat()
}

type Flyer interface {
    Fly()
}
```

---

## Backend Usage

```go
type UserRepository interface {
    GetUser(id int)
}
```

Service depends on interface, not implementation.

---

# Golden Interview Answer

### Why Are Interfaces So Important In Go?

Interfaces allow code to depend on behavior rather than concrete implementations.

This makes applications:

- Flexible
- Testable
- Maintainable
- Loosely Coupled

That's why interfaces are heavily used in modern Go backend systems.

---

# End of Interfaces Revision