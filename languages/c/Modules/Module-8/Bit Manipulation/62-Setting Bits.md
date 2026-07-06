# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 62 — Setting Bits

---

# Most beginners think:

> "To turn ON one bit, I have to rewrite the entire binary number."

You don't.

That's exactly why **Bit Masks** and the **Bitwise OR (`|`)** operator exist.

Suppose a device has:

```
Power

WiFi

Bluetooth

GPS

Camera

Speaker

Microphone

Flashlight
```

Each feature is controlled by **one bit**.

If you want to enable only **Bluetooth**,

you shouldn't modify the other seven bits.

Instead,

you create a mask and set only the required bit.

This technique is used everywhere:

- Linux Kernel
- Device Drivers
- TCP/IP Stack
- Embedded Systems
- Databases
- Hardware Registers

---

# Learning Objectives

After this chapter you will understand:

- What "setting a bit" means
- Why OR (`|`) is used
- How to create masks
- Setting one bit
- Setting multiple bits
- Real-world examples
- Common mistakes

---

# What Does "Setting a Bit" Mean?

Setting a bit means:

```
Change

0

↓

1
```

If the bit is already:

```
1
```

Nothing changes.

---

# Why Use OR (`|`)?

Recall the OR truth table.

```
0 | 0 = 0

0 | 1 = 1

1 | 0 = 1

1 | 1 = 1
```

Notice something.

Whenever the mask contains:

```
1
```

the result becomes:

```
1
```

Exactly what we need.

---

# Example

Current value:

```
00000000
```

Need to enable:

```
Bit 2
```

Mask:

```
00000100
```

Operation:

```
00000000

|

00000100

------------

00000100
```

Result:

```
Bit 2

↓

ON
```

---

# Creating the Mask

Instead of writing:

```
00000100
```

Professional programmers write:

```c
1 << 2
```

Because:

```
00000001

↓

<<2

↓

00000100
```

---

# Basic Example

```c
#include <stdio.h>

int main()
{
    unsigned char status = 0;

    status = status | (1 << 2);

    printf("%u\n", status);

    return 0;
}
```

Output:

```
4
```

Binary:

```
00000100
```

---

# Shorter Syntax

Instead of:

```c
status = status | (1 << 2);
```

Use:

```c
status |= (1 << 2);
```

Both mean exactly the same thing.

Professional C code almost always uses:

```c
|=
```

---

# Setting Multiple Bits

Current:

```
00000000
```

Need:

```
Bit 1

Bit 4

Bit 6
```

Masks:

```
00000010

00010000

01000000
```

Combine them:

```c
status |= (1 << 1);

status |= (1 << 4);

status |= (1 << 6);
```

Result:

```
01010010
```

---

# Combining Masks

Instead of three separate statements:

```c
status |= (1 << 1);

status |= (1 << 4);

status |= (1 << 6);
```

Combine them:

```c
status |= (1 << 1)
       |  (1 << 4)
       |  (1 << 6);
```

This is common in system programming.

---

# Visual Representation

Suppose:

```
Current

00100010
```

Need:

```
Set Bit 4
```

Mask:

```
00010000
```

Operation:

```
00100010

|

00010000

------------

00110010
```

Only Bit 4 changed.

Everything else remained the same.

---

# Real-World Example — Linux Permissions

Suppose:

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

Current:

```
100

(Read Only)
```

Need:

```
Write Permission
```

Mask:

```
010
```

Operation:

```
100

|

010

↓

110
```

Now:

```
Read ✔

Write ✔

Execute ❌
```

---

# Real-World Example — WiFi Register

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

Current:

```
001
```

Need Bluetooth.

```
001

|

100

↓

101
```

Power remains ON.

Bluetooth becomes ON.

---

# Real-World Example — TCP Flags

TCP Header:

```
SYN

ACK

FIN

RST
```

Suppose:

```
ACK
```

must be enabled.

The operating system sets only that bit.

Everything else remains unchanged.

---

# Why OR Works

Suppose:

```
Current

10101010
```

Mask:

```
00000100
```

Only one position contains:

```
1
```

Every other position contains:

```
0
```

OR changes only the selected bit.

---

# Memory Visualization

```
Status Register

↓

00100110

↓

OR Mask

↓

00010000

↓

Result

↓

00110110
```

Only one bit changed.

---

# Common Mistakes

---

## Using XOR Instead of OR

Wrong:

```c
status ^= (1 << 3);
```

XOR toggles.

OR always sets the bit.

---

## Writing Binary Manually

Instead of:

```
00100000
```

Write:

```c
1 << 5
```

Cleaner.

Easier to understand.

---

## Confusing Decimal with Bit Position

Wrong:

```c
status |= 5;
```

Correct:

```c
status |= (1 << 5);
```

The first sets bits according to the binary representation of decimal `5` (`00000101`), not "bit number 5".

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    unsigned char status = 0;

    status |= (1 << 0);   // Power

    status |= (1 << 2);   // Bluetooth

    status |= (1 << 5);   // Speaker

    printf("%u\n", status);

    return 0;
}
```

Binary:

```
00100101
```

Enabled:

```
Power ✔

Bluetooth ✔

Speaker ✔
```

---

# Hands-on Labs

## Lab 1

Set:

```
Bit 0
```

Print the result.

---

## Lab 2

Set:

```
Bit 2

Bit 5

Bit 7
```

Print the binary value.

---

## Lab 3

Create Linux permission bits.

Enable:

```
Read

Write
```

---

## Lab 4

Create a WiFi device register.

Enable:

```
Power

Bluetooth

GPS
```

---

## Lab 5

Create TCP flags.

Enable:

```
SYN

ACK
```

---

# Interview Questions

### What does "setting a bit" mean?

Changing a bit from:

```
0

↓

1
```

---

### Which operator is used to set a bit?

Bitwise OR:

```c
|
```

---

### Why does OR work?

Because:

```
Bit | 1

↓

Always 1
```

---

### How do programmers create masks?

Usually:

```c
1 << bitPosition
```

---

### Can OR accidentally clear a bit?

No.

OR only sets selected bits.

It never changes a `1` to `0`.

---

# Summary

```
Current Value

↓

Bit Mask

↓

OR Operation

↓

Selected Bit

↓

ON
```

Example:

```
00010010

|

00100000

↓

00110010
```

## Key Takeaways

- Setting a bit means changing it to `1`.
- Bitwise OR (`|`) is the standard operator for setting bits.
- Masks are typically created using `1 << bitPosition`.
- OR changes only the selected bit and leaves all others unchanged.
- Setting bits is used extensively in operating systems, networking, embedded systems, and hardware programming.

---

# Next Chapter

## Chapter 63 — Clearing Bits

You'll learn:

- How to turn a bit OFF
- Why AND (`&`) and NOT (`~`) are used together
- Clearing multiple bits
- Real-world examples from Linux permissions, TCP flags, and hardware registers