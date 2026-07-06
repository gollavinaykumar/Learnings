# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 70 — `fwrite()`

---

# Most beginners think:

> "`fwrite()` writes text into a file."

Not exactly.

Functions like:

```c
fprintf()

fputs()
```

are designed for writing **text**.

`fwrite()` writes **raw bytes**.

Those bytes may represent:

- Integers
- Structures
- Images
- Videos
- Databases
- Executables
- Memory Dumps

Everything stored on disk eventually becomes **bytes**.

Just as `fread()` copies bytes **from disk to memory**,

`fwrite()` copies bytes **from memory to disk**.

---

# Learning Objectives

After this chapter you will understand:

- What `fwrite()` does
- Why it exists
- Writing raw bytes
- Buffers
- Writing arrays
- Writing structures
- Return value
- Binary files
- Real-world examples

---

# First Understand This

Suppose your program has:

```c
int age = 25;
```

Memory:

```
RAM

↓

25
```

When you save it,

the value isn't stored as:

```
2

5
```

unless you're writing text.

Instead,

the binary representation is written.

Conceptually:

```
RAM

↓

Bytes

↓

Disk
```

---

# What is `fwrite()`?

`fwrite()` writes **raw bytes** from memory into a file.

Syntax:

```c
size_t fwrite(
    const void *buffer,
    size_t size,
    size_t count,
    FILE *stream
);
```

It is almost the opposite of:

```c
fread()
```

---

# Think Like This

```
Program

↓

Memory

↓

Bytes

↓

fwrite()

↓

Disk
```

Data flows from RAM to storage.

---

# Understanding the Parameters

Example:

```c
fwrite(buffer,
       4,
       10,
       fp);
```

Meaning:

```
Write

↓

10 Objects

↓

Each Object

↓

4 Bytes
```

Total:

```
40 Bytes
```

---

# Parameter 1 — Buffer

The data you want to save.

Example:

```c
int numbers[5];
```

Memory:

```
RAM

↓

numbers
```

`fwrite()` copies these bytes to the file.

---

# Parameter 2 — Size

Suppose:

```c
sizeof(int)
```

returns:

```
4 Bytes
```

Each object occupies:

```
4 Bytes
```

---

# Parameter 3 — Count

Suppose:

```
5 Integers
```

Need to save:

```
5 Objects
```

Then:

```
count

↓

5
```

---

# Parameter 4 — Stream

The file returned by:

```c
fopen()
```

Example:

```c
FILE *fp;
```

---

# Example 1 — Write Integers

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("numbers.bin","wb");

    if(fp == NULL)
        return 1;

    int numbers[5] =
    {
        10,
        20,
        30,
        40,
        50
    };

    fwrite(numbers,
           sizeof(int),
           5,
           fp);

    fclose(fp);

    return 0;
}
```

Memory:

```
10

20

30

40

50
```

↓

```
numbers.bin
```

---

# Why `"wb"`?

```
w

↓

Write
```

```
b

↓

Binary
```

Using binary mode avoids text translation on platforms like Windows.

On Linux/macOS,

`"w"` and `"wb"` behave the same for ordinary files,

but `"wb"` is recommended for portable binary programs.

---

# Example 2 — Write Characters

```c
char data[] =
{
    'A',
    'B',
    'C',
    'D'
};

fwrite(data,
       sizeof(char),
       4,
       fp);
```

File contains:

```
A

B

C

D
```

stored as bytes.

---

# Example 3 — Write a Structure

Suppose:

```c
struct Student
{
    int age;
};
```

Program:

```c
struct Student s =
{
    21
};

fwrite(&s,
       sizeof(struct Student),
       1,
       fp);
```

Entire structure is written in one call.

This is very common in database engines and binary file formats.

---

# Memory Visualization

Before:

```
RAM

↓

Structure

↓

Array

↓

Buffer
```

After:

```
RAM

↓

Bytes

↓

Copied

↓

Disk
```

The bytes are stored in the file.

---

# Return Value

`fwrite()` returns:

```
Number of Objects Successfully Written
```

Example:

```c
size_t written =
fwrite(...);
```

Suppose:

Need:

```
10 Objects
```

Actually written:

```
8 Objects
```

Return:

```
8
```

Always check the return value.

---

# File Position

Suppose the file initially contains:

```
Empty
```

Write:

```
ABC
```

File becomes:

```
ABC
```

File position:

```
↓

