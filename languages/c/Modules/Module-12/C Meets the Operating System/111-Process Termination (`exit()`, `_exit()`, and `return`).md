# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 111 — Process Termination (`exit()`, `_exit()`, and `return`)

---

# Most beginners think:

> "When `main()` finishes, the program simply stops."

That's only part of the story.

When a process finishes, Linux doesn't simply erase it.

The operating system performs a long sequence of cleanup operations.

Understanding **how a process dies** is just as important as understanding how it starts.

---

# Learning Objectives

After this chapter you will understand:

- Process termination
- `return` from `main()`
- `exit()`
- `_exit()`
- Exit status
- Process cleanup
- Zombie processes
- Parent notification
- Why `_exit()` exists

---

# How Can a Process End?

A process may terminate by:

```
return from main()

↓

exit()

↓

_exit()

↓

Signal

↓

Fatal Error
```

All eventually end the process,

but not in exactly the same way.

---

# Returning From `main()`

Example:

```c
int main()
{
    return 0;
}
```

Many beginners think:

```
Program Ends
```

Reality:

```
return

↓

exit(0)

↓

Kernel Cleanup
```

Returning from `main()` is conceptually equivalent to calling:

```c
exit(0);
```

---

# Exit Status

Every process returns an integer to its parent.

Example:

```c
return 0;
```

Means:

```
Success
```

Example:

```c
return 1;
```

Means:

```
Failure

(or some error)
```

The exact meaning depends on the program.

---

# Why Exit Codes Matter

Shell:

```bash
./program
```

After completion:

```bash
echo $?
```

Output:

```
0
```

Meaning:

```
Previous Program

↓

Success
```

---

# Example

Program:

```c
return 5;
```

Shell:

```bash
echo $?
```

Output:

```
5
```

The shell receives the child's exit status.

---

# `exit()`

Include:

```c
#include <stdlib.h>
```

Example:

```c
exit(0);
```

This ends the process **gracefully**.

---

# What Does `exit()` Do?

Conceptually:

```
Call exit()

↓

Flush stdio buffers

↓

Run atexit() handlers

↓

Close FILE streams

↓

Notify Kernel

↓

Process Ends
```

It performs orderly cleanup.

---

# Buffered Output Example

Program:

```c
printf("Hello");
```

Suppose the text is still in a buffer.

Calling:

```c
exit(0);
```

Flushes the buffer,

so:

```
Hello
```

appears.

---

# `_exit()`

Include:

```c
#include <unistd.h>
```

Example:

```c
_exit(0);
```

Unlike `exit()`,

it immediately terminates the process.

No C library cleanup occurs.

---

# Comparison

`exit()`

```
Flush Buffers

↓

Cleanup

↓

Terminate
```

`_exit()`

```
Terminate Immediately
```

---

# Why Does `_exit()` Exist?

Consider:

```
Parent

↓

fork()

↓

Child
```

Both processes inherit:

```
stdio Buffers
```

Suppose:

```c
printf("Hello");
```

Buffer not yet flushed.

If the child calls:

```c
exit();
```

the inherited buffer may be flushed again,

causing duplicate output.

---

# Visualization

Before `fork()`:

```
Buffer

↓

"Hello"
```

After `fork()`:

```
Parent Buffer

↓

Hello

---------------------

Child Buffer

↓

Hello
```

If both call:

```
exit()
```

Possible output:

```
HelloHello
```

---

# Correct Solution

Child process:

```c
_exec(...)

↓

Failure?

↓

_exit()
```

Avoids flushing inherited stdio buffers again.

This is one reason `_exit()` is commonly used after `fork()`.

---

# `atexit()`

Example:

```c
atexit(cleanup);
```

When:

```c
exit();
```

is called:

```
cleanup()

↓

Runs Automatically
```

Useful for releasing application-level resources.

---

# Signals

Processes can also terminate because of signals.

Examples:

```
SIGINT

Ctrl+C

----------------

SIGTERM

Terminate

----------------

SIGKILL

Immediate Kill
```

We'll study signals in later chapters.

---

# Fatal Errors

Example:

```
Segmentation Fault
```

Flow:

```
Invalid Memory

↓

CPU Exception

↓

Kernel

↓

SIGSEGV

↓

Process Ends
```

---

# Zombie Processes

Question:

When a child finishes,

does Linux erase it immediately?

No.

Instead:

```
Child Ends

↓

Zombie

↓

Parent Reads Status

↓

Kernel Removes Process
```

