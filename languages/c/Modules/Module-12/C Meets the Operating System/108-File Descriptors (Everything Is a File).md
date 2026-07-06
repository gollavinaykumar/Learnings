# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 108 — File Descriptors (Everything Is a File)

---

# Most beginners think:

> "Files are files, sockets are sockets, and terminals are terminals."

Linux doesn't think that way.

To Linux,

almost everything is treated as a:

```
File
```

Examples:

- Regular files
- Directories
- Terminals
- Pipes
- Sockets
- Devices
- USB ports
- Serial ports

How can one operating system treat all of these so similarly?

The answer is:

```
File Descriptors
```

---

# Learning Objectives

After this chapter you will understand:

- What file descriptors are
- Standard input
- Standard output
- Standard error
- Kernel file tables
- Process file descriptor tables
- `dup()`
- `dup2()`
- Unix philosophy

---

# What is a File Descriptor?

A file descriptor (FD) is:

> **A small integer used by the kernel to identify an open file or I/O resource for a process.**

Example:

```
3

↓

notes.txt
```

The number itself is not the file.

It is an identifier.

---

# First Example

Program:

```c
int fd =
open(
"notes.txt",
O_RDONLY);
```

Suppose:

Kernel returns:

```
3
```

Meaning:

```
fd = 3
```

Your program never manipulates the disk directly.

It uses:

```
3
```

to refer to the open resource.

---

# Visualization

```
Program

↓

fd = 3

↓

Kernel

↓

notes.txt
```

The descriptor is simply a handle.

---

# Standard File Descriptors

Every Unix process starts with three file descriptors.

| Number | Meaning |
|---------|----------|
| 0 | Standard Input (`stdin`) |
| 1 | Standard Output (`stdout`) |
| 2 | Standard Error (`stderr`) |

These are automatically opened when the process starts.

---

# Standard Input

Descriptor:

```
0
```

Usually connected to:

```
Keyboard
```

Example:

```c
read(
0,
buffer,
100);
```

Reads from standard input.

---

# Standard Output

Descriptor:

```
1
```

Usually connected to:

```
Terminal
```

Example:

```c
write(
1,
"Hello\n",
6);
```

Writes to standard output.

---

# Standard Error

Descriptor:

```
2
```

Usually connected to:

```
Terminal
```

Used for:

- Errors
- Warnings
- Diagnostics

Example:

```c
write(
2,
"Error\n",
6);
```

---

# Process File Descriptor Table

Each process owns its own FD table.

Example:

```
Process

↓

FD Table

-------------------

0 → Keyboard

1 → Terminal

2 → Terminal

3 → notes.txt

4 → socket
```

The numbers are meaningful only within that process.

---

# Kernel File Table

The process table entries point to kernel objects.

Conceptually:

```
Process FD Table

↓

FD 3

↓

Kernel File Object

↓

Filesystem

↓

Disk
```

Multiple descriptors can refer to the same kernel file object.

---

# Visualization

```
Process

↓

FD 3

↓

Kernel File Object

↓

Disk File
```

---

# Reading a File

Program:

```c
char buffer[100];

read(
fd,
buffer,
100);
```

Flow:

```
FD

↓

Kernel

↓

Find File

↓

Read Data

↓

Copy To Buffer
```

---

# Closing a File

Program:

```c
close(fd);
```

Kernel:

```
Release File Descriptor

↓

Decrease Reference Count

↓

Free Resources
```

The descriptor becomes available for reuse.

---

# Opening Multiple Files

Example:

```c
fd1 = open(...);

fd2 = open(...);

fd3 = open(...);
```

Possible result:

```
3

4

5
```

Descriptors are generally allocated using the lowest available number.

---

# Why Integers?

Instead of returning:

```
Huge Structure
```

Kernel returns:

```
3
```

Fast,

simple,

efficient.

The kernel stores the complex details internally.

---

# Everything Is a File

Examples:

```
Regular File

↓

FD

-------------------

Socket

↓

FD

-------------------

Pipe

↓

FD

-------------------

Terminal

↓

FD
```

All use:

```
read()

write()

close()
```

The same interface works across many resource types.

---

# Socket Example

Networking:

```c
socket(...)
```

Returns:

```
FD = 4
```

Then:

```c
read(4,...)

write(4,...)
```

Exactly like reading and writing a file.

---

# Pipe Example

Parent:

```
Pipe

↓

FD 3

FD 4
```

Child process communicates using ordinary file descriptor operations.

---

# Terminal Example

Keyboard:

```
FD 0
```

Screen:

```
FD 1
```

Error output:

```
FD 2
```

No special API required.

---

# `dup()`

Suppose:

```
FD 3

↓

notes.txt
```

Call:

```c
dup(3);
```

Returns:

```
FD 4
```

Now:

```
3

↓

notes.txt

↑

4
```

Both descriptors refer to the same open file description.

---

# Visualization

