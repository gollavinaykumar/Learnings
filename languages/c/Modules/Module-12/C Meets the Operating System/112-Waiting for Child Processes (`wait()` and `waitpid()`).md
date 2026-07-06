# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 112 — Waiting for Child Processes (`wait()` and `waitpid()`)

---

# Most beginners think:

> "When a child process finishes, Linux immediately removes it."

Not true.

When a child process exits,

Linux **does not immediately delete it**.

Instead, it becomes a:

```
Zombie Process
```

until its parent collects its exit status.

This is why every Unix programmer must understand:

```
wait()

and

waitpid()
```

---

# Learning Objectives

After this chapter you will understand:

- Why parents wait for children
- Zombie processes
- `wait()`
- `waitpid()`
- Exit status
- Blocking vs non-blocking waits
- Process synchronization
- Parent-child communication basics

---

# Life Cycle of a Child Process

Suppose:

```
Parent

↓

fork()

↓

Child

↓

Runs

↓

exit()

↓

Zombie

↓

wait()

↓

Removed
```

Notice:

```
Zombie

↓

Still Exists
```

until the parent waits.

---

# Why Keep a Zombie?

The operating system keeps:

- PID
- Exit status
- Resource usage (implementation-dependent)
- Accounting information

The parent may need this information.

If Linux deleted the child immediately,

the parent could never know:

```
Did It Succeed?

↓

Did It Fail?
```

---

# The Problem

Parent:

```c
fork();
```

Child:

```c
exit(5);
```

Question:

How does the parent discover:

```
5
```

The answer:

```
wait()
```

---

# What is `wait()`?

Prototype:

```c
#include <sys/wait.h>

pid_t wait(int *status);
```

Meaning:

```
Wait

↓

Any Child

↓

Return PID
```

It blocks until one child finishes.

---

# First Example

```c
pid_t pid = fork();

if(pid == 0)
{
    exit(0);
}

wait(NULL);

printf("Child Finished\n");
```

Output:

```
Child Finished
```

Only after the child exits.

---

# Visualization

```
Parent

↓

wait()

↓

Blocked

↓

Child Ends

↓

Kernel Wakes Parent

↓

Continue
```

---

# Blocking

Suppose:

Child:

```
Runs

↓

5 Seconds
```

Parent:

```
wait()

↓

Sleep

↓

5 Seconds

↓

Continue
```

The parent cannot continue until the child terminates.

---

# Return Value

Suppose:

```c
pid = wait(NULL);
```

Returns:

```
Child PID
```

Failure:

```
-1
```

---

# Waiting for Exit Status

Example:

```c
int status;

wait(&status);
```

Now the parent receives encoded information about how the child terminated.

---

# Why Isn't Status Just an Integer?

Because Linux stores multiple pieces of information:

- Normal exit?
- Signal?
- Exit code?
- Core dump? (platform-dependent)

Instead of reading the raw integer directly,

use helper macros.

---

# Status Macros

Check:

```c
WIFEXITED(status)
```

Meaning:

```
Did Child Exit Normally?
```

If true:

```
Retrieve Exit Code
```

using:

```c
WEXITSTATUS(status)
```

---

# Example

Child:

```c
exit(7);
```

Parent:

```c
int status;

wait(&status);

printf("%d\n",
WEXITSTATUS(status));
```

Output:

```
7
```

---

# Complete Flow

```
Child

↓

exit(7)

↓

Kernel Stores Status

↓

Zombie

↓

Parent wait()

↓

Kernel Returns Status

↓

Zombie Removed
```

---

# Multiple Children

Suppose:

```
Parent

↓

fork()

↓

Child A

↓

fork()

↓

Child B
```

Calling:

```c
wait(NULL);
```

collects:

```
One Child
```

The next:

```c
wait(NULL);
```

collects another.

---

# `waitpid()`

Sometimes you don't want:

```
Any Child
```

You want:

```
Specific Child
```

Prototype:

```c
waitpid(
pid,
&status,
options);
```

---

# Example

```c
waitpid(
childPid,
NULL,
0);
```

Meaning:

```
Wait

↓

Only This Child
```

---

# Non-Blocking Wait

Sometimes the parent shouldn't stop.

Use:

```c
waitpid(
pid,
&status,
WNOHANG);
```

If child is still running:

```
Return Immediately
```

No blocking.

---

# Visualization

Blocking:

```
Parent

↓

Sleep

↓

Child Ends

↓

Continue
```

Non-Blocking:

```
Parent

↓

Check

↓

Still Running?

↓

Continue Working
```

---

