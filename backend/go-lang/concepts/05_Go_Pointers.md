# Go Pointers – Complete Interview Revision Guide

> Pointers are one of the most important Go concepts.
>
> Understanding pointers is essential for:
>
> - Memory optimization
> - Struct updates
> - Methods with pointer receivers
> - Large object handling
> - Backend development

---

# 1. What is a Pointer?

A pointer is a variable that stores the memory address of another variable.

Normally:

```go
x := 10
```

Variable:

```text
x = 10
```

Pointer:

```text
p = address of x
```

Instead of storing the value:

```text
10
```

it stores:

```text
0xc0000120a0
```

(memory address)

---

# Why Do We Need Pointers?

Without pointers:

```go
x := 10
y := x
```

Go copies the value.

```text
x -> 10
y -> 10
```

Two separate values.

Sometimes we want:

```text
Modify original value
```

without creating copies.

That's where pointers help.

---

# Memory Address

Every variable is stored somewhere in memory.

Example:

```go
x := 10
```

Visual:

```text
Memory Address    Value

0x1000           10
```

---

# Getting Memory Address (&)

Use:

```go
&
```

called the Address Operator.

---

## Example

```go
x := 10

fmt.Println(&x)
```

Output:

```text
0xc0000120a0
```

(actual address will vary)

---

# Creating a Pointer

```go
x := 10

p := &x
```

Visual:

```text
x = 10

p --> address of x
```

---

# Pointer Declaration

```go
var p *int
```

Meaning:

```text
p is a pointer to an int
```

---

## Example

```go
x := 10

var p *int = &x

fmt.Println(p)
```

Output:

```text
0xc0000120a0
```

---

# Dereferencing

Most Asked Interview Topic.

---

# What is Dereferencing?

Getting the actual value from a pointer.

Use:

```go
*
```

called the Dereference Operator.

---

## Example

```go
x := 10

p := &x

fmt.Println(*p)
```

Output:

```
10
```

---

# Visual

```text
x = 10

p ----> x
```

Pointer stores:

```go
p
```

Value stored at address:

```go
*p
```

---

# Example

```go
x := 100

p := &x

fmt.Println("Address:", p)
fmt.Println("Value:", *p)
```

Output:

```text
Address: 0xc0000120a0
Value: 100
```

---

# Modifying Value Through Pointer

Very Important.

---

## Example

```go
x := 10

p := &x

*p = 50

fmt.Println(x)
```

Output:

```
50
```

---

# Why?

Because:

```go
*p = 50
```

changes the value stored at x's memory address.

Visual:

Before:

```text
x = 10
```

After:

```text
x = 50
```

---

# Pass By Value

Extremely Important Interview Question.

---

# Does Go Use Pass By Reference?

No.

Go uses:

```text
Pass By Value
```

---

## Example

```go
func update(num int) {
    num = 100
}

func main() {
    x := 10

    update(x)

    fmt.Println(x)
}
```

Output:

```
10
```

---

# Why Didn't It Change?

When calling:

```go
update(x)
```

Go creates a copy.

Visual:

```text
x = 10

update receives:

num = 10
```

Changing:

```go
num = 100
```

only changes the copy.

Original remains unchanged.

---

# Pass By Value Visualization

```text
Original:

x = 10

Function Receives:

num = 10 (copy)

num becomes 100

x remains 10
```

---

# Modifying Original Objects Using Pointers

If we want to modify the original value:

Pass a pointer.

---

## Example

```go
func update(num *int) {
    *num = 100
}
```

Usage:

```go
x := 10

update(&x)

fmt.Println(x)
```

Output:

```
100
```

---

# How It Works

```go
update(&x)
```

passes:

```text
address of x
```

Function receives:

```go
*num
```

which points to original x.

---

# Visual

```text
x = 10

update(&x)

num ---> x

*num = 100

x = 100
```

---

# Pointers with Structs

Most Common Real-World Usage.

---

# Struct Example

```go
type User struct {
    Name string
    Age  int
}
```

---

## Without Pointer

```go
func update(user User) {
    user.Age = 50
}
```

Usage:

```go
u := User{
    Name: "Vinay",
    Age: 25,
}

update(u)

fmt.Println(u.Age)
```

Output:

```
25
```

---

# Why?

Because struct was copied.

Visual:

```text
Original User
      ↓
Copy Created
      ↓
Function Modifies Copy
```

Original remains unchanged.

---

# Using Pointer

