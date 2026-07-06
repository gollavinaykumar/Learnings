# 👑 C Mastery Roadmap

# Module 7 — Structures & Unions

# Chapter 56 — Bit Fields

---

# Most beginners think:

> "The smallest memory unit I can use in C is one byte."

That's true for normal variables.

But sometimes **one byte is too large**.

Suppose you need to store:

- Is User Logged In? (Yes/No)
- Is Email Verified? (Yes/No)
- Is Admin? (Yes/No)
- Is Account Locked? (Yes/No)

Each answer needs only **one bit**.

Using:

```c
bool isLoggedIn;
bool isVerified;
bool isAdmin;
bool isLocked;
```

may consume multiple bytes.

Instead, C allows us to store data **bit by bit**.

This feature is called **Bit Fields**.

Bit fields are widely used in:

- Linux Kernel
- Device Drivers
- Embedded Systems
- Network Protocols
- Hardware Registers

---

# Learning Objectives

After this chapter you will understand:

- What bit fields are
- Why bit fields exist
- How bit fields work
- Memory layout
- Packing data into bits
- Advantages and disadvantages
- Real-world uses

---

# Why Do We Need Bit Fields?

Imagine a classroom attendance sheet.

Instead of writing:

```
Present

Absent

Present

Absent
```

You could simply write:

```
1 0 1 0
```

One bit is enough.

Now imagine millions of records.

Saving even a few bytes per record becomes significant.

---

# What is a Bit Field?

A bit field allows you to specify **how many bits** a structure member should occupy.

Example:

```c
struct Status
{
    unsigned int isLoggedIn : 1;
    unsigned int isVerified : 1;
    unsigned int isAdmin    : 1;
    unsigned int isLocked   : 1;
};
```

Each member occupies only **1 bit**.

---

# Understanding the Syntax

```c
unsigned int isAdmin : 1;
```

Break it down:

- `unsigned int` → Base type
- `isAdmin` → Field name
- `: 1` → Reserve **1 bit**

Another example:

```c
unsigned int age : 7;
```

This reserves **7 bits**.

A 7-bit unsigned value can store:

```
0 → 127
```

---

# Memory Layout

Suppose:

```c
struct Flags
{
    unsigned int A : 1;
    unsigned int B : 1;
    unsigned int C : 1;
    unsigned int D : 1;
};
```

Memory:

```
Bit Position

7 6 5 4 3 2 1 0

↓

0 0 0 0 D C B A
```

Only four bits are used.

The remaining bits are unused.

---

# Example Program

```c
#include <stdio.h>

struct Status
{
    unsigned int isLoggedIn : 1;
    unsigned int isVerified : 1;
    unsigned int isAdmin    : 1;
};

int main()
{
    struct Status user = {1, 0, 1};

    printf("%u\n", user.isLoggedIn);
    printf("%u\n", user.isVerified);
    printf("%u\n", user.isAdmin);

    return 0;
}
```

Output:

```
1
0
1
```

---

# Bit Field Ranges

Suppose:

```c
unsigned int value : 3;
```

Three bits can represent:

```
000 → 0

001 → 1

010 → 2

011 → 3

100 → 4

101 → 5

110 → 6

111 → 7
```

Maximum value:

```
7
```

If you assign:

```c
value = 10;
```

The result is implementation-defined or truncated depending on the compiler and settings.

---

# Memory Saving Example

Without bit fields:

```c
struct User
{
    unsigned int loggedIn;
    unsigned int verified;
    unsigned int admin;
    unsigned int locked;
};
```

Memory:

```
4 Bytes

4 Bytes

4 Bytes

4 Bytes

=

16 Bytes
```

Using bit fields:

```c
struct User
{
    unsigned int loggedIn : 1;
    unsigned int verified : 1;
    unsigned int admin    : 1;
    unsigned int locked   : 1;
};
```

Conceptually:

```
4 Bits
```

