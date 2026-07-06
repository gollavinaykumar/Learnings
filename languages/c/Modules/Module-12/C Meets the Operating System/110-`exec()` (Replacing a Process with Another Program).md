# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 110 — `exec()` (Replacing a Process with Another Program)

---

# Most beginners think:

> "`fork()` starts another program."

This is one of the biggest misconceptions in Linux.

`fork()` **does NOT** start another program.

It simply creates a copy of the current process.

Suppose you're running:

```
my_program
```

After:

```c
fork();
```

You now have:

```
my_program

↓

Parent

↓

my_program

↓

Child
```

Notice:

Both are still running:

```
my_program
```

So how does Linux run:

```
ls

cat

grep

python

docker

git
```

The answer is:

```
exec()
```

---

# Learning Objectives

After this chapter you will understand:

- Why `exec()` exists
- The exec family
- Process replacement
- Memory replacement
- PID preservation
- Shell execution flow
- `fork()` + `exec()`
- Why `exec()` never returns on success

---

# The Problem

Suppose the shell executes:

```bash
ls
```

If the shell only called:

```c
fork();
```

Then the child would still execute:

```
Shell Code
```

Not:

```
ls
```

The child must completely replace itself.

---

# Enter `exec()`

`exec()` tells the kernel:

> **"Replace this process with a completely different program."**

The process remains.

Everything else changes.

---

# Visualization

Before:

```
PID 500

↓

Shell
```

After:

```
PID 500

↓

ls
```

Same process.

Different program.

---

# Important Concept

`fork()` creates:

```
New Process
```

`exec()` replaces:

```
Current Program
```

These are completely different operations.

---

# The exec Family

Linux provides several related functions.

Examples:

```
execl()

execv()

execle()

execve()

execlp()

execvp()
```

Internally,

they all eventually rely on the underlying `execve()` system call.

---

# Simple Example

```c
#include <unistd.h>

int main()
{
    execl(
        "/bin/ls",
        "ls",
        NULL);

    return 0;
}
```

Output:

```
Directory Listing
```

Notice:

Your program disappears.

`ls` takes over.

---

# What Happens Internally?

Before:

```
PID 100

↓

Program A
```

Call:

```
exec()
```

After:

```
PID 100

↓

Program B
```

No new process.

Only replacement.

---

# Memory Before exec()

Suppose:

```
Program A

↓

Code

↓

Heap

↓

Stack

↓

Global Variables
```

---

# Memory After exec()

Everything is replaced.

```
Program B

↓

New Code

↓

New Heap

↓

New Stack

↓

New Globals
```

Old program memory is discarded.

---

# What Does NOT Change?

The following remain associated with the process:

- Process ID (PID)
- Parent Process ID (PPID)
- Open file descriptors (unless marked otherwise)
- Some process attributes

Everything else related to the old program image is replaced.

---

# PID Demonstration

Before:

```
PID

↓

500
```

After:

```
exec()

↓

Still

500
```

Same process.

New program.

---

# Why Doesn't `exec()` Return?

Consider:

```c
printf("A");

exec(...);

printf("B");
```

Output:

```
A
```

No:

```
B
```

Why?

Because:

```
Program

↓

Replaced
```

There is no old program left to continue executing.

---

# When Does `exec()` Return?

Only if it fails.

Example:

```c
execl(
"/wrong/path",
"ls",
NULL);
```

Result:

```
Failure

↓

Return -1
```

Then your original program continues.

---

# Complete Flow

Program:

```c
printf("Before");

exec(...);

printf("After");
```

Success:

```
Before
```

Failure:

```
Before

After
```

---

# Why Shell Uses Both

Suppose you type:

```bash
ls
```

Shell executes:

```
fork()

↓

Child

↓

exec(ls)

↓

ls Runs

↓

Parent Shell Waits
```

Without `fork()`:

The shell itself would become:

```
ls
```

and disappear.

---

# Visualization

```
Shell

↓

fork()

↓

Parent

↓

Wait

--------------------

Child

↓

exec(ls)

↓

ls
```

This is one of the most important workflows in Unix.

---

# Passing Arguments

Example:

```c
execl(
"/bin/echo",
"echo",
"Hello",
"World",
NULL);
```

