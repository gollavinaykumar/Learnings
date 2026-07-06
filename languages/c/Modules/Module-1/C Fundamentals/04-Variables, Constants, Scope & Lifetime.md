# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 4 --- Variables, Constants, Scope & Lifetime

------------------------------------------------------------------------

## Most beginners think:

> "A variable is just a place to store a value."

That is only partially true.

A variable is actually a **named location in memory**.

When you write:

``` c
int age = 25;
```

the compiler reserves memory, stores the value `25`, and remembers that
the memory location is called `age`.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What variables are
-   How variables are stored in memory
-   Constants
-   C keywords
-   Scope
-   Lifetime
-   Local vs Global variables
-   Common mistakes

------------------------------------------------------------------------

# What is a Variable?

A variable is a named memory location.

``` text
Variable Name     Memory Address     Value

age               0x1000             25
```

Instead of remembering `0x1000`, we use the name `age`.

------------------------------------------------------------------------

# Declaring Variables

``` c
int age;
float salary;
char grade;
```

This tells the compiler how much memory to reserve.

------------------------------------------------------------------------

# Initializing Variables

``` c
int age = 25;
```

Declaration + assignment together is called initialization.

------------------------------------------------------------------------

# Common Data Types

``` c
int count = 10;
char grade = 'A';
float price = 99.5f;
double pi = 3.1415926535;
```

------------------------------------------------------------------------

# Variables Live in Memory

``` text
RAM

+-------------------+
| age = 25          |
+-------------------+
| count = 10        |
+-------------------+
| grade = 'A'       |
+-------------------+
```

The exact location is chosen by the operating system and compiler.

------------------------------------------------------------------------

# Changing a Variable

``` c
int score = 10;

score = 20;
```

The value changes, but the variable still refers to the same memory
location.

------------------------------------------------------------------------

# Constants

Sometimes values should never change.

``` c
const float PI = 3.14159;
```

Attempting to modify `PI` results in a compiler error.

------------------------------------------------------------------------

# Why Use Constants?

Examples:

-   Pi
-   Tax rate
-   Maximum buffer size
-   Configuration values

Using constants makes programs safer and easier to understand.

------------------------------------------------------------------------

# C Keywords

Keywords are reserved words.

Examples:

``` text
int
char
float
double
if
else
while
for
return
const
static
```

You cannot use keywords as variable names.

------------------------------------------------------------------------

# Scope

Scope answers:

> "Where can this variable be accessed?"

## Local Scope

``` c
int main()
{
    int x = 5;
}
```

`x` exists only inside `main()`.

------------------------------------------------------------------------

## Global Scope

``` c
int count = 0;

int main()
{
    count++;
}
```

Global variables can be accessed by multiple functions in the same
source file.

------------------------------------------------------------------------

# Lifetime

Lifetime answers:

> "How long does the variable exist?"

Local variable:

``` text
main()
 │
 ▼
Created

...

Destroyed
```

Global variable:

``` text
Program Starts
      │
      ▼
Created
      │
      ▼
Program Ends
```

------------------------------------------------------------------------

# Local vs Global

  Local                Global
  -------------------- ------------------
  Inside function      Outside function
  Short lifetime       Entire program
  Limited visibility   Wide visibility

------------------------------------------------------------------------

# Common Mistakes

## Using an Uninitialized Variable

``` c
int x;

printf("%d", x);
```

The value is undefined.

Always initialize variables before using them.

------------------------------------------------------------------------

## Modifying Constants

``` c
const int MAX = 100;

MAX = 50;
```

Compilation error.

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Create variables of different types.

Print them using `printf()`.

------------------------------------------------------------------------

## Lab 2

Create a global variable.

Access it from multiple functions.

------------------------------------------------------------------------

## Lab 3

Declare a local variable.

Try accessing it outside the function.

Observe the compiler error.

------------------------------------------------------------------------

# Interview Questions

### What is a variable?

A named memory location used to store data.

### What is the difference between declaration and initialization?

Declaration creates the variable.

Initialization gives it an initial value.

### What is scope?

The region of code where a variable can be accessed.

### What is lifetime?

The duration for which a variable exists in memory.

### Why use constants?

To prevent accidental modification and improve readability.

------------------------------------------------------------------------

# Summary

``` text
Variable
    │
    ▼
Memory Location
    │
    ▼
Scope
    │
    ▼
Lifetime
    │
    ▼
Program Uses Data
```

Key Takeaways:

-   Variables are named memory locations.
-   Constants protect values from modification.
-   Scope controls visibility.
-   Lifetime controls how long data exists.
-   Choosing the right variable type is fundamental to writing correct C
    programs.

------------------------------------------------------------------------

# Next Chapter

## Chapter 5 --- Data Types

You'll learn:

-   int
-   char
-   short
-   long
-   float
-   double
-   signed
-   unsigned
-   Size of each type
-   Choosing the correct type
