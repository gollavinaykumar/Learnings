# 👑 C Mastery Roadmap

# Module 2 --- Control Flow

# Chapter 9 --- `if`, `else`, and `switch`

------------------------------------------------------------------------

## Most beginners think:

> "Programs execute every line from top to bottom."

That's true only until the program needs to **make a decision**.

Real software constantly asks questions:

-   Is the password correct?
-   Is the user an admin?
-   Did the payment succeed?
-   Is the network connected?

Without decision making, software would always do the same thing.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why control flow exists
-   `if`
-   `if...else`
-   `else if`
-   Nested `if`
-   `switch`
-   `break`
-   When to use `if` vs `switch`
-   Common mistakes

------------------------------------------------------------------------

# Why Control Flow?

``` text
Start
  │
  ▼
Condition?
 ├── Yes ─► Action A
 └── No  ─► Action B
```

A program chooses different paths based on conditions.

------------------------------------------------------------------------

# The `if` Statement

``` c
int age = 20;

if (age >= 18)
{
    printf("Adult\n");
}
```

The code inside the block runs only when the condition is true.

------------------------------------------------------------------------

# `if...else`

``` c
if (age >= 18)
{
    printf("Adult\n");
}
else
{
    printf("Minor\n");
}
```

Exactly one branch executes.

------------------------------------------------------------------------

# `else if`

Useful when there are multiple choices.

``` c
int marks = 82;

if (marks >= 90)
    printf("Grade A");
else if (marks >= 75)
    printf("Grade B");
else if (marks >= 50)
    printf("Grade C");
else
    printf("Fail");
```

Conditions are checked from top to bottom.

------------------------------------------------------------------------

# Nested `if`

``` c
if (loggedIn)
{
    if (isAdmin)
    {
        printf("Admin Panel");
    }
}
```

One decision can contain another.

------------------------------------------------------------------------

# Boolean Expressions

C treats:

``` text
0     → False

Non-zero → True
```

Example:

``` c
if (5)
{
    printf("Runs");
}
```

The condition is true because `5` is non-zero.

------------------------------------------------------------------------

# The `switch` Statement

When comparing one value against many constant options, `switch` is
often cleaner.

``` c
int day = 3;

switch(day)
{
case 1:
    printf("Monday");
    break;

case 2:
    printf("Tuesday");
    break;

case 3:
    printf("Wednesday");
    break;

default:
    printf("Unknown");
}
```

------------------------------------------------------------------------

# Why `break`?

Without `break`, execution continues into the next case.

``` c
case 1:
    printf("One");
case 2:
    printf("Two");
```

Output for value `1`:

``` text
One
Two
```

This is called **fall-through**.

------------------------------------------------------------------------

# `if` vs `switch`

  if                   switch
  -------------------- -----------------
  Complex conditions   One variable
  Ranges               Constant values
  Logical operators    Cleaner menus

------------------------------------------------------------------------

# Common Mistakes

## Assignment Instead of Comparison

Wrong:

``` c
if (x = 10)
```

Correct:

``` c
if (x == 10)
```

------------------------------------------------------------------------

## Missing `break`

Forgetting `break` may execute multiple cases unexpectedly.

------------------------------------------------------------------------

# Hands-on Labs

1.  Check whether a number is positive, negative, or zero.
2.  Build a simple grading system using `else if`.
3.  Build a calculator menu using `switch`.
4.  Experiment with switch fall-through by removing `break`.

------------------------------------------------------------------------

# Interview Questions

### When should you use `if`?

When conditions involve comparisons, ranges, or logical operators.

### When should you use `switch`?

When selecting one option based on a single variable with constant
values.

### Why is `break` used in `switch`?

To stop execution from continuing into the next case.

### What does `default` do?

It runs when none of the cases match.

------------------------------------------------------------------------

# Summary

``` text
Start
  │
  ▼
Decision
 ├── if
 ├── else if
 ├── else
 └── switch
       │
       ▼
Execute Selected Block
```

Key Takeaways:

-   Control flow lets programs make decisions.
-   `if` is flexible and handles complex conditions.
-   `switch` is ideal for selecting among constant values.
-   `break` prevents unwanted fall-through.

------------------------------------------------------------------------

# Next Chapter

## Chapter 10 --- Loops (`for`, `while`, `do...while`)

You'll learn how programs repeat tasks efficiently without writing the
same code multiple times.
