# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 72 — `fclose()`

---

# Most beginners think:

> "Once my program finishes, the file is automatically saved."

Sometimes that's true.

But that's **not** why we use `fclose()`.

The real purpose of `fclose()` is much deeper.

When you write:

```c
fprintf(fp, "Hello");
```

or

```c
fwrite(...);
```

the data is usually **not written immediately** to the disk.

Instead, it is first stored in a **memory buffer**.

Only later does the C library send that buffered data to the Operating System.

If you forget to close the file,

some buffered data may never be written.

---

# Learning Objectives

After this chapter you will understand:

- Why `fclose()` exists
- What happens internally
- File buffers
- Buffer flushing
- Resource leaks
- File descriptors
- Error handling
- Real-world examples

---

# Why Do We Need `fclose()`?

Suppose:

```c
fprintf(fp,
        "Hello");
```

Does the SSD immediately receive:

```
Hello
```

No.

Usually,

the data first goes into a buffer.

```
Program

↓

fprintf()

↓

Memory Buffer

↓

Operating System

↓

SSD
```

---

# What is a Buffer?

A buffer is a temporary memory area.

Think of it as a waiting room.

Instead of writing:

```
1 Byte

↓

Disk
```

again and again,

the library collects many bytes.

Example:

```
Hello

World

123

ABC
```

Everything stays inside the buffer until it's ready to be written efficiently.

---

# Why Buffers Exist

Imagine writing:

```
1 Character
```

to an SSD.

Without buffering:

```
CPU

↓

OS

↓

Disk
```

Thousands of times.

That's slow.

Instead:

```
CPU

↓

Buffer

↓

4096 Bytes

↓

OS

↓

Disk
```

One large write is much faster.

---

# What Does `fclose()` Do?

When you call:

```c
fclose(fp);
```

the C library performs several tasks.

Conceptually:

```
Flush Buffer

↓

Release FILE Structure

↓

Close OS File

↓

Release Resources
```

Only after these steps is the file completely closed.

---

# Syntax

```c
int fclose(FILE *stream);
```

Returns:

```
0
```

Success

or

```
EOF
```

Failure

(`EOF` is typically `-1`.)

---

# Basic Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("demo.txt","w");

    if(fp == NULL)
        return 1;

    fprintf(fp,
            "Hello World");

    fclose(fp);

    return 0;
}
```

After:

```c
fclose(fp);
```

The buffer is flushed,

and the file is safely closed.

---

# Internal Flow

```
Program

↓

fprintf()

↓

Memory Buffer

↓

fclose()

↓

Flush Buffer

↓

Operating System

↓

Disk
```

---

# What Happens If You Forget?

Suppose:

```c
fprintf(fp,
        "Important Data");
```

Program crashes before:

```c
fclose(fp);
```

Possible result:

```
Buffer

↓

Lost
```

File:

```
Incomplete
```

This is one reason why proper cleanup is important.

---

# Why `fclose()` Releases Resources

Opening a file allocates:

- A `FILE` structure
- An operating system file descriptor
- Internal buffers

After:

```c
fclose(fp);
```

these resources are released.

Without closing,

your program may eventually run out of available file descriptors.

---

# File Descriptor Relationship

Remember:

```
FILE *

↓

C Library

↓

File Descriptor

↓

Operating System
```

`fclose()` ultimately closes the underlying operating system file descriptor as well.

We'll study file descriptors in a later chapter.

---

# Closing Multiple Files

```c
FILE *a =
fopen("a.txt","w");

FILE *b =
fopen("b.txt","w");

FILE *c =
fopen("c.txt","w");

fclose(a);

fclose(b);

fclose(c);
```

Each open file should be closed exactly once.

---

# What Happens After Closing?

Suppose:

```c
fclose(fp);
```

Then:

```c
fprintf(fp,
        "Hello");
```

This is **undefined behavior**.

The stream is no longer valid.

After closing,

don't use that pointer again unless it is assigned a new stream.

A common practice is:

```c
fclose(fp);

