# Go Pass By Value vs Pass By Reference – Complete Interview Revision Guide

> This is one of the most confusing topics for Go beginners.
>
> Many developers coming from JavaScript, Java, Python, or C++ ask:
>
> - Is Go pass-by-value or pass-by-reference?
> - Do slices copy data?
> - Do maps copy data?
> - Are structs deep copied?
> - What is similar to JS shallow copy and deep copy?
>
> Understanding this topic will help you avoid many bugs.

---

# Golden Rule

## Go is ALWAYS Pass By Value

This is the most important thing to remember.

```text
Go does NOT have pass-by-reference.
```

Every function argument is copied.

---

Example:

```go
func update(x int) {
    x = 100
}

func main() {
    a := 10

    update(a)

    fmt.Println(a)
}
```

Output:

```text
10
```

---

Why?

Because:

```go
a
```

was copied into:

```go
x
```

---

Visual:

```text
a = 10

update(a)

copy

x = 10
```

Changing:

```go
x
```

does not affect:

```go
a
```

---

# Value Types vs Reference-Like Types

Although Go is always pass-by-value,

some types contain internal pointers.

This creates "reference-like" behavior.

---

# Easy Classification

## Pure Value Types

```go
int
float64
bool
string
array
struct
```

---

## Reference-Like Types

```go
slice
map
channel
function
pointer
interface
```

---

These are still copied,

but the copy points to the same underlying data.

---

# 1. Integers

## Pass By Value

```go
func update(x int) {
    x = 100
}
```

---

```go
a := 10

update(a)

fmt.Println(a)
```

Output:

```text
10
```

---

Memory:

```text
a ---> 10

copy

x ---> 10
```

---

# 2. Strings

Strings are immutable.

---

Example:

```go
func update(s string) {
    s = "Go"
}
```

---

```go
name := "Vinay"

update(name)

fmt.Println(name)
```

Output:

```text
Vinay
```

---

Strings behave like value types.

---

# 3. Arrays

Arrays are true value types.

---

Example

```go
func update(arr [3]int) {

    arr[0] = 100
}
```

---

```go
nums := [3]int{1, 2, 3}

update(nums)

fmt.Println(nums)
```

Output:

```text
[1 2 3]
```

---

Entire array is copied.

---

Visual

```text
nums

[1 2 3]

copy

arr

[1 2 3]
```

---

# Array Copy Example

```go
a := [3]int{1,2,3}

b := a

b[0] = 100
```

Output:

```go
a = [1 2 3]

b = [100 2 3]
```

---

Arrays behave like:

```text
Deep Copy
```

in JavaScript.

---

# 4. Structs

Structs are value types.

---

Example

```go
type User struct {
    Name string
}
```

---

```go
func update(u User) {

    u.Name = "Alex"
}
```

---

```go
user := User{
    Name: "Vinay",
}

update(user)

fmt.Println(user.Name)
```

Output:

```text
Vinay
```

---

Struct copied.

---

Visual

```text
user

{Name: Vinay}

copy

u

{Name: Vinay}
```

---

# Struct Assignment

```go
u1 := User{
    Name: "Vinay",
}

u2 := u1
```

Entire struct copied.

---

```go
u2.Name = "Alex"
```

Result:

```go
u1.Name = Vinay

u2.Name = Alex
```

---

Structs are similar to:

```js
structuredClone()
```

for primitive fields.

---

# 5. Pointers

Pointers are value types too.

This surprises people.

---

Example

```go
func update(x *int) {

    *x = 100
}
```

---

```go
a := 10

update(&a)

fmt.Println(a)
```

Output:

```text
100
```

---

What gets copied?

```go
&a
```

(pointer)

---

Visual

```text
a ---> 10

pointer copied

p ---> same address
```

---

Both pointers reference same memory.

---

# Pointer Assignment

```go
a := 10

p1 := &a

p2 := p1
```

---

Memory

```text
p1 ----+
       |
       v
      10
       ^
       |
p2 ----+
```

---

# 6. Slices

Most Important Topic.

---

# Slice Internals

A slice contains:

```text
Pointer
Length
Capacity
```

---

Visual

```text
Slice Header

Pointer ---> Array
Length
Capacity
```

---

Example

```go
nums := []int{1,2,3}
```

---

Memory

```text
nums

Pointer ---> [1 2 3]
Len = 3
Cap = 3
```

---