End
```

Write:

```
XYZ
```

Final file:

```
ABCXYZ
```

The file position moves automatically after each write.

---

# Overwriting vs Appending

Mode:

```c
"wb"
```

Existing file:

```
Hello
```

Write:

```
ABC
```

Result:

```
ABC
```

Old contents are removed.

---

Mode:

```c
"ab"
```

Existing file:

```
Hello
```

Write:

```
ABC
```

Result:

```
HelloABC
```

The data is appended.

---

# `fwrite()` + `fread()`

Write:

```
Memory

↓

Disk
```

Later:

```
Disk

↓

Memory
```

These two functions are complementary.

---

# Real-World Example — PostgreSQL

When a database page changes:

```
Memory

↓

8 KB Page

↓

Disk
```

The database writes binary pages efficiently.

---

# Real-World Example — Image Editor

Suppose you edit:

```
photo.jpg
```

After modification:

```
Image Memory

↓

Binary Bytes

↓

Disk
```

The image is written as raw bytes.

---

# Real-World Example — Game Save File

A game stores:

```
Player

↓

Health

↓

Level

↓

Inventory
```

The game writes these structures to a save file.

Later,

it reads them back using `fread()`.

---

# Common Mistakes

---

## Ignoring Return Value

Wrong:

```c
fwrite(...);
```

Correct:

```c
if (fwrite(...) != expectedCount)
{
    /* Handle write failure */
}
```

---

## Using Wrong Size

Wrong:

```c
fwrite(numbers,
       1,
       5,
       fp);
```

when writing integers.

Correct:

```c
fwrite(numbers,
       sizeof(int),
       5,
       fp);
```

---

## Forgetting Binary Mode

For binary files,

prefer:

```c
"wb"
```

for portable code.

---

## Forgetting to Close the File

Without:

```c
fclose(fp);
```

some buffered data may not be written immediately.

Always close the file when you're done.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("ages.bin","wb");

    if(fp == NULL)
        return 1;

    int ages[] =
    {
        20,
        21,
        22,
        23
    };

    size_t written =
        fwrite(ages,
               sizeof(int),
               4,
               fp);

    printf("Written: %zu\n",
           written);

    fclose(fp);

    return 0;
}
```

Output:

```
Written: 4
```

---

# Hands-on Labs

## Lab 1

Write:

```
10 Integers
```

to a binary file.

Read them back using:

```c
fread()
```

Verify the values.

---

## Lab 2

Create a `Student` structure.

Write it to a file.

Read it back.

---

## Lab 3

Print the return value of:

```c
fwrite()
```

Observe what happens if the disk becomes full (if possible) or if a write error occurs.

---

## Lab 4

Create a binary copy of a file using:

```
fread()

↓

fwrite()
```

---

## Lab 5

Write several structures into a file.

Read them back into an array.

---

# Interview Questions

### What does `fwrite()` do?

It writes raw bytes from memory to a file.

---

### What does `fwrite()` return?

The number of objects successfully written.

---

### Why is `sizeof()` commonly used?

To write the correct number of bytes for each object.

---

### What is the difference between `fprintf()` and `fwrite()`?

`fprintf()` writes formatted text.

`fwrite()` writes raw binary data.

---

### Where is `fwrite()` commonly used?

- Databases
- Image editors
- Game save files
- Binary file formats
- Operating systems
- Embedded systems

---

# Summary

```
Memory

↓

Bytes

↓

fwrite()

↓

Disk
```

Example:

```
Structures

Arrays

Objects

↓

Binary File
```

## Key Takeaways

- `fwrite()` writes raw bytes to a file.
- It is designed for binary data.
- The source buffer must contain the data to be written.
- Always check the return value.
- `sizeof()` helps write complete objects correctly.
- `fwrite()` and `fread()` are complementary operations used extensively in databases, operating systems, and high-performance software.

---

# Next Chapter

## Chapter 71 — `fprintf()`

You'll learn:

- Formatted output to files
- Writing human-readable text
- Format specifiers
- Logging
- CSV generation
- Configuration files
- The difference between `fprintf()` and `fwrite()`