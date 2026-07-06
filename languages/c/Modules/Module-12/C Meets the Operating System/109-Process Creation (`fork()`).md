# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 109 — Process Creation (`fork()`)

---

# Most beginners think:

> "When I run a program, the operating system executes the `.exe` or binary."

That's only part of the story.

When you execute:

```bash
./my_program
```

Linux **doesn't execute the file directly**.

Instead it creates something much more important:

```
Process
```

The executable file is just **code stored on disk**.

The operating system creates a **running instance** of that program called a process.

---

# Learning Objectives

After this chapter you will understand:

- What a process is
- Program vs Process
- `fork()`
- Parent process
- Child process
- PID
- PPID
- Process trees
- Copy-on-Write
- Why shells use `fork()`

---

# Program vs Process

Many beginners confuse these.

A program is:

```
Code

↓

Stored On Disk
```

Example:

```
/bin/ls

/usr/bin/git

./my_program
```

Nothing is running.

---

A process is:

```
Running Program

↓

Memory

↓

CPU

↓

Stack

↓

Heap

↓

Registers
```

One program can create many processes.

---

# Visualization

```
Program

↓

my_program

↓

fork()

↓

Process A

↓

Process B
```

One executable.

Multiple running processes.

---

# Think About Chrome

Suppose you open:

```
Chrome
```

Then:

```
10 Tabs
```

Do you think Chrome has only one process?

Usually:

```
Chrome

↓

Process

↓

Tab 1

↓

Process

↓

Tab 2

↓

Process

↓

GPU

↓

Process
```

Many processes.

One program.

---

# What is `fork()`?

`fork()` is the Linux system call that creates a new process.

Prototype:

```c
#include <unistd.h>

pid_t fork(void);
```

The operating system duplicates the calling process.

---

# Before `fork()`

Suppose:

```
Process

↓

PID 100
```

Only one process exists.

---

# After `fork()`

```
Process 100

↓

fork()

↓

Parent (100)

↓

Child (101)
```

Now two processes exist.

---

# Amazing Fact

Both processes continue executing **from the next instruction after `fork()`**.

Example:

```c
printf("Before\n");

fork();

printf("After\n");
```

Output:

```
Before

After

After
```

Why?

Because:

```
Parent

↓

printf()

----------------

Child

↓

printf()
```

Both execute the remaining code.

---

# Return Value of `fork()`

This is one of the most important concepts.

Parent receives:

```
Child PID
```

Child receives:

```
0
```

Failure:

```
-1
```

---

# Visualization

```
fork()

↓

Parent

↓

101

----------------

Child

↓

0

----------------

Failure

↓

-1
```

---

# Example

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    pid_t pid = fork();

    if(pid == 0)
    {
        printf("Child\n");
    }
    else
    {
        printf("Parent\n");
    }

    return 0;
}
```

Possible output:

```
Parent

Child
```

Or:

```
Child

Parent
```

Order is **not guaranteed**.

The scheduler decides.

---

# Process Memory

After `fork()`:

```
Parent

↓

Code

↓

Stack

↓

Heap

---------------------

Child

↓

Code

↓

Stack

↓

Heap
```

Initially,

they appear identical.

---

# Are They Sharing Memory?

Many beginners think:

```
Parent

↓

Same Variables

↓

Child
```

Wrong.

Each process has its own virtual address space.

Changing a variable in one process does not change it in the other.

---

# Example

```c
int x = 10;

fork();

x++;

printf("%d\n", x);
```

Output:

```
11

11
```

Each process increments its own copy.

---

# Copy-on-Write (COW)

Question:

Does Linux immediately copy every byte of memory?

No.

That would be very slow.

Instead,

Linux uses:

```
Copy-On-Write
```

---

# Before Writing

```
Parent

↓

Page A

↑

Child
```

Both processes share the same physical memory page.

---

# Parent Writes

Suppose:

```
Parent

↓

Changes Variable
```

Kernel:

```
Copy Page

↓

Parent Gets New Copy

↓

Child Keeps Old Copy
```

Only modified pages are duplicated.

This is why `fork()` is efficient.

---

# Process IDs

Every process has a unique:

```
PID

(Process ID)
```

Example:

```
100

101

102

103
```

No two running processes share the same PID.

---

# Parent Process ID

Every process also has:

```
PPID

(Parent Process ID)
```

Example:

```
Shell

PID 500

↓

Program

