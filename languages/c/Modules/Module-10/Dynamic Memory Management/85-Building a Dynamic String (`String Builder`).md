# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 85 — Building a Dynamic String (String Builder)

---

# Most beginners think:

> "Strings in C automatically grow when I append text."

They don't.

Consider:

```c
char name[10] = "Vinay";
```

Memory:

```
V i n a y \0
```

Now try:

```
Append:

Kumar
```

Result?

Impossible.

The array has a fixed size.

Modern applications rarely know the final length of a string.

Examples:

- Chat Messages
- HTTP Responses
- JSON Documents
- HTML Pages
- SQL Queries
- Logs

All of these grow dynamically.

This is why we build **dynamic strings**.

---

# Learning Objectives

After this chapter you will understand:

- Why fixed strings are limiting
- Dynamic strings
- String builders
- Capacity vs Length
- Automatic growth
- Efficient concatenation
- Real-world implementations

---

# The Problem

Suppose:

```c
char text[10];
```

Store:

```
Hello
```

Memory:

```
H

e

l

l

o

\0
```

Need to append:

```
World
```

No space remains.

Overflow becomes possible.

---

# Dynamic String Idea

Instead of:

```
Fixed Array
```

Store:

```
Pointer

↓

Heap
```

Now the string can grow whenever necessary.

---

# Length vs Capacity

Exactly like dynamic arrays.

Suppose:

```
Length

↓

5
```

```
Capacity

↓

20
```

Visualization:

```
H

e

l

l

o

\0

_

_

_

...
```

Five characters are used.

Twenty bytes are available.

---

# Dynamic String Structure

```c
typedef struct
{
    char *data;

    int length;

    int capacity;

} StringBuilder;
```

Contains:

```
Pointer

↓

Heap

----------------

Length

↓

Characters Used

----------------

Capacity

↓

Allocated Bytes
```

---

# Initialization

Suppose:

```
Capacity

↓

16
```

Allocate:

```c
builder.data =
malloc(16);
```

State:

```
Length

↓

0

Capacity

↓

16
```

Store:

```
'\0'
```

at index:

```
0
```

Now it represents an empty string.

---

# Appending Characters

Append:

```
H
```

Memory:

```
H

\0
```

Length:

```
1
```

Append:

```
e
```

Memory:

```
H

e

\0
```

Length:

```
2
```

Continue until full.

---

# Capacity Full

Suppose:

```
Capacity

↓

8
```

Length:

```
8
```

Need:

```
9th Character
```

Grow:

```
8

↓

16
```

using:

```c
realloc()
```

---

# Appending a Character

Pseudo Logic:

```
Enough Space?

↓

Yes

↓

Append

↓

Update Length

↓

Write '\0'

----------------

No

↓

Grow

↓

Append
```

---

# Why Always Keep `'\0'`?

C strings end with:

```
Null Terminator
```

Example:

```
H

e

l

l

o

\0
```

Without:

```
'\0'
```

functions like:

```c
printf("%s")
```

cannot determine where the string ends.

---

# Appending Another String

Suppose:

```
Hello
```

Append:

```
 World
```

Result:

```
Hello World
```

Internally:

```
Check Capacity

↓

Grow If Needed

↓

Copy Characters

↓

Update Length

↓

Write '\0'
```

---

# Why Not Use `strcat()`?

Suppose:

```c
strcat(dest,
       src);
```

Problem:

It assumes:

```
Enough Memory Exists
```

If not,

buffer overflow occurs.

A dynamic string grows automatically before copying.

---

# Memory Visualization

Before:

```
Heap

↓

Hello

\0

_

_

_
```

Append:

```
World
```

After:

```
Hello World

\0
```

If necessary,

the heap allocation is resized first.

---

# Simple Append Function

```c
void appendChar(
    StringBuilder *s,
    char ch)
{
    if(s->length + 1 >= s->capacity)
    {
        s->capacity *= 2;

        char *temp =
            realloc(
                s->data,
                s->capacity);

        if(temp == NULL)
            return;

        s->data = temp;
    }

    s->data[s->length++] = ch;

    s->data[s->length] = '\0';
}
```

Notice:

