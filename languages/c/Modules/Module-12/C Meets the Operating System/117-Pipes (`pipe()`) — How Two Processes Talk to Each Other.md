# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 117 — Pipes (`pipe()`) — How Two Processes Talk to Each Other

---

# Most beginners think:

> "Processes cannot communicate."

Actually...

Processes communicate all the time.

Examples:

```bash
ls | grep txt

cat file.txt | wc -l

ps aux | grep nginx

docker ps | grep redis
```

The `|` symbol is one of the most powerful ideas in Unix.

It allows one process to send its output **directly** to another process.

This happens using:

```
Pipe
```

---

# Learning Objectives

After this chapter you will understand:

- What a pipe is
- Anonymous pipes
- `pipe()`
- Read end
- Write end
- Parent-child communication
- File descriptors in pipes
- Shell pipelines

---

# The Problem

Suppose:

Program A produces data.

```
Numbers

↓

???

↓

Program B
```

How can Program B receive that data without using files?

Linux provides:

```
Pipe
```

---

# What is a Pipe?

A pipe is:

> **A unidirectional communication channel between processes.**

Think of it as an in-memory tunnel.

```
Process A

↓

Pipe

↓

Process B
```

---

# Think About a Water Pipe

Imagine:

```
Tap

↓

Water Pipe

↓

Bucket
```

Water flows only one direction.

A Unix pipe works similarly.

```
Writer

↓

Pipe

↓

Reader
```

---

# Anonymous Pipe

Created using:

```c
pipe(fd);
```

Kernel returns:

```
fd[0]

↓

Read End

----------------

fd[1]

↓

Write End
```

---

# Visualization

```
+----------------------+
|      Pipe            |
+----------------------+

fd[1]

↓

Write

=====================>

Read

↓

fd[0]
```

Data always flows in one direction.

---

# Creating a Pipe

Example:

```c
int fd[2];

pipe(fd);
```

After success:

```
fd[0]

↓

Read

fd[1]

↓

Write
```

---

# Writing

Program:

```c
write(
fd[1],
"Hello",
5);
```

Data enters:

```
Pipe Buffer
```

---

# Reading

Program:

```c
read(
fd[0],
buffer,
5);
```

Reads:

```
Hello
```

from the pipe.

---

# Complete Example

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    int fd[2];

    pipe(fd);

    write(fd[1], "Hello", 5);

    char buf[6];

    read(fd[0], buf, 5);

    buf[5] = '\0';

    printf("%s\n", buf);

    return 0;
}
```

Output:

```
Hello
```

---

# But Why Use `fork()`?

A pipe becomes truly useful when two processes communicate.

Example:

```
Parent

↓

Pipe

↓

Child
```

---

# Parent Writes

```c
write(
fd[1],
"Linux",
5);
```

Child:

```c
read(
fd[0],
buffer,
5);
```

Output:

```
Linux
```

---

# Visualization

```
Parent

↓

fd[1]

↓

Pipe

↓

fd[0]

↓

Child
```

---

# After `fork()`

Remember:

```
fork()

↓

Copies

↓

File Descriptors
```

So:

```
Parent

↓

fd[0]

fd[1]

----------------------

Child

↓

fd[0]

fd[1]
```

Both processes inherit both pipe ends.

---

# Closing Unused Ends

This is extremely important.

Suppose:

Parent only writes.

Close:

```c
close(fd[0]);
```

Child only reads.

Close:

```c
close(fd[1]);
```

---

# Why Close Them?

Suppose the write end stays open forever.

Reader waits:

```
More Data?

↓

Yes?

↓

Wait Forever
```

Closing unused descriptors tells the kernel:

```
No More Data
```

---

# Parent-Child Example

```text
Parent

↓

pipe()

↓

fork()

↓

Parent

↓

close(read)

↓

write()

-------------------------

Child

↓

close(write)

↓

read()
```

This is the classic Unix communication pattern.

---

# Pipe Buffer

Kernel maintains an internal buffer.

Visualization:

```
Writer

↓

AAAA

BBBB

CCCC

↓

Pipe Buffer

↓

Reader
```

The writer and reader do not have to execute at exactly the same moment.

---

# Blocking Behavior

Suppose:

Reader:

```c
read()
```

Pipe empty.

Result:

```
Block

↓

Wait
```

Until data arrives.

---

# Another Case

Writer:

```c
write()
```

Pipe full.

Result:

```
Block

↓

Wait

↓

Reader Reads
```

Then writing continues.

---

# End of File (EOF)

Suppose:

All writers close:

```c
close(fd[1]);
```

Reader:

```c
read()
```

Returns:

```
0

