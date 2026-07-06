# 👑 C Mastery Roadmap

# Module 7 — Structures & Unions

# Chapter 55 — Union

---

# Most beginners think:

> "A union is just another kind of structure."

It isn't.

A **structure** stores **all its members at the same time**.

A **union** stores **only one member at a time**.

This makes unions extremely memory efficient.

They are widely used in:

- Linux Kernel
- Device Drivers
- Embedded Systems
- Network Protocols
- Compilers
- Database Engines

Whenever memory is limited or multiple data types share the same storage, unions become very useful.

---

# Learning Objectives

After this chapter you will understand:

- What a union is
- Why unions exist
- Union vs Structure
- Memory layout
- Shared memory
- Size of a union
- Tagged unions
- Real-world uses
- Common mistakes

---

# Why Do We Need Unions?

Imagine a payment application.

A payment can be made by:

- Credit Card
- UPI
- Bank Transfer

But **only one method is used for a transaction**.

Why allocate memory for all three?

Instead, all payment methods can **share the same memory**.

That's exactly what a union does.

---

# What is a Union?

A union is a user-defined data type where **all members share the same memory location**.

Only one member should contain meaningful data at a time.

---

# Declaring a Union

```c
union Data
{
    int i;
    float f;
    char c;
};
```

This creates a blueprint.

No memory is allocated yet.

---

# Creating a Union Variable

```c
union Data value;
```

Now memory is allocated.

Unlike a structure, **all members overlap**.

---

# Memory Layout

Suppose:

```c
union Data
{
    int i;
    float f;
    char c;
};
```

Memory:

```
+----------------------+
|                      |
| Shared Memory Block  |
|                      |
+----------------------+
      ▲
      │
 ┌────┼────┐
 │    │    │
 i    f    c
```

Every member starts at the **same address**.

---

# Writing to a Union

```c
union Data value;

value.i = 100;
```

Memory:

```
Shared Memory

↓

100
```

Now:

```c
printf("%d", value.i);
```

Output:

```
100
```

---

# Overwriting Data

```c
value.i = 100;

value.f = 3.14;
```

Memory:

```
Shared Memory

↓

3.14
```

The old integer value is overwritten.

The union now contains meaningful data only as a `float`.

---

# Why?

Because there is only **one memory block**.

```
Union

↓

Shared Memory

↓

Member A

or

Member B

or

Member C
```

Never all simultaneously.

---

# Union vs Structure

Suppose:

```c
struct Data
{
    int i;
    float f;
    char c;
};
```

Memory:

```
+---------+
| int     |
+---------+
| float   |
+---------+
| char    |
+---------+
```

Every member has its own storage.

---

Union:

```
+----------------+
| Shared Memory  |
+----------------+
```

All members use the same bytes.

---

# Size of a Union

Example:

```c
union Data
{
    int i;        // 4 bytes
    double d;     // 8 bytes
    char c;       // 1 byte
};
```

Question:

How large is the union?

Answer:

```
8 Bytes
```

The union size equals the size of its **largest member** (plus any alignment requirements).

---

# Example Program

```c
#include <stdio.h>

union Data
{
    int i;
    float f;
};

int main()
{
    union Data value;

    value.i = 100;

    printf("%d\n", value.i);

    value.f = 3.14f;

    printf("%.2f\n", value.f);

    return 0;
}
```

Output:

```
100

3.14
```

If you print `value.i` after storing `value.f`, you'll see the bit pattern of the float interpreted as an integer, not `100`.

---

# Tagged Union

A union alone doesn't tell us **which member is currently valid**.

We solve this by combining a structure with:

- A **tag** (type information)
- A **union**

Example:

```c
enum PaymentType
{
    CREDIT_CARD,
    UPI,
    BANK_TRANSFER
};

struct Payment
{
    enum PaymentType type;

    union
    {
        char card[20];
        char upi[50];
        char account[20];
    } data;
};
```

Memory:

```
Payment

↓

Type

↓

Shared Data
```

Now we always know which union member is active.

---

# Accessing Tagged Union

```c
payment.type = UPI;

strcpy(payment.data.upi,
       "vinay@upi");
```

Later:

```c
if(payment.type == UPI)
{
    printf("%s",
           payment.data.upi);
}
```

The tag tells us how to interpret the shared memory.

---

# Real-World Example — Linux Kernel

Linux uses unions in many kernel structures.

Conceptually:

```
Socket

↓

IPv4

or

IPv6
```

Only one protocol is active.

A union avoids wasting memory.

---

# Real-World Example — Network Packet

A packet may contain:

```
TCP Header

or

UDP Header
```

Not both.

A union allows them to share memory.

---

# Real-World Example — Compiler

A compiler's Abstract Syntax Tree (AST) node might represent:

- Integer literal
- String literal
- Variable
- Function call

Only one node type is active.

Compilers often implement this using tagged unions.

---

# Advantages

- Saves memory
- Efficient representation
- Useful for variant data
- Common in embedded systems
- Used in protocol implementations

---

# Disadvantages

- Only one member is valid at a time
- Easy to misuse without a tag
- Reading the wrong member produces incorrect results

---

# Common Mistakes

---

## Assuming All Members Hold Values

Wrong:

```text
i = 100

f = 3.14
```

Both cannot be valid simultaneously.

---

## Reading the Wrong Member

```c
value.i = 100;

printf("%f", value.f);
```

This prints the same bits interpreted as a float, which is usually meaningless.

---

## Forgetting the Tag

Always use a tag (like an `enum`) when multiple union members are possible.

---

# Hands-on Labs

## Lab 1

Create a union with:

- int
- float
- char

Store values one by one.

Observe how each assignment overwrites the previous one.

---

## Lab 2

Print:

```c
sizeof(union Data)
```

Compare it with:

```c
sizeof(struct Data)
```

---

## Lab 3

Print the addresses of every member.

```c
printf("%p\n",
       (void *)&value.i);

printf("%p\n",
       (void *)&value.f);
```

Notice that they are identical.

---

## Lab 4

Build a tagged union representing different payment methods.

---

## Lab 5

Model a network packet that can contain either a TCP or UDP header.

---

# Interview Questions

### What is a union?

A user-defined data type where all members share the same memory location.

---

### How is a union different from a structure?

A structure allocates separate memory for each member.

A union shares one memory block among all members.

---

### What determines the size of a union?

The size of its largest member (subject to alignment requirements).

---

### Why are tagged unions used?

To record which union member currently contains valid data.

---

### Where are unions commonly used?

- Linux Kernel
- Embedded Systems
- Network Protocols
- Compilers
- Device Drivers
- Database Engines

---

# Summary

```
Union

↓

Shared Memory

↓

Member A

or

Member B

or

Member C
```

Compared with a structure:

```
Structure

↓

Separate Memory

↓

Member A

Member B

Member C
```

## Key Takeaways

- A union allows multiple members to share the same memory.
- Only one member should be considered valid at a time.
- The size of a union is based on its largest member.
- Tagged unions combine a union with a type indicator for safety.
- Unions are widely used in systems programming to save memory and represent variant data efficiently.

---

# Next Chapter

## Chapter 56 — Bit Fields

You'll learn:

- What bit fields are
- Why they exist
- Packing data into individual bits
- Saving memory
- Hardware register representation
- Network protocol flags
- Linux kernel and embedded systems usage