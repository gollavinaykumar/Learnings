# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 60 — Bitwise Operators

---

# Most beginners think:

> "The operators `&`, `|`, `^`, `~`, `<<`, and `>>` are rarely used."

Actually, they're used everywhere.

Almost every operating system, network protocol, embedded device, database engine, and compiler relies on bitwise operations.

Examples:

- Linux Kernel
- Device Drivers
- TCP/IP
- IPv4 & IPv6
- Redis
- PostgreSQL
- Encryption
- Compression
- Hardware Registers

After this chapter, you'll understand how computers manipulate data one bit at a time.

---

# Learning Objectives

After this chapter you will understand:

- What bitwise operators are
- Why they exist
- Binary operations
- AND (`&`)
- OR (`|`)
- XOR (`^`)
- NOT (`~`)
- Left Shift (`<<`)
- Right Shift (`>>`)
- Real-world uses

---

# Why Do We Need Bitwise Operators?

Suppose you have:

```
8 switches
```

Each switch controls one feature.

```
Bit 7 Bit 6 Bit 5 Bit 4 Bit 3 Bit 2 Bit 1 Bit 0
```

Example:

```
00001010
```

Instead of changing all eight switches,

you want to change only one.

Bitwise operators make this possible.

---

# What are Bitwise Operators?

Bitwise operators work **bit by bit**.

Example:

```
5

↓

00000101
```

```
3

↓

00000011
```

Instead of treating them as decimal numbers,

the CPU operates on each bit individually.

---

# The Six Bitwise Operators

| Operator | Name |
|----------|------|
| `&` | AND |
| `|` | OR |
| `^` | XOR |
| `~` | NOT |
| `<<` | Left Shift |
| `>>` | Right Shift |

---

# 1. Bitwise AND (`&`)

Rule:

```
1 & 1 = 1

1 & 0 = 0

0 & 1 = 0

0 & 0 = 0
```

Think of AND like **two security guards**.

Both must approve.

---

# Example

```c
int a = 5;
int b = 3;

printf("%d", a & b);
```

Binary:

```
5

00000101

3

00000011

------------

AND

00000001
```

Result:

```
1
```

---

# Real-Life Analogy

Door opens only if:

```
Key

AND

Password
```

Both must be correct.

---

# 2. Bitwise OR (`|`)

Rule:

```
1 | 1 = 1

1 | 0 = 1

0 | 1 = 1

0 | 0 = 0
```

If **either bit** is 1,

the result becomes 1.

---

# Example

```
5

00000101

3

00000011

------------

OR

00000111
```

Result:

```
7
```

---

# Real-Life Analogy

A room gets power if:

```
Main Supply

OR

Generator
```

Either one is enough.

---

# 3. Bitwise XOR (`^`)

Rule:

```
1 ^ 1 = 0

0 ^ 0 = 0

1 ^ 0 = 1

0 ^ 1 = 1
```

XOR means:

**Different → 1**

**Same → 0**

---

# Example

```
5

00000101

3

00000011

------------

XOR

00000110
```

Result:

```
6
```

---

# Why XOR is Special

Suppose:

```
A ^ B ^ B
```

Result:

```
A
```

Because:

```
B ^ B = 0
```

XOR is widely used in:

- Encryption
- Error detection
- Swapping values
- Checksums

---

# 4. Bitwise NOT (`~`)

NOT flips every bit.

Rule:

```
1 → 0

0 → 1
```

Example:

```
00000101

↓

11111010
```