```
Length

↓

Updated

↓

Null Terminator

↓

Maintained
```

---

# Real-World Example — JSON Builder

Suppose:

```
{
```

Append:

```
"name":
```

Append:

```
"Vinay"
```

Append:

```
}
```

The JSON grows piece by piece.

No fixed size required.

---

# Real-World Example — HTML Generator

Server creates:

```
<html>
```

Append:

```
<body>
```

Append:

```
Content
```

Append:

```
</body>
```

The response grows dynamically.

---

# Real-World Example — SQL Query Builder

Start:

```
SELECT *
```

Append:

```
FROM Users
```

Append:

```
WHERE age > 18
```

Query is built gradually.

---

# Real-World Example — Java

Java:

```java
StringBuilder
```

stores:

```
Character Array

↓

Length

↓

Capacity
```

When full:

```
Allocate Larger Array

↓

Copy

↓

Continue
```

Exactly the same concept.

---

# Real-World Example — C#

```
StringBuilder
```

uses a similar strategy to avoid repeatedly allocating new immutable strings.

---

# Real-World Example — Go

Go strings are immutable,

but packages like:

```
strings.Builder
```

internally grow a byte buffer dynamically.

---

# Common Mistakes

---

## Forgetting the Null Terminator

Wrong:

```
Hello
```

without:

```
'\0'
```

The result is not a valid C string.

---

## Confusing Length and Capacity

Length:

```
Characters Used
```

Capacity:

```
Allocated Bytes
```

They are not the same.

---

## Forgetting `realloc()` Failure

Always use:

```
Temporary Pointer
```

before replacing the original pointer.

---

## Forgetting `free()`

Heap memory must eventually be released.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
    char *data;
    int length;
    int capacity;
} StringBuilder;

int main()
{
    StringBuilder s;

    s.capacity = 16;
    s.length = 0;

    s.data = malloc(s.capacity);

    if(s.data == NULL)
        return 1;

    s.data[0] = '\0';

    char word[] = "Hello";

    for(int i = 0; word[i] != '\0'; i++)
    {
        s.data[s.length++] = word[i];
    }

    s.data[s.length] = '\0';

    printf("%s\n", s.data);

    free(s.data);

    return 0;
}
```

Output:

```
Hello
```

---

# Hands-on Labs

## Lab 1

Implement:

```
appendChar()
```

---

## Lab 2

Implement:

```
appendString()
```

---

## Lab 3

Automatically double capacity when needed.

---

## Lab 4

Append:

```
1000 Characters
```

Verify the string grows correctly.

---

## Lab 5

Build a simple JSON generator using your dynamic string.

---

# Interview Questions

### Why are fixed-size strings limiting?

Because they cannot grow after allocation.

---

### What is the difference between length and capacity?

- Length = number of characters currently stored.
- Capacity = allocated bytes available.

---

### Why is the null terminator important?

It marks the end of a C string so string functions know where to stop.

---

### Why use `realloc()` in a string builder?

To increase capacity while preserving existing characters.

---

### Which languages use similar ideas?

- Java (`StringBuilder`)
- C# (`StringBuilder`)
- Go (`strings.Builder`)
- Many template engines and serialization libraries

---

# Summary

```
StringBuilder

↓

Pointer

↓

Heap

----------------

Length

↓

Characters Used

----------------

Capacity

↓

Allocated Bytes
```

When Full:

```
realloc()

↓

Double Capacity

↓

Continue Appending
```

Always:

```
Last Character

↓

'\0'
```

## Key Takeaways

- Fixed-size character arrays cannot grow.
- Dynamic strings store characters on the heap.
- A string builder tracks **length** and **capacity**.
- `realloc()` allows the buffer to expand while preserving existing text.
- Always maintain the null terminator.
- Dynamic string builders are widely used in web servers, JSON generators, HTML renderers, and many modern programming languages.

---

# Next Chapter

## Chapter 86 — Stack vs Heap (Deep Dive)

You'll learn:

- Why stack allocation is extremely fast
- Why heap allocation is slower
- Stack frames
- Function call internals
- Heap allocator overhead
- Cache locality
- Performance comparisons
- When to choose the stack vs the heap in real-world systems