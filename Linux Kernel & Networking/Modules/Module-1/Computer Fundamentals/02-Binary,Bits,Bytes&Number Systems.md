Most developers use strings, numbers, images, and files every day.

They write:

```js
let age = 25;
let name = "Vinay";
```

But very few ask:

> **How does the computer actually store `25` or `"Vinay"`?**

The answer is simple:

**Everything inside a computer is just 0s and 1s.**

Numbers, text, videos, Docker images, databases, AI models—even the Linux kernel itself—is ultimately stored as binary.

After this chapter, you'll understand how computers represent information using bits, bytes, binary, hexadecimal, and character encoding.

---

# 👑 Linux Kernel & Networking Mastery

# Module 1 — Computer Fundamentals

# Chapter 2 — Binary, Bits, Bytes & Number Systems

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why computers use binary
> - What is a Bit
> - What is a Byte
> - Binary Number System
> - Decimal Number System
> - Hexadecimal Number System
> - Octal Number System
> - Binary to Decimal Conversion
> - Decimal to Binary Conversion
> - Hexadecimal Conversion
> - ASCII
> - Unicode
> - UTF-8
> - Endianness
> - How numbers, text, images and programs are stored

---

# 📖 Why Do Computers Understand Only 0 and 1?

Inside a computer are billions of tiny electronic switches called **transistors**.

A transistor has only two stable states:

```
OFF

ON
```

We represent them as:

```
OFF = 0

ON  = 1
```

Because hardware naturally works with two electrical states, computers use the **Binary Number System**.

---

# What is Binary?

Binary is a numbering system that uses only two digits.

```
0

1
```

Unlike the decimal system, there are no digits like:

```
2

5

8

9
```

Everything is represented using only 0 and 1.

---

# Decimal Number System

Humans normally use decimal.

```
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

It has:

```
10 Symbols
```

Each position represents a power of 10.

Example:

```
548

=

5 × 100

+

4 × 10

+

8 × 1
```

---

# Binary Number System

Binary has only two symbols.

```
0

1
```

Each position represents a power of 2.

Example:

```
1011
```

means

```
1 × 8

+

0 × 4

+

1 × 2

+

1 × 1

=

11
```

---

# Powers of 2

```
2⁰ = 1

2¹ = 2

2² = 4

2³ = 8

2⁴ = 16

2⁵ = 32

2⁶ = 64

2⁷ = 128

2⁸ = 256

2⁹ = 512
```

You will use these constantly.

Memorize them.

---

# What is a Bit?

A Bit is the smallest unit of data.

It can contain only:

```
0

or

1
```

Examples:

```
1

0

1

1

0
```

Each of these is **one bit**.

---

# What is a Byte?

A Byte consists of:

```
8 Bits
```

Example:

```
01000001
```

This entire sequence is:

```
1 Byte
```

---

# Why 8 Bits?

Because 8 bits can represent:

```
2⁸

=

256

Different Values
```

Range:

```
00000000

↓

11111111
```

Decimal:

```
0

↓

255
```

---

# Storage Units

```
1 Bit

↓

1 Byte = 8 Bits

↓

1 KB = 1024 Bytes

↓

1 MB = 1024 KB

↓

1 GB = 1024 MB

↓

1 TB = 1024 GB

↓

1 PB = 1024 TB
```

---

# Binary to Decimal Conversion

Example:

```
101101
```

Write powers of 2.

```
32 16 8 4 2 1

1  0  1 1 0 1
```

Multiply.

```
32

+

8

+

4

+

1

=

45
```

So

```
101101₂

=

45₁₀
```

---

# Decimal to Binary Conversion

Convert:

```
25
```

Repeatedly divide by 2.

```
25 ÷ 2 = 12 R1

12 ÷ 2 = 6  R0

6 ÷ 2 = 3   R0

3 ÷ 2 = 1   R1

1 ÷ 2 = 0   R1
```

Read remainders from bottom.

```
11001
```

Therefore

```
25

=

11001₂
```

---

# Binary Addition

Example:

```
  1010

+ 0011

------

  1101
```

Decimal:

```
10

+

3

=

13
```

---

# Binary Multiplication

Similar to decimal multiplication.

```
101

×

11

=

1111
```

Computers perform arithmetic using binary circuits.

---

# What is Octal?

Octal uses:

```
0

↓

7
```

Base:

```
8
```

Example:

```
17₈

=

15₁₀
```

Octal is less common today.

---

# What is Hexadecimal?

Binary numbers become very long.

Example:

```
1111111111111111
```

Hard to read.

Instead use hexadecimal.

Digits:

```
0

↓

9

A

B

C

D

E

F
```

Base:

```
16
```

---

# Hexadecimal Table

```
Decimal

10 → A

11 → B

12 → C

13 → D

14 → E

