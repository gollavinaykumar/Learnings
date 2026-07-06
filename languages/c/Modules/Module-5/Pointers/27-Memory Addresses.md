# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 27 --- Memory Addresses

------------------------------------------------------------------------

## Most beginners think:

> "Variables store values."

That's only half the story.

A variable stores a value **at a specific memory address**.

The CPU never works with variable names like `age` or `count`.

It only understands **memory addresses**.

Understanding memory addresses is the first step toward mastering
pointers, operating systems, and the Linux kernel.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a memory address is
-   Why addresses exist
-   RAM organization
-   Variables and addresses
-   The address-of operator (`&`)
-   Printing addresses
-   Why addresses matter
-   Common mistakes

------------------------------------------------------------------------

# Why Do Memory Addresses Exist?

Imagine RAM as a huge apartment building.

Each apartment has:

-   A unique apartment number
-   Space to store things

Memory works the same way.

``` text
RAM

Address        Value

0x1000  ---->   25
0x1004  ---->   40
0x1008  ---->   99
```

The CPU finds data using addresses.

------------------------------------------------------------------------

# What is a Memory Address?

A memory address is the location where a value is stored.

``` c
int age = 25;
```

Conceptually:

``` text
Variable

age

↓

Address

0x1000

↓

Value

25
```

`age` is just a name given by the programmer.

The CPU accesses `0x1000`.

------------------------------------------------------------------------

# Variables vs Addresses

``` text
Source Code

age

↓

Compiler

↓

Memory Address

0x1000

↓

CPU
```

Variable names disappear after compilation.

The executable contains addresses and instructions.

------------------------------------------------------------------------

# The Address-of Operator (`&`)

C provides the `&` operator to obtain a variable's address.

``` c
int age = 25;

printf("%p\n", (void *)&age);
```

Example output:

``` text
0x7ffd92f2a8cc
```

Every running program receives its own memory layout, so addresses
change between runs.

------------------------------------------------------------------------

# Why `%p`?

Addresses are not ordinary integers.

Use `%p` to print pointer values.

Always cast to `void *` when printing with `printf()`.

``` c
printf("%p\n", (void *)&age);
```

------------------------------------------------------------------------

# Multiple Variables

``` c
int a = 10;
int b = 20;
int c = 30;
```

Memory might look like:

``` text
Address        Variable    Value

0x1000  ---->  a           10
0x1004  ---->  b           20
0x1008  ---->  c           30
```

The compiler decides the exact layout.

------------------------------------------------------------------------

# Arrays and Addresses

``` c
int numbers[4] = {10,20,30,40};
```

Memory:

``` text
Address      Value

0x2000 ---> 10
0x2004 ---> 20
0x2008 ---> 30
0x200C ---> 40
```

Each element has its own address.

------------------------------------------------------------------------

# Stack Addresses

Local variables are usually stored on the stack.

``` c
int main(void)
{
    int x = 5;
    int y = 8;
}
```

Conceptually:

``` text
Stack

Address        Variable

0x7ffd...      y
0x7ffd...      x
```

The exact order depends on the compiler and architecture.

------------------------------------------------------------------------

# Why Addresses Matter

Without addresses we couldn't build:

-   Pointers
-   Dynamic memory
-   Linked lists
-   Trees
-   Operating systems
-   Device drivers
-   Network buffers

Addresses are the foundation of systems programming.

------------------------------------------------------------------------

# Common Mistakes

## Confusing Value and Address

``` c
printf("%d\n", age);
```

Prints:

``` text
25
```

But:

``` c
printf("%p\n", (void *)&age);
```

Prints the address.

------------------------------------------------------------------------

## Assuming Addresses Never Change

Modern operating systems use techniques such as Address Space Layout
Randomization (ASLR), so addresses often differ each time a program
runs.

------------------------------------------------------------------------

# Hands-on Labs

1.  Declare three variables and print their values and addresses.
2.  Print the address of every element in an integer array.
3.  Compare the addresses of `char` and `int` variables.
4.  Observe how addresses change after restarting the program.

------------------------------------------------------------------------

# Interview Questions

### What is a memory address?

The location in RAM where a value is stored.

### Which operator returns a variable's address?

The address-of operator `&`.

### Which format specifier prints an address?

`%p`

### Why do operating systems use virtual addresses?

To isolate processes and simplify memory management. (You'll study this
in the Linux Internals module.)

------------------------------------------------------------------------

# Summary

``` text
Variable Name
      │
      ▼
Memory Address
      │
      ▼
Stored Value
      │
      ▼
CPU Access
```

Key Takeaways:

-   Every variable has a memory address.
-   The CPU accesses memory using addresses, not variable names.
-   The `&` operator returns a variable's address.
-   `%p` prints addresses.
-   Understanding memory addresses is the first step toward learning
    pointers.

------------------------------------------------------------------------

# Next Chapter

## Chapter 28 --- Pointer Basics

You'll learn what pointers are, how they store addresses, how to
dereference them, and why pointers are one of the most powerful features
of C.
