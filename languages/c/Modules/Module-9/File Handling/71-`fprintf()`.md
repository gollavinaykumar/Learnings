# 👑 C Mastery Roadmap

# Module 9 — File Handling

# Chapter 71 — `fprintf()`

---

# Most beginners think:

> "`fprintf()` and `fwrite()` do the same thing."

They don't.

This is one of the biggest misunderstandings in C.

`fwrite()` writes **raw bytes**.

`fprintf()` writes **formatted human-readable text**.

Imagine storing:

```c
int age = 25;
```

Using `fwrite()`:

```
Memory

↓

Raw Binary Bytes

↓

Disk
```

Using `fprintf()`:

```
Memory

↓

Convert to Text

↓

"25"

↓

Disk
```

Both store data,

but in completely different ways.

---

# Learning Objectives

After this chapter you will understand:

- What `fprintf()` is
- Why it exists
- Formatted output
- Format specifiers
- Logging
- CSV generation
- Configuration files
- Difference from `printf()`
- Difference from `fwrite()`

---

# Why Do We Need `fprintf()`?

Suppose we have:

```c
int marks = 95;
```

Need to save:

```
95
```

inside:

```
marks.txt
```

If we use:

```c
fwrite()
```

the file contains binary bytes.

A text editor won't show a readable number.

Instead,

we want:

```
95
```

as characters.

That's exactly what `fprintf()` does.

---

# What is `fprintf()`?

`fprintf()` writes **formatted text** into a file.

Syntax:

```c
int fprintf(
    FILE *stream,
    const char *format,
    ...
);
```

Looks almost identical to:

```c
printf()
```

The only difference is:

```
printf()

↓

Console
```

```
fprintf()

↓

File
```

---

# Think Like This

```
Program

↓

Variables

↓

Format

↓

Characters

↓

File
```

---

# First Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("marks.txt","w");

    if(fp == NULL)
        return 1;

    int marks = 95;

    fprintf(fp,
            "%d",
            marks);

    fclose(fp);

    return 0;
}
```

File:

```
95
```

Human readable.

---

# Multiple Values

```c
fprintf(fp,
        "%d %d %d",
        10,
        20,
        30);
```

File:

```
10 20 30
```

---

# Writing Strings

```c
char name[] = "Vinay";

fprintf(fp,
        "%s",
        name);
```

File:

```
Vinay
```

---

# Writing Multiple Data Types

```c
char name[] = "Vinay";

int age = 21;

float cgpa = 8.7;

fprintf(fp,
        "%s %d %.1f",
        name,
        age,
        cgpa);
```

File:

```
Vinay 21 8.7
```

---

# Common Format Specifiers

| Specifier | Meaning |
|-----------|---------|
| `%d` | Integer |
| `%u` | Unsigned Integer |
| `%f` | Floating Point |
| `%.2f` | Float with 2 decimal places |
| `%c` | Character |
| `%s` | String |
| `%x` | Hexadecimal |
| `%p` | Pointer Address |

Exactly the same specifiers used by:

```c
printf()
```

---

# Writing New Lines

```c
fprintf(fp,
        "Apple\n");

fprintf(fp,
        "Banana\n");

fprintf(fp,
        "Orange\n");
```

File:

```
Apple

Banana

Orange
```

---

# Writing a Table

```c
fprintf(fp,
        "ID Name Age\n");

fprintf(fp,
        "1 Vinay 21\n");

fprintf(fp,
        "2 Rahul 22\n");
```

File:

```
ID Name Age

1 Vinay 21

2 Rahul 22
```

---

# Writing CSV Files

CSV:

```
Comma Separated Values
```

Program:

```c
fprintf(fp,
        "ID,Name,Age\n");

fprintf(fp,
        "1,Vinay,21\n");

fprintf(fp,
        "2,Rahul,22\n");
```

Generated file:

```
ID,Name,Age

1,Vinay,21

2,Rahul,22
```

Open it using:

- Microsoft Excel
- Google Sheets
- LibreOffice Calc

---

# Logging

Servers continuously generate logs.

Example:

```c
fprintf(logFile,
        "Server Started\n");

fprintf(logFile,
        "User Login\n");

fprintf(logFile,
        "Database Connected\n");
```

Log File:

```
Server Started

User Login

Database Connected
```

Logging is one of the most common uses of `fprintf()`.

---

# Configuration Files

Suppose:

```
config.txt
```

contains:

```
Port=8080

Host=localhost

Timeout=30
```

Program:

```c
fprintf(fp,
        "Port=%d\n",
        8080);

fprintf(fp,
        "Host=%s\n",
        "localhost");
```

Readable.

Easy to edit.

---

# `printf()` vs `fprintf()`

`printf()`:

```
Variables

↓

