# Go Structs & Methods – Complete Interview Revision Guide

> Simple explanations + examples + interview questions.

---

# 1. Structs

A struct is a custom data type that groups related data together.

Think of a struct as a blueprint for creating objects.

### Real-Life Example

A User has:

- Name
- Age
- Email

Instead of creating separate variables:

```go
name := "Vinay"
age := 25
email := "vinay@gmail.com"
```

We can group them together:

```go
type User struct {
    Name  string
    Age   int
    Email string
}
```

---

# Why Use Structs?

Without structs:

```go
name := "John"
age := 25
city := "Hyderabad"
```

Managing many related variables becomes difficult.

With structs:

```go
type User struct {
    Name string
    Age  int
    City string
}
```

Everything stays together.

---

# Define a Struct

```go
type User struct {
    Name string
    Age  int
}
```

---

# Create Struct Object

## Method 1: Named Fields (Recommended)

```go
user := User{
    Name: "Vinay",
    Age:  25,
}
```

Output:

```go
{Vinay 25}
```

---

## Method 2: Positional Values

```go
user := User{
    "Vinay",
    25,
}
```

Works but not recommended.

Why?

Because field order matters.

---

# Access Struct Fields

```go
fmt.Println(user.Name)
fmt.Println(user.Age)
```

Output:

```
Vinay
25
```

---

# Modify Struct Fields

```go
user.Age = 30

fmt.Println(user.Age)
```

Output:

```
30
```

---

# Zero Values in Struct

If fields are not initialized, Go assigns zero values.

```go
var user User

fmt.Println(user)
```

Output:

```go
{ 0}
```

Meaning:

```go
Name = ""
Age = 0
```

---

# Anonymous Struct

Struct without creating a type.

```go
user := struct {
    Name string
    Age  int
}{
    Name: "Vinay",
    Age:  25,
}
```

Used for temporary data.

---

# Nested Struct

A struct can contain another struct.

### Example

```go
type Address struct {
    City string
}

type User struct {
    Name    string
    Address Address
}
```

Create:

```go
user := User{
    Name: "Vinay",
    Address: Address{
        City: "Hyderabad",
    },
}
```

Access:

```go
fmt.Println(user.Address.City)
```

Output:

```
Hyderabad
```

---

# Struct Comparison

Structs can be compared if all fields are comparable.

### Example

```go
u1 := User{
    Name: "Vinay",
    Age: 25,
}

u2 := User{
    Name: "Vinay",
    Age: 25,
}

fmt.Println(u1 == u2)
```

Output:

```
true
```

---

# Struct Interview Questions

### What is a Struct?

A custom data type that groups related fields together.

---

### Why Use Structs?

To organize related data into a single unit.

---

### Can Structs Be Compared?

Yes, if all fields are comparable.

---

# 2. Methods

A method is a function attached to a struct.

Think:

```text
Function + Struct = Method
```

---

# Normal Function

```go
func SayHello() {
    fmt.Println("Hello")
}
```

This function belongs to nobody.

---

# Method

```go
func (u User) SayHello() {
    fmt.Println("Hello", u.Name)
}
```

This method belongs to User.

---

# Method Syntax

```go
func (receiver Type) MethodName() {
}
```

Example:

```go
func (u User) SayHello() {
    fmt.Println("Hello", u.Name)
}
```

---

# Example

```go
type User struct {
    Name string
}

func (u User) SayHello() {
    fmt.Println("Hello", u.Name)
}

func main() {
    user := User{Name: "Vinay"}

    user.SayHello()
}
```

Output:

```
Hello Vinay
```

---

# Receiver

This part:

```go
(u User)
```

is called the receiver.

It tells Go:

```text
This method belongs to User
```

---

# Value Receiver

Most asked interview topic.

---

## Example

```go
type User struct {
    Name string
}

func (u User) ChangeName() {
    u.Name = "Alex"
}
```

Usage:

```go
user := User{Name: "Vinay"}

user.ChangeName()

fmt.Println(user.Name)
```

Output:

```
Vinay
```

---

# Why Didn't It Change?

Because value receivers work on a copy.

Visual:

```text
Original User
      ↓
Copy Created
      ↓
Method Uses Copy
```

The original struct remains unchanged.

---

# Value Receiver Summary

```go
func (u User)
```

Characteristics:

- Creates copy
- Original data unchanged
- Good for read-only operations

Example:

```go
func (u User) Display()
```

---

# Pointer Receiver

Allows modifying original struct.

---

## Example

```go
func (u *User) ChangeName() {
    u.Name = "Alex"
}
```

Usage:

```go
user := User{Name: "Vinay"}

user.ChangeName()

fmt.Println(user.Name)
```

Output:

```
Alex
```

---

# Why Did It Change?

Pointer receiver gets memory address.

Visual:

```text
Method
   ↓
Original Struct Memory
```

Changes affect original object.

---

