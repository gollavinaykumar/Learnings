# Go Fundamentals – Complete Interview Revision Guide

> Simple explanations + examples for quick interview preparation.

---

# 1. Variables

Variables are used to store data in memory.

Think of a variable as a box that holds a value.

## Syntax

```go
var name string = "John"
```

### Example

```go
package main

import "fmt"

func main() {
    var age int = 25
    fmt.Println(age)
}
```

Output:

```
25
```

## Short Declaration

Most commonly used inside functions.

```go
name := "John"
age := 25
```

Go automatically determines the type.

### Example

```go
language := "Go"
fmt.Println(language)
```

---

# 2. Constants

Constants are values that cannot be changed after they are declared.

## Syntax

```go
const PI = 3.14
```

### Example

```go
const country = "India"

fmt.Println(country)
```

Trying to change it causes an error:

```go
country = "USA" // Error
```

## Why Use Constants?

- Fixed values
- Prevent accidental changes
- Better readability

### Interview Point

**Variable → Can change**

**Constant → Cannot change**

---

# 3. Data Types

Data type tells Go what kind of value is stored.

## int

Stores whole numbers.

```go
var age int = 25
```

## float64

Stores decimal numbers.

```go
var price float64 = 99.99
```

## string

Stores text.

```go
var name string = "Alex"
```

## bool

Stores true or false.

```go
var isAdmin bool = true
```

### Example

```go
var age int = 25
var salary float64 = 50000.50
var name string = "John"
var active bool = true

fmt.Println(age, salary, name, active)
```

---

# 4. Type Inference

Type inference means Go automatically figures out the type from the assigned value.

### Example

```go
name := "John"
age := 25
salary := 5000.50
```

Go understands:

```go
name   -> string
age    -> int
salary -> float64
```

### Check Type

```go
fmt.Printf("%T\n", age)
```

Output:

```
int
```

### Interview Question

**What is Type Inference?**

Type inference is Go's ability to automatically determine a variable's type from its assigned value.

---

# 5. Zero Values

When a variable is declared but not initialized, Go gives it a default value.

These default values are called Zero Values.

| Type | Zero Value |
|--------|------------|
| int | 0 |
| float64 | 0.0 |
| string | "" |
| bool | false |
| pointer | nil |

### Example

```go
var age int
var name string
var active bool

fmt.Println(age)
fmt.Println(name)
fmt.Println(active)
```

Output:

```
0

false
```

### Interview Question

**What is a Zero Value?**

The default value automatically assigned by Go when a variable is declared but not initialized.

---

# 6. Operators

Operators perform operations on values.

---

## Arithmetic Operators

| Operator | Meaning |
|-----------|----------|
| + | Addition |
| - | Subtraction |
| * | Multiplication |
| / | Division |
| % | Modulus |

### Example

```go
a := 10
b := 3

fmt.Println(a + b)
fmt.Println(a - b)
fmt.Println(a * b)
fmt.Println(a / b)
fmt.Println(a % b)
```

Output:

```
13
7
30
3
1
```

---

## Comparison Operators

Used to compare values.

| Operator | Meaning |
|-----------|----------|
| == | Equal |
| != | Not Equal |
| > | Greater Than |
| < | Less Than |
| >= | Greater Than or Equal |
| <= | Less Than or Equal |

### Example

```go
fmt.Println(10 > 5)
fmt.Println(10 == 5)
```

Output:

```
true
false
```

---

## Logical Operators

| Operator | Meaning |
|-----------|----------|
| && | AND |
| \|\| | OR |
| ! | NOT |

### Example

```go
age := 20

fmt.Println(age > 18 && age < 30)
```

Output:

```
true
```

---

# 7. Type Conversion

Go does not automatically convert types.

You must explicitly convert them.

### Example

```go
var a int = 10
var b float64 = 5.5

result := float64(a) + b

fmt.Println(result)
```

Output:

```
15.5
```

### Example

```go
num := 100

f := float64(num)

fmt.Println(f)
```

### Interview Question

**Does Go support implicit type conversion?**

No.

Go only supports explicit type conversion.

---

# 8. if / else

