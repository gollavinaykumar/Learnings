# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 94 — Bit Manipulation (Bitwise Operators)

---

# Most beginners think:

> "Computers understand decimal numbers."

They don't.

Computers understand only:

```
0

and

1
```

Everything else is built from these two values.

When you write:

```c
int x = 13;
```

The CPU never stores:

```
13
```

Instead, it stores:

```
00001101
```

(assuming an 8-bit illustration)

Professional programmers manipulate these individual bits directly.

This is called:

```
Bit Manipulation
```

It is heavily used in:

- Linux Kernel
- Device Drivers
- Networking
- Embedded Systems
- Operating Systems
- Databases
- Compression Algorithms
- Cryptography

---

# Learning Objectives

After this chapter you will understand:

- Binary representation
- Bitwise operators
- Bit masks
- Setting bits
- Clearing bits
- Toggling bits
- Testing bits
- Real-world applications

---

# Binary Review

Suppose:

```c
int x = 13;
```

Binary:

```
Decimal

13

↓

Binary

00001101
```

Position values:

```
128

64

32

16

8

4

2

1
```

Number:

```
0

0

0

0

1

1

0

1
```

Meaning:

```
8

+

4

+

1

=

13
```

---

# Bitwise Operators

C provides:

| Operator | Meaning |
|----------|---------|
| `&` | AND |
| `|` | OR |
| `^` | XOR |
| `~` | NOT |
| `<<` | Left Shift |
| `>>` | Right Shift |

These operators work on individual bits,

not entire numbers.

---

# Bitwise AND (`&`)

Rule:

```
1 & 1

↓

1

Everything Else

↓

0
```

Truth Table:

| A | B | A & B |
|---|---|--------|
|0|0|0|
|0|1|0|
|1|0|0|
|1|1|1|

---

# Example

```c
int a = 12;
int b = 10;
```

Binary:

```
12

1100

10

1010
```

AND:

```
1100

1010

----

1000
```

Result:

```
8
```

Program:

```c
printf("%d",
12 & 10);
```

Output:

```
8
```

---

# Bitwise OR (`|`)

Rule:

```
If Either Bit Is 1

↓

Result Is 1
```

Truth Table:

| A | B | A \| B |
|---|---|---------|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|

Example:

```
1100

1010

----

1110
```

Result:

```
14
```

---

# Bitwise XOR (`^`)

Rule:

```
Different

↓

1

Same

↓

0
```

Truth Table:

| A | B | A ^ B |
|---|---|--------|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|0|

Example:

```
1100

1010

----

0110
```

Result:

```
6
```

---

# Bitwise NOT (`~`)

NOT flips every bit.

Example:

```
00001101

↓

11110010
```

Program:

```c
int x = 13;

printf("%d",
~x);
```

