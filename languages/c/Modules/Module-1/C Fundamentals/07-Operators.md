# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 7 --- Operators

------------------------------------------------------------------------

## Most beginners think:

> "Operators are just symbols like `+` and `-`."

But operators are how you tell the CPU **what work to perform**.

Whenever you calculate, compare values, update variables, or manipulate
bits, you are using operators.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Arithmetic operators
-   Relational operators
-   Logical operators
-   Assignment operators
-   Bitwise operators
-   Increment & decrement
-   Operator precedence
-   Common mistakes

------------------------------------------------------------------------

# What is an Operator?

An operator performs an operation on one or more operands.

``` c
int sum = 5 + 3;
```

``` text
5   +   3
│   │   │
│ Operator
Operands
```

------------------------------------------------------------------------

# Arithmetic Operators

  Operator   Meaning
  ---------- ----------------
  \+         Addition
  \-         Subtraction
  \*         Multiplication
  /          Division
  \%         Modulus

Example:

``` c
int a = 10;
int b = 3;

printf("%d\n", a + b);
printf("%d\n", a % b);
```

------------------------------------------------------------------------

# Relational Operators

Used for comparisons.

  Operator   Meaning
  ---------- -----------------------
  ==         Equal
  !=         Not Equal
  \>         Greater Than
  \<         Less Than
  \>=        Greater Than or Equal
  \<=        Less Than or Equal

Example:

``` c
if (age >= 18)
{
    printf("Adult");
}
```

These expressions evaluate to either **true (1)** or **false (0)**.

------------------------------------------------------------------------

# Logical Operators

Combine conditions.

  Operator   Meaning
  ---------- ---------
  &&         AND
  \|\|       OR
  !          NOT

Example:

``` c
if (age >= 18 && citizen)
{
    printf("Eligible");
}
```

------------------------------------------------------------------------

# Assignment Operators

``` c
int x = 10;

x += 5;
x -= 2;
x *= 3;
x /= 2;
```

Equivalent to:

``` c
x = x + 5;
```

and so on.

------------------------------------------------------------------------

# Increment & Decrement

Increment:

``` c
x++;
++x;
```

Decrement:

``` c
x--;
--x;
```

## Difference

``` c
int x = 5;

printf("%d\n", x++);
```

Output:

``` text
5
```

Then `x` becomes `6`.

``` c
int x = 5;

printf("%d\n", ++x);
```

Output:

``` text
6
```

The increment happens before the value is used.

------------------------------------------------------------------------

# Bitwise Operators

Operate directly on binary bits.

  Operator   Meaning
  ---------- -------------
  &          AND
  \|         OR
  \^         XOR
  \~         NOT
  \<\<       Left Shift
  \>\>       Right Shift

Example:

``` c
5 & 3
```

``` text
5 = 0101
3 = 0011
--------------
    0001
```

Result:

``` text
1
```

Bitwise operations are heavily used in operating systems, networking and
embedded programming.

------------------------------------------------------------------------

# Operator Precedence

Not every expression is evaluated left to right.

Example:

``` c
2 + 3 * 4
```

Actually becomes:

``` text
2 + (3 * 4)
```

Result:

``` text
14
```

Use parentheses to make expressions clear.

``` c
(2 + 3) * 4
```

Result:

``` text
20
```

------------------------------------------------------------------------

# Common Mistakes

## Assignment vs Comparison

Wrong:

``` c
if (x = 10)
```

Correct:

``` c
if (x == 10)
```

The first assigns a value instead of comparing it.

------------------------------------------------------------------------

## Integer Division

``` c
5 / 2
```

Result:

``` text
2
```

Use casting for floating-point division.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Create a calculator using arithmetic operators.

## Lab 2

Compare two numbers using relational operators.

## Lab 3

Experiment with `x++` and `++x`.

## Lab 4

Perform bitwise AND, OR and XOR on small integers.

------------------------------------------------------------------------

# Interview Questions

### What is an operator?

A symbol that performs an operation on one or more operands.

### What is the difference between `=` and `==`?

`=` assigns a value.

`==` compares two values.

### What is the difference between `x++` and `++x`?

`x++` returns the current value, then increments.

`++x` increments first, then returns the new value.

### Where are bitwise operators commonly used?

Operating systems, networking, embedded systems, device drivers and
performance-critical software.

------------------------------------------------------------------------

# Summary

``` text
Operands
    │
    ▼
Operators
    │
    ▼
CPU Performs Work
    │
    ▼
Result
```

Key Takeaways:

-   Operators are fundamental building blocks of C.
-   Arithmetic operators perform calculations.
-   Relational and logical operators control decisions.
-   Assignment operators update variables efficiently.
-   Bitwise operators manipulate individual bits.
-   Operator precedence determines evaluation order.

------------------------------------------------------------------------

# Next Chapter

## Chapter 8 --- Input & Output

You'll learn:

-   printf()
-   scanf()
-   getchar()
-   putchar()
-   puts()
-   fgets()
-   Format specifiers
-   Safe user input
