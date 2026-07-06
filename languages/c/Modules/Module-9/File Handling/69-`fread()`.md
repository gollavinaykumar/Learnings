# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 69 — `fread()`

---

# Most beginners think:

> "`fread()` reads a file line by line."

It doesn't.

That's what functions like:

```c
fgets()

fscanf()
```

are designed for.

`fread()` is much lower level.

It reads **raw bytes** from a file.

Those bytes could represent:

- Text
- Images
- Videos
- Database records
- Linux executables
- Network packets
- Memory dumps

Everything stored in a file is ultimately just **bytes**.

After this chapter, you'll understand how databases, operating systems, and file systems read binary data efficiently.

---

# Learning Objectives

After this chapter you will understand:

- What `fread()` does
- Why it exists
- Reading raw bytes
- Buffers
- Reading arrays
- Reading structures
- Binary files
- File position
- Real-world examples

---

# First Understand This

Suppose we have:

```
students.txt
```

Contents:

```
Vinay

Rahul

Anil
```

Looks like text.

But on disk,

the Operating System stores:

```
56

69

6E

61

79

0A

52

61

68

75

6C
...
```

These are **bytes**.

The computer never sees:

```
Vinay
```

It only sees bytes.

---

# What is `fread()`?

`fread()` reads **raw bytes** from a file into memory.

Syntax:

```c
size_t fread(
    void *buffer,
    size_t size,
    size_t count,
    FILE *stream
);
```

Looks complicated.

Let's simplify it.

---

# Think Like This

```
Disk

↓

Bytes

↓

Buffer

↓

Program
```

`fread()` copies bytes

from the file

into memory.

---

# Understanding the Parameters

Suppose:

```c
fread(buffer,
      4,
      10,
      fp);
```

Meaning:

```
Read

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

```c
buffer
```

Where should the data go?

Example:

```c
char buffer[100];
```

Memory:

```
RAM

↓

buffer
```

The bytes are copied here.

---

# Parameter 2 — Size

Suppose:

```c
sizeof(int)
```

equals:

```
4 Bytes
```

Then:

```c
size = 4
```

Each object occupies:

```
4 Bytes
```

---

# Parameter 3 — Count

Suppose:

```
10 Integers
```

Need to read:

```
10 Objects
```

Then:

```
count

↓

10
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

# Example 1 — Read Characters

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("data.bin","rb");

    char buffer[10];

    fread(buffer,
          sizeof(char),
          10,
          fp);

    fclose(fp);

    return 0;
}
```

Reads:

```
10 Bytes
```

---

# Why `"rb"`?

```
r

↓

Read
```

```
b

↓

Binary
```

Binary mode avoids text-specific translations on platforms like Windows.

On Linux and macOS, `"r"` and `"rb"` behave the same for regular files, but using `"rb"` is good practice for binary data.

---

# Example 2 — Read Integers

Suppose file contains:

```
10

20

30

40
```

Stored as binary.

Program:

```c
int numbers[4];

fread(numbers,
      sizeof(int),
      4,
      fp);
```

Memory:

```
numbers

↓

10

20

30

40
```

---

# Example 3 — Read a Structure

Suppose:

```c
struct Student
{
    int age;
};
```

Read:

```c
struct Student s;

fread(&s,
      sizeof(struct Student),
      1,
      fp);
```

Meaning:

```
Read

↓

One Structure
```

Very common in databases.

---

# Memory Visualization

Before:

```
Buffer

↓

???

???

???

???
```

After:

```
Disk

↓

Bytes

↓

Copied

↓

Buffer
```

The bytes now exist in RAM.

---

# Return Value

Many beginners ignore this.

`fread()` returns:

```
Number of Objects Successfully Read
```

Example:

```c
size_t n =
fread(...);
```

Suppose:

Need:

```
10 Objects
```

Actually read:

```
7 Objects
```

Return:

```
7
```

Always check this value.

---

# Reading Until End of File

Suppose:

```
100 Integers
```

Need to read one at a time.

```c
while(
fread(&value,
      sizeof(int),
      1,
      fp) == 1)
{
    printf("%d\n",value);
}
```

The loop stops when no more complete objects can be read.

---

# File Position

Suppose file contains:

```
A