fp = NULL;
```

This helps prevent accidental reuse.

---

# Return Value

Example:

```c
if (fclose(fp) == EOF)
{
    printf("Close Failed\n");
}
```

Most programs ignore this,

but checking it can help detect errors such as failures while flushing buffered data.

---

# `fflush()` vs `fclose()`

Many beginners confuse them.

---

`fflush()`

```
Flush Buffer

↓

Keep File Open
```

---

`fclose()`

```
Flush Buffer

↓

Close File

↓

Release Resources
```

---

# Real-World Example — Database

Suppose:

```
Transaction Log
```

Database:

```
Memory

↓

Write Log

↓

Flush

↓

Close During Shutdown
```

Closing ensures all pending writes are completed before shutdown.

---

# Real-World Example — Log File

A web server writes:

```
User Login

↓

Page Request

↓

Database Error
```

When the server shuts down,

it closes the log file to ensure buffered entries are written.

---

# Real-World Example — Image Editor

Suppose you edit:

```
photo.png
```

After saving:

```
Image Data

↓

Buffer

↓

Disk

↓

Close File
```

Only then is the save operation complete.

---

# Common Mistakes

---

## Forgetting `fclose()`

Wrong:

```c
FILE *fp =
fopen("a.txt","w");

fprintf(fp,
        "Hello");
```

Program exits without explicitly closing.

While many operating systems clean up resources at process exit, it's still good practice to call `fclose()` so buffered data is flushed and errors can be detected.

---

## Using a Closed File

Wrong:

```c
fclose(fp);

fprintf(fp,
        "Hello");
```

The stream is no longer valid.

---

## Closing Twice

Wrong:

```c
fclose(fp);

fclose(fp);
```

Closing the same stream twice is **undefined behavior**.

A safer pattern:

```c
fclose(fp);

fp = NULL;
```

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("log.txt","w");

    if(fp == NULL)
        return 1;

    fprintf(fp,
            "Program Started\n");

    fprintf(fp,
            "Initialization Complete\n");

    if (fclose(fp) == EOF)
    {
        printf("Error closing file\n");
        return 1;
    }

    return 0;
}
```

Generated file:

```
Program Started

Initialization Complete
```

---

# Hands-on Labs

## Lab 1

Write data.

Close the file.

Open it using a text editor.

Verify that everything was saved.

---

## Lab 2

Experiment with:

```c
fflush()
```

Observe that the file remains open.

---

## Lab 3

Attempt to write after:

```c
fclose()
```

Observe the incorrect behavior (do **not** rely on the result—it is undefined).

---

## Lab 4

Open five files.

Close them all properly.

---

## Lab 5

Check the return value of:

```c
fclose()
```

Handle any reported errors.

---

# Interview Questions

### Why do we call `fclose()`?

To flush buffered data, close the file, and release associated resources.

---

### What does `fclose()` return?

`0` on success and `EOF` on failure.

---

### What happens if we forget to close a file?

Buffered data may not be written correctly, and resources remain allocated until the operating system cleans them up at process termination.

---

### What is the difference between `fflush()` and `fclose()`?

`fflush()` writes buffered data but keeps the file open.

`fclose()` flushes the data, closes the stream, and releases resources.

---

### Can we use a file pointer after `fclose()`?

No.

Using a closed stream results in undefined behavior.

---

# Summary

```
Program

↓

Write Data

↓

Memory Buffer

↓

fclose()

↓

Flush

↓

Operating System

↓

Disk

↓

Resources Released
```

## Key Takeaways

- `fclose()` safely finishes work on a file.
- It flushes buffered data before closing.
- It releases both C library and operating system resources.
- Every successful `fopen()` should have a matching `fclose()`.
- Never use a stream after closing it.
- Proper file closing is essential in databases, web servers, compilers, operating systems, and production software.

---

# Next Chapter

## Chapter 73 — Text Files vs Binary Files

You'll learn:

- What makes a file a text file
- What makes a file a binary file
- Human-readable vs machine-readable formats
- Advantages and disadvantages
- When to choose each
- How databases, images, executables, and configuration files use different file formats