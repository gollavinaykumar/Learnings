# 👑 C Mastery Roadmap

# Module 8 — Bit Manipulation

# Chapter 67 — Two's Complement

---

# Most beginners think:

> "Computers store negative numbers by simply putting a minus (-) sign in front."

That's how **humans** write numbers.

Computers don't have a minus key inside the CPU.

The CPU only understands:

```
0

1
```

So how does a computer store:

```
-1

-5

-100

-1000
```

The answer is **Two's Complement**.

Two's Complement is one of the most important concepts in Computer Science.

Without it,

there would be:

- No negative numbers
- No subtraction
- No signed integers
- No operating systems
- No compilers

Every modern CPU uses Two's Complement.

---

# Learning Objectives

After this chapter you will understand:

- Why Two's Complement exists
- Why signed numbers are difficult
- One's Complement vs Two's Complement
- How negative numbers are stored
- Why `~5` becomes `-6`
- Signed vs Unsigned
- Integer overflow
- Real-world CPU arithmetic

---

# Why Is This Problem Difficult?

Suppose you store:

```
5
```

Binary:

```
00000101
```

Easy.

Now store:

```
-5
```

What binary should we use?

Maybe:

```
-00000101
```

Can we store a minus sign inside memory?

No.

Memory stores only:

```
0

1
```

Nothing else.

---

# The CPU Knows Only Binary

Imagine the CPU.

```
CPU

↓

0

1
```

It never sees:

```
+

-

*

/
```

Everything is converted into binary operations.

---

# Early Idea — Sign Bit

Suppose we use:

```
First Bit

↓

Sign
```

```
0

↓

Positive
```

```
1

↓

Negative
```

Example:

```
00000101

↓

+5
```

```
10000101

↓

-5
```

Looks simple.

But a problem appears.

---

# The Problem With Sign Bit

Suppose:

```
+5

+

-5
```

Should become:

```
0
```

The CPU now needs completely different hardware for:

- Addition
- Subtraction

This makes processor design more complex.

Computer engineers wanted one circuit that could do both.

---

# Another Problem

Using only a sign bit gives us:

```
+0

and

-0
```

Example:

```
00000000

↓

+0
```

```
10000000

↓

-0
```

But mathematically,

there should only be **one zero**.

This wastes one binary pattern.

---

# The Solution — Two's Complement

Two's Complement solves all these problems.

It provides:

- One representation for zero
- Simple arithmetic
- Easy subtraction
- Fast CPU implementation

That's why every modern processor uses it.

---

# How To Find Two's Complement

Suppose:

```
5
```

Binary:

```
00000101
```

Step 1

Invert every bit.

```
11111010
```

Step 2

Add:

```
1
```

```
11111010

+

00000001

------------

11111011
```

Result:

```
11111011
```

This represents:

```
-5
```

---

# Verify It

Positive:

```
5

↓

00000101
```

Negative:

```
-5

↓

11111011
```

Add them.

```
00000101

+

11111011

------------

1 00000000
```

Ignore the carry beyond 8 bits.

Result:

```
00000000
```

Exactly:

```
0
```

Amazing!

---

# Why CPUs Love Two's Complement

Notice something.

The CPU **did not perform subtraction**.

It only performed:

```
Addition
```

This means the CPU can build **one arithmetic circuit** that handles both addition and subtraction.

Simpler hardware.

Faster execution.

---

# Example

Suppose:

```
7

-

5
```

Instead of subtracting,

the CPU performs:

```
7

+

(-5)
```

Binary:

```
00000111

+

11111011

------------

00000010
```

Result:

```
2
```

No subtraction circuit required.

---

# Understanding `~5`

Earlier we learned:

```c
~5
```

returns:

```
-6
```

Why?

Let's see.

Binary:

```
5

↓

00000101
```

NOT:

```
11111010
```

Is this:

```
-5
```

No.

Remember:

To obtain a negative number,

we invert and add:

```
1
```

Here we only inverted.

```
11111010
```

represents:

```
-6
```

Let's verify.

Take Two's Complement again.

Invert:

```
00000101
```

Add one:

```
00000110
```

Decimal:

```
6
```

Therefore:

```
11111010

↓

-6
```

So:

```c
~5

↓

-6
```

---

# Signed vs Unsigned

Suppose:

```
8 Bits
```

Unsigned:

```
00000000

↓

0
```

Maximum:

```
11111111

↓

255
```

Range:

```
0

↓

255
```

---

