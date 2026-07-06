# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 65 — Checking Bits

---

# Most beginners think:

> "To know whether a feature is enabled, I have to compare the entire number."

For example:

```
Status

↓

10110110
```

Suppose you only want to know:

```
Is Bluetooth ON?
```

Why compare all 8 bits?

You only care about **one bit**.

This is where **Bit Masks** and the **Bitwise AND (`&`)** operator become extremely useful.

Checking bits is one of the most common operations in systems programming.

Every second,

Linux,

Windows,

routers,

CPUs,

and embedded devices check millions of bits.

---

# Learning Objectives

After this chapter you will understand:

- What checking a bit means
- Why AND (`&`) is used
- Creating checking masks
- Testing one bit
- Testing multiple bits
- Helper functions
- Real-world examples
- Common mistakes

---

# What Does "Checking a Bit" Mean?

Checking means asking:

```
Is this bit

ON

or

OFF?
```

We do **not** change the value.

We simply inspect it.

---

# Why Use AND?

Recall the AND truth table.

```
1 & 1 = 1

1 & 0 = 0

0 & 1 = 0

0 & 0 = 0
```

If we AND a number with a mask,

every bit becomes:

```
0
```

except the bit we want to inspect.

---

# Example

Current value:

```
00101100
```

Need to check:

```
Bit 3
```

Mask:

```
00001000
```

Operation:

```
00101100

&

00001000

------------

00001000
```

Result is **not zero**.

Therefore:

```
Bit 3

↓

ON
```

---

# Another Example

Current:

```
00100100
```

Need:

```
Check Bit 3
```

Mask:

```
00001000
```

Operation:

```
00100100

&

00001000

------------

00000000
```

Result:

```
0
```

Therefore:

```
Bit 3

↓

OFF
```

---

# Creating the Mask

Professional programmers use:

```c
1 << bitPosition
```

Example:

```c
1 << 5
```

Binary:

```
00100000
```

Mask for:

```
Bit 5
```

---

# Basic Example

```c
#include <stdio.h>

int main()
{
    unsigned char status = 44;

    if (status & (1 << 3))
        printf("Bit 3 is ON\n");
    else
        printf("Bit 3 is OFF\n");

    return 0;
}
```

Output:

```
Bit 3 is ON
```

---

# Why We Don't Compare with 1

Many beginners write:

```c
if ((status & (1 << 3)) == 1)
```

This is wrong.

Why?

Suppose:

```
Bit 3
```

Mask:

```
00001000
```

Result:

```
00001000
```

Decimal:

```
8
```

Not:

```
1
```

Correct:

```c
if (status & (1 << 3))
```

or

```c
if ((status & (1 << 3)) != 0)
```

---

# Checking Multiple Bits

Suppose:

Need to verify:

```
Bit 1

Bit 4
```

Mask:

```c
(1 << 1) | (1 << 4)
```

Operation:

```c
status & (
            (1 << 1)
          | (1 << 4)
         );
```

The result tells you whether those bits are present.

---

# Checking if ALL Bits Are Set

Suppose:

```
Read

Write
```

Mask:

```c
(1 << READ)

|

(1 << WRITE)
```

Check:

```c
if ((status & mask) == mask)
{
    printf("Both permissions exist");
}
```

Example:

```
Status

↓

110

Mask

↓

110

Result

↓

110

Equal

↓

Yes
```

---

# Checking if ANY Bit Is Set

```c
if (status & mask)
{
    printf("At least one permission exists");
}
```

Example:

```
Status

↓

100

Mask

↓

110

↓

100

↓

Not Zero

↓

Yes
```

---

# Writing a Helper Function

```c
int isBitSet(unsigned int value,
             int bit)
{
    return (value & (1 << bit)) != 0;
}
```

Usage:

```c
if (isBitSet(status, 4))
{
    printf("Bit 4 is ON");
}
```

This makes code easier to read and reuse.

---

# Real-World Example — Linux Permissions

Permissions:

```
Read

↓

Bit 2

Write

↓

Bit 1

Execute

↓

Bit 0
```

Need to check:

```
Can the user execute this file?
```

Kernel:

```
Permissions

&

Execute Mask

↓

Result
```

If the result is non-zero,

execution is allowed.

---

# Real-World Example — TCP Flags

TCP Header:

```
SYN

ACK

FIN

RST
```

When a packet arrives,

the operating system checks:

```
ACK
```