```
FD 3

↓

Kernel File Object

↑

FD 4
```

Two handles,

one underlying open file.

---

# `dup2()`

Example:

```c
dup2(
fd,
1);
```

Meaning:

```
Replace

Standard Output

↓

fd
```

Now:

```
printf()

↓

Writes To File
```

Instead of the terminal.

---

# Output Redirection

Command:

```bash
ls > output.txt
```

How?

Conceptually:

```
Shell

↓

Open output.txt

↓

dup2(fd,1)

↓

Execute ls
```

`ls` still writes to:

```
stdout

↓

FD 1
```

But FD 1 now refers to the file.

---

# Why `stderr` Exists

Suppose:

```bash
program > output.txt
```

Normal output:

↓

```
output.txt
```

Errors:

↓

Still appear on the terminal,

because they use:

```
FD 2
```

This separation is extremely useful.

---

# Real-World Example — Web Server

Connection accepted:

```
Socket

↓

FD 15
```

Server reads:

```c
read(15,...)
```

and writes:

```c
write(15,...)
```

The socket behaves like any other file descriptor.

---

# Real-World Example — Docker

Container logs:

```
stdout

↓

FD 1

↓

Docker Logging Driver
```

Docker captures output by observing standard descriptors.

---

# Real-World Example — Shell

Shell redirects:

```
stdin

stdout

stderr
```

using:

```
dup2()
```

before launching child processes.

---

# Real-World Example — Linux Devices

Examples:

```
/dev/null

/dev/tty

/dev/random
```

Open them:

```
open()

↓

FD
```

Interact using:

```
read()

write()
```

Just like ordinary files.

---

# Common Mistakes

---

## Thinking FD Is the File

Wrong.

The descriptor is an integer identifier.

The kernel maintains the actual file object.

---

## Forgetting to Close Files

Every successful:

```c
open()
```

should eventually be matched with:

```c
close()
```

to avoid resource leaks.

---

## Confusing `FILE *` with File Descriptor

`FILE *`

```
C Standard Library

Buffered I/O
```

File Descriptor

```
Kernel Interface

Unbuffered System Calls
```

We'll study their relationship later.

---

## Assuming FD Numbers Are Global

Descriptor numbers are local to each process.

Two different processes can both have:

```
FD = 3
```

pointing to completely different resources.

---

# Complete Example

```c
#include <fcntl.h>
#include <unistd.h>

int main()
{
    int fd =
        open(
            "notes.txt",
            O_RDONLY);

    if(fd == -1)
        return 1;

    close(fd);

    return 0;
}
```

Conceptual Flow:

```
open()

↓

Kernel

↓

FD = 3

↓

close()

↓

Release Descriptor
```

---

# Hands-on Labs

## Lab 1

Write a program that:

- Opens a file
- Prints the returned file descriptor
- Closes it

---

## Lab 2

Replace:

```c
printf()
```

with:

```c
write(1,...)
```

Observe the result.

---

## Lab 3

Use:

```bash
ls > output.txt
```

Explain how `dup2()` enables redirection.

---

## Lab 4

Run:

```bash
ls -l /proc/self/fd
```

(on Linux)

Inspect the currently open file descriptors.

---

## Lab 5

Experiment with:

```c
dup()

dup2()
```

Observe how duplicated descriptors share the same underlying file.

---

# Interview Questions

### What is a file descriptor?

A small integer that identifies an open file or I/O resource within a process.

---

### What are the standard file descriptors?

- 0 → Standard Input
- 1 → Standard Output
- 2 → Standard Error

---

### What does `dup()` do?

Creates a new file descriptor referring to the same open file description.

---

### Why does Unix say "Everything is a file"?

Because many different resources—files, sockets, pipes, terminals, and devices—are accessed through the same file descriptor interface.

---

### What is the difference between `FILE *` and a file descriptor?

`FILE *` is a buffered C library abstraction.

A file descriptor is the low-level kernel interface used by system calls.

---

# Summary

```
Process

↓

FD Table

↓

File Descriptor

↓

Kernel File Object

↓

Hardware
```

Standard Descriptors:

```
0

↓

stdin

----------------

1

↓

stdout

----------------

2

↓

stderr
```

## Key Takeaways

- File descriptors are integer handles to open I/O resources.
- Every process maintains its own file descriptor table.
- Standard input, output, and error correspond to descriptors 0, 1, and 2.
- Files, sockets, pipes, terminals, and devices all use the same descriptor model.
- `dup()` and `dup2()` enable descriptor duplication and I/O redirection.
- The Unix philosophy of "Everything is a file" simplifies programming by providing a consistent interface for many different resources.

---

# Next Chapter

## Chapter 109 — Process Creation (`fork()`)

You'll learn:

- What a process really is
- How `fork()` creates new processes
- Parent vs child processes
- Copy-on-write
- Process IDs (PID/PPID)
- Process trees
- Why every Unix shell depends on `fork()`