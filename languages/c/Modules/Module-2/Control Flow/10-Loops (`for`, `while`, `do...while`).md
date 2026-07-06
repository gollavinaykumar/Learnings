# 👑 C Mastery Roadmap

# Module 2 --- Control Flow

# Chapter 10 --- Loops (`for`, `while`, `do...while`)

------------------------------------------------------------------------

## Most beginners think:

> "If I want to print something 100 times, I have to write it 100
> times."

Imagine writing:

``` c
printf("Hello\n");
printf("Hello\n");
printf("Hello\n");
```

...1000 times.

That would be impossible to maintain.

Loops solve this problem.

They allow a block of code to execute repeatedly until a condition
changes.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why loops exist
-   `for` loop
-   `while` loop
-   `do...while` loop
-   Loop initialization
-   Loop condition
-   Loop update
-   Infinite loops
-   Nested loops
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Loops?

``` text
Start
  │
  ▼
Condition?
  │
Yes
  ▼
Run Code
  │
Update
  │
  └─────────────┐
                │
             Condition?
                │
              No ▼
               End
```

A loop keeps repeating while its condition is true.

------------------------------------------------------------------------

# The `for` Loop

Best when you know how many times to repeat.

``` c
for (int i = 1; i <= 5; i++)
{
    printf("%d\n", i);
}
```

Structure:

``` text
for (
 Initialization;
 Condition;
 Update
)
```

-   Initialization runs once.
-   Condition is checked before every iteration.
-   Update runs after each iteration.

------------------------------------------------------------------------

# How a `for` Loop Executes

``` text
Initialize i = 1
      │
      ▼
i <= 5 ?
 ├── Yes
 │     ▼
 │  Execute Body
 │     ▼
 │   i++
 │     │
 └─────┘
       ▼
      No
       │
       ▼
      End
```

------------------------------------------------------------------------

# The `while` Loop

Use `while` when you don't know in advance how many iterations are
needed.

``` c
int count = 1;

while (count <= 5)
{
    printf("%d\n", count);
    count++;
}
```

The condition is checked **before** the body executes.

------------------------------------------------------------------------

# The `do...while` Loop

A `do...while` loop executes the body **at least once**.

``` c
int choice;

do
{
    printf("Enter a number: ");
    scanf("%d", &choice);

} while (choice != 0);
```

Execution order:

``` text
Execute Body
      │
      ▼
Check Condition
 ├── True → Repeat
 └── False → End
```

------------------------------------------------------------------------

# `for` vs `while` vs `do...while`

  Loop           Best Used When
  -------------- ----------------------------------
  `for`          Number of iterations is known
  `while`        Repeat until a condition changes
  `do...while`   Must execute at least once

------------------------------------------------------------------------

# Nested Loops

A loop can exist inside another loop.

``` c
for (int i = 1; i <= 3; i++)
{
    for (int j = 1; j <= 3; j++)
    {
        printf("(%d,%d)\n", i, j);
    }
}
```

Useful for:

-   Tables
-   Matrices
-   Patterns
-   Games

------------------------------------------------------------------------

# Infinite Loops

If the condition never becomes false:

``` c
while (1)
{
    printf("Running...\n");
}
```

The loop never ends unless interrupted.

Servers and operating systems often use controlled infinite loops.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting the Update

``` c
int i = 1;

while (i <= 5)
{
    printf("%d\n", i);
}
```

`i` never changes.

Result:

Infinite loop.

------------------------------------------------------------------------

## Wrong Condition

``` c
for (int i = 0; i < 5; i++)
```

prints:

``` text
0
1
2
3
4
```

Not `5`.

Understand the difference between `<` and `<=`.

------------------------------------------------------------------------

# Hands-on Labs

1.  Print numbers from 1 to 100 using a `for` loop.
2.  Print even numbers using a `while` loop.
3.  Create a menu using `do...while`.
4.  Print a multiplication table with nested loops.
5.  Create an intentional infinite loop and stop it using **Ctrl+C**.

------------------------------------------------------------------------

# Interview Questions

### When should you use a `for` loop?

When the number of iterations is known beforehand.

### When should you use a `while` loop?

When repetition depends on a changing condition.

### Why use `do...while`?

Because it guarantees that the loop body executes at least once.

### What causes an infinite loop?

A condition that never becomes false or forgetting to update loop
variables.

### What is a nested loop?

A loop placed inside another loop.

------------------------------------------------------------------------

# Summary

``` text
Loop
 │
 ├── for
 ├── while
 └── do...while
        │
        ▼
Repeat Code
        │
        ▼
Condition Becomes False
        │
        ▼
End
```

Key Takeaways:

-   Loops eliminate repetitive code.
-   `for` is ideal for counted iterations.
-   `while` is ideal for condition-based repetition.
-   `do...while` always runs once.
-   Nested loops solve two-dimensional problems.
-   Infinite loops are useful when intentionally controlled.

------------------------------------------------------------------------

# Next Chapter

## Chapter 11 --- `break`, `continue`, and `goto`

You'll learn how to alter the normal flow of loops and jump between
sections of code.
