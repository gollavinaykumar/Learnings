# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 66 — Endianness

---

# Most beginners think:

> "If I store the number `0x12345678` in memory, every computer stores it exactly the same way."

That's not true.

Different CPUs can store the **same number** in **different byte orders**.

This concept is called **Endianness**.

Understanding endianness is extremely important for:

- Linux Kernel
- Networking
- Device Drivers
- File Systems
- Databases
- Embedded Systems
- Reverse Engineering

If you don't understand endianness,

you'll eventually encounter mysterious bugs when communicating between different systems.

---

# Learning Objectives

After this chapter you will understand:

- What Endianness is
- Why it exists
- Bytes vs Bits
- Little Endian
- Big Endian
- How CPUs store numbers
- Detecting Endianness
- Network Byte Order
- Real-world examples

---

# First Understand One Important Thing

Many beginners confuse:

```
Bits
```

with

```
Bytes
```

They are **not** the same.

---

# A Byte

One byte contains:

```
8 Bits
```

Example:

```
01100101
```

One byte.

---

# A 32-bit Integer

Suppose:

```c
int number = 0x12345678;
```

Hexadecimal:

```
12

34

56

78
```

Each pair represents:

```
1 Byte
```

Memory needs:

```
4 Bytes
```

Visualization:

```
Byte 1

↓

12

------------

Byte 2

↓

34

------------

Byte 3

↓

56

------------

Byte 4

↓

78
```

The question is:

**Which byte should be stored first?**

That's what Endianness answers.

---

# What is Endianness?

Endianness defines:

> **The order in which bytes are stored in memory.**

Notice:

It talks about **bytes**,

not individual bits.

---

# Little Endian

Little Endian stores the:

```
Least Significant Byte

↓

First
```

Suppose:

```
0x12345678
```

Bytes:

```
12

34

56

78
```

Memory:

```
Address

1000

↓

78

1001

↓

56

1002

↓

34

1003

↓

12
```

The smallest byte comes first.

---

# Why is 78 the Least Significant Byte?

Binary:

```
0x12

0x34

0x56

0x78
```

The last byte contains the lowest-value part of the number.

Think of decimal:

```
1234
```

The digit:

```
4
```

is the least significant digit.

Similarly:

```
78
```

is the least significant byte.

---

# Big Endian

Big Endian stores the:

```
Most Significant Byte

↓

First
```

Memory:

```
Address

1000

↓

12

1001

↓

34

1002

↓

56

1003

↓

78
```

Exactly the opposite.

---

# Comparison

Number:

```
0x12345678
```

Little Endian

```
Address

↓

78

56

34

12
```

Big Endian

```
Address

↓

12

34

56

78
```

The **number is identical**.

Only the storage order changes.

---

# Why Does Endianness Exist?

Different CPU designers made different architectural choices.

Today:

Most desktop CPUs:

```
Intel

AMD
```

use:

```
Little Endian
```

Many network protocols use:

```
Big Endian
```

Some ARM processors can operate in either mode.

---

# Real Memory Example

Suppose:

```c
int number = 0x12345678;
```

Address:

```
1000
```

Little Endian

```
1000 → 78

1001 → 56

1002 → 34

1003 → 12
```

Big Endian

```
1000 → 12

1001 → 34

1002 → 56

1003 → 78
```

---

# How C Reads Memory

Suppose:

```c
int x = 0x12345678;
```

Memory:

```
1000

↓

78
```

If we create:

```c
char *p = (char *)&x;
```

Then:

```c
*p
```

reads the **first byte** in memory.

On a Little Endian machine:

```
0x78
```

On a Big Endian machine:

```
0x12
```

---

# Detecting Endianness

Example:

```c
#include <stdio.h>

int main()
{
    int x = 1;

    char *p = (char *)&x;

    if (*p == 1)
        printf("Little Endian\n");
    else
        printf("Big Endian\n");

    return 0;
}
```

Why does this work?

---

# Understanding the Trick

Suppose:

```c
int x = 1;
```

Binary:

```
00000000

00000000

00000000

00000001
```

Little Endian Memory

```
Address

↓

01

00

00

00
```

First byte:

```
01
```

Big Endian Memory

```
Address

↓

00

00

00

01
```

First byte:

```
00
```

That's why checking the first byte tells us the system's endianness.

---