B

C

D
```

Initially:

```
↓

A
```

Read one byte.

Now:

```
↓

B
```

Read again.

```
↓

C
```

Every call moves the file position forward.

---

# Binary vs Text Reading

Text:

```
Hello
```

Read using:

```
fgets()

fscanf()
```

Binary:

```
Image

Video

Database

Executable

Memory Dump
```

Read using:

```
fread()
```

---

# Real-World Example — PostgreSQL

A database page:

```
8 KB
```

Database:

```
Disk

↓

8 KB Page

↓

RAM
```

This is conceptually a binary read.

---

# Real-World Example — Image Viewer

Open:

```
photo.jpg
```

The viewer reads:

```
Binary Bytes

↓

Memory

↓

Decode Image
```

It doesn't read "lines."

It reads bytes.

---

# Real-World Example — Linux ELF Executable

When Linux starts a program:

```
Executable

↓

Read Binary Bytes

↓

Memory

↓

Execute
```

The kernel reads binary data,

not text.

---

# Common Mistakes

---

## Ignoring Return Value

Wrong:

```c
fread(...);
```

Correct:

```c
if (fread(...) != expectedCount)
{
    /* Handle partial read or EOF/error */
}
```

---

## Using Wrong Size

Wrong:

```c
fread(numbers,
      1,
      10,
      fp);
```

when reading integers.

Correct:

```c
fread(numbers,
      sizeof(int),
      10,
      fp);
```

---

## Forgetting Binary Mode

For binary files,

use:

```c
"rb"
```

especially for portable code.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("numbers.bin","rb");

    if(fp == NULL)
    {
        return 1;
    }

    int numbers[5];

    size_t read =
        fread(numbers,
              sizeof(int),
              5,
              fp);

    printf("Read %zu integers\n", read);

    fclose(fp);

    return 0;
}
```

---

# Hands-on Labs

## Lab 1

Create a binary file.

Read:

```
10 Integers
```

using:

```c
fread()
```

---

## Lab 2

Store a structure in a file.

Read it back.

---

## Lab 3

Print the return value of:

```c
fread()
```

Read beyond the end of the file and observe the result.

---

## Lab 4

Read an image file using:

```c
fread()
```

Print the first:

```
32 Bytes
```

in hexadecimal.

---

## Lab 5

Use a debugger.

Observe how the buffer changes after:

```c
fread()
```

---

# Interview Questions

### What does `fread()` do?

It reads raw bytes from a file into memory.

---

### What does `fread()` return?

The number of objects successfully read.

---

### Why is `sizeof()` commonly used?

To ensure the correct number of bytes is read for each object.

---

### What is the difference between `fread()` and `fgets()`?

`fread()` reads raw binary data.

`fgets()` reads a line of text.

---

### Where is `fread()` commonly used?

- Databases
- Image processing
- Executable loaders
- Operating systems
- Binary file formats
- Embedded systems

---

# Summary

```
Disk

↓

Bytes

↓

fread()

↓

Buffer

↓

Program
```

Example:

```
Binary File

↓

Raw Bytes

↓

RAM

↓

Structures

Arrays

Objects
```

## Key Takeaways

- `fread()` reads raw bytes from a file.
- It is primarily used for binary data.
- The destination buffer must have enough memory.
- Always check the return value to detect partial reads or end-of-file.
- `sizeof()` helps read the correct object size.
- `fread()` is fundamental to databases, operating systems, executable loaders, and high-performance file processing.

---

# Next Chapter

## Chapter 70 — `fwrite()`

You'll learn:

- How to write binary data to files
- Writing arrays and structures
- Buffers
- File position while writing
- How databases and operating systems save data efficiently