# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 74 — File Position (`ftell()`, `fseek()`, `rewind()`)

---

# Most beginners think:

> "When a program reads a file, it always starts from the beginning."

That's only true for the **first read**.

Every opened file maintains an internal value called the **file position indicator** (often called the file pointer or cursor).

It tells the C library:

> **"Which byte should be read or written next?"**

Understanding file positions is one of the most important concepts in file handling.

Without it, databases, video players, operating systems, and search engines could never jump directly to the data they need.

---

# Learning Objectives

After this chapter you will understand:

- What the file position indicator is
- Sequential access
- Random access
- `ftell()`
- `fseek()`
- `rewind()`
- Why databases use random access
- Real-world examples

---

# Imagine a Book

Suppose you are reading a book.

Initially:

```
Page

↓

1
```

Read five pages.

Now you're here:

```
Page

↓

6
```

The book remembers where you stopped.

Files work exactly the same way.

---

# File Position Indicator

Suppose:

```
students.txt

↓

ABCDEFGHIJ
```

Initially:

```
↓

A B C D E F G H I J
```

The next read gets:

```
A
```

After reading:

```
A

↓

B C D E F G H I J
```

The position automatically moves.

---

# Sequential Reading

Suppose:

```c
fgetc(fp);
```

reads:

```
A
```

Next call:

```c
fgetc(fp);
```

returns:

```
B
```

Then:

```
C
```

Then:

```
D
```

You never tell it where to go.

The file position advances automatically.

---

# What is `ftell()`?

`ftell()` tells you:

> **Where is the current file position?**

Syntax:

```c
long ftell(FILE *stream);
```

Returns:

```
Current Byte Offset
```

from the beginning of the file.

---

# Example

Suppose:

```
ABCDEFGH
```

Initially:

```
↓

A
```

Position:

```
0
```

Read:

```
A
```

Now:

```
↓

B
```

Position:

```
1
```

Read again.

```
↓

C
```

Position:

```
2
```

---

# Program Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("text.txt","r");

    if(fp == NULL)
        return 1;

    printf("%ld\n",
           ftell(fp));

    fgetc(fp);

    printf("%ld\n",
           ftell(fp));

    fclose(fp);

    return 0;
}
```

Output:

```
0

1
```

---

# What is `fseek()`?

Suppose you don't want to read sequentially.

You want:

```
Byte Number

5000
```

directly.

That's exactly what `fseek()` does.

Syntax:

```c
int fseek(
    FILE *stream,
    long offset,
    int origin
);
```

---

# The Three Origins

`fseek()` needs a starting point.

| Constant | Meaning |
|----------|---------|
| `SEEK_SET` | Beginning of file |
| `SEEK_CUR` | Current position |
| `SEEK_END` | End of file |

---

# SEEK_SET

Move from the beginning.

Example:

```c
fseek(fp,
      5,
      SEEK_SET);
```

Meaning:

```
Beginning

↓

Move

↓

5 Bytes
```

---

Visualization:

```
A B C D E F G H

↓

Move To

↓

F
```

---

# SEEK_CUR

Move relative to the current position.

Suppose:

Current:

```
↓

D
```

Call:

```c
fseek(fp,
      2,
      SEEK_CUR);
```

Result:

```
↓

F
```

---

# Moving Backwards

Negative offsets are also possible.

Suppose:

```
↓

G
```

Call:

```c
fseek(fp,
     -2,
      SEEK_CUR);
```

Result:

```
↓

E
```

---

# SEEK_END

Move relative to the end.

Suppose file contains:

```
ABCDEFGH
```

Call:

```c
fseek(fp,
     -1,
      SEEK_END);
```

Moves to:

```
↓

H
```

Very useful when reading file trailers or checking file sizes.

---

# Random Access

Suppose a database has:

```
1 Million Records
```

Need:

```
Record

500000
```

Should it read:

```
1

↓

2

↓

3

↓

...

↓

500000
```

No.

Instead:

```
Jump

↓

Record

500000
```

This is random access.

`fseek()` makes it possible.

---

# What is `rewind()`?

Suppose you've finished reading a file.

Need to read it again.

Instead of:

```c
fseek(fp,
      0,
      SEEK_SET);
```

you can simply write:

```c
rewind(fp);
```

It moves the file position back to the beginning.

---

# Example

Before:

```
↓

