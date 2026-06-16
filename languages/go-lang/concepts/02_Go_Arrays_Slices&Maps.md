# Go Arrays, Slices & Maps – Complete Interview Revision Guide

> Simple explanations + examples + interview questions.

---

# 1. Arrays

An array is a collection of elements of the same type stored in contiguous memory locations.

### Important Points

- Fixed size
- Size cannot change after creation
- Stores same data type elements
- Size is part of the array type

---

## Array Declaration

```go
var arr [3]int
```

This creates an array of size 3.

Default values:

```go
[0 0 0]
```

---

## Array Initialization

```go
arr := [3]int{1, 2, 3}
```

Output:

```go
[1 2 3]
```

---

## Access Elements

```go
arr := [3]int{10, 20, 30}

fmt.Println(arr[0])
fmt.Println(arr[1])
```

Output:

```
10
20
```

---

## Modify Elements

```go
arr[1] = 100
```

Result:

```go
[10 100 30]
```

---

## Array Length

```go
fmt.Println(len(arr))
```

Output:

```
3
```

---

## Iterate Array

```go
arr := [3]int{10, 20, 30}

for i, v := range arr {
    fmt.Println(i, v)
}
```

Output:

```
0 10
1 20
2 30
```

---

## Array is Value Type

Arrays are copied when assigned.

### Example

```go
a := [3]int{1, 2, 3}

b := a

b[0] = 100

fmt.Println(a)
fmt.Println(b)
```

Output:

```
[1 2 3]
[100 2 3]
```

### Why?

Because array assignment creates a copy.

---

## Interview Question

### Why are arrays rarely used in Go?

Because arrays have fixed size.

Most real-world applications need dynamic size collections, so slices are used.

---

# 2. Slices

A slice is a dynamic view of an array.

Think of a slice as a flexible array.

### Most Important Topic in Go Interviews

Most Go programs use slices instead of arrays.

---

## Create Slice

```go
nums := []int{1, 2, 3}
```

Output:

```go
[1 2 3]
```

---

## Slice Internally Contains

A slice has 3 things:

```go
Pointer
Length
Capacity
```

Visual:

```text
Array:
[1][2][3][4][5]

Slice:
Pointer -> first element
Length  -> number of usable elements
Capacity -> available space
```

---

# Length vs Capacity

Very Important Interview Question

---

## Length

Number of elements currently in the slice.

```go
nums := []int{10,20,30}

fmt.Println(len(nums))
```

Output:

```
3
```

---

## Capacity

Number of elements the slice can hold before reallocation.

```go
fmt.Println(cap(nums))
```

Output:

```
3
```

---

## Example

```go
arr := [5]int{1,2,3,4,5}

slice := arr[1:4]
```

Visual:

```text
Array:
[1][2][3][4][5]

Slice:
   [2][3][4]
```

Length:

```go
3
```

Capacity:

```go
4
```

Why?

Capacity counts from slice start until array end.

```text
[2][3][4][5]
```

Capacity = 4

---

# Append

Used to add elements.

---

## Example

```go
nums := []int{1,2,3}

nums = append(nums, 4)

fmt.Println(nums)
```

Output:

```
[1 2 3 4]
```

---

## Append Multiple Elements

```go
nums = append(nums, 5, 6, 7)
```

Output:

```
[1 2 3 4 5 6 7]
```

---

# Internal Memory Allocation

Most Asked Interview Topic

---

## What Happens During Append?

```go
nums := []int{1,2,3}
```

Current:

```text
Length = 3
Capacity = 3
```

Now:

```go
nums = append(nums, 4)
```

Capacity is full.

Go:

1. Creates a larger array
2. Copies old elements
3. Adds new element
4. Slice points to new array

Visual:

Before:

```text
Old Array
[1][2][3]
```

After:

```text
New Array
[1][2][3][4][ ][ ]
```

---

## Capacity Growth

Usually grows approximately:

```go
1 -> 2
2 -> 4
4 -> 8
8 -> 16
```

(Not guaranteed exactly, implementation may vary.)

---

## Example

```go
nums := []int{}

for i := 1; i <= 10; i++ {
    nums = append(nums, i)

    fmt.Println(
        "Len:", len(nums),
        "Cap:", cap(nums),
    )
}
```

Output (similar):

```text
Len:1 Cap:1
Len:2 Cap:2
Len:3 Cap:4
Len:4 Cap:4
Len:5 Cap:8
```

---

# Slicing Behavior

A slice shares memory with the original array.

This is extremely important.

---

## Example

```go
arr := [5]int{1,2,3,4,5}

slice := arr[1:4]

fmt.Println(slice)
```

Output:

```
[2 3 4]
```

---

## Modify Slice

```go
slice[0] = 100
```

Now:

```go
fmt.Println(arr)
```

Output:

```
[1 100 3 4 5]
```

---

## Why?

Slice and array share the same memory.

Visual:

```text
Array:
[1][2][3][4][5]

Slice:
   ↑
 shares memory
```

