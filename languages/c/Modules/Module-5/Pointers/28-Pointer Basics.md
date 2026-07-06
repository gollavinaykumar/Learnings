# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 28 --- Pointer Basics

------------------------------------------------------------------------

## Most beginners think:

> "A pointer stores a value."

It doesn't.

A pointer stores the **memory address** of another object.

Think of a house.

The house contains your belongings.

The address tells people where the house is.

A pointer is like the house address.

The variable is the house.

Without pointers, C could not implement dynamic memory, linked lists,
operating systems, device drivers, or network buffers.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a pointer is
-   Why pointers exist
-   Pointer declaration
-   Pointer initialization
-   Address-of operator (`&`)
-   Dereference operator (`*`)
-   Memory diagrams
-   NULL pointers (preview)
-   Common mistakes

------------------------------------------------------------------------

# Why Do Pointers Exist?

Suppose we have:

``` c
int age = 25;
```

Memory:

``` text
Address      Value

0x1000  ---> 25
```

How can another variable refer to this memory location?

Using a pointer.

------------------------------------------------------------------------

# What is a Pointer?

A pointer is a variable that stores a memory address.

``` c
int *p;
```

Read it as:

> "`p` is a pointer to an integer."

It does **not** store an integer.

It stores the address of an integer.

------------------------------------------------------------------------

# Declaring a Pointer

``` c
int age = 25;

int *p = &age;
```

``` text
age

Address: 0x1000

Value: 25

      ▲
      │

p

Value: 0x1000
```

`p` stores the address of `age`.

------------------------------------------------------------------------

# The Address-of Operator (`&`)

`&` returns the address of a variable.

``` c
int age = 25;

printf("%p\n",(void*)&age);
```

Result:

``` text
0x1000
```

Conceptually.

------------------------------------------------------------------------

# The Dereference Operator (`*`)

`*` means:

> "Go to this address and access the value stored there."

``` c
printf("%d\n", *p);
```

Execution:

``` text
p

↓

0x1000

↓

25
```

Output:

``` text
25
```

------------------------------------------------------------------------

# Visualizing a Pointer

``` text
Stack Memory

┌──────────────┐
│ age = 25     │
│ Address 1000 │
└──────────────┘
        ▲
        │
        │
┌──────────────┐
│ p = 1000     │
└──────────────┘
```

The pointer points to the variable.

------------------------------------------------------------------------

# Changing Data Through a Pointer

``` c
int age = 25;

int *p = &age;

*p = 40;
```

Memory:

Before:

``` text
age = 25
```

After:

``` text
age = 40
```

Changing `*p` changes the original variable.

------------------------------------------------------------------------

# Pointer and Variable Stay Connected

``` c
int score = 100;

int *p = &score;

score = 200;
```

Now:

``` text
*p

↓

200
```

Because the pointer still points to the same memory.

------------------------------------------------------------------------

# Pointer Size

``` c
printf("%zu\n", sizeof(int*));
printf("%zu\n", sizeof(char*));
printf("%zu\n", sizeof(double*));
```

On most modern 64-bit systems:

``` text
8
8
8
```

Notice:

Pointer size usually depends on the **architecture**, not on the type it
points to.

------------------------------------------------------------------------

# NULL Pointer (Preview)

A pointer that points nowhere.

``` c
int *p = NULL;
```

``` text
p

↓

NULL
```

Never dereference a NULL pointer.

We'll study this in detail later.

------------------------------------------------------------------------

# Common Mistakes

## Forgetting Initialization

Wrong:

``` c
int *p;

printf("%d", *p);
```

`p` contains an unknown address.

Dereferencing it causes undefined behavior.

------------------------------------------------------------------------

## Confusing Address and Value

``` c
printf("%p\n",(void*)p);
```

Prints:

Address stored in the pointer.

``` c
printf("%d\n",*p);
```

Prints:

Value stored at that address.

------------------------------------------------------------------------

## Dereferencing NULL

``` c
int *p = NULL;

*p = 5;
```

Usually crashes the program.

------------------------------------------------------------------------

# Real-World Uses

Pointers are used everywhere:

-   Dynamic memory (`malloc`)
-   Linked lists
-   Trees
-   Linux kernel
-   Device drivers
-   Network packet buffers
-   File systems
-   Database engines

------------------------------------------------------------------------

# Hands-on Labs

1.  Create an integer and a pointer to it.
2.  Print the variable, its address, the pointer value, and `*pointer`.
3.  Modify the variable through the pointer.
4.  Compare pointer sizes for different types.
5.  Observe what happens when the variable changes after the pointer is
    created.

------------------------------------------------------------------------

# Interview Questions

### What is a pointer?

A variable that stores the memory address of another object.

### What does `&` do?

Returns the address of a variable.

### What does `*` do?

Dereferences a pointer and accesses the value stored at the pointed-to
address.

### Does a pointer store data?

No. It stores an address.

### Why are pointers important?

They enable efficient memory management, dynamic allocation, data
structures, operating systems, networking, and low-level programming.

------------------------------------------------------------------------

# Summary

``` text
Variable
   │
   ▼
Memory Address
   ▲
   │
Pointer
   │
Dereference (*)
   │
   ▼
Original Value
```

Key Takeaways:

-   A pointer stores an address, not the actual data.
-   `&` obtains an address.
-   `*` accesses the value at an address.
-   Modifying `*pointer` modifies the original variable.
-   Pointers are the foundation of systems programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 29 --- Pointer Arithmetic

You'll learn why adding `1` to a pointer doesn't add one byte, how
arrays and pointers are connected, and how CPUs navigate memory
efficiently.