PID 700

PPID 500
```

The child knows who created it.

---

# Getting PID

Example:

```c
printf("%d\n",
getpid());
```

Returns:

```
Current Process ID
```

---

# Getting Parent PID

Example:

```c
printf("%d\n",
getppid());
```

Returns:

```
Parent Process ID
```

---

# Process Tree

Imagine:

```
init/systemd

↓

bash

↓

gcc

↓

my_program

↓

fork()

↓

Child
```

Processes form a tree,

not a flat list.

---

# Why Shell Uses `fork()`

Suppose you type:

```bash
ls
```

Does the shell disappear?

No.

Instead:

```
Shell

↓

fork()

↓

Child

↓

exec(ls)

↓

Shell Waits
```

The shell survives.

The child runs the command.

---

# Scheduler

After `fork()`:

```
Parent

↓

CPU?

----------------

Child

↓

CPU?
```

The scheduler decides which process runs next.

Never assume execution order.

---

# Real-World Example — Web Server

Server:

```
Accept Client

↓

fork()

↓

Child Handles Client

↓

Parent Accepts Next Client
```

Classic Unix server model.

---

# Real-World Example — Apache

Older Apache versions commonly used:

```
Master

↓

fork()

↓

Worker

↓

fork()

↓

Worker
```

Multiple worker processes handled requests.

---

# Real-World Example — Shell

Every command:

```bash
pwd

ls

cat

grep
```

Typically begins with:

```
fork()

↓

exec()
```

---

# Real-World Example — Docker

Inside a container,

processes are still ordinary Linux processes.

The kernel manages them just like any others.

---

# Common Mistakes

---

## Thinking `fork()` Runs Another Program

Wrong.

`fork()` duplicates the current process.

Running another program is done later using `exec()`.

---

## Assuming Parent Runs First

Wrong.

Scheduling order is not guaranteed.

---

## Thinking Variables Are Shared

Each process has its own virtual memory.

---

## Forgetting the Return Value

Always check:

```c
pid == 0
```

to distinguish child from parent.

---

# Complete Example

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    pid_t pid = fork();

    if(pid == 0)
    {
        printf("Child PID: %d\n",
               getpid());
    }
    else
    {
        printf("Parent PID: %d\n",
               getpid());
    }

    return 0;
}
```

Possible output:

```
Parent PID: 4321

Child PID: 4322
```

(The order may vary.)

---

# Hands-on Labs

## Lab 1

Create a program that calls:

```c
fork();
```

Print:

- PID
- PPID

for both parent and child.

---

## Lab 2

Modify a variable after `fork()`.

Observe that parent and child have independent copies.

---

## Lab 3

Run:

```bash
ps -ef
```

while your program is executing.

Identify both processes.

---

## Lab 4

Draw the process tree created by your program.

---

## Lab 5

Read about Copy-on-Write and explain why Linux doesn't immediately duplicate all memory during `fork()`.

---

# Interview Questions

### What is a process?

A running instance of a program with its own memory, CPU state, and operating system resources.

---

### What does `fork()` do?

Creates a new process by duplicating the calling process.

---

### What does `fork()` return?

- `0` in the child
- Child PID in the parent
- `-1` on failure

---

### Do parent and child share variables after `fork()`?

No.

Each process has its own virtual address space.

---

### What is Copy-on-Write?

A memory optimization where parent and child initially share pages until one process modifies a page.

---

# Summary

```
Program

↓

Process

↓

fork()

↓

Parent

↓

Child
```

Return Values:

```
Parent

↓

Child PID

---------------------

Child

↓

0

---------------------

Failure

↓

-1
```

Memory:

```
Initially

↓

Shared Pages

↓

Copy-On-Write

↓

Independent Memory
```

## Key Takeaways

- A program is passive code on disk; a process is a running instance.
- `fork()` duplicates the current process.
- Parent and child continue execution after the `fork()` call.
- The return value distinguishes parent from child.
- Linux uses Copy-on-Write to make `fork()` efficient.
- `fork()` is fundamental to shells, servers, and Unix process management.

---

# Next Chapter

## Chapter 110 — `exec()` (Replacing a Process with Another Program)

You'll learn:

- Why `fork()` alone isn't enough
- The `exec` family of functions
- How shells launch commands
- What happens to process memory during `exec()`
- Why the PID stays the same
- The complete `fork()` → `exec()` workflow used throughout Unix