H
```

After:

```c
rewind(fp);
```

```
↓

A
```

---

# File Size Trick

One common technique:

```c
fseek(fp,
      0,
      SEEK_END);

long size =
ftell(fp);

rewind(fp);
```

Explanation:

```
Go To End

↓

Current Position

↓

File Size

↓

Return To Start
```

Very common.

---

# Memory Visualization

Suppose:

```
File

↓

ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

Current position:

```
↓

K
```

Call:

```c
fseek(fp,
      5,
      SEEK_CUR);
```

Result:

```
↓

P
```

---

# Real-World Example — Video Player

Suppose you drag a video to:

```
1 Hour

25 Minutes
```

The player doesn't read the first hour again.

Instead:

```
Jump

↓

Desired Position

↓

Continue Playback
```

Random access.

---

# Real-World Example — PostgreSQL

Database page:

```
8192 Bytes
```

Need:

```
Page

250
```

Compute:

```
250 × 8192
```

Then:

```
fseek()

↓

Correct Offset

↓

Read Page
```

No unnecessary scanning.

---

# Real-World Example — PDF Reader

Click:

```
Page 200
```

Reader:

```
Jump

↓

Correct Offset

↓

Display Page
```

It doesn't render pages 1–199 first.

---

# Common Mistakes

---

## Confusing Bytes with Characters

`ftell()` reports the current byte position.

In text mode, due to platform-specific newline translation, the value may not correspond exactly to a visible character count.

---

## Forgetting `rewind()`

After reaching the end of a file,

attempting to read again without repositioning will not work.

Use:

```c
rewind(fp);
```

or

```c
fseek(fp, 0, SEEK_SET);
```

---

## Using Invalid Offsets

Suppose:

```
File

↓

100 Bytes
```

Moving to:

```
100000
```

may cause later read operations to fail or return end-of-file.

Always ensure your offsets make sense for the file.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("text.txt","r");

    if(fp == NULL)
        return 1;

    printf("Position: %ld\n",
           ftell(fp));

    fseek(fp,
          5,
          SEEK_SET);

    printf("Position: %ld\n",
           ftell(fp));

    rewind(fp);

    printf("Position: %ld\n",
           ftell(fp));

    fclose(fp);

    return 0;
}
```

Output:

```
Position: 0

Position: 5

Position: 0
```

---

# Hands-on Labs

## Lab 1

Read one character at a time.

Print the value of:

```c
ftell()
```

after each read.

---

## Lab 2

Jump to:

```
Byte

20
```

using:

```c
fseek()
```

Read the next character.

---

## Lab 3

Find the size of a file using:

```c
fseek()

↓

ftell()

↓

rewind()
```

---

## Lab 4

Create a binary file containing:

```
100 Integers
```

Jump directly to the:

```
50th Integer
```

Read it without reading the previous 49 integers.

---

## Lab 5

Build a simple record viewer.

Allow the user to jump directly to any record using:

```c
fseek()
```

---

# Interview Questions

### What is the file position indicator?

It tracks the next byte that will be read from or written to a file.

---

### What does `ftell()` return?

The current byte offset from the beginning of the file.

---

### What does `fseek()` do?

It moves the file position indicator to a specified location.

---

### What does `rewind()` do?

It resets the file position indicator to the beginning of the file.

---

### Why is `fseek()` important?

It enables random access, allowing programs to jump directly to the required data without reading everything before it.

---

# Summary

```
File

↓

Position Indicator

↓

Read

Write

↓

Moves Automatically
```

Need a different location?

```
fseek()

↓

Jump

↓

Continue Reading
```

Need the beginning again?

```
rewind()
```

Need the current position?

```
ftell()
```

## Key Takeaways

- Every open file has a file position indicator.
- Sequential reads automatically advance the position.
- `ftell()` reports the current byte offset.
- `fseek()` moves the position anywhere in the file.
- `rewind()` returns to the beginning.
- Random access is essential for databases, video players, file systems, and high-performance software.

---

# Next Chapter

## Chapter 75 — Standard Streams (`stdin`, `stdout`, `stderr`)

You'll learn:

- What standard streams are
- Why every C program has them
- `stdin`
- `stdout`
- `stderr`
- Input/output redirection
- Pipes
- How Linux shells connect programs together using standard streams