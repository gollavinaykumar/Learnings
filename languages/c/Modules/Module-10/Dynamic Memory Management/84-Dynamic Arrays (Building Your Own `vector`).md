# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 84 — Dynamic Arrays (Building Your Own `vector`)

---

# Most beginners think:

> "Arrays in C can never change their size."

That's true for **normal arrays**.

Example:

```c
int arr[5];
```

Once created:

```
Size

↓

5
```

Forever.

You cannot make it:

```
10

↓

20

↓

100
```

But modern software rarely knows in advance how much data it will receive.

Examples:

- Browser tabs
- Database rows
- Chat messages
- HTTP requests
- File contents

All of these grow dynamically.

This is why dynamic arrays exist.

---

# Learning Objectives

After this chapter you will understand:

- Why dynamic arrays exist
- Capacity vs Size
- Automatic resizing
- Using `realloc()`
- Growth strategies
- Amortized complexity
- Real-world implementations

---

# Fixed Array Problem

Suppose:

```c
int arr[5];
```

Memory:

```
10

20

30

40

50
```

Need:

```
6th Element
```

Impossible.

You must create another array.

---

# Dynamic Array Idea

Instead of:

```
Fixed

↓

5
```

We store:

```
Pointer

↓

Heap
```

Now the heap allocation can grow.

---

# Size vs Capacity

This is the most important concept.

Many beginners think these are the same.

They are not.

Suppose:

```
Capacity

↓

10
```

Current elements:

```
7
```

Visualization:

```
Capacity

↓

10 Slots

----------------

Size

↓

7 Used
```

Three slots remain free.

---

# Example

```
Index

0

1

2

3

4

5

6

7

8

9
```

Contents:

```
10

20

30

40

50

60

70

_

_

_
```

Size:

```
7
```

Capacity:

```
10
```

---

# Dynamic Array Structure

A simple implementation:

```c
typedef struct
{
    int *data;

    int size;

    int capacity;

} Vector;
```

Three things are stored:

```
Pointer

↓

Heap Memory

----------------

Current Size

----------------

Maximum Capacity
```

---

# Initialization

Suppose:

```c
capacity = 4;
```

Allocate:

```c
data =
malloc(4 *
sizeof(int));
```

State:

```
Size

↓

0

----------------

Capacity

↓

4
```

---

# Inserting Elements

Insert:

```
10
```

State:

```
Size

↓

1

Capacity

↓

4
```

Insert:

```
20
```

State:

```
Size

↓

2

Capacity

↓

4
```

Continue until:

```
Size

↓

4

Capacity

↓

4
```

Now the array is full.

---

# What Happens Next?

Need:

```
5th Element
```

No free space.

Time to grow.

---

# Growing the Array

Most implementations:

```
Capacity

↓

4

↓

8

↓

16

↓

32

↓

64
```

Double the capacity.

---

# Using `realloc()`

Example:

```c
capacity *= 2;

int *temp =
realloc(data,
capacity *
sizeof(int));

if(temp != NULL)
{
    data = temp;
}
```

Now:

```
Old Capacity

↓

4

----------------

New Capacity

↓

8
```

---

# Visualization

Before:

```
10

20

30

40
```

After growing:

```
10

20

30

40

_

_

_

_
```

Existing data is preserved.

New slots are available.

---

# Why Double Capacity?

Imagine increasing by:

```
1 Element
```

every time.

```
5

↓

6

↓

7

↓

8
```

Every insertion after reaching capacity would require:

- New allocation
- Copying all existing elements

Very slow.

Instead:

```
4

↓

8

↓

16

↓

32
```

Far fewer reallocations.

---

# Amortized Complexity

Growing an array sometimes requires copying many elements.

But it doesn't happen on every insertion.

Result:

```
Average Insert

↓

O(1)
```

This is called:

```
Amortized O(1)
```

Even though an occasional resize takes longer.

---

# Removing Elements

Suppose:

```
10

20

30

40

50
```

Remove:

```
30
```

Shift remaining elements:

```
10

20

40

50
```

Size:

```
4
```

Capacity remains unchanged.

---

# Should We Shrink?

Suppose:

```
Capacity

↓

1024
```

Current size:

```
2
```

Should we reduce memory?

Sometimes.

Many implementations shrink when:

```
Size

<

Capacity / 4
```

This avoids wasting large amounts of memory.

---

# Complete Vector Example

```c
typedef struct
{
    int *data;

    int size;

    int capacity;

} Vector;
```

Initialize:

```
size

↓

0

capacity

↓

4
```

Insert until full.

Then:

```
realloc()

↓

capacity *= 2
```

---