15 → F
```

---

# Why Developers Love Hex

Instead of

```
11111111
```

We write

```
FF
```

Instead of

```
11110000
```

We write

```
F0
```

Much easier.

---

# Where Hex is Used

- Memory Addresses
- Network Packets
- MAC Addresses
- Debuggers
- Kernel Logs
- Color Codes

Example:

```
#FFFFFF

#000000

0xFF

0x1000
```

---

# ASCII

Computers store letters as numbers.

ASCII assigns each character a number.

Example:

```
A

↓

65
```

```
B

↓

66
```

```
C

↓

67
```

Binary:

```
A

↓

01000001
```

---

# Example

Word:

```
CAT
```

ASCII:

```
C = 67

A = 65

T = 84
```

Stored as:

```
01000011

01000001

01010100
```

---

# Problem with ASCII

ASCII supports only:

```
128 Characters
```

Not enough for:

- Telugu
- Hindi
- Chinese
- Japanese
- Arabic
- Emojis

---

# Unicode

Unicode gives every character a unique number.

Example:

```
A

↓

U+0041
```

Emoji:

```
😀

↓

U+1F600
```

Telugu:

```
అ

↓

Unicode Code Point
```

Now every language can be represented.

---

# UTF-8

UTF-8 is the most common Unicode encoding.

Advantages:

✔ Backward compatible with ASCII

✔ Variable length

✔ Supports every language

✔ Used everywhere on the Internet

Examples:

```
HTML

JSON

Linux

Docker

Git

Web APIs
```

All primarily use UTF-8.

---

# Endianness

Suppose a number occupies multiple bytes.

Question:

Which byte comes first?

There are two methods.

---

## Big Endian

```
Highest Byte

↓

Lowest Byte
```

Example:

```
12 34 56 78
```

Stored exactly like this.

---

## Little Endian

```
Lowest Byte

↓

Highest Byte
```

Example:

```
78 56 34 12
```

Most modern CPUs like x86 use Little Endian.

---

# How Text is Stored

Suppose:

```
Hi
```

ASCII:

```
H = 72

i = 105
```

Binary:

```
01001000

01101001
```

The computer stores these bytes in memory.

---

# How Images are Stored

An image is simply millions of pixels.

Each pixel contains numbers.

Example:

```
Pixel

↓

Red

Green

Blue
```

Example:

```
255

0

0
```

means

```
Pure Red
```

Every image is ultimately just binary.

---

# How Programs are Stored

Suppose you write:

```java
System.out.println("Hello");
```

The compiler converts it into machine instructions.

Eventually:

```
101011...

110010...

001010...
```

The CPU understands only these binary instructions.

---

# Why Linux Kernel Developers Use Hex

Kernel logs often show:

```
0x7ffc1234

0xffff888102ab0000
```

Instead of long binary numbers.

Hex is shorter and maps neatly to binary.

---

# Hands-on Lab

## Lab 1 — Convert Decimal to Binary

Use Linux/macOS:

```bash
echo "obase=2;25" | bc
```

---

## Lab 2 — Convert Binary to Decimal

```bash
echo "ibase=2;11001" | bc
```

---

## Lab 3 — Print ASCII Values

Python:

```python
print(ord('A'))
print(chr(65))
```

---

## Lab 4 — View File in Hex

Linux/macOS:

```bash
hexdump -C README.md
```

Observe that files are just bytes.

---

# Interview Questions

## Why do computers use binary?

Because electronic transistors naturally have two stable states: ON and OFF.

---

## What is the difference between a Bit and a Byte?

A Bit is a single binary digit (0 or 1).

A Byte is a group of 8 bits.

---

## Why is hexadecimal commonly used?

It is a compact, human-friendly representation of binary and is widely used for memory addresses, debugging, and networking.

---

## Why was Unicode introduced?

ASCII could not represent all world languages and symbols.

Unicode provides a unique code point for every character.

---

## What is UTF-8?

UTF-8 is the most widely used Unicode encoding format and is compatible with ASCII.

---

# Summary

Everything inside a computer is represented as binary.

```
Transistors

↓

Bits

↓

Bytes

↓

Binary

↓

Hexadecimal

↓

Characters

↓

Files

↓

Programs

↓

Operating Systems
```

Key concepts:

- Binary uses only 0 and 1
- A Bit is the smallest unit of data
- A Byte contains 8 Bits
- Decimal, Binary, Octal and Hexadecimal are different number systems
- ASCII maps characters to numbers
- Unicode supports every language
- UTF-8 is the standard text encoding
- Images, text, videos and programs are all stored as binary

Understanding binary is essential because every layer of computing—from the Linux kernel to network packets—ultimately operates on sequences of bits.

---

# Next Chapter

## Chapter 3 — Memory: RAM, ROM, Stack, Heap & Virtual Memory

We'll explore:

- How memory is organized
- RAM vs ROM
- Stack memory
- Heap memory
- Memory allocation
- Memory leaks
- Virtual memory
- Paging
- Swap memory
- Why memory management is critical for operating systems and high-performance software