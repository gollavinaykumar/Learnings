# 👑 C Mastery Roadmap

# Module 7 — Structures & Unions

# Chapter 51 — Structures (`struct`)

---

# Most beginners think:

> "A structure is just multiple variables grouped together."

That's only half the story.

A structure is much more than that.

Structures are the **foundation of almost every software system**.

Everything around you uses structures:

- Linux Kernel
- PostgreSQL
- Redis
- Docker
- Kubernetes
- Browsers
- Databases
- Device Drivers
- Network Packets

Almost every object in C is represented using a structure.

After this chapter, you'll start seeing the entire Linux kernel as millions of interconnected structures.

---

# Learning Objectives

After this chapter you will understand:

- What a structure is
- Why structures exist
- How memory is organized
- Structure declaration
- Structure initialization
- Accessing members
- Structure assignment
- Memory layout
- Structure padding (introduction)
- Real-world uses

---

# Why Do We Need Structures?

Suppose you're building a Student Management System.

Without structures:

```c
char name[50];
int age;
float marks;
```

Now imagine storing **1000 students**.

You would need:

```text
name1
age1
marks1

name2
age2
marks2

name3
age3
marks3

...
```

Very difficult to manage.

Instead,

we group related information together.

That's exactly what a structure does.

---

# What is a Structure?

A structure is a **user-defined data type** that groups multiple variables into one object.

Think of it like a person's ID card.

```
Student

↓

Name

Age

Marks

Roll Number
```

Instead of storing each field separately,

everything belongs to one Student object.

---

# Declaring a Structure

```c
struct Student
{
    char name[50];
    int age;
    float marks;
};
```

This only creates a **blueprint**.

No memory has been allocated yet.

Think of it like an architectural drawing of a house.

```
Blueprint

↓

No House Yet
```

---

# Creating a Structure Variable

```c
struct Student s1;
```

Now memory is allocated.

```
Stack

↓

s1
```

Memory layout:

```
+--------------------+
| name[50]           |
+--------------------+
| age                |
+--------------------+
| marks              |
+--------------------+
```

---

# Accessing Members

Use the dot (`.`) operator.

```c
s1.age = 21;

s1.marks = 92.5;
```

Reading values:

```c
printf("%d\n", s1.age);

printf("%.2f\n", s1.marks);
```

---

# Complete Example

```c
#include <stdio.h>

struct Student
{
    char name[50];
    int age;
    float marks;
};

int main()
{
    struct Student s1;

    s1.age = 21;
    s1.marks = 95.6;

    printf("Age : %d\n", s1.age);
    printf("Marks : %.2f\n", s1.marks);

    return 0;
}
```

Output

```
Age : 21

Marks : 95.60
```

---

# Initializing a Structure

Instead of assigning one member at a time,

you can initialize it directly.

```c
struct Student s1 =
{
    "Vinay",
    21,
    95.5
};
```

Memory

```
Student

↓

Name = Vinay

Age = 21

Marks = 95.5
```

---

# Structure Assignment

Unlike arrays,

structures can be copied directly.

```c
struct Student s2;

s2 = s1;
```

Now

```
s1

↓

Copied

↓

s2
```

Every member is copied automatically.

---

# Memory Layout

Suppose

```c
struct Student
{
    char grade;
    int age;
};
```

Many beginners think

```
1 + 4

=

5 Bytes
```

But actually

```
sizeof(struct Student)

↓

8 Bytes
```

Why?

Because of **padding**.

We'll study padding in detail later.

---

# Structures in Memory

```
+--------+
| grade  |
+--------+
|Padding |
+--------+
| age    |
+--------+
```

The compiler inserts extra bytes for alignment.

---

# Why Structures Are Powerful

Imagine a company.

Without structures:

```
Employee Name

Employee Age

Employee Salary

Employee Department
```

Everything is separate.

With structures:

```
Employee

↓

Name

Age

Salary

Department
```

Everything becomes one logical object.

---

# Real-World Example

Linux represents a process using a structure.

Conceptually:

```c
struct task_struct
{
    int pid;

    char name[16];

    int state;

    ...
};
```

Every running process has its own structure.

Similarly,

a network socket is represented by a structure.

A file is represented by a structure.

A device driver is represented by structures.

The Linux kernel is essentially millions of interconnected structures.

---

# Common Mistakes

---

## Forgetting `struct`

Wrong

```c
Student s1;
```

Unless `typedef` is used,

correct syntax is

```c
struct Student s1;
```

---

## Assuming Members Are Stored Separately

They are stored together in one contiguous block of memory.

---

## Confusing Structure Definition with Variable Creation

```c
struct Student
{
    int age;
};
```

This defines a type.

It does **not** create a variable.

---

# Hands-on Labs

## Lab 1

Create a Student structure.

Store:

- Name
- Age
- Marks

Print all values.

---

## Lab 2

Create two Student objects.

Copy one into another.

Observe that all fields are copied.

---

## Lab 3

Print:

```c
sizeof(struct Student);
```

Compare it with the sum of member sizes.

---

## Lab 4

Print the addresses of every member.

```c
printf("%p\n", (void *)&s1.age);
```

Observe the memory layout.

---

## Lab 5

Create structures for:

- Book
- Employee
- Car
- Product

Practice grouping related data.

---

# Interview Questions

### What is a structure?

A structure is a user-defined data type that groups related variables into one object.

---

### Why are structures used?

To represent real-world entities by combining related data into a single unit.

---

### Can structures contain different data types?

Yes.

Example:

- `int`
- `char`
- `float`
- arrays
- pointers
- other structures

---

### Can one structure be assigned to another?

Yes.

Structure assignment copies all members.

---

### Where are structure variables stored?

It depends on where they are declared.

- Local structure → Stack
- Global structure → Global Memory
- Dynamically allocated structure → Heap

---

# Summary

```
Structure Definition

↓

Blueprint

↓

Create Variable

↓

Memory Allocated

↓

Access Members

↓

Use as One Object
```

## Key Takeaways

- A structure groups related data into one object.
- Structures are user-defined data types.
- The structure definition is only a blueprint.
- Memory is allocated when structure variables are created.
- Members are accessed using the dot (`.`) operator.
- Structures can be copied directly.
- Structures are the foundation of operating systems, databases, networking, and the Linux kernel.

---

# Next Chapter

## Chapter 52 — Nested Structures

You'll learn:

- Structures inside structures
- Real-world object modeling
- Memory layout of nested structures
- How operating systems model complex objects
- Why nested structures are used throughout the Linux kernel