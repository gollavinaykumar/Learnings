# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 68 — `fopen()`

---

# Most beginners think:

> "When I write `fopen()`, C directly opens a file."

Not exactly.

Your C program **cannot open files by itself**.

Only the **Operating System** has permission to access disks.

When you call:

```c
fopen()
```

your program is actually asking the **Operating System**:

> "Please open this file for me."

If the OS approves,

it returns a handle that your program can use.

If the OS rejects the request,

the operation fails.

Understanding this is the first step toward understanding databases, compilers, shells, web servers, and the Linux kernel.

---

# Learning Objectives

After this chapter you will understand:

- Why `fopen()` exists
- What actually happens internally
- File Streams
- `FILE *`
- File Modes
- Opening files
- Error handling
- Real-world examples

---

# Why Do We Need `fopen()`?

Suppose you want to read:

```
students.txt
```

Can your program directly access the SSD?

```
Program

↓

SSD
```

No.

The program must ask the Operating System.

```
Program

↓

Operating System

↓

File System

↓

SSD
```

Only the Operating System talks to the storage device.

---

# What is `fopen()`?

`fopen()` is a Standard C Library function used to **open a file**.

Syntax:

```c
FILE *fopen(const char *filename,
            const char *mode);
```

It returns:

```
FILE *
```

or

```
NULL
```

if opening fails.

---

# Understanding `FILE *`

Many beginners think:

```
FILE
```

is the actual file.

It isn't.

```
FILE
```

is a structure maintained by the C Standard Library.

Conceptually:

```
FILE

↓

Buffer

↓

File Position

↓

Status Flags

↓

OS File Descriptor
```

Your program works with this structure instead of directly communicating with the operating system every time.

---

# What Happens Internally?

Suppose:

```c
FILE *fp =
fopen("notes.txt","r");
```

Internally:

```
Your Program

↓

fopen()

↓

C Standard Library

↓

System Call

↓

Operating System

↓

File System

↓

SSD
```

If successful:

```
FILE *

↓

Returned
```

Otherwise:

```
NULL
```

---

# The Most Common File Modes

| Mode | Meaning |
|------|---------|
| `"r"` | Read an existing file |
| `"w"` | Write (create or overwrite) |
| `"a"` | Append to the end |
| `"r+"` | Read and write |
| `"w+"` | Read/write (overwrite or create) |
| `"a+"` | Read and append |

We'll explore each mode in more detail later.

---

# Mode: `"r"` (Read)

```c
FILE *fp =
fopen("data.txt","r");
```

Requirements:

```
File Must Exist
```

If the file doesn't exist:

```
NULL
```

is returned.

---

# Mode: `"w"` (Write)

```c
FILE *fp =
fopen("data.txt","w");
```

Behavior:

If the file exists:

```
Old Contents

↓

Deleted
```

If it doesn't exist:

```
New File

↓

Created
```

---

# Mode: `"a"` (Append)

Suppose the file contains:

```
Hello
```

Open:

```c
"a"
```

Write:

```
World
```

Final file:

```
HelloWorld
```

Nothing is deleted.

New data is added to the end.

---

# Opening a File

Example:

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("sample.txt","r");

    if(fp == NULL)
    {
        printf("Unable to open file\n");
        return 1;
    }

    printf("File opened successfully\n");

    return 0;
}
```

---

# Why Check for NULL?

Suppose:

```
sample.txt
```

doesn't exist.

Then:

```c
fp
```

becomes:

```
NULL
```

Trying to use:

```c
fp
```

without checking causes errors.

Always verify:

```c
if(fp == NULL)
```

---

# Memory Visualization

```
Stack

↓

fp

↓

FILE Structure

↓

OS File Descriptor

↓

Disk File
```

Notice:

The pointer doesn't point directly to the file.

It points to a `FILE` object managed by the C library.

---

# File Stream

When a file is opened,

the C library creates a **stream**.

Think of a stream as a flowing pipe.

```
Disk

↓

Stream

↓