The compiler packs these fields into the underlying storage unit, greatly reducing memory usage.

---

# Real-World Example — Linux Permissions

Linux file permissions:

```
Read

Write

Execute
```

Each permission needs only:

```
1 Bit
```

Conceptually:

```
111

↓

Read

Write

Execute
```

---

# Real-World Example — Network Header

An IPv4 header contains many small fields.

Examples:

```
Version

Header Length

Flags

Fragment Offset
```

Many of these fields occupy only a few bits.

Bit fields can model these compact layouts, although many networking libraries prefer explicit bit masking for portability.

---

# Real-World Example — Hardware Register

Suppose a hardware register:

```
Bit 0

Power

Bit 1

Reset

Bit 2

Interrupt

Bit 3

Error
```

Each flag occupies one bit.

Bit fields provide a natural representation.

---

# Advantages

- Saves memory
- Easy to read
- Models hardware registers
- Represents protocol flags
- Makes boolean flags compact

---

# Disadvantages

- Compiler-dependent layout
- Endianness considerations
- Not ideal for portable binary formats
- Slower than manual bit masks on some architectures

Because of these issues, Linux kernel code often prefers explicit bitwise operations over C bit fields for externally visible data structures.

---

# Bit Fields vs Bit Masks

Bit field:

```c
status.isAdmin = 1;
```

Bit mask:

```c
status |= (1 << 2);
```

Bit fields are easier to read.

Bit masks are often more portable and predictable.

Both are widely used.

---

# Common Mistakes

---

## Assuming Exact Memory Layout

Different compilers may pack bit fields differently.

Never use bit fields for portable file formats without understanding your compiler's ABI.

---

## Storing Large Values

Wrong:

```c
unsigned int age : 3;

age = 20;
```

Three bits cannot represent 20.

---

## Ignoring Signedness

Prefer:

```c
unsigned int
```

for flags.

Signed bit fields may behave differently across compilers.

---

# Hands-on Labs

## Lab 1

Create bit fields for:

- Read
- Write
- Execute

Print their values.

---

## Lab 2

Create:

```c
unsigned int age : 7;
```

Store values from:

```
0

↓

127
```

Observe what happens beyond the supported range.

---

## Lab 3

Print:

```c
sizeof(struct Flags)
```

Compare it with an equivalent structure using normal integers.

---

## Lab 4

Represent Linux file permissions using bit fields.

---

## Lab 5

Implement the same flags using bit masks.

Compare both approaches.

---

# Interview Questions

### What is a bit field?

A structure member that occupies a specified number of bits instead of an entire byte or word.

---

### Why are bit fields used?

To save memory and efficiently represent small values or flags.

---

### Where are bit fields commonly used?

- Hardware registers
- Device drivers
- Embedded systems
- Linux kernel
- Network protocols

---

### Are bit fields always portable?

No.

Their layout is implementation-defined and can vary between compilers and architectures.

---

### What is the difference between bit fields and bit masks?

Bit fields provide convenient syntax.

Bit masks offer greater portability and control over binary layouts.

---

# Summary

```
Structure

↓

Bit Fields

↓

1 Bit

1 Bit

3 Bits

5 Bits

↓

Compact Memory
```

Example:

```
Flags

↓

Read

Write

Execute

Admin
```

## Key Takeaways

- Bit fields store data using individual bits.
- They reduce memory usage for small values and flags.
- They are useful for hardware, embedded systems, and kernel programming.
- Their layout is compiler-dependent.
- For portable binary protocols, explicit bit masks are often preferred.
- Understanding bit fields prepares you for low-level systems programming and hardware interaction.

---

# Next Chapter

## Chapter 57 — `typedef`

You'll learn:

- Why `typedef` exists
- Simplifying complex type declarations
- `typedef` with structures, unions, pointers, and function pointers
- How large C projects improve readability using `typedef`
- Common conventions in the Linux kernel and C libraries