# Slice Assignment

```go
a := []int{1,2,3}

b := a
```

Only slice header copied.

---

Visual

```text
a ----------+
            |
            v

       [1 2 3]

            ^
            |
b ----------+
```

---

Both share same underlying array.

---

Example

```go
b[0] = 100
```

Output

```go
a = [100 2 3]

b = [100 2 3]
```

---

This is like:

```js
const b = a
```

for arrays in JavaScript.

---

# Slice Is Shallow Copy

```go
a := []int{1,2,3}

b := a
```

Result:

```text
Shallow Copy
```

---

# Deep Copy Slice

```go
a := []int{1,2,3}

b := make([]int, len(a))

copy(b, a)
```

---

Now:

```go
b[0] = 100
```

Output:

```go
a = [1 2 3]

b = [100 2 3]
```

---

Equivalent to:

```js
structuredClone()
```

or

```js
[...arr]
```

for primitives.

---

# 7. Maps

Most Asked Interview Question.

---

Map internally contains pointers.

---

Example

```go
users := map[string]int{
    "vinay": 25,
}
```

---

Assignment

```go
m1 := users

m2 := m1
```

---

Visual

```text
m1 ----------+
             |
             v

      Map Data

             ^
             |
m2 ----------+
```

---

Example

```go
m2["vinay"] = 30
```

Output

```go
m1["vinay"] = 30
```

---

Map assignment is:

```text
Shallow Copy
```

---

# Deep Copy Map

```go
newMap := make(map[string]int)

for k, v := range oldMap {

    newMap[k] = v
}
```

---

Now both maps are independent.

---

# 8. Channels

Channels are reference-like.

---

Example

```go
ch1 := make(chan int)

ch2 := ch1
```

Both point to same channel.

---

Visual

```text
ch1 ----+
        |
        v

     Channel

        ^
        |
ch2 ----+
```

---

# 9. Functions

Functions are reference-like values.

---

Example

```go
f1 := someFunc

f2 := f1
```

Both reference same function.

---

# 10. Interfaces

Depends on underlying value.

---

Example

```go
var i interface{} = []int{1,2,3}
```

The interface contains:

```text
Type Info
Value
```

---

Copying interface copies:

```text
Interface Header
```

not necessarily underlying data.

---

# JS Comparison

## JavaScript Primitive

```js
let a = 10
let b = a
```

Copy.

---

Equivalent Go:

```go
a := 10

b := a
```

---

# JS Object

```js
let a = {name:"Vinay"}

let b = a
```

Shallow.

---

Equivalent Go:

```go
m1 := map[string]string{
    "name":"Vinay",
}

m2 := m1
```

---

# JS Array

```js
const a = [1,2,3]

const b = a
```

Shallow.

---

Equivalent Go:

```go
a := []int{1,2,3}

b := a
```

---

# JS structuredClone()

```js
const b = structuredClone(a)
```

---

Equivalent Go Slice

```go
b := make([]int, len(a))

copy(b, a)
```

---

Equivalent Go Map

```go
for k,v := range a {

    b[k] = v
}
```

---

# Interview Table

| Type | Pass By Value? | Shares Underlying Data? | JS Equivalent |
|--------|------------|------------------|---------------|
| int | Yes | No | Primitive |
| bool | Yes | No | Primitive |
| string | Yes | No | Primitive |
| array | Yes | No | Deep Copy |
| struct | Yes | No | structuredClone |
| pointer | Yes | Yes | Reference |
| slice | Yes | Yes | Array/Object Reference |
| map | Yes | Yes | Object Reference |
| channel | Yes | Yes | Shared Resource |
| function | Yes | Yes | Function Reference |
| interface | Yes | Depends | Wrapper |

---

# Golden Interview Answer

### Is Go Pass By Value or Pass By Reference?

Go is **always pass-by-value**.

However, some types such as:

```go
slice
map
channel
pointer
function
interface
```

contain internal references to shared data.

When these types are copied, the reference is copied, not the underlying data.

Therefore they behave like **shallow copies**.

Types like:

```go
int
string
array
struct
```

create independent copies and behave more like **deep copies** (unless the struct contains slices, maps, or pointers inside it).

---

# Quick Memory Rule

```text
Always Value Copy

But:

int/string/array/struct
    ↓
Independent Copy

slice/map/channel/pointer
    ↓
Shared Underlying Data
```

# End