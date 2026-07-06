# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 6 --- Type Conversion

------------------------------------------------------------------------

## Most beginners think:

> "The compiler only does exactly what I write."

But that's not always true.

The C compiler automatically converts data types in many situations.

Sometimes this is helpful.

Sometimes it introduces subtle bugs.

Understanding type conversion is essential for writing correct and
efficient C programs.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What type conversion is
-   Implicit conversion
-   Explicit casting
-   Integer promotion
-   Overflow
-   Underflow
-   Precision loss
-   Common conversion pitfalls

------------------------------------------------------------------------

# What is Type Conversion?

Type conversion means converting a value from one data type into
another.

``` text
int
 │
 ▼
float
```

or

``` text
double
 │
 ▼
int
```

Conversions happen either automatically or manually.

------------------------------------------------------------------------

# Implicit Conversion

The compiler performs these conversions automatically.

``` c
int a = 10;
double b = a;
```

Result:

``` text
a = 10
b = 10.0
```

No cast is required.

------------------------------------------------------------------------

# Why Does the Compiler Convert?

When two different data types appear in one expression, the compiler
chooses a common type.

``` c
int a = 10;
double b = 5.5;

double c = a + b;
```

``` text
10
 │
 ▼
10.0
 │
 ▼
10.0 + 5.5
 │
 ▼
15.5
```

The integer is promoted to a double.

------------------------------------------------------------------------

# Integer Division

Many beginners expect:

``` c
int a = 5;
int b = 2;

printf("%f", a / b);
```

Expected:

``` text
2.5
```

Actual:

``` text
2
```

Because both operands are integers.

To get a decimal result:

``` c
printf("%f", (double)a / b);
```

------------------------------------------------------------------------

# Explicit Casting

Sometimes you want to control the conversion yourself.

``` c
double pi = 3.14159;

int value = (int)pi;
```

Result:

``` text
3
```

The decimal part is discarded.

------------------------------------------------------------------------

# Precision Loss

``` c
double price = 99.99;

int x = (int)price;
```

``` text
99.99
   │
   ▼
99
```

Information is lost.

Always be careful when converting from a larger type to a smaller type.

------------------------------------------------------------------------

# Integer Promotion

Small integer types are automatically promoted during arithmetic.

``` c
char a = 10;
char b = 20;

int c = a + b;
```

Even though `a` and `b` are `char`, the addition happens as `int`.

------------------------------------------------------------------------

# Overflow

Overflow happens when a value exceeds the maximum range of its type.

``` c
unsigned char x = 255;
x++;
```

Result:

``` text
0
```

The value wraps around.

------------------------------------------------------------------------

# Underflow

Underflow occurs when a value becomes smaller than the minimum
representable value.

Example:

``` c
unsigned int x = 0;
x--;
```

Result:

``` text
4294967295
```

(on a typical 32-bit unsigned integer)

------------------------------------------------------------------------

# Conversion Hierarchy

``` text
char
 │
 ▼
short
 │
 ▼
int
 │
 ▼
long
 │
 ▼
long long
 │
 ▼
float
 │
 ▼
double
```

Generally, smaller types are promoted to larger types.

------------------------------------------------------------------------

# Common Mistakes

## Mixing Signed and Unsigned

``` c
int a = -1;
unsigned int b = 1;

if (a < b)
```

This may not behave as expected because of implicit conversions.

------------------------------------------------------------------------

## Losing Fractional Values

``` c
double d = 8.9;
int i = d;
```

Fractional digits are discarded.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Convert an `int` to a `double` and print both values.

## Lab 2

Cast a `double` to an `int`.

Observe how the fractional part disappears.

## Lab 3

Create an overflow example using `unsigned char`.

## Lab 4

Experiment with integer division versus floating-point division.

------------------------------------------------------------------------

# Interview Questions

### What is implicit conversion?

Automatic conversion performed by the compiler.

### What is explicit conversion?

Manual conversion using a cast.

### What is integer promotion?

Automatic promotion of small integer types (such as `char` and `short`)
to `int` during arithmetic.

### What is overflow?

When a value exceeds the maximum range of its data type.

### Why is `(double)a / b` different from `a / b`?

Because the cast changes the arithmetic to floating-point division.

------------------------------------------------------------------------

# Summary

``` text
Different Types
      │
      ▼
Implicit Conversion
      │
      ▼
Common Type
      │
      ▼
Calculation
      │
      ▼
Result
```

Key Takeaways:

-   C automatically converts data types when necessary.
-   Use explicit casts when you need precise control.
-   Integer division discards fractional values.
-   Overflow and underflow can produce unexpected results.
-   Understanding conversions helps prevent subtle bugs.

------------------------------------------------------------------------

# Next Chapter

## Chapter 7 --- Operators

You'll learn:

-   Arithmetic operators
-   Relational operators
-   Logical operators
-   Assignment operators
-   Bitwise operators
-   Increment and decrement
-   Operator precedence