# Pointer Receiver Syntax

```go
func (u *User) MethodName() {
}
```

Example:

```go
func (u *User) UpdateAge() {
    u.Age++
}
```

---

# Value vs Pointer Receiver

| Feature | Value Receiver | Pointer Receiver |
|----------|---------------|------------------|
| Creates Copy | Yes | No |
| Modifies Original | No | Yes |
| Memory Efficient | Less | More |
| Large Structs | Not Recommended | Recommended |

---

# Example Comparison

### Value Receiver

```go
func (u User) UpdateAge() {
    u.Age = 50
}
```

Original stays same.

---

### Pointer Receiver

```go
func (u *User) UpdateAge() {
    u.Age = 50
}
```

Original changes.

---

# Interview Question

### When Should You Use Pointer Receivers?

Use pointer receivers when:

- Modifying struct fields
- Struct is large
- Avoid copying data

---

# Composition

Composition means building larger structs using smaller structs.

Instead of inheritance, Go prefers composition.

---

## Example

```go
type Address struct {
    City string
    State string
}

type User struct {
    Name string
    Address Address
}
```

Create:

```go
user := User{
    Name: "Vinay",
    Address: Address{
        City: "Hyderabad",
        State: "Telangana",
    },
}
```

Access:

```go
fmt.Println(user.Address.City)
```

Output:

```
Hyderabad
```

---

# Why Composition?

Composition helps:

- Reuse code
- Keep structs modular
- Build complex objects

Think:

```text
User HAS-A Address
```

Not:

```text
User IS-A Address
```

---

# Embedding

Most important struct topic after pointer receivers.

---

# What is Embedding?

Embedding allows one struct to be included inside another struct directly.

---

## Example

```go
type Address struct {
    City string
}

type User struct {
    Name string
    Address
}
```

Notice:

```go
Address
```

No field name.

This is embedding.

---

# Create Object

```go
user := User{
    Name: "Vinay",
    Address: Address{
        City: "Hyderabad",
    },
}
```

---

# Access Embedded Fields

Without embedding:

```go
user.Address.City
```

With embedding:

```go
user.City
```

Output:

```
Hyderabad
```

---

# Why Does This Work?

Go automatically promotes embedded fields.

Visual:

```text
User
 ├── Name
 └── Address
       └── City

Promoted To:

User.City
```

---

# Embedding Methods

Methods are promoted too.

---

## Example

```go
type Address struct{}

func (a Address) ShowCity() {
    fmt.Println("Hyderabad")
}

type User struct {
    Name string
    Address
}
```

Usage:

```go
user := User{}

user.ShowCity()
```

Output:

```
Hyderabad
```

Even though User doesn't have ShowCity(), it gets it from Address.

---

# Composition vs Embedding

## Composition

```go
type User struct {
    Address Address
}
```

Access:

```go
user.Address.City
```

---

## Embedding

```go
type User struct {
    Address
}
```

Access:

```go
user.City
```

---

# Real-World Example

```go
type Logger struct{}

func (l Logger) Log(msg string) {
    fmt.Println(msg)
}

type Service struct {
    Logger
}
```

Usage:

```go
s := Service{}

s.Log("Application Started")
```

Output:

```
Application Started
```

This is heavily used in Go projects.

---

# Struct & Methods Interview Questions

---

## What is a Struct?

A custom data type that groups related fields together.

---

## What is a Method?

A function attached to a struct.

---

## What is a Receiver?

The object that owns the method.

Example:

```go
func (u User) SayHello()
```

`u User` is the receiver.

---

## Difference Between Value Receiver and Pointer Receiver?

### Value Receiver

```go
func (u User)
```

- Works on copy
- Cannot modify original

### Pointer Receiver

```go
func (u *User)
```

- Works on original object
- Can modify original

---

## When Should Pointer Receivers Be Used?

- Modify data
- Large structs
- Better performance

---

## What is Composition?

Creating bigger structs using smaller structs.

```go
User HAS-A Address
```

---

## What is Embedding?

Including a struct directly inside another struct.

```go
type User struct {
    Address
}
```

Promotes fields and methods automatically.

---

# Quick Revision Cheat Sheet

## Struct

```go
type User struct {
    Name string
    Age  int
}
```

---

## Create Struct

```go
user := User{
    Name: "Vinay",
    Age: 25,
}
```

---

## Method

```go
func (u User) SayHello() {}
```

---

## Value Receiver

```go
func (u User)
```

- Copy created
- Cannot modify original

---

## Pointer Receiver

```go
func (u *User)
```

- Uses original memory
- Can modify original

---

## Composition

```go
type User struct {
    Address Address
}
```

Access:

```go
user.Address.City
```

---

## Embedding

```go
type User struct {
    Address
}
```

Access:

```go
user.City
```

---

## Promoted Methods

```go
user.ShowCity()
```

Works because embedded methods are promoted.

---

# End of Structs & Methods Revision