# Why Networking Uses Big Endian

Computers communicate over networks.

Imagine:

```
Intel

↓

Little Endian
```

sends data to

```
Big Endian Machine
```

If both used their own formats,

numbers would be interpreted incorrectly.

To avoid confusion,

Internet protocols define a standard called:

```
Network Byte Order
```

which is:

```
Big Endian
```

---

# Network Byte Order

Whenever data is sent through sockets,

many network APIs expect numbers in **Network Byte Order**.

Example:

```
Host

↓

Convert

↓

Network Byte Order

↓

Send

↓

Receive

↓

Convert Back
```

In C,

the standard library provides helper functions:

```c
htons()   // Host to Network Short

htonl()   // Host to Network Long

ntohs()   // Network to Host Short

ntohl()   // Network to Host Long
```

We'll use these extensively in the Networking module.

---

# Real-World Example — TCP/IP

Suppose a port number:

```
8080
```

Before transmission,

it is converted into:

```
Network Byte Order
```

The receiving machine converts it back into its native format.

---

# Real-World Example — PNG File

Many binary file formats define their own byte order.

When reading a PNG, JPEG, or executable file,

software must interpret bytes according to the file specification,

not necessarily the CPU's native order.

---

# Real-World Example — Linux Kernel

The Linux kernel frequently converts between:

```
CPU Byte Order

↓

Network Byte Order
```

to ensure correct communication across machines.

---

# Common Mistakes

---

## Confusing Bits with Bytes

Endianness changes the **order of bytes**.

It does **not** reverse the bits inside a byte.

Example:

Byte:

```
01100101
```

remains:

```
01100101
```

---

## Assuming Every CPU Is Little Endian

Most desktops are,

but not every processor.

Some architectures support both modes.

---

## Ignoring Byte Order in Networking

Sending raw integers without conversion can cause incorrect values on machines with different byte orders.

Always use the appropriate conversion functions for network protocols.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    int value = 0x12345678;

    unsigned char *ptr = (unsigned char *)&value;

    for (int i = 0; i < sizeof(value); i++)
    {
        printf("%02X ", ptr[i]);
    }

    printf("\n");

    return 0;
}
```

On a typical Intel or AMD computer, the output is:

```
78 56 34 12
```

because these processors are Little Endian.

---

# Hands-on Labs

## Lab 1

Run the endianness detection program.

Determine whether your computer is:

- Little Endian
- Big Endian

---

## Lab 2

Store:

```c
0x11223344
```

Print each byte using a `char *`.

Observe the memory order.

---

## Lab 3

Experiment with:

```c
short

int

long long
```

Observe how multiple-byte values are stored.

---

## Lab 4

Use:

```c
htons()

htonl()
```

to convert values to Network Byte Order.

(You'll revisit these in the Networking module.)

---

## Lab 5

Use a debugger like `gdb` to inspect memory byte by byte.

---

# Interview Questions

### What is Endianness?

The order in which bytes of a multi-byte value are stored in memory.

---

### What is Little Endian?

The Least Significant Byte is stored at the lowest memory address.

---

### What is Big Endian?

The Most Significant Byte is stored at the lowest memory address.

---

### Which byte order does TCP/IP use?

Network Byte Order,

which is Big Endian.

---

### How can you detect Endianness in C?

By examining the first byte of a multi-byte integer through a `char *` pointer.

---

# Summary

```
Integer

↓

Multiple Bytes

↓

Memory

↓

Little Endian

or

Big Endian
```

Example:

```
0x12345678

Little Endian

↓

78 56 34 12

-------------------

Big Endian

↓

12 34 56 78
```

## Key Takeaways

- Endianness determines the order of **bytes**, not bits.
- Little Endian stores the Least Significant Byte first.
- Big Endian stores the Most Significant Byte first.
- Most Intel and AMD processors use Little Endian.
- Internet protocols use Big Endian (Network Byte Order).
- Understanding endianness is essential for networking, operating systems, device drivers, file formats, and reverse engineering.

---

# Next Chapter

## Chapter 67 — Two's Complement

You'll learn:

- Why computers can represent negative numbers
- Why `~5` becomes `-6`
- Signed vs Unsigned integers
- Two's Complement representation
- Binary arithmetic
- Integer overflow
- How CPUs perform subtraction using addition
- Why every operating system and compiler uses Two's Complement