Console
```

Example:

```c
printf("Hello");
```

Displays:

```
Hello
```

---

`fprintf()`:

```
Variables

↓

File
```

Example:

```c
fprintf(fp,
        "Hello");
```

Writes:

```
Hello
```

to the file.

---

# `fprintf()` vs `fwrite()`

`fprintf()`:

```
25

↓

Characters

↓

'2'

'5'
```

---

`fwrite()`:

```
25

↓

Binary

↓

Memory Representation
```

The binary format depends on the machine's representation.

---

# Memory Visualization

Variable:

```
95
```

↓

```
fprintf()
```

↓

Characters:

```
'9'

'5'
```

↓

```
File
```

---

# Return Value

`fprintf()` returns:

```
Number of Characters Written
```

or

```
Negative Value
```

if an error occurs.

Example:

```c
int n =
fprintf(fp,
        "Hello");
```

Return:

```
5
```

Because:

```
Hello
```

contains five characters.

---

# Real-World Example — Apache Logs

Apache stores entries like:

```
IP Address

↓

Date

↓

Request

↓

Status Code
```

Each line is written using formatted text.

---

# Real-World Example — CSV Export

Suppose a database exports customer information.

Program:

```
Database

↓

Rows

↓

fprintf()

↓

customers.csv
```

Open directly in Excel.

---

# Real-World Example — Configuration Generator

Many applications generate:

```
.env

config.ini

settings.conf
```

using formatted output.

---

# Common Mistakes

---

## Forgetting `\n`

Wrong:

```c
fprintf(fp,
        "One");

fprintf(fp,
        "Two");
```

File:

```
OneTwo
```

Correct:

```c
fprintf(fp,
        "One\n");

fprintf(fp,
        "Two\n");
```

---

## Using Wrong Format Specifier

Wrong:

```c
fprintf(fp,
        "%d",
        "Hello");
```

Correct:

```c
fprintf(fp,
        "%s",
        "Hello");
```

---

## Using `fprintf()` for Binary Files

Binary files should use:

```c
fwrite()
```

Text files should use:

```c
fprintf()
```

---

# Complete Example

```c
#include <stdio.h>

int main()
{
    FILE *fp =
        fopen("student.txt","w");

    if(fp == NULL)
        return 1;

    char name[] = "Vinay";

    int age = 21;

    float cgpa = 8.75;

    fprintf(fp,
            "Name : %s\n",
            name);

    fprintf(fp,
            "Age  : %d\n",
            age);

    fprintf(fp,
            "CGPA : %.2f\n",
            cgpa);

    fclose(fp);

    return 0;
}
```

Generated file:

```
Name : Vinay

Age  : 21

CGPA : 8.75
```

---

# Hands-on Labs

## Lab 1

Create:

```
students.txt
```

Store:

- Name
- Age
- Marks

using `fprintf()`.

---

## Lab 2

Generate:

```
employees.csv
```

Open it in Excel.

---

## Lab 3

Create:

```
server.log
```

Write:

```
Server Started

Client Connected

Client Disconnected
```

---

## Lab 4

Generate:

```
config.txt
```

using formatted output.

---

## Lab 5

Compare:

```
fprintf()

↓

student.txt
```

with

```
fwrite()

↓

student.bin
```

Open both files using a text editor and compare the results.

---

# Interview Questions

### What does `fprintf()` do?

It writes formatted, human-readable text to a file.

---

### What does `fprintf()` return?

The number of characters written, or a negative value if an error occurs.

---

### What is the difference between `printf()` and `fprintf()`?

`printf()` writes to the console.

`fprintf()` writes to a file stream.

---

### What is the difference between `fprintf()` and `fwrite()`?

`fprintf()` writes formatted text.

`fwrite()` writes raw binary bytes.

---

### Where is `fprintf()` commonly used?

- Log files
- CSV files
- Configuration files
- Reports
- Text exports

---

# Summary

```
Variables

↓

Formatting

↓

Characters

↓

fprintf()

↓

Text File
```

Example:

```
Name

Age

Marks

↓

student.txt
```

## Key Takeaways

- `fprintf()` writes formatted text to a file.
- It works like `printf()`, but writes to a file stream instead of the console.
- It is ideal for logs, reports, CSV files, and configuration files.
- Use the correct format specifiers (`%d`, `%s`, `%f`, etc.).
- For binary data, prefer `fwrite()` instead of `fprintf()`.
- `fprintf()` is widely used in servers, utilities, reporting tools, and configuration generators.

---

# Next Chapter

## Chapter 72 — `fclose()`

You'll learn:

- Why closing files is essential
- What actually happens inside `fclose()`
- File buffers
- Flushing pending writes
- Resource leaks
- Why databases and operating systems always close files properly