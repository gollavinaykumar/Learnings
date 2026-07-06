# 👑 C Mastery Roadmap

# Module 7 — Structures & Unions

# Chapter 53 — Arrays of Structures

---

# Most beginners think:

> "If I want to store multiple students, I should create `student1`, `student2`, `student3`..."

That works for 3 students.

What about:

- 1,000 students?
- 1,000,000 customers?
- 50,000 processes in Linux?
- 100,000 network connections?

Creating separate variables is impossible.

Instead, we create **an array of structures**.

This is one of the most common data organization techniques in C.

Operating systems, databases, game engines, browsers, and networking software all use arrays of structures.

---

# Learning Objectives

After this chapter you will understand:

- What an array of structures is
- Why arrays of structures exist
- Memory layout
- Accessing elements
- Looping through structures
- Structure arrays vs normal arrays
- Real-world uses

---

# Why Do We Need Arrays of Structures?

Suppose a school has:

```
Student 1

↓

Name

Age

Marks
```

Another student:

```
Student 2

↓

Name

Age

Marks
```

Creating variables like:

```c
student1

student2

student3

...

student1000
```

is not practical.

Instead:

```
Students

↓

Array

↓

Student

Student

Student

Student
```

One variable can represent thousands of students.

---

# What is an Array of Structures?

It is simply an array where **each element is a structure**.

Example:

```c
struct Student
{
    char name[30];
    int age;
    float marks;
};

struct Student students[3];
```

Memory:

```
students

↓

Student 0

Student 1

Student 2
```

Each element is a complete `Student` object.

---

# Memory Layout

Suppose:

```c
sizeof(Student) = 40 Bytes
```

Memory becomes:

```
Address

1000

↓

Student[0]

1040

↓

Student[1]

1080

↓

Student[2]
```

Every structure is stored **contiguously**.

Just like arrays of integers.

---

# Accessing Elements

First choose the array element.

Then choose the member.

Syntax:

```c
students[0].age
```

Example:

```c
students[0].age = 20;

students[1].age = 22;

students[2].age = 19;
```

---

# Complete Example

```c
#include <stdio.h>
#include <string.h>

struct Student
{
    char name[30];
    int age;
};

int main()
{
    struct Student students[2];

    strcpy(students[0].name, "Vinay");
    students[0].age = 21;

    strcpy(students[1].name, "Rahul");
    students[1].age = 22;

    printf("%s %d\n",
           students[0].name,
           students[0].age);

    printf("%s %d\n",
           students[1].name,
           students[1].age);

    return 0;
}
```

Output:

```
Vinay 21

Rahul 22
```

---

# Using Loops

Arrays become powerful with loops.

```c
for(int i = 0; i < 3; i++)
{
    printf("%d\n",
           students[i].age);
}
```

Instead of writing the same code repeatedly,

the loop processes every structure.

---

# Visualization

```
students

↓

+----------------------+
| Student 0            |
+----------------------+
| Student 1            |
+----------------------+
| Student 2            |
+----------------------+
| Student 3            |
+----------------------+
```

Each element contains multiple fields.

---

# Arrays of Structures vs Structure of Arrays

Suppose you have:

### Array of Structures (AoS)

```c
struct Student
{
    int age;
    float marks;
};

struct Student students[3];
```

Memory:

```
Age Marks

Age Marks

Age Marks
```

Everything about one student stays together.

---

### Structure of Arrays (SoA)

```c
struct Data
{
    int age[3];
    float marks[3];
};
```

Memory:

```
Age

Age

Age

Marks

Marks

Marks
```

Both designs are useful.

Most C programs start with **Array of Structures (AoS)** because it matches real-world objects.

High-performance applications sometimes prefer **Structure of Arrays (SoA)** for better cache efficiency.

---

# Real-World Example — Linux Kernel

The Linux kernel maintains many collections of objects.

Conceptually:

```
Process Table

↓

Process

Process

Process

Process
```

Each process is represented by a structure.

Similarly:

```
Open File Table

↓

File

File

File
```

Each file descriptor corresponds to a structure.

---

# Real-World Example — Database

A table:

```
Customers

↓

Customer

Customer

Customer
```

Each customer record can be represented by a structure.

The table is conceptually an array (or collection) of structures.

---

# Real-World Example — Networking

A web server keeps track of active connections.

```
Connections

↓

Socket

Socket

Socket
```

Each socket is represented by a structure.

The server manages many of them.

---

# Advantages

- Easy to manage many objects
- Contiguous memory
- Efficient iteration
- Simple indexing
- Matches real-world data

---

# Common Mistakes

---

## Forgetting the Array Index

Wrong:

```c
students.age
```

Correct:

```c
students[0].age
```

---

## Going Out of Bounds

Wrong:

```c
students[10]
```

when only:

```c
students[5]
```

exists.

This causes undefined behavior.

---

## Confusing Array Size with Structure Size

Remember:

```text
Total Memory

=

Number of Structures

×

Size of One Structure
```

---

# Hands-on Labs

## Lab 1

Create an array of five students.

Store:

- Name
- Age
- Marks

Print all records.

---

## Lab 2

Use a loop to initialize every student.

---

## Lab 3

Print:

```c
sizeof(students)
```

and

```c
sizeof(students[0])
```

Understand the difference.

---

## Lab 4

Print the addresses of each structure.

```c
printf("%p\n",
(void *)&students[i]);
```

Observe contiguous memory.

---

## Lab 5

Create arrays for:

- Employees
- Books
- Cars
- Products

Practice accessing members.

---

# Interview Questions

### What is an array of structures?

An array where each element is a complete structure.

---

### How are elements accessed?

Using:

```c
array[index].member
```

---

### Are structures stored contiguously in an array?

Yes.

Just like arrays of integers.

---

### Why use arrays of structures?

To efficiently manage many objects of the same type.

---

### Where are arrays of structures used?

- Linux Kernel
- Databases
- Browsers
- Networking
- Embedded Systems
- Game Engines

---

# Summary

```
Array

↓

Structure

↓

Members
```

Example:

```
Students

↓

Student 0

↓

Name

Age

Marks

------------------

Student 1

↓

Name

Age

Marks
```

## Key Takeaways

- Arrays of structures store many objects of the same type.
- Every element is a complete structure.
- Structures are stored contiguously in memory.
- Members are accessed using `array[index].member`.
- Arrays and loops work together to process large collections efficiently.
- Arrays of structures are used throughout operating systems, databases, and networking software.

---

# Next Chapter

## Chapter 54 — Pointer to Structure

You'll learn:

- Why pointers to structures are used
- The `->` (arrow) operator
- Passing structures efficiently to functions
- Dynamic allocation of structures
- How linked lists, trees, Linux kernel objects, and device drivers rely on structure pointers