# Memory Visualization

```
Vector

↓

Pointer

↓

Heap

↓

10

20

30

40

----------------

Size

↓

4

----------------

Capacity

↓

4
```

Grow:

```
Capacity

↓

8
```

---

# Real-World Example — C++ `std::vector`

When you write:

```cpp
vector.push_back(x);
```

Internally:

```
Enough Space?

↓

Yes

↓

Insert

----------------

No

↓

Grow

↓

Copy

↓

Insert
```

Exactly the same concept.

---

# Real-World Example — Java `ArrayList`

`ArrayList` stores:

```
Object Array
```

When full:

```
Allocate Larger Array

↓

Copy Elements

↓

Continue
```

Very similar to our dynamic array.

---

# Real-World Example — Go Slices

Go slices contain:

- Pointer
- Length
- Capacity

When capacity is exceeded,

Go allocates a larger backing array and copies the existing elements.

---

# Real-World Example — Python Lists

Python lists are also dynamic arrays.

Appending usually runs in amortized constant time because the runtime occasionally grows the underlying array.

---

# Common Mistakes

---

## Confusing Size and Capacity

Wrong:

```
Size == Capacity
```

Always.

No.

Size changes frequently.

Capacity changes only when the array grows or shrinks.

---

## Forgetting `realloc()` Failure

Wrong:

```c
data =
realloc(...);
```

Use a temporary pointer.

---

## Accessing Beyond Size

Suppose:

```
Capacity

↓

10

Size

↓

3
```

Only:

```
0

1

2
```

contain valid elements.

---

## Forgetting `free()`

Dynamic arrays allocate heap memory.

Eventually:

```c
free(data);
```

must be called.

---

# Simplified Implementation

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
    int *data;
    int size;
    int capacity;
} Vector;

int main()
{
    Vector v;

    v.size = 0;
    v.capacity = 4;

    v.data =
        malloc(v.capacity *
               sizeof(int));

    if(v.data == NULL)
        return 1;

    for(int i = 0; i < 10; i++)
    {
        if(v.size == v.capacity)
        {
            v.capacity *= 2;

            int *temp =
                realloc(v.data,
                        v.capacity *
                        sizeof(int));

            if(temp == NULL)
            {
                free(v.data);
                return 1;
            }

            v.data = temp;
        }

        v.data[v.size++] = i;
    }

    for(int i = 0; i < v.size; i++)
    {
        printf("%d ", v.data[i]);
    }

    printf("\n");

    free(v.data);

    return 0;
}
```

Output:

```
0 1 2 3 4 5 6 7 8 9
```

---

# Hands-on Labs

## Lab 1

Implement a dynamic array for integers.

---

## Lab 2

Automatically double the capacity when full.

---

## Lab 3

Print the capacity after every resize.

Observe:

```
4

↓

8

↓

16

↓

32
```

---

## Lab 4

Implement element deletion.

Update the size correctly.

---

## Lab 5

Implement automatic shrinking when the array becomes mostly empty.

---

# Interview Questions

### What is the difference between size and capacity?

- Size = number of elements currently stored.
- Capacity = total number of elements that can be stored before resizing.

---

### Why use `realloc()` in dynamic arrays?

To grow or shrink the underlying heap allocation while preserving existing elements.

---

### Why double the capacity?

To reduce the number of expensive reallocations and achieve amortized constant-time insertion.

---

### What is the average complexity of appending?

Amortized **O(1)**.

---

### Which languages use dynamic arrays internally?

- C++ (`std::vector`)
- Java (`ArrayList`)
- Go (Slices)
- Python (Lists)
- JavaScript (Arrays)

---

# Summary

```
Vector

↓

Pointer

↓

Heap Memory

----------------

Size

↓

Used Elements

----------------

Capacity

↓

Allocated Space
```

When Full:

```
realloc()

↓

Double Capacity

↓

Continue
```

## Key Takeaways

- Dynamic arrays overcome the fixed-size limitation of normal C arrays.
- They maintain both **size** and **capacity**.
- `realloc()` is used to resize the underlying storage.
- Doubling capacity minimizes reallocations and provides amortized **O(1)** insertion.
- This design is used by many modern programming languages and libraries.
- Understanding dynamic arrays is the foundation for implementing higher-level data structures efficiently.

---

# Next Chapter

## Chapter 85 — Building a Dynamic String (`String Builder`)

You'll learn:

- Why fixed-size character arrays are limiting
- Growing strings dynamically
- Automatic resizing with `realloc()`
- Efficient string concatenation
- Null terminators in dynamic strings
- How languages like Java, C#, JavaScript, and Go build strings internally