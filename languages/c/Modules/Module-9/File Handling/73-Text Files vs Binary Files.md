# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 73 — Text Files vs Binary Files

---

# Most beginners think:

> "Every file is just text."

Actually...

**Every file is binary.**

This is one of the biggest misconceptions in computing.

Whether it's:

- PDF
- JPG
- PNG
- MP4
- EXE
- ZIP
- Database
- TXT

everything stored on disk is ultimately just **bytes (0s and 1s).**

The difference is:

**How those bytes are interpreted.**

---

# Learning Objectives

After this chapter you will understand:

- What is a Text File?
- What is a Binary File?
- Human-readable vs Machine-readable
- Advantages & Disadvantages
- Performance comparison
- Storage comparison
- Real-world examples
- When to use each

---

# Everything is Binary

Suppose you create:

```
hello.txt
```

Contents:

```
Hello
```

Looks like text.

But inside the SSD:

```
01001000

01100101

01101100

01101100

01101111
```

Everything is binary.

---

# Then Why Call It a Text File?

Because those bytes represent **characters**.

Example:

```
01000001
```

ASCII:

```
A
```

The operating system or text editor interprets the bytes as characters.

---

# What is a Text File?

A text file stores data using a **character encoding**.

Common encodings:

- ASCII
- UTF-8
- UTF-16

Examples:

```
notes.txt

README.md

config.ini

settings.conf

data.csv

index.html

script.js

main.c
```

All of these are text files.

---

# Example

Suppose:

```
Age = 25
```

Text file stores:

```
'2'

↓

00110010

'5'

↓

00110101
```

The number is stored as characters.

---

# What is a Binary File?

A binary file stores **raw bytes**.

Example:

```
25
```

Memory:

```
00000000

00000000

00000000

00011001
```

Those bytes are written directly.

No conversion into characters.

---

# Text File vs Binary File

Suppose:

```
Integer = 25
```

Text:

```
Characters

↓

'2'

'5'
```

Binary:

```
Memory Bytes

↓

19 Hex

↓

00011001
```

Different storage.

Same value.

---

# Memory Visualization

Program:

```
int age = 25;
```

Using:

```c
fprintf()
```

```
25

↓

Characters

↓

Text File
```

---

Using:

```c
fwrite()
```

```
25

↓

Memory Bytes

↓

Binary File
```

---

# File Size Comparison

Suppose:

```
100000
```

Text:

```
6 Characters
```

Binary:

```
4 Bytes
```

Binary is usually more compact for numeric data.

---

# Speed Comparison

Text:

```
Read Characters

↓

Convert To Integer

↓

Use
```

Binary:

```
Read Bytes

↓

Use Immediately
```

Binary usually requires less parsing.

---

# Human Readability

Text:

```
Name: Vinay

Age: 21
```

Easy to read.

---

Binary:

```
8A 22 1F 9C ...
```

Looks meaningless without knowing the format.

---

# Text File Example

Program:

```c
fprintf(fp,
        "%d",
        25);
```

File:

```
25
```

Open using:

```
Notepad

VS Code

Vim
```

Readable.

---

# Binary File Example

Program:

```c
fwrite(&age,
       sizeof(int),
       1,
       fp);
```

Open using:

```
Notepad
```

Result:

```
Unreadable Symbols
```

Because the bytes are not intended as text.

---

# When Should You Use Text Files?

When humans need to read or edit the data.

Examples:

```
Configuration

Logs

CSV

JSON

XML

Source Code

README
```

---

# When Should You Use Binary Files?

When performance and compact storage matter.

Examples:

```
Images

Videos

Games

Database Pages

Compiled Programs

Audio

Machine Learning Models
```

---

# Text File Advantages

- Human readable
- Easy to edit
- Easy to debug
- Portable across many systems
- Great for configuration

---

# Text File Disadvantages

- Larger file size
- Slower parsing
- Conversion overhead
- Not ideal for large numeric datasets

---

# Binary File Advantages

- Smaller
- Faster
- Efficient
- No conversion required
- Better for structured data

---

# Binary File Disadvantages

- Difficult to read manually
- Requires software to interpret
- Can have portability concerns (endianness, structure padding, etc.)