---

# Copy Slice

If you want separate memory:

```go
src := []int{1,2,3}

dst := make([]int, len(src))

copy(dst, src)
```

Now modifications won't affect original slice.

---

# make Function

Creates slices efficiently.

```go
nums := make([]int, 5)
```

Output:

```go
[0 0 0 0 0]
```

---

## Length and Capacity

```go
nums := make([]int, 3, 10)
```

Meaning:

```go
length = 3
capacity = 10
```

---

# Slice Interview Questions

---

## What is a Slice?

A dynamic view of an array.

Internally contains:

```go
Pointer
Length
Capacity
```

---

## Difference Between Length and Capacity?

Length:

```go
Current elements
```

Capacity:

```go
Maximum elements before reallocation
```

---

## What Happens During Append?

When capacity is full:

1. New array allocated
2. Elements copied
3. New element added
4. Slice points to new array

---

## Do Slices Copy Data?

No.

Slices share memory with underlying array.

---

# 3. Maps

Maps store data as key-value pairs.

Similar to:

```text
Dictionary
HashMap
Object
```

in other languages.

---

## Create Map

```go
users := map[string]int{
    "vinay": 25,
}
```

Output:

```go
map[vinay:25]
```

---

## Empty Map

```go
users := make(map[string]int)
```

---

## Add Data

```go
users["john"] = 30
users["alex"] = 40
```

Output:

```go
map[john:30 alex:40]
```

---

## Access Value

```go
fmt.Println(users["john"])
```

Output:

```
30
```

---

# Map Lookup

Most Asked Interview Topic

---

## Normal Lookup

```go
age := users["john"]
```

---

## Safe Lookup

```go
age, exists := users["john"]
```

Example:

```go
age, exists := users["john"]

fmt.Println(age)
fmt.Println(exists)
```

Output:

```
30
true
```

---

## Missing Key

```go
age, exists := users["bob"]
```

Output:

```
0
false
```

---

## Why?

Maps return:

```go
value
found/not found
```

This avoids confusion with zero values.

---

# Delete from Map

Use delete()

### Example

```go
delete(users, "john")
```

Before:

```go
map[john:30 alex:40]
```

After:

```go
map[alex:40]
```

---

# Iterate Map

```go
for key, value := range users {
    fmt.Println(key, value)
}
```

Output:

```text
john 30
alex 40
```

Order is not guaranteed.

---

# Why Map Order Changes?

Go intentionally randomizes map iteration order.

Never depend on map order.

---

# Concurrent Map Access Issue

Extremely Important Interview Topic

---

## Problem

Maps are NOT thread-safe.

Multiple goroutines accessing a map simultaneously can crash.

---

## Example

```go
m := make(map[string]int)

go func() {
    m["a"] = 1
}()

go func() {
    m["b"] = 2
}()
```

Possible Error:

```text
fatal error:
concurrent map writes
```

---

# Why Does It Happen?

Map internally modifies memory structures.

Multiple goroutines writing simultaneously can corrupt data.

---

# Solution 1: Use Mutex

```go
var mu sync.Mutex

mu.Lock()
m["a"] = 1
mu.Unlock()
```

---

## Example

```go
var mu sync.Mutex
m := make(map[string]int)

go func() {
    mu.Lock()
    m["a"] = 1
    mu.Unlock()
}()
```

---

# Solution 2: sync.RWMutex

Allows:

```go
Multiple readers
Single writer
```

Better performance for read-heavy workloads.

---

# Solution 3: sync.Map

Built-in concurrent-safe map.

```go
var m sync.Map

m.Store("name", "vinay")

value, ok := m.Load("name")
```

---

# Map Interview Questions

---

## What is a Map?

A key-value data structure.

Example:

```go
map[string]int
```

---

## How to Check if Key Exists?

```go
value, ok := m["key"]
```

---

## How to Delete Key?

```go
delete(m, "key")
```

---

## Is Map Iteration Ordered?

No.

Order is random.

---

## Are Maps Thread Safe?

No.

Concurrent access can cause crashes.

Use:

```go
sync.Mutex
sync.RWMutex
sync.Map
```

---

# Quick Revision Cheat Sheet

## Array

```go
arr := [3]int{1,2,3}
```

- Fixed size
- Value type
- Rarely used

---

## Slice

```go
nums := []int{1,2,3}
```

- Dynamic size
- Most commonly used
- Has length and capacity

---

## Append

```go
nums = append(nums, 4)
```

---

## Length

```go
len(nums)
```

Current elements.

---

## Capacity

```go
cap(nums)
```

Available storage before reallocation.

---

## Map

```go
users := map[string]int{
    "vinay": 25,
}
```

---

## Lookup

```go
value, ok := users["vinay"]
```

---

## Delete

```go
delete(users, "vinay")
```

---

## Concurrent Access

Maps are NOT thread-safe.

Use:

```go
sync.Mutex
sync.RWMutex
sync.Map
```

---

# End of Arrays, Slices & Maps Revision