# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 13 --- Function Declaration, Definition & Calling Functions

------------------------------------------------------------------------

## Most beginners think:

> "A function is just a reusable block of code."

That's true, but functions are much more important.

Every modern application is built from thousands of functions.

When you click a button in a browser, send a network request, or save a
file, one function calls another until the work is complete.

Functions help us divide large problems into smaller, reusable pieces.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why functions exist
-   Function declaration
-   Function definition
-   Function call
-   Parameters
-   Arguments
-   Return values
-   Function prototypes
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Functions?

Imagine writing the same code 20 times.

``` text
Calculate Tax
Calculate Tax
Calculate Tax
Calculate Tax
```

Instead, write it once.

``` text
calculateTax()
```

Reuse it whenever needed.

Functions improve:

-   Readability
-   Reusability
-   Testing
-   Maintenance

------------------------------------------------------------------------

# What is a Function?

A function is a named block of code that performs one specific task.

``` text
Program
 │
 ├── login()
 ├── calculateTax()
 ├── sendEmail()
 └── printReport()
```

Each function has a single responsibility.

------------------------------------------------------------------------

# Function Declaration

A declaration tells the compiler that a function exists.

``` c
int add(int a, int b);
```

This is also called a **function prototype**.

It specifies:

-   Function name
-   Return type
-   Parameter types

but not the implementation.

------------------------------------------------------------------------

# Function Definition

The definition contains the actual code.

``` c
int add(int a, int b)
{
    return a + b;
}
```

------------------------------------------------------------------------

# Calling a Function

``` c
int result = add(10, 20);

printf("%d\n", result);
```

Execution flow:

``` text
main()
   │
   ▼
add(10,20)
   │
   ▼
Return 30
   │
   ▼
main() continues
```

------------------------------------------------------------------------

# Parameters vs Arguments

Parameters belong to the function definition.

``` c
int add(int a, int b)
```

`a` and `b` are parameters.

Arguments are the actual values supplied.

``` c
add(10, 20);
```

`10` and `20` are arguments.

------------------------------------------------------------------------

# Return Value

A function may return data.

``` c
int square(int x)
{
    return x * x;
}
```

Example:

``` c
int ans = square(5);
```

Result:

``` text
25
```

Functions that return nothing use:

``` c
void greet(void)
{
    printf("Hello\n");
}
```

------------------------------------------------------------------------

# Program Flow

``` text
main()
 │
 ▼
login()
 │
 ▼
validateUser()
 │
 ▼
databaseQuery()
 │
 ▼
Return
 │
 ▼
main()
```

Programs spend most of their time moving between functions.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting a Prototype

Calling a function before the compiler knows about it causes errors.

Declare it first or place the definition above `main()`.

------------------------------------------------------------------------

## Wrong Return Type

Wrong:

``` c
int greet()
{
    printf("Hello");
}
```

If nothing is returned, use:

``` c
void greet(void)
{
    printf("Hello");
}
```

------------------------------------------------------------------------

## Wrong Number of Arguments

``` c
add(10);
```

If the function expects two arguments, the compiler reports an error.

------------------------------------------------------------------------

# Hands-on Labs

1.  Write a function that adds two integers.
2.  Write a function that returns the larger of two numbers.
3.  Write a `void` function that prints your name.
4.  Split a small program into multiple functions.
5.  Add prototypes before `main()` and compare with defining functions
    first.

------------------------------------------------------------------------

# Interview Questions

### What is a function?

A named block of code that performs a specific task.

### What is a function prototype?

A declaration that tells the compiler about a function before it is
used.

### What is the difference between parameters and arguments?

Parameters are variables in the function definition.

Arguments are the actual values passed during a function call.

### What is the purpose of `void`?

It indicates that a function returns no value (or takes no parameters
when written as `void` in the parameter list).

### Why do large applications use many small functions?

They improve readability, reuse, testing, and maintenance.

------------------------------------------------------------------------

# Summary

``` text
Caller
 │
 ▼
Function Call
 │
 ▼
Parameters
 │
 ▼
Function Body
 │
 ▼
Return Value
 │
 ▼
Caller Continues
```

Key Takeaways:

-   Functions divide programs into manageable units.
-   Declarations tell the compiler a function exists.
-   Definitions contain the implementation.
-   Arguments are copied into parameters.
-   Return values send results back to the caller.

------------------------------------------------------------------------

# Next Chapter

## Chapter 14 --- Call Stack

You'll learn how function calls are managed internally, how the
operating system tracks active functions, and why recursive calls build
up on the stack.