```go
func update(user *User) {
    user.Age = 50
}
```

Usage:

```go
update(&u)
```

Output:

```
50
```

---

# Why Does This Work?

Function receives address of original struct.

No copy is created.

Visual:

```text
Function
   ↓
Original Struct Memory
```

Changes affect original object.

---

# Shortcut Syntax

Go automatically dereferences struct pointers.

---

## Example

```go
func update(user *User) {
    user.Age = 50
}
```

Actually Go internally treats:

```go
user.Age
```

as:

```go
(*user).Age
```

---

# Explicit Form

```go
func update(user *User) {
    (*user).Age = 50
}
```

Same result.

Most Go developers use:

```go
user.Age
```

because it's cleaner.

---

# Example: Update User

Most Interview-Friendly Example.

---

```go
type User struct {
    Name string
    Age  int
}

func update(u *User) {
    u.Age = 30
}

func main() {

    user := User{
        Name: "Vinay",
        Age: 25,
    }

    update(&user)

    fmt.Println(user)
}
```

Output:

```go
{Vinay 30}
```

---

# Pointer to Struct

Create directly.

```go
user := &User{
    Name: "Vinay",
    Age: 25,
}
```

Type:

```go
*User
```

---

# Access Fields

```go
fmt.Println(user.Name)
```

Output:

```
Vinay
```

Even though:

```go
user
```

is a pointer.

Go automatically dereferences it.

---

# new Keyword

Creates memory and returns pointer.

---

## Example

```go
user := new(User)
```

Equivalent to:

```go
user := &User{}
```

---

## Usage

```go
user.Name = "Vinay"
```

---

# Nil Pointers

Pointers can be nil.

---

## Example

```go
var p *int

fmt.Println(p)
```

Output:

```go
<nil>
```

---

# Danger

```go
fmt.Println(*p)
```

Output:

```text
panic:
nil pointer dereference
```

---

# Safe Check

```go
if p != nil {
    fmt.Println(*p)
}
```

---

# When to Use Pointers?

Use pointers when:

---

## 1. Need to Modify Original Data

```go
func update(u *User)
```

---

## 2. Large Structs

Avoid expensive copying.

```go
type User struct {
    LargeData [10000]int
}
```

Passing pointers is more efficient.

---

## 3. Shared Data

Multiple functions can work on same object.

---

## 4. Pointer Receivers

```go
func (u *User) UpdateAge()
```

Allows modification of original struct.

---

# Value vs Pointer

| Feature | Value | Pointer |
|----------|---------|----------|
| Stores | Actual Value | Address |
| Copy Created | Yes | No |
| Modify Original | No | Yes |
| Memory Efficient | Less | More |
| Large Structs | Costly | Efficient |

---

# Interview Questions

---

## What is a Pointer?

A variable that stores the memory address of another variable.

---

## What Does & Mean?

Returns memory address.

```go
&x
```

---

## What Does * Mean?

Dereference operator.

Gets value stored at address.

```go
*p
```

---

## Is Go Pass By Reference?

No.

Go is pass-by-value.

---

## Then How Do Pointers Work?

The pointer itself is passed by value.

But it contains the address of the original object.

Therefore the original data can be modified.

---

## Why Use Pointers With Structs?

- Avoid copying large structs
- Modify original object
- Better performance

---

## What Happens If Pointer Is Nil?

Dereferencing causes panic.

```go
*p
```

on nil pointer crashes.

---

## Difference Between Value and Pointer Receiver?

Value Receiver:

```go
func (u User)
```

Works on copy.

---

Pointer Receiver:

```go
func (u *User)
```

Works on original struct.

---

# Quick Revision Cheat Sheet

## Get Address

```go
x := 10

p := &x
```

---

## Dereference

```go
*p
```

---

## Modify Original Value

```go
*p = 100
```

---

## Pass Pointer

```go
update(&x)
```

---

## Pointer Parameter

```go
func update(x *int)
```

---

## Struct Pointer

```go
func update(u *User)
```

---

## Create Struct Pointer

```go
user := &User{}
```

---

## new Keyword

```go
user := new(User)
```

---

## Nil Pointer

```go
var p *int
```

Value:

```go
nil
```

---

# Golden Interview Answer

### Why Are Pointers Important In Go?

Pointers allow functions and methods to work with the original data instead of copies.

They help:

- Modify original objects
- Improve performance
- Avoid copying large structs
- Enable efficient backend applications

---

# End of Pointers Revision