Output:

```
Hello World
```

Notice:

```
argv[0]

↓

Program Name
```

is passed explicitly.

---

# Environment Variables

Another member of the exec family allows specifying a custom environment.

Conceptually:

```
Program

↓

Arguments

↓

Environment

↓

exec()
```

We'll study environment variables in a later chapter.

---

# Open File Descriptors

Suppose:

```
Open File

↓

FD 3
```

Call:

```
exec()
```

Descriptor usually remains open,

unless it has been configured to close automatically during `exec()`.

This behavior is important for:

- Shell redirection
- Pipes
- Servers

---

# Why This Matters

Suppose:

```bash
ls > output.txt
```

Shell:

```
Open File

↓

dup2()

↓

FD 1

↓

exec(ls)
```

`ls` writes to:

```
stdout

↓

output.txt
```

Even though it never opened the file itself.

---

# Real-World Example — Bash

Every command:

```
ls

cat

git

docker

python
```

Typically follows:

```
fork()

↓

exec()
```

---

# Real-World Example — Apache

Worker process:

```
fork()

↓

exec(CGI Script)
```

Runs another executable while preserving the process model.

---

# Real-World Example — Docker

Container runtime eventually executes:

```
execve()
```

to start the container's initial process inside the configured environment.

---

# Real-World Example — SSH

After successful login:

```
fork()

↓

exec(shell)
```

Starts the user's login shell.

---

# Common Mistakes

---

## Thinking `fork()` Runs Another Program

Wrong.

Only:

```
exec()
```

loads a different executable.

---

## Forgetting `exec()` Doesn't Return

Successful `exec()` replaces the current program completely.

---

## Assuming PID Changes

The process ID remains the same.

Only the program image changes.

---

## Ignoring Failure

Always check whether `exec()` returned.

If it did,

an error occurred.

---

# Complete Example

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    printf("Before exec()\n");

    execl(
        "/bin/echo",
        "echo",
        "Hello Linux",
        NULL);

    printf("This prints only if exec fails\n");

    return 0;
}
```

Output:

```
Before exec()

Hello Linux
```

(The last `printf()` executes only if `execl()` fails.)

---

# Hands-on Labs

## Lab 1

Write a program that replaces itself with:

```bash
/bin/ls
```

using `execl()`.

---

## Lab 2

Run:

```bash
ps
```

before and after `exec()`.

Observe that the PID remains unchanged.

---

## Lab 3

Pass command-line arguments to:

```c
exec()
```

using `/bin/echo`.

---

## Lab 4

Deliberately provide an invalid executable path.

Handle the resulting error.

---

## Lab 5

Draw the complete sequence for:

```bash
ls
```

showing:

- Shell
- `fork()`
- Child
- `exec()`
- `ls`

---

# Interview Questions

### What does `exec()` do?

It replaces the current process image with a new program.

---

### Does `exec()` create a new process?

No.

It reuses the existing process.

---

### What happens to the PID after `exec()`?

It remains the same.

---

### Why doesn't `exec()` normally return?

Because the original program has been completely replaced.

---

### Why do shells use both `fork()` and `exec()`?

`fork()` creates a child process.

`exec()` replaces the child with the requested command while allowing the parent shell to continue running.

---

# Summary

```
fork()

↓

Creates

↓

New Process

----------------------------

exec()

↓

Replaces

↓

Current Program
```

Typical Shell Workflow:

```
Shell

↓

fork()

↓

Child

↓

exec(command)

↓

Command Runs

↓

Parent Waits
```

## Key Takeaways

- `fork()` duplicates a process; `exec()` replaces its program image.
- `exec()` does not create a new process.
- The PID remains unchanged across `exec()`.
- On success, `exec()` never returns to the old program.
- The `fork()` → `exec()` pattern is the foundation of Unix command execution.
- Understanding this workflow is essential for shells, servers, containers, and process management.

---

# Next Chapter

## Chapter 111 — Process Termination (`exit()`, `_exit()`, and `return`)

You'll learn:

- How processes end
- `exit()` vs `_exit()`
- Returning from `main()`
- Exit status codes
- Zombie processes
- Process cleanup
- Why child processes often use `_exit()` after `fork()`