Used to make decisions.

### Syntax

```go
if condition {
    // code
}
```

### Example

```go
age := 20

if age >= 18 {
    fmt.Println("Adult")
}
```

Output:

```
Adult
```

---

## if else

```go
age := 15

if age >= 18 {
    fmt.Println("Adult")
} else {
    fmt.Println("Minor")
}
```

Output:

```
Minor
```

---

## if else if

```go
marks := 75

if marks >= 90 {
    fmt.Println("A")
} else if marks >= 70 {
    fmt.Println("B")
} else {
    fmt.Println("C")
}
```

Output:

```
B
```

---

## if with Initialization

```go
if age := 20; age >= 18 {
    fmt.Println("Adult")
}
```

The variable exists only inside the if block.

---

# 9. switch

Switch is used when multiple conditions need checking.

Cleaner than many if-else blocks.

### Example

```go
day := 2

switch day {
case 1:
    fmt.Println("Monday")

case 2:
    fmt.Println("Tuesday")

default:
    fmt.Println("Unknown")
}
```

Output:

```
Tuesday
```

---

## Multiple Values

```go
day := 1

switch day {
case 1, 7:
    fmt.Println("Weekend")
default:
    fmt.Println("Weekday")
}
```

Output:

```
Weekend
```

---

## Switch Without Expression

```go
age := 25

switch {
case age < 18:
    fmt.Println("Minor")

case age >= 18:
    fmt.Println("Adult")
}
```

---

# 10. for Loop

Go has only one looping keyword:

```go
for
```

---

## Traditional Loop

```go
for i := 1; i <= 5; i++ {
    fmt.Println(i)
}
```

Output:

```
1
2
3
4
5
```

---

## While Style Loop

```go
count := 1

for count <= 5 {
    fmt.Println(count)
    count++
}
```

---

## Infinite Loop

```go
for {
    fmt.Println("Running")
}
```

Stop using:

```go
break
```

---

## Continue

Skips current iteration.

```go
for i := 1; i <= 5; i++ {

    if i == 3 {
        continue
    }

    fmt.Println(i)
}
```

Output:

```
1
2
4
5
```

---

# 11. range

Used to iterate over collections.

- Arrays
- Slices
- Maps
- Strings
- Channels

---

## range with Slice

```go
nums := []int{10, 20, 30}

for index, value := range nums {
    fmt.Println(index, value)
}
```

Output:

```
0 10
1 20
2 30
```

---

## Ignore Index

```go
for _, value := range nums {
    fmt.Println(value)
}
```

Output:

```
10
20
30
```

---

## range with String

```go
name := "Go"

for index, char := range name {
    fmt.Println(index, string(char))
}
```

Output:

```
0 G
1 o
```

---

## range with Map

```go
student := map[string]int{
    "John": 90,
    "Alex": 80,
}

for key, value := range student {
    fmt.Println(key, value)
}
```

---

# 12. Functions

Functions are reusable blocks of code.

Instead of writing the same code again and again, write it once inside a function.

---

## Basic Function

```go
func greet() {
    fmt.Println("Hello")
}
```

Usage:

```go
greet()
```

Output:

```
Hello
```

---

## Function with Parameters

```go
func greet(name string) {
    fmt.Println("Hello", name)
}
```

Usage:

```go
greet("John")
```

Output:

```
Hello John
```

---

## Function Returning Value

```go
func add(a int, b int) int {
    return a + b
}
```

Usage:

```go
result := add(10, 20)

fmt.Println(result)
```

Output:

```
30
```

---

# 13. Multiple Return Values

Go functions can return more than one value.

### Example

```go
func calculate(a, b int) (int, int) {
    return a+b, a-b
}
```

Usage:

```go
sum, diff := calculate(10, 5)

fmt.Println(sum)
fmt.Println(diff)
```

Output:

```
15
5
```

---

## Ignoring Values

```go
sum, _ := calculate(10, 5)
```

The underscore (`_`) ignores unwanted values.

---

### Real World Example

```go
value, err := strconv.Atoi("123")
```

Most Go functions return:

```go
result, error
```

---

# 14. Named Return Values

