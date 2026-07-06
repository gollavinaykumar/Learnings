# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 16 --- Pass by Value

------------------------------------------------------------------------

## Most beginners think:

> "When I pass a variable to a function, the function receives the
> original variable."

That is **not** how C works.

C uses **pass by value**.

The function receives a **copy** of the value, not the original
variable.

This is one of the most important concepts in C because it explains why
changing a parameter usually does **not** change the caller's variable.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What pass by value is
-   How arguments are copied
-   Parameters vs arguments
-   Why changes inside a function don't affect the caller
-   Memory view of function calls
-   Common mistakes
-   Why pointers are used to simulate pass by reference

------------------------------------------------------------------------

# What is Pass by Value?

When you call a function, C copies each argument into the function's
parameters.

``` c
void show(int x)
{
    printf("%d\n", x);
}

int main(void)
{
    int n = 10;
    show(n);
}
```

`x` is **not** the same variable as `n`.

It is a copy.

------------------------------------------------------------------------

# Memory View

Before the call:

``` text
main()

n = 10
```

After calling `show(n)`:

``` text
main()              show()

n = 10        --->  x = 10
                  (copy)
```

Both variables have the same value but live in different memory
locations.

------------------------------------------------------------------------

# Changing the Parameter

``` c
void increase(int x)
{
    x++;
}

int main(void)
{
    int n = 10;

    increase(n);

    printf("%d\n", n);
}
```

Output:

``` text
10
```

Why?

Because only the copy changed.

------------------------------------------------------------------------

# Visualizing the Stack

``` text
Call increase(n)

┌──────────────────┐
│ x = 10           │
└──────────────────┘

main()

┌──────────────────┐
│ n = 10           │
└──────────────────┘
```

`x` and `n` are stored in different stack frames.

------------------------------------------------------------------------

# Parameters vs Arguments

Function definition:

``` c
void add(int a, int b)
```

`a` and `b` are **parameters**.

Function call:

``` c
add(5, 8);
```

`5` and `8` are **arguments**.

The arguments are copied into the parameters.

------------------------------------------------------------------------

# Why C Uses Pass by Value

Advantages:

-   Prevents accidental modification of the caller's data.
-   Keeps function behavior predictable.
-   Simpler implementation for the compiler and CPU.

------------------------------------------------------------------------

# When Is This a Problem?

Sometimes you want a function to modify the caller's variable.

Example:

``` c
void reset(int x)
{
    x = 0;
}
```

Calling:

``` c
int score = 100;
reset(score);
```

`score` remains:

``` text
100
```

The function changed only its local copy.

------------------------------------------------------------------------

# Simulating Pass by Reference

To modify the original variable, pass its **address**.

``` c
void reset(int *x)
{
    *x = 0;
}
```

Call:

``` c
reset(&score);
```

This is covered in the next chapter.

------------------------------------------------------------------------

# Common Mistakes

## Expecting the Original Variable to Change

``` c
void set(int x)
{
    x = 99;
}
```

The caller's variable is unchanged.

------------------------------------------------------------------------

## Confusing Copies with References

Remember:

``` text
Pass by Value

Copy

≠

Original Variable
```

------------------------------------------------------------------------

# Hands-on Labs

1.  Write a function that doubles a number. Observe that the original
    variable does not change.
2.  Print the addresses of a variable in `main()` and its parameter
    inside the function.
3.  Modify the parameter and compare both values.
4.  Predict outputs before running each program.

------------------------------------------------------------------------

# Interview Questions

### What is pass by value?

Passing a copy of the argument into the function.

### Does C support pass by reference?

No. C supports pass by value. Pass-by-reference behavior is simulated
using pointers.

### Why doesn't changing a parameter modify the caller's variable?

Because the parameter is a separate copy stored in the function's stack
frame.

### What is copied during a function call?

The values of the arguments.

------------------------------------------------------------------------

# Summary

``` text
Caller
 │
 │ value = 10
 ▼
Function Call
 │
 ▼
Copy Value
 │
 ▼
Parameter = 10
 │
 ▼
Modify Copy
 │
 ▼
Caller Unchanged
```

Key Takeaways:

-   C always passes arguments by value.
-   Parameters are copies of arguments.
-   Changing a parameter does not change the caller's variable.
-   Pointers are used to simulate pass-by-reference behavior.

------------------------------------------------------------------------

# Next Chapter

## Chapter 17 --- Simulating Pass by Reference

You'll learn how pointers allow functions to modify variables owned by
the caller and why this pattern is used throughout operating systems and
the Linux kernel.
