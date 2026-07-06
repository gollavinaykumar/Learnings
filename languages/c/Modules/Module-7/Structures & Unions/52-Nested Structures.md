# 👑 C Mastery Roadmap

# Module 7 — Structures & Unions

# Chapter 52 — Nested Structures

---

# Most beginners think:

> "A structure can only store simple variables like `int` or `char`."

That's not true.

A structure can also contain **another structure**.

This is called a **Nested Structure**.

Nested structures allow us to model complex real-world objects naturally.

Everything from the Linux kernel to PostgreSQL, Redis, Docker, and network protocols uses nested structures extensively.

After this chapter, you'll understand how large software systems organize data using structures inside structures.

---

# Learning Objectives

After this chapter you will understand:

- What nested structures are
- Why nested structures exist
- How to declare nested structures
- Accessing nested members
- Memory layout
- Structure composition
- Real-world examples
- Common mistakes

---

# Why Do We Need Nested Structures?

Suppose you're storing information about an employee.

Without nested structures:

```text
Employee

↓

Name

Age

Street

City

State

Pincode
```

Everything is mixed together.

But notice something.

The last four fields belong to **Address**.

So instead of treating them as independent fields, we can group them.

```
Employee

↓

Name

Age

Address

↓

Street

City

State

Pincode
```

This makes the design cleaner and easier to understand.

---

# What is a Nested Structure?

A nested structure is simply **a structure that contains another structure as one of its members**.

Think of it like folders.

```
Employee Folder

↓

Address Folder

↓

Street

City

State
```

One object contains another object.

---

# Declaring Nested Structures

```c
struct Address
{
    char city[30];
    char state[30];
};

struct Student
{
    char name[50];
    int age;

    struct Address address;
};
```

Notice:

`Student` contains an `Address`.

---

# Creating an Object

```c
struct Student s1;
```

Memory:

```
Student

↓

Name

Age

Address

↓

City

State
```

Everything is stored in one contiguous block of memory.

---

# Accessing Nested Members

Use multiple dot (`.`) operators.

```c
s1.address.city
```

Example:

```c
strcpy(s1.address.city, "Hyderabad");

strcpy(s1.address.state, "Telangana");
```

Reading values:

```c
printf("%s\n", s1.address.city);
```

---

# Complete Example

```c
#include <stdio.h>
#include <string.h>

struct Address
{
    char city[30];
    char state[30];
};

struct Student
{
    char name[50];
    int age;
    struct Address address;
};

int main()
{
    struct Student s1;

    strcpy(s1.name, "Vinay");
    s1.age = 21;

    strcpy(s1.address.city, "Hyderabad");
    strcpy(s1.address.state, "Telangana");

    printf("Name : %s\n", s1.name);
    printf("City : %s\n", s1.address.city);

    return 0;
}
```

Output:

```
Name : Vinay

City : Hyderabad
```

---

# Memory Layout

Memory is still one continuous block.

```
+----------------------+
| Name                 |
+----------------------+
| Age                  |
+----------------------+
| City                 |
+----------------------+
| State                |
+----------------------+
```

The nested structure does **not** allocate separate memory elsewhere.

Its members become part of the parent structure.

---

# Nested Structures Can Be Deeper

Example:

```text
Company

↓

Employee

↓

Address

↓

Country
```

Or in C:

```c
struct Country
{
    char name[30];
};

struct Address
{
    char city[30];
    struct Country country;
};

struct Employee
{
    char name[30];
    struct Address address;
};
```

Access:

```c
employee.address.country.name
```

You simply keep following the objects.

---

# Structure Composition

Nested structures are an example of **composition**.

Instead of saying:

> "Employee **is an** Address"

We say:

> "Employee **has an** Address"

This is one of the most common design principles in software engineering.

---

# Real-World Example — Linux Kernel

A Linux process (`task_struct`) contains many other structures.

Conceptually:

```text
task_struct

↓

Memory Info

↓

File Info

↓

Signal Info

↓

Scheduler Info
```

Each of these is another structure.

The kernel builds large objects by combining many smaller structures.

---

# Real-World Example — Network Packet

Imagine a network packet.

```
Packet

↓

Ethernet Header

↓

IP Header

↓

TCP Header

↓

Payload
```

Each header is naturally represented as its own structure.

The packet structure simply contains them.

---

# Real-World Example — Database

A database record might look like:

```
Customer

↓

Personal Info

↓

Address

↓

Bank Details
```

Each section becomes its own structure.

---

# Advantages of Nested Structures

- Better organization
- Reusable components
- Easier maintenance
- Cleaner code
- Matches real-world objects

---

# Common Mistakes

---

## Forgetting the Parent Structure

Wrong:

```c
city
```

Correct:

```c
student.address.city
```

---

## Duplicating Data

Instead of writing:

```text
Student

City

State

Employee

City

State
```

Create one reusable `Address` structure.

---

## Thinking Nested Structures Are Separate Objects

They are part of the parent object's memory.

They are not allocated independently unless you use pointers.

---

# Hands-on Labs

## Lab 1

Create:

```text
Student

↓

Address
```

Store and print all fields.

---

## Lab 2

Create:

```text
Employee

↓

Department

↓

Manager
```

Practice accessing deeply nested members.

---

## Lab 3

Print:

```c
sizeof(struct Student);
```

Observe how nested structures affect the total size.

---

## Lab 4

Print addresses of nested members.

```c
printf("%p\n",
(void *)&student.address.city);
```

Observe contiguous memory layout.

---

## Lab 5

Model a university.

```
University

↓

Department

↓

Professor

↓

Address
```

Represent each object using nested structures.

---

# Interview Questions

### What is a nested structure?

A structure that contains another structure as one of its members.

---

### Why are nested structures used?

To model complex objects by grouping related data into reusable components.

---

### How do you access nested members?

Using multiple dot operators.

Example:

```c
student.address.city
```

---

### Are nested structures stored separately in memory?

No.

They are stored as part of the parent structure's memory.

---

### Where are nested structures used?

- Linux Kernel
- Device Drivers
- Network Protocols
- Databases
- Embedded Systems
- Browsers

---

# Summary

```
Structure

↓

Another Structure

↓

Another Structure

↓

Members
```

Example:

```
Student

↓

Address

↓

City

State
```

## Key Takeaways

- A nested structure contains another structure.
- Nested structures model real-world relationships naturally.
- They use composition ("has-a" relationship).
- Members are accessed using multiple dot operators.
- Nested structures occupy one contiguous block of memory.
- They are heavily used in operating systems, networking, databases, and embedded software.

---

# Next Chapter

## Chapter 53 — Arrays of Structures

You'll learn:

- Why arrays of structures are used
- Memory layout of structure arrays
- Iterating through structure arrays
- How databases and operating systems store collections of objects
- Real-world examples like process tables, file tables, and routing tables