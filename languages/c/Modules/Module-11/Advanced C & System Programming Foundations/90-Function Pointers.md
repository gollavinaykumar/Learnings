# 👑 C Mastery Roadmap

# Module 11 — Advanced C & System Programming Foundations

# Chapter 90 — Function Pointers

---

# Most beginners think:

> "Functions are different from variables."

Actually...

functions also have **addresses**.

Just like this variable:

```c
int age = 21;
```

has an address,

```
age

↓

0x1000
```

this function also has an address:

```c
void hello()
{
    printf("Hello\n");
}
```

```
hello()

↓

0x401250
```

Because functions have addresses,

we can store those addresses inside variables.

Those variables are called:

```
Function Pointers
```

Function pointers are one of the most powerful features in C.

They make possible:

- Callbacks
- Event systems
- Operating systems
- Linux kernel drivers
- Network libraries
- GUI frameworks
- Plugin systems

---

# Learning Objectives

After this chapter you will understand:

- Function addresses
- Function pointers
- Calling functions through pointers
- Syntax
- Memory representation
- Why function pointers exist
- Real-world applications

---

# Variables Have Addresses

Example:

```c
int x = 10;
```

Memory:

```
Variable

↓

10

Address

↓

0x1000
```

Pointer:

```c
int *p = &x;
```

```
p

↓

0x1000
```

Exactly the same idea applies to functions.

---

# Functions Have Addresses

Example:

```c
void hello()
{
    printf("Hello\n");
}
```

Memory:

```
Code Segment

↓

Machine Instructions

↓

Address

↓

0x401200
```

The function name refers to its entry address in most expressions.

---

# Printing Function Address

Example:

```c
#include <stdio.h>

void hello()
{
    printf("Hello\n");
}

int main()
{
    printf("%p\n",
           (void *)hello);

    return 0;
}
```

Possible output:

```
0x401200
```

(The exact address varies.)

---

# What is a Function Pointer?

A function pointer stores:

```
Function Address
```

Instead of:

```
Integer Address
```

Example:

```c
void (*func)();
```

Meaning:

```
func

↓

Pointer

↓

Function
```

---

# Understanding the Syntax

Many beginners find this confusing:

```c
void (*func)();
```

Let's break it down.

Without parentheses:

```c
void *func();
```

Means:

```
Function

↓

Returns Pointer
```

Not what we want.

With parentheses:

```c
void (*func)();
```

Means:

```
Pointer

↓

Function
```

The parentheses change the meaning.

---

# Assigning a Function

Example:

```c
void hello()
{
    printf("Hello\n");
}

int main()
{
    void (*func)();

    func = hello;
}
```

Memory:

```
func

↓

Address Of hello()
```

---

# Calling Through Pointer

Instead of:

```c
hello();
```

Call:

```c
func();
```

or

```c
(*func)();
```

Both forms are equivalent.

Output:

```
Hello
```

---

# Visualization

```
func

↓

0x401200

↓

hello()

↓

Machine Instructions
```

Calling:

```
func()

↓

Jump

↓

Execute hello()
```

---

# Example

```c
#include <stdio.h>

void hello()
{
    printf("Hello\n");
}

int main()
{
    void (*func)();

    func = hello;

    func();

    return 0;
}
```

Output:

```
Hello
```

---

# Function Parameters

Suppose:

```c
int add(int a,
        int b)
{
    return a + b;
}
```

Function pointer:

```c
int (*operation)(
    int,
    int
);
```

Assignment:

```c
operation = add;
```

Call:

```c
int result =
operation(10, 20);
```

Output:

```
30
```

---

# Matching Signatures

The pointer type must match the function.

Correct:

```c
int add(int, int);

int (*f)(int, int);

f = add;
```

Wrong:

```c
void (*f)();

f = add;
```

Different function types.

---

# Arrays of Function Pointers

Example:

```c
void one()
{
    printf("One\n");
}

void two()
{
    printf("Two\n");
}

int main()
{
    void (*arr[2])();

    arr[0] = one;
    arr[1] = two;

    arr[0]();
    arr[1]();
}
```

Output:

```
One

Two
```

---

# Memory Layout

```
Stack

↓

Function Pointer

↓

0x401250

-------------------

Code Segment

↓

Function Instructions
```

Notice:

The pointer lives on the stack,

while the function itself lives in the code segment.

---

# Why Not Just Call Functions Directly?

Suppose:

Need:

```
Choose Function

↓

Runtime
```

Impossible with direct calls alone.

Function pointers allow:

```
Store Function

↓

Pass Function

↓

Execute Later
```

---

# Real-World Example — Calculator

User chooses:

```
+

-

*

/
```

Program selects:

```
Corresponding Function
```

Instead of many `if` statements,

it can use a function pointer.

---

# Real-World Example — Linux Kernel

The kernel stores function pointers for:

- Device Drivers
- File Systems
- Network Protocols
- Schedulers

When hardware triggers an event,

the kernel calls the appropriate function through a pointer.

---

# Real-World Example — GUI

Button:

```
Clicked
```

Framework:

```
Call Registered Function
```

The button doesn't know which function to call until runtime.

---

# Real-World Example — Networking

HTTP Server:

```
Request

↓

GET Handler

POST Handler

DELETE Handler
```

Handlers are often stored as function pointers.

---

# Common Mistakes

---

## Wrong Syntax

Wrong:

```c
void *func();
```

This declares:

```
Function

↓

Returns Pointer
```

Not a function pointer.

---

## Wrong Function Signature

Pointer type must exactly match:

- Return type
- Parameter types

---

## Calling Uninitialized Pointer

Wrong:

```c
void (*func)();

func();
```

Undefined behavior.

Initialize the pointer before calling it.

---

## Forgetting Parentheses

Correct declaration:

```c
void (*func)();
```

The parentheses are essential.

---

# Complete Example

```c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main()
{
    int (*operation)(int, int);

    operation = add;

    printf("%d\n",
           operation(5, 7));

    return 0;
}
```

Output:

```
12
```

---

# Hands-on Labs

## Lab 1

Create a function pointer to:

```c
void hello();
```

Call it.

---

## Lab 2

Create function pointers for:

- Add
- Subtract
- Multiply
- Divide

Invoke each through the pointer.

---

## Lab 3

Store multiple function pointers in an array.

Call them one after another.

---

## Lab 4

Print the addresses of several functions.

Observe that functions occupy different locations in the code segment.

---

## Lab 5

Implement a simple calculator using function pointers instead of `if` or `switch`.

---

# Interview Questions

### What is a function pointer?

A pointer that stores the address of a function.

---

### Where is the function stored?

In the program's code (text) segment.

---

### Where is the function pointer variable stored?

Typically on the stack (if it's a local variable), though it can also be global or dynamically allocated.

---

### Why are function pointers useful?

They allow selecting and calling functions dynamically at runtime.

---

### Can functions be passed as arguments?

Functions themselves are not passed by value, but pointers to functions can be passed as arguments.

---

# Summary

```
Function

↓

Address

↓

Function Pointer

↓

Call Through Pointer
```

Visualization:

```
Function Pointer

↓

0x401250

↓

Code Segment

↓

Machine Instructions
```

## Key Takeaways

- Functions have addresses just like variables.
- Function pointers store those addresses.
- Function pointers enable dynamic behavior at runtime.
- The pointer type must match the function signature.
- Arrays of function pointers enable dispatch tables.
- Function pointers are fundamental to callbacks, event systems, operating systems, networking libraries, and plugin architectures.

---

# Next Chapter

## Chapter 91 — Callback Functions

You'll learn:

- What callbacks are
- Why callbacks exist
- Passing functions as arguments
- Event-driven programming
- Sorting with callbacks (`qsort`)
- Linux kernel callbacks
- Real-world callback design patterns