---

# Why Zombies Exist

Parent needs:

```
Exit Status

↓

PID

↓

Accounting Information
```

Kernel keeps minimal information until the parent collects it.

We'll study:

```
wait()

waitpid()
```

next.

---

# Process Cleanup

When a process exits,

the kernel releases:

- Virtual memory
- Open resources owned by the process
- Scheduling state
- Other kernel-managed process resources

The operating system reclaims these resources.

---

# Open Files

Suppose:

```
FD 3

↓

notes.txt
```

Process exits.

Kernel closes remaining open file descriptors owned by the process.

---

# Parent Notification

When child exits:

```
Kernel

↓

Stores Exit Status

↓

Notifies Parent

↓

Parent Calls wait()
```

The child is fully removed after the parent reaps it.

---

# Real-World Example — Shell

Shell:

```
fork()

↓

exec(command)

↓

wait()

↓

Exit Status

↓

Prompt Returns
```

The shell waits for most foreground commands to finish.

---

# Real-World Example — Apache

Worker:

```
Handle Request

↓

exit()

↓

Master Creates New Worker
```

---

# Real-World Example — Docker

Container:

```
Main Process Ends

↓

Container Stops
```

The container's lifecycle depends on its main process.

---

# Real-World Example — Git

Git returns:

```
0

↓

Success

----------------

Non-Zero

↓

Error
```

Shell scripts rely on these exit codes.

---

# Common Mistakes

---

## Thinking `return` and `_exit()` Are the Same

Returning from `main()` behaves like `exit()`,

not `_exit()`.

---

## Using `_exit()` in Normal Applications

Most applications should use:

```
return

or

exit()
```

Reserve `_exit()` for special situations such as child processes after `fork()`.

---

## Ignoring Exit Codes

Exit status is important for:

- Shell scripts
- Automation
- Build systems
- CI/CD pipelines

---

## Forgetting Zombies

A terminated child remains a zombie until the parent collects its status.

---

# Complete Example

```c
#include <stdlib.h>

int main()
{
    exit(0);
}
```

Equivalent to:

```c
int main()
{
    return 0;
}
```

for most programs.

---

# Hands-on Labs

## Lab 1

Return different values from `main()`.

Check them using:

```bash
echo $?
```

---

## Lab 2

Replace:

```c
return 0;
```

with:

```c
exit(0);
```

Observe identical behavior.

---

## Lab 3

Create a parent and child using:

```c
fork()
```

Experiment with:

```c
exit()

_exit()
```

Observe differences in buffered output.

---

## Lab 4

Register an `atexit()` function.

Verify that it executes before program termination.

---

## Lab 5

Use:

```bash
ps
```

and later `wait()` (next chapter) to observe zombie processes.

---

# Interview Questions

### What is the difference between `return 0;` and `exit(0);`?

Returning from `main()` is conceptually equivalent to calling `exit(0)`.

---

### What does `exit()` do?

Performs C library cleanup, flushes buffered streams, executes `atexit()` handlers, and then terminates the process.

---

### What does `_exit()` do?

Terminates the process immediately without performing C library cleanup.

---

### Why is `_exit()` commonly used after `fork()`?

To avoid running cleanup code or flushing inherited stdio buffers twice in the child process.

---

### What is a zombie process?

A terminated child process that remains in the process table until its parent collects its exit status.

---

# Summary

```
return

↓

exit()

↓

Cleanup

↓

Kernel

↓

Process Ends
```

Immediate Termination:

```
_exit()

↓

Kernel

↓

Process Ends
```

Process Lifecycle:

```
fork()

↓

Child

↓

exec()

↓

Program Runs

↓

exit()

↓

Zombie

↓

wait()

↓

Removed
```

## Key Takeaways

- Returning from `main()` is equivalent to calling `exit()` in normal programs.
- `exit()` performs orderly cleanup before terminating the process.
- `_exit()` skips C library cleanup and terminates immediately.
- Exit status codes communicate success or failure to the parent process.
- Child processes become zombies until their parent reaps them with `wait()`.
- Understanding process termination is essential before learning process synchronization.

---

# Next Chapter

## Chapter 112 — Waiting for Child Processes (`wait()` and `waitpid()`)

You'll learn:

- Why parents must wait for children
- Zombie process cleanup
- `wait()`
- `waitpid()`
- Exit status retrieval
- Blocking vs non-blocking waits
- Process synchronization in Unix