↓

EOF
```

Meaning:

```
No More Data
```

---

# Shell Example

Command:

```bash
ls | grep txt
```

Internally:

```
Shell

↓

pipe()

↓

fork()

↓

Child 1

↓

exec(ls)

↓

Pipe

↓

Child 2

↓

exec(grep)
```

Amazing.

That's exactly how Unix pipelines work.

---

# Visualization

```
ls

↓

stdout

↓

Pipe

↓

stdin

↓

grep
```

No temporary file.

Pure streaming.

---

# Standard Input/Output Redirection

Shell performs:

```
dup2(pipe_write, 1)

↓

ls

-----------------------

dup2(pipe_read, 0)

↓

grep
```

Remember:

```
stdout

↓

FD 1

stdin

↓

FD 0
```

The shell redirects them to the pipe.

---

# Real-World Example — Shell

Every pipeline:

```bash
cat file | sort | uniq
```

Creates multiple pipes.

Each command reads from one pipe and writes to the next.

---

# Real-World Example — Compiler

Some compiler toolchains connect internal stages using pipes instead of temporary files.

---

# Real-World Example — Docker

Docker captures container output through pipes before forwarding it to the logging subsystem.

---

# Real-World Example — SSH

SSH creates communication channels that conceptually behave similarly,

allowing local input to become remote program input.

---

# Common Mistakes

---

## Forgetting to Close Unused Pipe Ends

May cause:

```
Reader

↓

Never Gets EOF

↓

Blocks Forever
```

---

## Reading from Write End

Wrong:

```c
read(fd[1], ...);
```

---

## Writing to Read End

Wrong:

```c
write(fd[0], ...);
```

---

## Thinking Pipes Are Bidirectional

Anonymous pipes are:

```
One Direction
```

Two-way communication typically requires two pipes or another IPC mechanism.

---

# Complete Parent-Child Example

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    int fd[2];

    pipe(fd);

    if(fork() == 0)
    {
        close(fd[1]);

        char buf[100];

        int n = read(fd[0], buf, sizeof(buf)-1);

        buf[n] = '\0';

        printf("Child: %s\n", buf);
    }
    else
    {
        close(fd[0]);

        write(fd[1], "Hello Child", 11);
    }

    return 0;
}
```

Possible output:

```
Child: Hello Child
```

---

# Hands-on Labs

## Lab 1

Create a pipe.

Write a string.

Read it back.

---

## Lab 2

Create a parent and child.

Send messages through the pipe.

---

## Lab 3

Experiment with closing different pipe ends.

Observe blocking behavior.

---

## Lab 4

Run:

```bash
ls | wc -l
```

Draw the internal process and pipe diagram.

---

## Lab 5

Use:

```bash
strace ls | grep pipe
```

or trace a shell pipeline to observe the underlying `pipe()` system calls.

---

# Interview Questions

### What is a pipe?

A unidirectional communication channel used for inter-process communication.

---

### What does `pipe()` return?

Two file descriptors:

- Read end
- Write end

---

### Why are unused pipe ends closed?

To avoid resource leaks and to ensure readers receive EOF when writing is complete.

---

### Why does `read()` block on a pipe?

Because no data is currently available.

---

### How does `ls | grep txt` work internally?

The shell creates a pipe, forks child processes, redirects standard input/output using `dup2()`, and then executes `ls` and `grep`.

---

# Summary

```
pipe()

↓

fd[0]

↓

Read

----------------------

fd[1]

↓

Write
```

Parent-Child Communication:

```
Parent

↓

write()

↓

Pipe

↓

read()

↓

Child
```

Shell Pipeline:

```
ls

↓

Pipe

↓

grep

↓

Pipe

↓

wc
```

## Key Takeaways

- Pipes provide simple one-way communication between processes.
- `pipe()` creates two file descriptors: one for reading and one for writing.
- Parent and child processes inherit pipe descriptors after `fork()`.
- Closing unused pipe ends is essential for correct behavior.
- Shell pipelines are implemented using `pipe()`, `fork()`, `dup2()`, and `exec()`.
- Pipes are one of the foundational IPC mechanisms in Unix.

---

# Next Chapter

## Chapter 118 — Named Pipes (FIFOs)

You'll learn:

- What a FIFO is
- Anonymous pipe vs named pipe
- `mkfifo()`
- Communication between unrelated processes
- Blocking behavior
- Real-world uses of FIFOs
- How Linux exposes FIFOs as special files