You can name return variables.

### Example

```go
func add(a int, b int) (result int) {
    result = a + b
    return
}
```

Usage:

```go
fmt.Println(add(10, 20))
```

Output:

```
30
```

### What Happens?

Go creates:

```go
result int
```

automatically.

Then:

```go
return
```

returns that variable.

### Interview Note

Named returns are rarely used in production code because explicit returns are easier to read.

---

# 15. Variadic Functions

Variadic functions accept any number of arguments.

### Syntax

```go
func functionName(values ...int)
```

---

### Example

```go
func sum(nums ...int) int {

    total := 0

    for _, num := range nums {
        total += num
    }

    return total
}
```

Usage:

```go
fmt.Println(sum(1, 2))
fmt.Println(sum(1, 2, 3))
fmt.Println(sum(1, 2, 3, 4))
```

Output:

```
3
6
10
```

### Interview Question

**Why use variadic functions?**

When the number of arguments is not fixed.

---

# 16. Anonymous Functions

Functions without a name.

### Example

```go
func() {
    fmt.Println("Hello")
}()
```

Output:

```
Hello
```

The final `()` immediately executes the function.

---

## Store in Variable

```go
greet := func() {
    fmt.Println("Hello")
}

greet()
```

Output:

```
Hello
```

### Use Cases

- Goroutines
- Callbacks
- Closures

---

# 17. Closures

A closure is an anonymous function that remembers variables from its outer scope.

### Example

```go
func counter() func() int {

    count := 0

    return func() int {
        count++
        return count
    }
}
```

Usage:

```go
c := counter()

fmt.Println(c())
fmt.Println(c())
fmt.Println(c())
```

Output:

```
1
2
3
```

### How It Works

Even after `counter()` finishes, the anonymous function remembers:

```go
count
```

This is called a Closure.

### Real Life Analogy

Think of it as a backpack.

The function carries its variables with it wherever it goes.

---

# 18. Recursion

Recursion means a function calls itself.

### Example

```go
func countdown(n int) {

    if n == 0 {
        return
    }

    fmt.Println(n)

    countdown(n - 1)
}
```

Usage:

```go
countdown(5)
```

Output:

```
5
4
3
2
1
```

---

## Factorial Example

Factorial:

```
5! = 5 × 4 × 3 × 2 × 1
```

### Recursive Solution

```go
func factorial(n int) int {

    if n == 1 {
        return 1
    }

    return n * factorial(n-1)
}
```

Usage:

```go
fmt.Println(factorial(5))
```

Output:

```
120
```

---

# Interview Quick Revision

### Variables

```go
var age int = 25
name := "John"
```

### Constants

```go
const PI = 3.14
```

### Zero Values

```go
int -> 0
string -> ""
bool -> false
pointer -> nil
```

### Type Conversion

```go
float64(age)
int(price)
```

### Loop

```go
for i := 0; i < 5; i++ {}
```

### Range

```go
for i, v := range nums {}
```

### Function

```go
func add(a, b int) int
```

### Multiple Returns

```go
return value, err
```

### Named Return

```go
func add() (result int)
```

### Variadic

```go
func sum(nums ...int)
```

### Anonymous Function

```go
func() {}
```

### Closure

Function remembers outer variables.

### Recursion

Function calls itself.

---

# Most Asked Interview Questions

### Q1. Difference between var and := ?

`var`

```go
var age int = 25
```

`:=`

```go
age := 25
```

`:=` can only be used inside functions.

---

### Q2. What is a Zero Value?

Default value assigned when a variable is declared but not initialized.

---

### Q3. Does Go support implicit type conversion?

No.

Only explicit conversion.

---

### Q4. Why multiple return values?

To return data and error together.

Example:

```go
value, err := getUser()
```

---

### Q5. What is a Closure?

A function that remembers variables from its outer scope.

---

### Q6. What is Recursion?

A function calling itself until a base condition is reached.

---

### Q7. Why use Variadic Functions?

To accept a variable number of arguments.

Example:

```go
sum(1,2)
sum(1,2,3,4)
```

---

# End of Go Fundamentals Revision