Program
```

Data flows through the stream.

Later we'll use:

```
fread()

fwrite()

fprintf()
```

to move data through it.

---

# Multiple Open Files

A program can open several files simultaneously.

```
Program

↓

FILE*

↓

notes.txt

----------------

FILE*

↓

data.csv

----------------

FILE*

↓

log.txt
```

Each open file has its own stream and internal state.

---

# Real-World Example — Database

When PostgreSQL starts,

it opens:

```
Data Files

↓

Transaction Logs

↓

Configuration Files
```

Each file is opened before being read or written.

---

# Real-World Example — Compiler

When GCC compiles:

```
main.c
```

It first:

```
Open File

↓

Read Source Code

↓

Compile
```

The compiler cannot read the file until it has been opened.

---

# Real-World Example — Linux Shell

When you run:

```bash
cat notes.txt
```

The `cat` program internally performs an operation equivalent to:

```
Open File

↓

Read

↓

Display
```

---

# Common Mistakes

---

## Forgetting to Check for NULL

Wrong:

```c
FILE *fp =
fopen("a.txt","r");

fprintf(fp,"Hello");
```

If the file couldn't be opened,

`fp` is `NULL`.

Always check first.

---

## Using the Wrong Mode

Suppose:

```c
"w"
```

is used accidentally.

Existing data is overwritten.

If you only wanted to add data,

use:

```c
"a"
```

---

## Assuming `fopen()` Reads the File

It doesn't.

It only opens the file.

Reading happens later using:

```
fread()

fgets()

fscanf()
```

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("students.txt","w");

    if(fp == NULL)
    {
        printf("Cannot open file\n");
        return 1;
    }

    printf("File opened successfully\n");

    fclose(fp);

    return 0;
}
```

Output:

```
File opened successfully
```

If:

```
students.txt
```

didn't exist,

it is created.

---

# What is `fclose()`?

Every opened file should eventually be closed.

```c
fclose(fp);
```

This tells the C library and the Operating System:

```
Finished Using File

↓

Release Resources
```

We'll study `fclose()` in detail in the next chapter.

---

# Hands-on Labs

## Lab 1

Open an existing file using:

```c
"r"
```

Check whether it opens successfully.

---

## Lab 2

Create a new file using:

```c
"w"
```

Verify that the file appears in your directory.

---

## Lab 3

Open the same file using:

```c
"a"
```

Observe that new content is appended instead of replacing the old content.

---

## Lab 4

Try opening a file that doesn't exist using:

```c
"r"
```

Observe that `fopen()` returns `NULL`.

---

## Lab 5

Open three different files simultaneously.

Print a message if each one opens successfully.

---

# Interview Questions

### What does `fopen()` do?

It requests the Operating System to open a file and returns a `FILE *` stream for accessing it.

---

### What does `FILE *` represent?

A pointer to a C library structure that manages the file stream and communicates with the Operating System.

---

### What happens if `fopen()` fails?

It returns:

```c
NULL
```

---

### Why should we check for `NULL`?

Because attempting to use an invalid file pointer can lead to errors or crashes.

---

### Does `fopen()` read file contents?

No.

It only opens the file.

Reading and writing are performed by other functions.

---

# Summary

```
Program

↓

fopen()

↓

C Standard Library

↓

Operating System

↓

File System

↓

Disk
```

If successful:

```
FILE *

↓

Returned
```

Otherwise:

```
NULL
```

## Key Takeaways

- Programs cannot access storage devices directly.
- `fopen()` requests the Operating System to open a file.
- It returns a `FILE *` stream, not the actual file.
- Always check whether the returned pointer is `NULL`.
- Choose the correct file mode (`"r"`, `"w"`, `"a"`, etc.).
- Every opened file should eventually be closed using `fclose()`.

---

# Next Chapter

## Chapter 69 — `fread()`

You'll learn:

- How to read binary data from files
- The `fread()` function
- Buffers
- Reading structures from files
- Binary vs text reading
- How databases and operating systems read data efficiently