using:

```
Flags

&

ACK Mask
```

This determines how the packet should be processed.

---

# Real-World Example — Hardware Register

Suppose:

```
Bit 0

↓

Power

Bit 1

↓

WiFi

Bit 2

↓

Bluetooth
```

A driver checks:

```
Power Bit
```

If it's OFF,

the device isn't ready.

---

# Real-World Example — CPU Status Register

CPUs maintain flags like:

```
Zero Flag

Carry Flag

Overflow Flag

Interrupt Flag
```

The processor checks these bits continuously to decide what instruction to execute next.

---

# Common Mistakes

---

## Comparing with 1

Wrong:

```c
(status & mask) == 1
```

Correct:

```c
(status & mask) != 0
```

or simply:

```c
if (status & mask)
```

---

## Using OR

Wrong:

```c
status | mask
```

OR changes data.

AND checks data.

---

## Forgetting Parentheses

Wrong:

```c
status & 1 << 3
```

Although `<<` has higher precedence and this expression works as intended, writing it without parentheses is harder to read.

Prefer:

```c
status & (1 << 3)
```

Clear code is better than clever code.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    unsigned char permissions = 6;

    if (permissions & (1 << 2))
        printf("Read Allowed\n");

    if (permissions & (1 << 1))
        printf("Write Allowed\n");

    if (permissions & (1 << 0))
        printf("Execute Allowed\n");

    return 0;
}
```

Output:

```
Read Allowed

Write Allowed
```

---

# Hands-on Labs

## Lab 1

Check:

```
Bit 0
```

Print:

```
ON

OFF
```

---

## Lab 2

Check:

```
Bit 2

Bit 5

Bit 7
```

---

## Lab 3

Write:

```c
isBitSet()
```

Use it in several examples.

---

## Lab 4

Represent Linux permissions.

Check:

```
Read

Write

Execute
```

---

## Lab 5

Represent TCP flags.

Check:

```
ACK

SYN

FIN
```

---

# Interview Questions

### What does checking a bit mean?

Determining whether a specific bit is `1` or `0` without modifying it.

---

### Which operator is used?

Bitwise AND:

```c
&
```

---

### Why is AND used?

Because it isolates the selected bit while turning all other bits into `0`.

---

### How do you check if a bit is ON?

```c
if (value & (1 << bit))
```

or

```c
if ((value & (1 << bit)) != 0)
```

---

### What is the difference between checking ANY bits and checking ALL bits?

To check if **any** selected bit is set:

```c
if (value & mask)
```

To check if **all** selected bits are set:

```c
if ((value & mask) == mask)
```

---

# Summary

```
Current Value

↓

Create Mask

↓

AND

↓

Result

↓

Zero?

↓

Bit OFF

------------

Non-Zero?

↓

Bit ON
```

Example:

```
00101100

&

00001000

↓

00001000

↓

Bit 3 is ON
```

## Key Takeaways

- Checking a bit means inspecting its value without changing it.
- Bitwise AND (`&`) is the standard operator for testing bits.
- Masks are typically created using:

```c
1 << bitPosition
```

- A non-zero result means the bit is ON.
- A zero result means the bit is OFF.
- Bit checking is fundamental in operating systems, networking, device drivers, CPUs, and embedded systems.

---

# 🎯 You Can Now Manipulate Any Bit

After Chapters **61–65**, you now know the four fundamental bit operations used throughout systems programming:

| Operation | Formula |
|-----------|---------|
| **Set a Bit** | `value |= (1 << bit);` |
| **Clear a Bit** | `value &= ~(1 << bit);` |
| **Toggle a Bit** | `value ^= (1 << bit);` |
| **Check a Bit** | `value & (1 << bit)` |

These four operations are used millions of times every second inside:

- ✅ Linux Kernel
- ✅ Device Drivers
- ✅ TCP/IP Stack
- ✅ File Systems
- ✅ CPUs
- ✅ Databases
- ✅ Embedded Systems
- ✅ Network Routers

Master them—they are among the most important low-level programming techniques you'll ever learn.

---

# Next Chapter

## Chapter 66 — Endianness

You'll learn:

- What Endianness is
- Little Endian vs Big Endian
- How CPUs store multi-byte numbers
- Why networking uses Big Endian
- How Intel, AMD, and ARM handle byte order
- How to detect your system's endianness in C
- Why endianness matters in networking, file formats, and operating systems