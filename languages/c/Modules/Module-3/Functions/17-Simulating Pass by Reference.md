# 👑 C Mastery Roadmap

# Module 3 --- Functions

# Chapter 17 --- Simulating Pass by Reference

------------------------------------------------------------------------

## Most beginners think:

> "C supports pass by reference."

Actually, it doesn't.

C always uses **pass by value**.

So how do functions change variables that belong to the caller?

The trick is simple:

Instead of copying the **value**, we copy the **address**.

The function receives the address of the original variable and modifies
the data stored there.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why C has no pass by reference
-   Passing addresses with pointers
-   Dereferencing pointers
-   Modifying the caller's variable
-   Memory diagrams
-   Multiple output values
-   Common mistakes

------------------------------------------------------------------------

# Review: Pass by Value

``` c
void increase(int x)
{
    x++;
}
```

Calling:

``` c
int n = 10;
increase(n);
```

Result:

``` text
n = 10
```

Only the copy changes.

------------------------------------------------------------------------

# Passing an Address

Instead of passing the value:

``` c
increase(n);
```

Pass its address:

``` c
increase(&n);
```

Function:

``` c
void increase(int *x)
{
    (*x)++;
}
```

Now the original variable changes.

------------------------------------------------------------------------

# Memory View

Before call:

``` text
main()

n = 10
Address: 0x1000
```

Function call:

``` text
increase(&n)

x = 0x1000
```

Dereference:

``` text
*x

↓

Memory at 0x1000

↓

10
```

After:

``` text
(*x)++

↓

11
```

`n` is now 11.

------------------------------------------------------------------------

# Complete Example

``` c
#include <stdio.h>

void swap(int *a, int *b)
{
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void)
{
    int x = 5;
    int y = 8;

    swap(&x, &y);

    printf("%d %d\n", x, y);
}
```

Output:

``` text
8 5
```

Without pointers, swapping would not affect the caller.

------------------------------------------------------------------------

# Why Pointers Work

``` text
Caller

x = 5
Address 0x1000

      ▲
      │
Pointer

a = 0x1000

      │
      ▼

*a

↓

5
```

Pointers let a function access another variable's memory.

------------------------------------------------------------------------

# Multiple Output Values

Functions return only one value.

Pointers allow many outputs.

``` c
void calculate(int a, int b, int *sum, int *product)
{
    *sum = a + b;
    *product = a * b;
}
```

Caller:

``` c
int s, p;

calculate(3,4,&s,&p);
```

Now both values are returned through pointers.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting '&'

Wrong:

``` c
swap(x, y);
```

Correct:

``` c
swap(&x, &y);
```

------------------------------------------------------------------------

## Forgetting '\*'

Wrong:

``` c
a = b;
```

Correct:

``` c
*a = *b;
```

------------------------------------------------------------------------

## Dereferencing NULL

``` c
int *p = NULL;

*p = 5;
```

This causes undefined behavior, often a crash.

Always validate pointers before dereferencing when appropriate.

------------------------------------------------------------------------

# Hands-on Labs

1.  Write a function that increments a variable using a pointer.
2.  Implement `swap()` using pointers.
3.  Return both minimum and maximum values through pointer parameters.
4.  Print the address and value before and after modification.

------------------------------------------------------------------------

# Interview Questions

### Does C support pass by reference?

No. C supports only pass by value.

### How do we simulate pass by reference?

By passing the address of a variable and using pointers.

### Why do we use `&`?

To obtain the address of a variable.

### Why do we use `*`?

To access or modify the value stored at an address.

### Why are pointers important?

They allow functions to modify caller-owned data, manage dynamic memory,
and interact with hardware and operating system APIs.

------------------------------------------------------------------------

# Summary

``` text
Caller Variable
      │
      ▼
Address (&)
      │
      ▼
Pointer Parameter
      │
      ▼
Dereference (*)
      │
      ▼
Modify Original Variable
```

Key Takeaways:

-   C always passes arguments by value.
-   Passing an address lets a function access the caller's variable.
-   `&` gets an address.
-   `*` accesses the value stored at that address.
-   Pointer parameters are the foundation of systems programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 18 --- Function Pointers

You'll learn how functions can be stored in variables, passed to other
functions, and used to build callbacks, event systems, and plugin
architectures.
