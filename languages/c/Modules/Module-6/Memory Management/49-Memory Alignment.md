# 👑 C Mastery Roadmap

# Module 6 — Memory Management

# Chapter 49 — Memory Alignment

---

## Most beginners think:

> "Memory is just a sequence of bytes, so variables can start anywhere."

Technically, every byte has an address.

But modern CPUs are designed to access memory **much faster** when data is stored at specific addresses.

This is called **Memory Alignment**.

Poor alignment can cause:

- Slower programs
- Extra CPU work
- Hardware exceptions (on some CPUs)
- Larger structures
- Wasted memory

Understanding memory alignment is essential for:

- Operating Systems
- Linux Kernel
- Device Drivers
- Networking
- Embedded Systems
- Database Engines

---

# Learning Objectives

After this chapter you will understand:

- What memory alignment is
- Why CPUs need alignment
- Alignment requirements
- Structure padding
- Packed structures
- Cache performance
- Misaligned access
- Real-world uses

---

# What is Memory Alignment?

Memory alignment means placing data at addresses that are **multiples of its alignment requirement**.

Example:

Suppose an `int` requires **4-byte alignment**.

Valid addresses:

```text
0
4
8
12
16
20
```

Invalid (misaligned) addresses:

```text
1
2
3
5
6
7
```

---

# Why Does the CPU Prefer Alignment?

Imagine reading a book.

Case 1:

One sentence is completely on one page.

Easy.

Case 2:

Half the sentence is on one page.

Half is on the next page.

Now you need two page reads.

Memory works similarly.

Aligned data is usually read in one memory access.

Misaligned data may require multiple accesses.

---

# Example

Suppose an `int` occupies **4 bytes**.

Aligned:

```text
Address

1000
1001
1002
1003

Integer
```

The CPU reads everything at once.

---

Misaligned:

```text
999
1000
1001
1002
```

Now the integer spans alignment boundaries.

Some CPUs handle this slowly.

Some CPUs raise an exception.

---

# Alignment Requirements

Typical alignment rules on many 64-bit systems:

| Data Type | Size | Typical Alignment |
|-----------|------|-------------------|
| `char` | 1 | 1 byte |
| `short` | 2 | 2 bytes |
| `int` | 4 | 4 bytes |
| `float` | 4 | 4 bytes |
| `double` | 8 | 8 bytes |
| `long long` | 8 | 8 bytes |

**Note:** Exact alignment depends on the compiler, ABI, and CPU architecture.

---

# Structure Padding

Consider:

```c
struct Student
{
    char grade;
    int age;
};
```

Many beginners think:

```text
1 + 4 = 5 Bytes
```

But on many systems:

```text
sizeof(Student)

↓

8 Bytes
```

Why?

Padding.

---

# Memory Layout

Without padding:

```text
grade

age
```

Addresses:

```text
0

1

2

3

4
```

`age` begins at address:

```text
1
```

Misaligned.

---

Compiler adds padding:

```text
Byte 0

grade

Byte 1

Padding

Byte 2

Padding

Byte 3

Padding

Byte 4-7

age
```

Now:

```text
age

↓

Address 4
```

Properly aligned.

---

# Visual Diagram

```text
Without Padding

+---+----+----+----+---------+
| g | age begins here         |
+---+----+----+----+---------+

Misaligned ❌

-------------------------------

With Padding

+---+---+---+---+---------+
| g | P | P | P |   age   |
+---+---+---+---+---------+

Aligned ✔
```

---

# Why Padding Exists

Padding improves:

- CPU performance
- Memory access speed
- Cache efficiency

The compiler inserts it automatically.

---

# Reducing Padding

Suppose:

```c
struct Bad
{
    char c;
    int i;
    double d;
};
```

Better ordering:

```c
struct Good
{
    double d;
    int i;
    char c;
};
```

Both structures contain the same fields,

but the second usually wastes less memory.

Professional programmers often reorder members to reduce padding.

---

# Packed Structures

Sometimes exact memory layout is required.

Example:

- Network packets
- Disk formats
- Hardware registers

Compilers provide attributes such as:

```c
#pragma pack(1)
```

or

```c
__attribute__((packed))
```

Now padding is removed.

Example:

```text
grade

age
```

stored back-to-back.

---

# Warning About Packed Structures

Packed structures save memory,

but may reduce performance.

Some CPUs must perform extra work for misaligned accesses.

Others may generate hardware faults.

Use packed structures only when exact byte layout is required.

---

# Cache Performance

Modern CPUs load memory in blocks called **cache lines**.

Example:

```text
64 Bytes
```

Proper alignment helps:

- Fewer cache misses
- Faster execution
- Better throughput

This is extremely important for:

- Databases
- Game engines
- Operating systems

---

# Alignment and Arrays

Suppose:

```c
int numbers[4];
```

Memory:

```text
1000

1004

1008

1012
```

Every element is naturally aligned.

This allows CPUs to read array elements efficiently.

---

# Real-World Uses

Memory alignment matters in:

- Linux Kernel
- PostgreSQL
- Redis
- Device Drivers
- Embedded Systems
- Database Storage Engines
- Network Protocols
- High-performance Servers

---

# Common Mistakes

## Assuming `sizeof()` Equals Sum of Members

Wrong.

Padding increases structure size.

---

## Ignoring Member Order

Poor ordering wastes memory.

---

## Packing Everything

Packed structures are **not** always faster.

Use them only when exact layout is required.

---

# Hands-on Labs

## Lab 1

Create several structures.

Compare:

```c
sizeof(struct)
```

---

## Lab 2

Reorder structure members.

Observe size reduction.

---

## Lab 3

Print member addresses:

```c
printf("%p\n",(void*)&student.age);
```

Observe alignment.

---

## Lab 4

Use:

```c
offsetof()
```

to inspect member offsets.

Example:

```c
#include <stddef.h>

printf("%zu\n", offsetof(struct Student, age));
```

---

## Lab 5

Compare a normal structure with a packed structure.

Observe:

- `sizeof()`
- Memory layout
- Performance (if possible)

---

# Interview Questions

### What is memory alignment?

Placing data at addresses that satisfy the alignment requirements of the CPU and ABI.

---

### Why do CPUs prefer aligned memory?

Because aligned accesses are usually faster and may require fewer memory operations.

---

### What is structure padding?

Extra bytes inserted by the compiler to align members correctly.

---

### Why does `sizeof(struct)` sometimes exceed the sum of its members?

Because of compiler-inserted padding.

---

### Should packed structures always be used?

No.

They reduce padding but can hurt performance and may cause problems on some architectures.

---

# Summary

```text
Variable
     │
     ▼
Alignment Requirement
     │
     ▼
Correct Address
     │
     ▼
Fast CPU Access

-----------------------

Misaligned Data
     │
     ▼
Extra CPU Work
     │
     ▼
Lower Performance
```

Key Takeaways:

- Memory alignment improves CPU efficiency.
- Compilers insert padding automatically.
- `sizeof(struct)` often includes padding.
- Reordering members can reduce wasted memory.
- Packed structures are useful only when exact layouts are required.
- Understanding alignment is essential for systems programming, networking, databases, and the Linux kernel.

---

# Next Chapter

## Chapter 50 — Memory Corruption

You'll learn:

- What memory corruption is
- Buffer overflows
- Stack smashing
- Heap corruption
- Out-of-bounds access
- Security vulnerabilities
- How tools like AddressSanitizer detect corruption