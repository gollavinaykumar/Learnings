# 👑 C Mastery Roadmap

# Module 10 — Dynamic Memory Management

# Chapter 87 — Memory Alignment & Padding

---

# Most beginners think:

> "`sizeof(struct)` is simply the sum of the sizes of its members."

For example:

```c
struct Student
{
    char grade;
    int age;
};
```

Many beginners calculate:

```
char

↓

1 Byte

+

int

↓

4 Bytes

=

5 Bytes
```

Then they run:

```c
printf("%zu",
sizeof(struct Student));
```

Output:

```
8
```

Wait...

Where did the extra **3 bytes** come from?

The answer is:

```
Memory Alignment
```

and

```
Padding
```

Every modern CPU depends on alignment for efficient memory access.

---

# Learning Objectives

After this chapter you will understand:

- What memory alignment is
- Why CPUs require alignment
- Structure padding
- Compiler behavior
- Performance impact
- Packed structures
- Real-world examples

---

# Why Alignment Exists

Suppose:

A CPU wants to read an:

```
int

↓

4 Bytes
```

Memory:

```
Address

0

1

2

3

4

5

6

7
```

If the integer begins at:

```
Address

4
```

Everything is easy.

```
4

5

6

7
```

One aligned read.

---

# Misaligned Access

Suppose instead:

```
Address

3
```

Memory:

```
3

4

5

6
```

Now the integer crosses a natural boundary.

Some CPUs must perform:

```
Read One Block

↓

Read Another Block

↓

Combine Results
```

Slower.

Some architectures even raise a hardware exception for certain misaligned accesses.

---

# Natural Alignment

Most data types have a preferred alignment.

Typical example on many modern systems:

| Type | Size | Preferred Alignment |
|------|------|---------------------|
| `char` | 1 | 1 |
| `short` | 2 | 2 |
| `int` | 4 | 4 |
| `float` | 4 | 4 |
| `double` | 8 | 8 |
| Pointer | 8 (64-bit systems) | 8 |

Exact values depend on the platform and ABI.

---

# Example Structure

```c
struct Student
{
    char grade;
    int age;
};
```

Naively:

```
grade

↓

1 Byte

age

↓

4 Bytes
```

Total:

```
5 Bytes
```

But that's not what happens.

---

# Actual Layout

```
Offset

0

↓

grade

1

↓

Padding

2

↓

Padding

3

↓

Padding

4

↓

age

5

6

7
```

Total:

```
8 Bytes
```

---

# Why Padding?

The compiler wants:

```
int age
```

to begin at an address divisible by:

```
4
```

So it inserts:

```
3 Bytes

↓

Padding
```

Automatically.

---

# Another Example

```c
struct Data
{
    char a;
    double b;
};
```

Typical layout:

```
Offset

0

↓

a

1

↓

Padding

...

7

↓

Padding

8

↓

b

8 Bytes
```

Typical total size:

```
16 Bytes
```

Not:

```
9 Bytes
```

---

# End Padding

Consider:

```c
struct Example
{
    int a;
    char b;
};
```

Layout:

```
a

↓

4 Bytes

b

↓

1 Byte

Padding

↓

3 Bytes
```

Total:

```
8 Bytes
```

Why pad the end?

Suppose:

```c
struct Example arr[2];
```

Each structure should begin properly aligned.

End padding ensures every array element starts at the correct boundary.

---

# Visualizing an Array

Without end padding:

```
Struct1

↓

5 Bytes

Struct2

↓

Starts Misaligned
```

With padding:

```
Struct1

↓

8 Bytes

Struct2

↓

Aligned
```

Much better for the CPU.

---

# Member Ordering Matters

Example:

```c
struct Bad
{
    char a;
    double b;
    char c;
};
```

Large amount of padding.

Better:

```c
struct Good
{
    double b;
    char a;
    char c;
};
```

Usually fewer padding bytes.

Reordering members can reduce memory usage.

---

# Packed Structures

Some compilers support packed structures.

Example (compiler-specific):

```c
#pragma pack(1)
```

or

```c
__attribute__((packed))
```

Now:

```
No Padding
```

Memory becomes compact.

---

# Why Not Always Pack?

Packed structures may cause:

- Slower memory access
- Misaligned loads
- Hardware exceptions on some CPUs