Signed (Two's Complement):

```
10000000

↓

-128
```

Maximum:

```
01111111

↓

127
```

Range:

```
-128

↓

127
```

Same:

```
8 Bits
```

Different interpretation.

---

# Why Half the Values Become Negative

One bit is effectively used to distinguish negative and non-negative values in Two's Complement.

With 8 bits:

```
256 Total Patterns
```

Split into:

```
128 Negative Values

+

128 Non-Negative Values
```

Range:

```
-128

↓

127
```

---

# Integer Overflow

Suppose:

```
127

+

1
```

Binary:

```
01111111

+

00000001

------------

10000000
```

Binary:

```
10000000
```

represents:

```
-128
```

This is an overflow for an 8-bit signed integer.

**Important:** In C, **signed integer overflow is undefined behavior**. The wraparound shown here reflects typical Two's Complement hardware but is not guaranteed by the C language standard.

---

# Real-World Example — Temperature Sensor

Suppose a sensor stores:

```
-10°C
```

Binary:

```
Two's Complement
```

The operating system reads those bits and interprets them as a signed integer.

---

# Real-World Example — CPU Registers

CPUs don't distinguish between:

```
Signed

Unsigned
```

The bits are identical.

The interpretation depends on the instruction and the data type chosen by the program.

---

# Real-World Example — Linux Kernel

Linux stores many values like:

```
Process Priority

CPU Load

Temperature

Error Codes
```

using signed integers represented internally with Two's Complement.

---

# Common Mistakes

---

## Thinking `~x` Means Negative

Wrong.

```c
~5
```

does **not** mean:

```
-5
```

It means:

Invert every bit.

On modern systems:

```
~5

=

-6
```

---

## Confusing Signed and Unsigned

Same bits.

Different interpretation.

Example:

```
11111111
```

Unsigned:

```
255
```

Signed:

```
-1
```

---

## Assuming Signed Overflow Is Safe

Many CPUs wrap around,

but the C standard treats signed overflow as **undefined behavior**.

Do not rely on it in portable programs.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    signed char a = 5;

    printf("%d\n", a);

    printf("%d\n", ~a);

    return 0;
}
```

Typical Output:

```
5

-6
```

---

# Hands-on Labs

## Lab 1

Convert:

```
5

10

25
```

to binary.

Find their Two's Complement.

---

## Lab 2

Verify:

```
x

+

(-x)

=

0
```

using binary addition.

---

## Lab 3

Print:

```c
~5

~10

~100
```

Observe the pattern.

---

## Lab 4

Compare:

```c
unsigned char

signed char
```

using the same binary values.

---

## Lab 5

Experiment with:

```
127 + 1
```

and

```
255 + 1
```

Observe the results for signed and unsigned types, and remember that signed overflow is undefined behavior in C.

---

# Interview Questions

### What is Two's Complement?

A method used by modern computers to represent negative integers in binary.

---

### Why is Two's Complement used?

Because it allows addition and subtraction to use the same hardware and provides a unique representation for zero.

---

### How do you calculate Two's Complement?

1. Invert all bits.
2. Add 1.

---

### Why does `~5` produce `-6`?

Because `~` only flips the bits. The resulting bit pattern corresponds to `-6` in Two's Complement representation.

---

### What is the range of an 8-bit signed integer?

```
-128

↓

127
```

---

# Summary

```
Positive Number

↓

Binary

↓

Invert Bits

↓

Add 1

↓

Negative Number
```

Example:

```
5

↓

00000101

↓

Invert

↓

11111010

↓

+1

↓

11111011

↓

-5
```

## Key Takeaways

- Computers do not store a separate minus sign.
- Modern CPUs represent negative numbers using Two's Complement.
- Two's Complement makes subtraction possible using addition.
- `~` flips bits; it does not directly compute a negative value.
- Signed and unsigned integers share the same bits but interpret them differently.
- Two's Complement is fundamental to CPUs, operating systems, compilers, and low-level programming.

---

# 🎉 Module 8 Complete

Congratulations!

You now understand:

- ✅ Binary Number System
- ✅ Bitwise Operators
- ✅ Bit Masks
- ✅ Setting Bits
- ✅ Clearing Bits
- ✅ Toggling Bits
- ✅ Checking Bits
- ✅ Endianness
- ✅ Two's Complement

These concepts are the foundation for:

- Linux Kernel Development
- Device Drivers
- Network Protocols
- Embedded Systems
- Cryptography
- File Systems
- CPU Architecture
- High-Performance Systems Programming

---

# 🚀 Next Module

# Module 9 — File Handling

You'll learn:

- `fopen()`
- `fclose()`
- `fread()`
- `fwrite()`
- `fprintf()`
- Binary Files
- File Descriptors
- `stdin`
- `stdout`
- `stderr`

By the end of Module 9, you'll understand how C programs interact with the operating system to read and write files—the same mechanisms used by databases, compilers, shells, web servers, and the Linux kernel.