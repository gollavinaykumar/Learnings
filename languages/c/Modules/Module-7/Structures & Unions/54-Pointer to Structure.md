# 👑 C Mastery Roadmap

# Module 7 — Structures & Unions

# Chapter 54 — Pointer to Structure

---

# Most beginners think:

> "If I already have a structure, why do I need a pointer to it?"

This is one of the biggest turning points in learning C.

Small programs can work with structure variables directly.

Large software systems cannot.

Almost every major C project uses **pointers to structures** instead of structure variables.

Examples:

- Linux Kernel
- Redis
- PostgreSQL
- Nginx
- SQLite
- Docker
- Device Drivers
- Networking Libraries

After this chapter, you'll understand why almost every Linux kernel function receives **structure pointers**, not structures.

---

# Learning Objectives

After this chapter you will understand:

- What a pointer to a structure is
- Why structure pointers exist
- Accessing members using pointers
- The `->` operator
- Passing structures to functions
- Dynamic allocation of structures
- Memory layout
- Real-world uses

---

# Why Do We Need Structure Pointers?

Suppose you have:

```c
struct Student
{
    char name[50];
    int age;
    float marks;
};
```

Creating one student is easy.

```
Student

↓

Name

Age

Marks
```

But what if:

- 1 million students
- Thousands of processes
- Millions of network packets

Copying complete structures repeatedly becomes expensive.

Instead,

we pass only their **address**.

---

# What is a Pointer to a Structure?

Exactly like an integer pointer.

Instead of pointing to an integer,

it points to an entire structure.

```c
struct Student
{
    char name[30];
    int age;
};

struct Student s1;

struct Student *ptr = &s1;
```

Memory:

```
ptr

↓

Student

↓

Name

Age
```

---

# Accessing Members

Without a pointer:

```c
s1.age = 21;
```

Using a pointer:

```c
(*ptr).age = 21;
```

Notice:

```
Pointer

↓

Structure

↓

Member
```

The pointer must first be dereferenced.

---

# The Arrow Operator (`->`)

Writing:

```c
(*ptr).age
```

again and again becomes annoying.

C provides a shortcut.

Instead of:

```c
(*ptr).age
```

write:

```c
ptr->age
```

Both mean exactly the same thing.

---

# Comparison

Normal structure:

```c
student.age
```

Pointer:

```c
ptr->age
```

Remember:

| Object | Operator |
|---------|----------|
| Structure Variable | `.` |
| Structure Pointer | `->` |

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
    struct Student s1;

    struct Student *ptr = &s1;

    strcpy(ptr->name,"Vinay");

    ptr->age = 21;

    printf("%s\n",ptr->name);

    printf("%d\n",ptr->age);

    return 0;
}
```

Output

```
Vinay

21
```

---

# Memory Layout

```
Stack

+----------------+

ptr

0x1000

+----------------+

↓

+-------------------------+

Student

Name

Age

+-------------------------+
```

The pointer only stores the address.

The structure stores the actual data.

---

# Passing Structures to Functions

Suppose:

```c
void printStudent(struct Student s)
{
    ...
}
```

Every function call copies the **entire structure**.

Large structures may contain:

```
500 Bytes

1000 Bytes

5000 Bytes
```

Copying becomes expensive.

Better:

```c
void printStudent(struct Student *s)
{
    printf("%d",s->age);
}
```

Only one pointer is copied.

Usually:

```
8 Bytes
```

on a 64-bit machine.

Much faster.

---

# Modifying Structures

Passing by value:

```c
void update(struct Student s)
{
    s.age = 30;
}
```

Original object:

```
Unchanged
```

Because only a copy was modified.

Passing a pointer:

```c
void update(struct Student *s)
{
    s->age = 30;
}
```

Now the original structure changes.

---

# Dynamic Structures

Pointers become even more useful with heap memory.

```c
struct Student *s =
malloc(sizeof(struct Student));
```

Memory:

```
Pointer

↓

Heap

↓

Student
```

Now the structure can live beyond the current function.

Don't forget:

```c
free(s);
```

---

# Multiple Structure Pointers

```
ptr1

↓

Student A

----------------

ptr2

↓

Student B

----------------

ptr3

↓

Student C
```

Pointers make it easy to manage many objects.

---

# Real-World Example — Linux Kernel

A Linux process is represented by:

```
task_struct
```

Kernel functions almost always receive:

```c
struct task_struct *task
```

instead of:

```c
struct task_struct task
```

Why?

Because copying an entire process structure would be extremely expensive.

---

# Real-World Example — Linked List

Each node stores a pointer.

```
Node

↓

Next Node

↓

Next Node

↓

NULL
```

Every node is accessed through structure pointers.

Without structure pointers,

linked lists couldn't exist.

---

# Real-World Example — Device Driver

A device structure may contain:

```
Device

↓

Configuration

↓

Status

↓

Driver Operations
```

Every subsystem manipulates devices using pointers.

---

# Advantages

- Faster function calls
- Less memory copying
- Dynamic allocation
- Shared access
- Essential for data structures

---

# Common Mistakes

---

## Using `.` Instead of `->`

Wrong

```c
ptr.age
```

Correct

```c
ptr->age
```

---

## Forgetting to Dereference

Wrong

```c
*ptr.age
```

Correct

```c
(*ptr).age
```

Or simply:

```c
ptr->age
```

---

## Forgetting Memory Allocation

Wrong

```c
struct Student *ptr;

ptr->age = 20;
```

`ptr` doesn't point anywhere.

Always assign a valid address first.

---

# Hands-on Labs

## Lab 1

Create a structure.

Create a pointer to it.

Access members using:

```c
->
```

---

## Lab 2

Pass a structure pointer to a function.

Modify one member.

Observe the original structure changes.

---

## Lab 3

Allocate a structure using:

```c
malloc()
```

Store values.

Release memory using:

```c
free()
```

---

## Lab 4

Print:

```c
sizeof(struct Student)

sizeof(struct Student *)
```

Observe the difference.

---

## Lab 5

Build a linked list node.

Store:

```
Data

↓

Next Pointer
```

This prepares you for linked lists.

---

# Interview Questions

### What is a pointer to a structure?

A pointer that stores the address of a structure.

---

### Why use structure pointers?

To avoid copying large structures and to allow functions to modify the original object.

---

### What is the difference between `.` and `->`?

`.` accesses members through a structure variable.

`->` accesses members through a structure pointer.

---

### Which is equivalent?

```c
(*ptr).age
```

and

```c
ptr->age
```

They are exactly the same.

---

### Why do operating systems use structure pointers?

Because kernel objects are often large and shared across many functions.

Copying them repeatedly would waste time and memory.

---

# Summary

```
Structure

↓

Pointer

↓

Address

↓

Member Access

↓

->
```

Example:

```
ptr

↓

Student

↓

Name

Age

Marks
```

## Key Takeaways

- Structure pointers store the address of structures.
- `->` is shorthand for `(*pointer).member`.
- Passing pointers avoids expensive structure copies.
- Structure pointers allow functions to modify original objects.
- Dynamic allocation of structures requires pointers.
- Almost every operating system, database, and networking library relies heavily on structure pointers.

---

# Next Chapter

## Chapter 55 — Union

You'll learn:

- What a union is
- Union vs Structure
- Shared memory
- Memory layout
- Tagged unions
- Real-world uses in operating systems, embedded systems, and network protocols