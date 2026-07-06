# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 5 --- Data Types

------------------------------------------------------------------------

## Most beginners think:

> "An integer is just a number."

But to a computer, every piece of data is simply **bits**.

A data type tells the compiler:

-   How many bytes to reserve
-   How to interpret those bytes
-   What operations are allowed

Without data types, the CPU would not know whether a value represents a
number, a character, or part of a floating-point value.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a data type is
-   Primitive C data types
-   Signed vs Unsigned
-   Integer types
-   Floating-point types
-   Characters
-   Memory size
-   Choosing the correct type

------------------------------------------------------------------------

# What is a Data Type?

A data type is a rule that describes how data is stored in memory.

``` text
Variable
   │
   ▼
Data Type
   │
   ▼
Memory Size + Interpretation
```

Example:

``` c
int age = 25;
char grade = 'A';
float price = 99.5f;
```

------------------------------------------------------------------------

# Integer Types

``` c
short s = 10;
int i = 100;
long l = 100000L;
long long ll = 10000000000LL;
```

Typical sizes (may vary by platform):

  Type                 Typical Size
  ----------- ---------------------
  short                     2 bytes
  int                       4 bytes
  long          8 bytes (Linux x64)
  long long                 8 bytes

------------------------------------------------------------------------

# Character Type

A `char` stores a single byte.

``` c
char letter = 'A';
```

Internally:

``` text
Character  ASCII

A          65
B          66
```

------------------------------------------------------------------------

# Floating-Point Types

``` c
float price = 10.5f;
double pi = 3.141592653589793;
```

  Type     Precision
  -------- -------------
  float    \~7 digits
  double   \~15 digits

Use `double` when more precision is required.

------------------------------------------------------------------------

# Signed vs Unsigned

Signed numbers can store positive and negative values.

``` c
int balance = -50;
```

Unsigned numbers store only zero or positive values.

``` c
unsigned int count = 100;
```

``` text
Signed
-5 -4 -3 -2 -1 0 1 2 3

Unsigned
0 1 2 3 4 5
```

Unsigned provides a larger positive range for the same number of bytes.

------------------------------------------------------------------------

# Memory Representation

``` text
RAM

+-----------+
| int age   |
+-----------+
| char A    |
+-----------+
| float     |
+-----------+
| double    |
+-----------+
```

Each type reserves a different amount of memory.

------------------------------------------------------------------------

# sizeof Operator

The `sizeof` operator tells you how many bytes a type or variable
occupies.

``` c
#include <stdio.h>

int main(void)
{
    printf("%zu\n", sizeof(int));
    printf("%zu\n", sizeof(double));
}
```

------------------------------------------------------------------------

# Choosing the Right Type

  Situation                 Recommended Type
  ------------------------- ------------------
  Age                       int
  Student Grade             char
  Temperature               float
  Scientific Calculations   double
  File Size                 long long

Choosing an appropriate type improves memory usage and correctness.

------------------------------------------------------------------------

# Common Mistakes

## Using float for money

Avoid:

``` c
float money = 10.10f;
```

Binary floating-point cannot represent many decimal fractions exactly.

Money is often stored as the smallest unit (for example, cents) using
integers.

------------------------------------------------------------------------

## Integer Overflow

``` c
unsigned char x = 255;
x++;
```

Result:

``` text
0
```

The value wraps around because it exceeded the maximum representable
value.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Print sizes:

``` c
printf("%zu\n", sizeof(char));
printf("%zu\n", sizeof(int));
printf("%zu\n", sizeof(long));
printf("%zu\n", sizeof(double));
```

------------------------------------------------------------------------

## Lab 2

Create one variable of every primitive type.

Print each value.

------------------------------------------------------------------------

## Lab 3

Experiment with integer overflow.

Observe how values wrap around.

------------------------------------------------------------------------

# Interview Questions

### What is a data type?

A data type defines how data is stored, interpreted, and manipulated in
memory.

### What is the difference between float and double?

`double` provides more precision and usually occupies more memory than
`float`.

### What does `sizeof` do?

It returns the size of a type or object in bytes.

### What is the difference between signed and unsigned integers?

Signed integers store both positive and negative values, while unsigned
integers store only non-negative values.

------------------------------------------------------------------------

# Summary

``` text
Data
 │
 ▼
Data Type
 │
 ▼
Memory Allocation
 │
 ▼
CPU Operations
```

Key Takeaways:

-   Every variable has a data type.
-   Data types determine memory size and interpretation.
-   `char`, integers, `float`, and `double` serve different purposes.
-   Use `sizeof` to inspect type sizes.
-   Choose the smallest appropriate type without sacrificing
    correctness.

------------------------------------------------------------------------

# Next Chapter

## Chapter 6 --- Type Conversion

You'll learn:

-   Implicit conversion
-   Explicit casting
-   Integer promotion
-   Overflow
-   Underflow
-   Precision loss
