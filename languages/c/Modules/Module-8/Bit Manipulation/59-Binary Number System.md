# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 59 — Binary Number System

---

# Most beginners think:

> "Computers understand numbers like humans do."

They don't.

Humans use the **Decimal Number System**.

Computers use the **Binary Number System**.

Everything inside your computer is represented using only **two values**.

```
0

1
```

That's all.

Every program,

every image,

every video,

every song,

every Linux process,

every Docker container,

every database,

and even this document you're reading,

is ultimately stored as billions of **0s and 1s**.

After this chapter, you'll understand how computers represent numbers internally and why binary is the foundation of all computing.

---

# Learning Objectives

After this chapter you will understand:

- What binary is
- Why computers use binary
- Decimal vs Binary
- Bits and Bytes
- Binary place values
- Decimal to Binary conversion
- Binary to Decimal conversion
- Real-world examples
- Common mistakes

---

# Why Don't Computers Use Decimal?

Imagine a light bulb.

It has only two states.

```
OFF

ON
```

We can represent them as:

```
OFF = 0

ON = 1
```

Modern computers are built using **billions of tiny electronic switches called transistors**.

A transistor also has only two stable states.

```
OFF

↓

0
```

```
ON

↓

1
```

Because hardware naturally works with two states,

computers use **Binary**.

---

# What is Binary?

Binary is a number system with only **two digits**.

```
0

1
```

Unlike Decimal:

```
0

1

2

3

...

9
```

Binary has only:

```
0

1
```

---

# Decimal vs Binary

Decimal:

```
Base 10
```

Uses:

```
0 - 9
```

Binary:

```
Base 2
```

Uses:

```
0

1
```

---

# What is a Bit?

A **Bit** (Binary Digit) is the smallest unit of data in a computer.

Possible values:

```
0

or

1
```

That's it.

---

# What is a Byte?

One byte consists of:

```
8 Bits
```

Visualization:

```
Bit

↓

0

Bit

↓

1

Bit

↓

1

Bit

↓

0

Bit

↓

1

Bit

↓

0

Bit

↓

0

Bit

↓

1
```

Together:

```
01101001
```

One Byte.

---

# Why 8 Bits?

Historically,

computer manufacturers experimented with different byte sizes.

Eventually,

8 bits became the industry standard because it can represent:

```
2⁸

=

256 Values
```

Range:

```
0

↓

255
```

---

# Binary Place Values

Just like decimal has place values:

```
1234

↓

1000

100

10

1
```

Binary has powers of **2**.

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

These are called **bit positions**.

---

# Example

Binary:

```
10110110
```

Write the place values.

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

Now multiply.

```
1 × 128 = 128

0 × 64 = 0

1 × 32 = 32

1 × 16 = 16

0 × 8 = 0

1 × 4 = 4

1 × 2 = 2

0 × 1 = 0
```

Add them.

```
128

+

32

+

16

+

4

+

2

=

182
```

Therefore:

```
10110110₂

=

182₁₀
```

---

# Another Example

Binary:

```
00001010
```

```
8

+

2

=

10
```

Result:

```
00001010₂

=

10₁₀
```

---

# Decimal to Binary

Suppose:

```
13
```

Repeatedly divide by:

```
2
```

```
13 ÷ 2 = 6 R1

6 ÷ 2 = 3 R0

3 ÷ 2 = 1 R1

1 ÷ 2 = 0 R1
```

Read remainders **from bottom to top**.

```
1101
```

Therefore:

```
13₁₀

=

1101₂
```

---

# Binary to Decimal

Example:

```
1101
```

Place values:

```
8

4

2

1
```

Multiply:

```
1×8

+

1×4

+

0×2

+

1×1
```

Result:

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

# Counting in Binary

Decimal:

```
0

1

2

3

4

5

6

7
```

Binary:

```
000

001

010

011

100

101

110

111
```

Notice how only **0** and **1** are used.

---

# Binary Overflow

Suppose we have only:

```
3 Bits
```

Maximum value:

```
111

=

7
```

What happens after:

```
111
```

?

It becomes:

```
1000
```

But:

```
1000
```

needs:

```
4 Bits
```

If only three bits are available,

overflow occurs.

---

# Why Binary Matters

Every operation inside a computer uses binary.

Examples:

```
Integer

↓

Binary
```

```
Character

↓

Binary
```

```
Floating Point

↓

Binary
```

```
Image

↓

Binary
```

```
Music

↓

Binary
```

```
Video

↓

Binary
```

Even memory addresses are represented in binary.

---

# Real-World Example — ASCII

Character:

```
A
```

ASCII Value:

```
65
```

Binary:

```
01000001
```

The computer stores:

```
01000001
```

Not the letter "A".

---

# Real-World Example — RGB Colors

Suppose:

```
Red

↓

255
```

Binary:

```
11111111
```

Green:

```
00000000
```

Blue:

```
00000000
```

Combined:

```
Pure Red
```

Images are simply millions of binary values.

---

# Real-World Example — CPU Registers

Suppose a CPU register stores:

```
0000000000001010
```

The CPU interprets it as:

```
10
```

Every arithmetic operation happens on binary numbers.

---

# Common Mistakes

---

## Thinking Binary Is Difficult

Binary is simply another number system.

Instead of powers of:

```
10
```

it uses powers of:

```
2
```

---

## Reading Bits Left to Right Incorrectly

Always remember the place values.

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

---

## Forgetting Leading Zeros

These are equivalent:

```
101

00000101
```

Leading zeros don't change the value.

---

# Hands-on Labs

## Lab 1

Convert the following decimal numbers to binary.

```
5

10

25

50

100
```

---

## Lab 2

Convert these binary numbers to decimal.

```
1010

1111

100000

110011
```

---

## Lab 3

Write a C program that prints the binary representation of an integer.

(Hint: You'll learn bitwise operators in the next chapter.)

---

## Lab 4

Print the binary representation of every number from:

```
0

↓

31
```

Observe the pattern.

---

## Lab 5

Open the Linux Calculator.

Switch to **Programmer Mode**.

Convert numbers between:

- Decimal
- Binary
- Hexadecimal

---

# Interview Questions

### What is Binary?

A number system that uses only:

```
0

1
```

---

### Why do computers use binary?

Because electronic circuits (transistors) naturally have two stable states: ON and OFF.

---

### What is a bit?

The smallest unit of data.

Possible values:

```
0

1
```

---

### What is a byte?

A group of:

```
8 Bits
```

---

### How many values can one byte represent?

```
2⁸

=

256
```

Values:

```
0

↓

255
```

---

# Summary

```
Transistor

↓

ON / OFF

↓

1 / 0

↓

Bits

↓

Bytes

↓

Binary Numbers

↓

Everything Inside Computer
```

## Key Takeaways

- Computers use binary because hardware naturally has two stable states.
- Binary is a base-2 number system.
- A bit stores one binary digit.
- A byte contains 8 bits.
- Binary numbers use powers of 2.
- Every piece of data inside a computer is ultimately represented as binary.
- Understanding binary is essential before learning bitwise operators, networking, memory management, and operating systems.

---

# Next Chapter

## Chapter 60 — Bitwise Operators

You'll learn:

- `&` (AND)
- `|` (OR)
- `^` (XOR)
- `~` (NOT)
- `<<` (Left Shift)
- `>>` (Right Shift)

These operators are the foundation of:

- Linux Kernel
- Device Drivers
- Embedded Systems
- Cryptography
- Networking
- High-Performance Programming