---

# Real-World Example — Database

PostgreSQL stores:

```
Tables

Indexes

Pages
```

as binary.

Reason:

```
Fast

Compact
```

---

# Real-World Example — Image

JPEG:

```
Pixels

↓

Binary
```

Millions of bytes.

Not text.

---

# Real-World Example — MP4

Movie:

```
Frames

↓

Binary
```

Video players understand the format.

Humans cannot read it directly.

---

# Real-World Example — Configuration File

Example:

```
config.ini
```

```
Host=localhost

Port=8080
```

Easy for humans.

---

# Real-World Example — Linux Executable

Program:

```
ls
```

File:

```
ELF Executable
```

Binary.

The Linux kernel loads it into memory and executes it.

---

# Comparison Table

| Feature | Text File | Binary File |
|----------|-----------|-------------|
| Human Readable | ✅ Yes | ❌ No |
| Smaller Size | ❌ Usually No | ✅ Yes |
| Faster | ❌ Usually No | ✅ Yes |
| Easy Debugging | ✅ Yes | ❌ No |
| Easy Editing | ✅ Yes | ❌ No |
| Direct Memory Storage | ❌ No | ✅ Yes |

---

# Common Mistakes

---

## Thinking Binary Means Encrypted

Wrong.

Binary simply means the data is stored as raw bytes.

Encryption is a completely different concept.

---

## Opening Binary Files in a Text Editor

Wrong tool.

Use:

- Hex Editor
- Binary Viewer

instead.

---

## Using `fprintf()` for Images

Wrong.

Images should use:

```c
fwrite()
```

---

## Assuming Binary Files Are Always Portable

Binary files containing native data structures may not be portable across systems with different:

- Endianness
- Integer sizes
- Structure padding
- Floating-point formats

Portable binary formats define these details explicitly.

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *text =
        fopen("age.txt","w");

    FILE *binary =
        fopen("age.bin","wb");

    int age = 25;

    fprintf(text,
            "%d",
            age);

    fwrite(&age,
           sizeof(age),
           1,
           binary);

    fclose(text);
    fclose(binary);

    return 0;
}
```

Result:

```
age.txt

↓

25
```

```
age.bin

↓

Raw Binary Bytes
```

---

# Hands-on Labs

## Lab 1

Create:

```
numbers.txt
```

using:

```c
fprintf()
```

Compare the file with:

```
numbers.bin
```

created using:

```c
fwrite()
```

---

## Lab 2

Open both files.

Compare:

- File size
- Contents

---

## Lab 3

Store:

```
100 Integers
```

Compare the storage size of text and binary versions.

---

## Lab 4

Create a binary file.

Inspect it using a hex editor.

---

## Lab 5

Save the same data as:

- CSV
- Binary

Measure loading time.

---

# Interview Questions

### What is a text file?

A file whose bytes represent characters using an encoding such as ASCII or UTF-8.

---

### What is a binary file?

A file that stores raw bytes without converting them into human-readable text.

---

### Which is faster?

Binary files are generally faster to read and write because they avoid text parsing.

---

### Which is easier for humans?

Text files.

---

### Why do databases use binary files?

Because they are compact and efficient for storing structured data.

---

# Summary

```
Everything

↓

Bytes

↓

Interpretation

↓

Text

or

Binary
```

Text:

```
Characters

↓

Humans
```

Binary:

```
Raw Bytes

↓

Programs
```

## Key Takeaways

- Every file is ultimately binary.
- Text files store bytes that represent characters.
- Binary files store raw memory bytes.
- Text files are easier for humans to read and edit.
- Binary files are usually smaller and faster.
- Choose text for readability and binary for efficiency.
- Understanding the difference is essential for databases, operating systems, networking, file systems, and systems programming.

---

# Next Chapter

## Chapter 74 — File Position (`ftell()`, `fseek()`, `rewind()`)

You'll learn:

- What the file pointer (position indicator) is
- How files are read sequentially
- `ftell()` — Find the current position
- `fseek()` — Move to any position
- `rewind()` — Return to the beginning
- Random access vs sequential access
- How databases and operating systems jump directly to specific records