In C, interpreting the result depends on the integer representation (typically two's complement), which you'll study in a later chapter.

---

# Example

```c
int x = 5;

printf("%d", ~x);
```

On most modern systems using two's complement:

```
~5

=

-6
```

We'll understand **why** in the chapter on Two's Complement.

---

# 5. Left Shift (`<<`)

Moves bits to the left.

Example:

```
5

00000101

↓

<<1

00001010
```

Result:

```
10
```

Every left shift by one position roughly multiplies an unsigned value by 2 (as long as no significant bits are lost).

---

# Example

```c
5 << 1

↓

10
```

```
5 << 2

↓

20
```

---

# Visualization

```
00000101

↓

00001010

↓

00010100
```

Bits move left.

---

# 6. Right Shift (`>>`)

Moves bits to the right.

Example:

```
20

00010100

↓

>>1

00001010
```

Result:

```
10
```

Every right shift by one position roughly divides an unsigned value by 2.

---

# Example

```c
20 >> 1

↓

10
```

```
20 >> 2

↓

5
```

---

# Visualization

```
00010100

↓

00001010

↓

00000101
```

Bits move right.

---

# Summary Table

| Operator | Meaning |
|----------|----------|
| `&` | Keep bits that are 1 in both numbers |
| `|` | Set bits that are 1 in either number |
| `^` | Set bits that are different |
| `~` | Flip every bit |
| `<<` | Shift bits left |
| `>>` | Shift bits right |

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    int a = 5;
    int b = 3;

    printf("AND : %d\n", a & b);

    printf("OR  : %d\n", a | b);

    printf("XOR : %d\n", a ^ b);

    printf("NOT : %d\n", ~a);

    printf("LEFT SHIFT : %d\n", a << 1);

    printf("RIGHT SHIFT: %d\n", a >> 1);

    return 0;
}
```

Output:

```
AND : 1

OR : 7

XOR : 6

NOT : -6

LEFT SHIFT : 10

RIGHT SHIFT : 2
```

---

# Real-World Example — Linux File Permissions

Permissions:

```
Read

Write

Execute
```

Stored as bits:

```
111

↓

RWX
```

Bitwise operations quickly check or modify permissions.

---

# Real-World Example — Hardware Registers

Suppose:

```
Bit 0

↓

Power

Bit 1

↓

Reset

Bit 2

↓

Interrupt
```

A driver can enable only one feature without affecting the others using bitwise operations.

---

# Real-World Example — TCP Flags

TCP packets contain flags like:

```
SYN

ACK

FIN

RST

PSH
```

Each flag occupies a single bit.

Routers and operating systems inspect these bits using bitwise operators.

---

# Common Mistakes

---

## Confusing Logical AND (`&&`) with Bitwise AND (`&`)

Wrong:

```c
a && b
```

This evaluates logical truth.

Bitwise AND:

```c
a & b
```

Operates on individual bits.

---

## Confusing OR (`||`) with Bitwise OR (`|`)

Logical OR:

```c
||
```

Bitwise OR:

```c
|
```

They serve different purposes.

---

## Assuming `~5` Equals `-5`

Wrong.

On two's complement systems:

```
~5

=

-6
```

We'll explain this mathematically in the Two's Complement chapter.

---

# Hands-on Labs

## Lab 1

Write a program demonstrating:

- AND
- OR
- XOR

Print both decimal and binary values.

---

## Lab 2

Experiment with:

```c
5 << 1

5 << 2

5 << 3
```

Observe the pattern.

---

## Lab 3

Experiment with:

```c
64 >> 1

64 >> 2

64 >> 3
```

---

## Lab 4

Print the binary representation before and after each operation.

---

## Lab 5

Use bitwise operators to manipulate permission flags.

---

# Interview Questions

### What are bitwise operators?

Operators that manipulate individual bits of integer values.

---

### What does AND (`&`) do?

A result bit is 1 only if both corresponding bits are 1.

---

### What does OR (`|`) do?

A result bit is 1 if either corresponding bit is 1.

---

### What is XOR (`^`) used for?

Detecting differences, toggling bits, encryption, checksums, and other low-level operations.

---

### Why are shift operators useful?

They efficiently move bits left or right and are commonly used in arithmetic, bit masks, and hardware programming.

---

# Summary

```
Integer

↓

Binary

↓

Bitwise Operator

↓

Bit-by-Bit Processing

↓

Result
```

Example:

```
5

↓

00000101

↓

AND

↓

00000001

↓

1
```

## Key Takeaways

- Bitwise operators manipulate individual bits.
- `&`, `|`, and `^` compare corresponding bits.
- `~` flips every bit.
- `<<` shifts bits left.
- `>>` shifts bits right.
- Bitwise operators are fundamental in operating systems, networking, embedded systems, cryptography, and hardware programming.

---

# Next Chapter

## Chapter 61 — Bit Masks

You'll learn:

- What a bit mask is
- Why bit masks are powerful
- Creating masks
- Combining masks with bitwise operators
- Real-world examples from Linux permissions, TCP flags, hardware registers, and device drivers