# Process Synchronization

Example:

```
Parent

↓

Create Child

↓

Wait

↓

Read Output

↓

Continue
```

This guarantees the parent proceeds only after the child has completed.

---

# Why Zombies Are Dangerous

Suppose:

Server:

```
Creates

10000

Children
```

Never calls:

```
wait()
```

Result:

```
10000 Zombies
```

Eventually,

the process table fills.

New processes cannot be created.

---

# Orphan Process

Different concept.

Suppose:

```
Parent Dies

↓

Child Still Running
```

Child becomes:

```
Orphan
```

The operating system assigns it a new parent (typically the init/systemd process on Linux).

This is **not** a zombie.

---

# Zombie vs Orphan

Zombie:

```
Child Dead

↓

Parent Alive

↓

Not Collected
```

Orphan:

```
Parent Dead

↓

Child Alive
```

Completely different concepts.

---

# Real-World Example — Bash

Shell:

```
fork()

↓

exec(command)

↓

wait()

↓

Prompt Returns
```

Without `wait()`,

the shell prompt would appear immediately,

even if the command hadn't finished.

---

# Real-World Example — Apache

Master:

```
fork()

↓

Worker

↓

waitpid()
```

Tracks worker termination and starts replacements if necessary.

---

# Real-World Example — Docker

Container runtime:

```
Wait For Main Process

↓

Container Stops
```

Container lifecycle depends on waiting for the main process to exit.

---

# Real-World Example — Build Systems

Tools like:

```
make

cmake

ninja
```

Launch multiple compiler processes and wait for them before continuing.

---

# Common Mistakes

---

## Forgetting `wait()`

Creates zombie processes.

---

## Reading Status Directly

Wrong:

```c
printf("%d", status);
```

Use:

```c
WIFEXITED()

WEXITSTATUS()
```

instead.

---

## Confusing Zombie and Orphan

Zombie:

```
Dead Child
```

Orphan:

```
Living Child
```

---

## Waiting for the Wrong Child

Use:

```c
waitpid()
```

when a specific child must be synchronized.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
    pid_t pid = fork();

    if(pid == 0)
    {
        exit(5);
    }

    int status;

    wait(&status);

    if(WIFEXITED(status))
    {
        printf("Exit Code: %d\n",
               WEXITSTATUS(status));
    }

    return 0;
}
```

Output:

```
Exit Code: 5
```

---

# Hands-on Labs

## Lab 1

Create a parent and child.

Use:

```c
wait()
```

Observe blocking behavior.

---

## Lab 2

Return different exit codes from the child.

Retrieve them using:

```c
WEXITSTATUS()
```

---

## Lab 3

Create multiple children.

Use repeated:

```c
wait()
```

calls to collect them all.

---

## Lab 4

Replace:

```c
wait()
```

with:

```c
waitpid()
```

Wait for a specific child.

---

## Lab 5

Use:

```bash
ps -el
```

(or another appropriate process listing command)

to observe zombie processes before calling `wait()`.

---

# Interview Questions

### Why does a parent call `wait()`?

To synchronize with child processes and collect their termination status.

---

### What is a zombie process?

A terminated child process that remains in the process table until its parent collects its exit status.

---

### What does `wait()` return?

The PID of the terminated child, or `-1` on failure.

---

### What is the difference between `wait()` and `waitpid()`?

`wait()` waits for any child.

`waitpid()` can wait for a specific child and supports additional options.

---

### What does `WEXITSTATUS()` do?

It extracts the child's exit code after confirming normal termination with `WIFEXITED()`.

---

# Summary

```
Parent

↓

fork()

↓

Child

↓

exit()

↓

Zombie

↓

wait()

↓

Removed
```

Comparison:

| `wait()` | `waitpid()` |
|-----------|-------------|
| Waits for any child | Waits for a chosen child |
| Blocking | Blocking or non-blocking |
| Simple | More flexible |

## Key Takeaways

- A child process becomes a zombie after exiting until its parent reaps it.
- `wait()` synchronizes with any child and retrieves its termination status.
- `waitpid()` provides finer control over which child to wait for.
- Use `WIFEXITED()` and `WEXITSTATUS()` to interpret exit status correctly.
- Forgetting to wait for children can create zombie processes.
- Process synchronization with `wait()` is fundamental to Unix process management.

---

# Next Chapter

## Chapter 113 — Process Memory Layout

You'll learn:

- Text segment
- Data segment
- BSS
- Heap
- Stack
- Environment variables
- Command-line arguments
- How memory changes before and after `fork()`
- Virtual memory layout of every Linux process