The numeric result depends on the integer representation (typically two's complement on modern systems).

---

# Left Shift (`<<`)

Example:

```c
5 << 1
```

Binary:

```
00000101

↓

00001010
```

Result:

```
10
```

Each left shift moves bits one position toward more significant bits.

For unsigned values,

shifting left by one is often equivalent to multiplying by 2,

provided no significant bits are lost.

---

# Right Shift (`>>`)

Example:

```c
20 >> 1
```

Binary:

```
00010100

↓

00001010
```

Result:

```
10
```

For unsigned integers,

right shift moves bits toward less significant positions.

For signed integers,

the behavior depends on the implementation.

---

# Setting a Bit

Suppose:

```
00001010
```

Need:

```
Set Bit 0
```

Mask:

```
00000001
```

Operation:

```c
x |= 1;
```

Result:

```
00001011
```

---

# Clearing a Bit

Need:

```
Clear Bit 3
```

Mask:

```
11110111
```

Operation:

```c
x &= ~(1 << 3);
```

Visualization:

```
Bit 3

↓

0
```

Everything else remains unchanged.

---

# Toggling a Bit

Need:

```
0

↓

1

or

1

↓

0
```

Use XOR.

```c
x ^= (1 << 2);
```

If the bit was:

```
0

↓

1
```

If it was:

```
1

↓

0
```

---

# Testing a Bit

Need to know whether:

```
Bit 5

↓

Set?
```

Program:

```c
if(x & (1 << 5))
{
    printf("ON");
}
```

Result:

```
Bit Exists?

↓

Yes

↓

Non-Zero

↓

True
```

---

# Bit Masks

Suppose:

Permissions:

```
Read

Write

Execute
```

Represent:

```
001

↓

Read

010

↓

Write

100

↓

Execute
```

Now:

```
Read + Write

↓

011
```

One integer stores multiple Boolean values.

---

# Permission Example

```c
#define READ    1
#define WRITE   2
#define EXECUTE 4

int permissions =
READ | WRITE;
```

Binary:

```
001

↓

Read

010

↓

Write

----

011
```

Check:

```c
if(permissions & WRITE)
```

---

# Flags

Operating systems often store flags like:

```
Hidden

System

Read Only

Archive
```

inside one integer using bit masks.

This saves memory and allows fast operations.

---

# Why Use Bits?

Suppose:

Need:

```
8 Boolean Variables
```

Using `bool`:

```
8 Bytes
```

Using one byte:

```
8 Bits
```

Memory:

```
8×

Smaller
```

---

# Real-World Example — Linux File Permissions

Linux stores permissions as bits.

Example:

```
rwxr-xr--
```

Internally,

each permission is represented by specific bits.

---

# Real-World Example — TCP Header

TCP contains flags:

```
SYN

ACK

FIN

RST

PSH
```

Each flag is stored as a single bit.

The receiver tests individual bits to determine packet behavior.

---

# Real-World Example — Embedded Systems

Microcontroller register:

```
Bit 0

↓

LED

Bit 1

↓

Motor

Bit 2

↓

Sensor
```

Setting or clearing a single bit controls hardware.

---

# Real-World Example — Game Engine

Game object:

```
Visible

↓

Bit 0

Alive

↓

Bit 1

Selected

↓

Bit 2
```

One integer stores many states efficiently.

---

# Common Mistakes

---

## Confusing Logical and Bitwise Operators

Logical:

```c
&&

||

!
```

Bitwise:

```c
&

|

^

~
```

They are completely different.

---

## Forgetting Parentheses

Wrong:

```c
x & 1 << 3
```

Better:

```c
x & (1 << 3)
```

Parentheses improve readability and avoid precedence mistakes.

---

## Shifting Too Far

Shifting by an amount greater than or equal to the width of the type results in undefined behavior.

---

## Using Signed Types Carelessly

Prefer unsigned integers for low-level bit manipulation.

Signed shifts can be implementation-defined or undefined in some situations.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    unsigned int x = 0;

    x |= (1 << 2);

    printf("%u\n", x);

    x ^= (1 << 2);

    printf("%u\n", x);

    return 0;
}
```

Output:

```
4
0
```

---

# Hands-on Labs

## Lab 1

Implement functions to:

- Set a bit
- Clear a bit
- Toggle a bit
- Test a bit

---

## Lab 2

Build a permission system using bit masks.

---

## Lab 3

Represent eight Boolean flags using one byte.

---

## Lab 4

Convert decimal numbers to binary manually and verify the results.

---

## Lab 5

Decode a simulated TCP flag field using bitwise operations.

---

# Interview Questions

### What is bit manipulation?

Performing operations directly on the binary representation of data.

---

### What does `&` do?

Bitwise AND.

---

### How do you set bit 5?

```c
x |= (1 << 5);
```

---

### How do you clear bit 5?

```c
x &= ~(1 << 5);
```

---

### How do you test bit 5?

```c
if(x & (1 << 5))
```

---

### Why are bit masks useful?

They allow multiple flags or Boolean values to be stored efficiently in a single integer.

---

# Summary

```
Integer

↓

Binary

↓

Bitwise Operators

↓

Manipulate Individual Bits
```

Common Operations:

```
Set

↓

|

----------------

Clear

↓

& ~

----------------

Toggle

↓

^

----------------

Test

↓

&
```

## Key Takeaways

- Computers operate on binary data.
- Bitwise operators manipulate individual bits efficiently.
- Bit masks allow multiple flags to be packed into a single integer.
- Bit manipulation is fundamental in operating systems, networking, embedded systems, device drivers, and performance-critical software.
- Prefer unsigned integers for portable low-level bit operations.
- Mastering bitwise programming is an essential skill for systems programmers.

---

# Next Chapter

## Chapter 95 — Bit Fields

You'll learn:

- What bit fields are
- How structures can store values using specific numbers of bits
- Memory savings
- Compiler layout considerations
- Hardware register mapping
- Protocol headers
- Advantages and limitations of bit fields