Packed structures are useful only when required,

such as binary protocols or file formats.

---

# Cache Efficiency

Suppose:

```
100 Million Structures
```

Saving:

```
8 Bytes
```

per structure saves:

```
800 MB
```

Good structure design can significantly improve cache performance and memory usage.

---

# Using `offsetof()`

C provides:

```c
offsetof()
```

Example:

```c
offsetof(
struct Student,
age);
```

Returns:

```
Offset

↓

4
```

Very useful in systems programming.

---

# Alignment in `malloc()`

When:

```c
malloc()
```

returns memory,

it is aligned suitably for any object type required by the platform.

You don't need to align normal allocations yourself.

---

# Real-World Example — Network Protocol

Network packet:

```
Header

↓

Exact Byte Layout
```

Packed structures are sometimes used,

though many projects instead serialize fields manually for portability.

---

# Real-World Example — Database

Database pages often arrange fields carefully to balance:

- Space efficiency
- Alignment
- Performance

---

# Real-World Example — Operating System

Kernel structures are designed with careful attention to:

- Alignment
- Cache lines
- False sharing
- Memory footprint

Poor layouts can hurt performance.

---

# Real-World Example — Game Engine

Millions of game objects:

```
Position

Velocity

Health
```

Optimizing structure layout improves cache locality and frame rate.

---

# Common Mistakes

---

## Assuming `sizeof(struct)` Is the Sum of Members

Wrong.

Padding may increase the total size.

---

## Ignoring Member Order

Poor ordering:

```
Small

Large

Small
```

Often wastes memory.

---

## Using Packed Structures Everywhere

Packed structures are not automatically faster.

They can reduce performance due to misaligned accesses.

---

## Hardcoding Structure Sizes

Wrong:

```c
malloc(20);
```

Better:

```c
malloc(sizeof(struct Student));
```

`sizeof()` automatically accounts for padding.

---

# Complete Example

```c
#include <stdio.h>

struct Student
{
    char grade;
    int age;
};

int main()
{
    printf("Size: %zu\n",
           sizeof(struct Student));

    return 0;
}
```

Typical output on many systems:

```
Size: 8
```

(The exact value depends on the platform and compiler.)

---

# Hands-on Labs

## Lab 1

Create several structures.

Predict their sizes.

Verify using:

```c
sizeof()
```

---

## Lab 2

Reorder members.

Observe how the structure size changes.

---

## Lab 3

Use:

```c
offsetof()
```

to print member offsets.

---

## Lab 4

Compare a packed structure with a normal structure.

Observe the size difference.

---

## Lab 5

Create an array of one million structures.

Estimate the memory savings after optimizing member order.

---

# Interview Questions

### What is memory alignment?

Arranging data at addresses that satisfy the alignment requirements of the CPU and platform.

---

### Why does the compiler insert padding?

To align members properly for efficient and correct memory access.

---

### Why is `sizeof(struct)` sometimes larger than expected?

Because padding bytes may be inserted between members and at the end of the structure.

---

### What is a packed structure?

A structure where padding is minimized or removed using compiler-specific features.

---

### Should packed structures always be used?

No.

They can reduce performance and may cause portability issues.

---

# Summary

```
Structure

↓

Members

↓

Compiler

↓

Padding

↓

Aligned Layout
```

Example:

```
char

↓

Padding

↓

int
```

instead of:

```
char

↓

int
```

Key Idea:

```
Space

vs

Performance
```

Compilers usually choose the layout that gives better performance.

## Key Takeaways

- CPUs perform best when data is properly aligned.
- Compilers insert padding automatically to satisfy alignment requirements.
- `sizeof(struct)` includes padding.
- Member ordering can significantly affect memory usage.
- Packed structures should only be used when exact layouts are required.
- Understanding alignment is essential for systems programming, operating systems, databases, networking, and performance optimization.

---

# Next Chapter

## Chapter 88 — Memory Debugging Tools (Valgrind & AddressSanitizer)

You'll learn:

- How Valgrind works internally
- How AddressSanitizer detects memory bugs
- Detecting leaks
- Detecting use-after-free
- Detecting buffer overflows
- Comparing Valgrind vs AddressSanitizer
- Professional debugging workflows used in production software