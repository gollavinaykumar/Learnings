# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 116 — Signals (How Linux Interrupts Running Processes)

---

# Most beginners think:

> "My program always executes line by line."

Example:

```c
while(1)
{
    printf("Running...\n");
}
```

Many imagine:

```
Line 1

↓

Line 2

↓

Line 3

↓

Forever
```

But what happens when you press:

```
Ctrl + C
```

Your program suddenly stops.

Who interrupted it?

How did Linux notify the process?

The answer is:

```
Signals
```

Signals are one of the oldest and most fundamental inter-process communication (IPC) mechanisms in Unix.

---

# Learning Objectives

After this chapter you will understand:

- What signals are
- Signal numbers
- Signal delivery
- `SIGINT`
- `SIGTERM`
- `SIGKILL`
- `SIGSEGV`
- Signal handlers
- `signal()`
- `sigaction()`

---

# What is a Signal?

A signal is:

> **An asynchronous notification sent to a process, informing it that an event has occurred.**

Examples:

```
Keyboard Interrupt

↓

Timer Expired

↓

Invalid Memory Access

↓

Child Process Finished

↓

Termination Request
```

---

# Think of a Phone Call

Imagine you're reading a book.

Suddenly:

```
Phone Rings
```

You immediately stop reading,

answer the call,

then continue reading.

Signals work similarly.

```
Running Process

↓

Signal Arrives

↓

Handler Executes

↓

Resume (Sometimes)
```

---

# Visualization

```
Program Running

↓

Signal

↓

Kernel

↓

Signal Handler

↓

Resume

or

Terminate
```

Signals interrupt normal execution.

---

# Where Do Signals Come From?

Signals may originate from:

- The keyboard
- Another process
- The kernel
- Hardware exceptions
- Timers

The receiving process doesn't know exactly when they will arrive.

---

# Common Signals

| Signal | Meaning |
|---------|---------|
| `SIGINT` | Interrupt (Ctrl+C) |
| `SIGTERM` | Graceful termination request |
| `SIGKILL` | Immediate termination |
| `SIGSEGV` | Invalid memory access |
| `SIGALRM` | Timer expired |
| `SIGCHLD` | Child process changed state |
| `SIGPIPE` | Write to closed pipe |

---

# `SIGINT`

Press:

```
Ctrl + C
```

Flow:

```
Keyboard

↓

Terminal

↓

Kernel

↓

SIGINT

↓

Process
```

By default,

the process terminates.

---

# Example

Program:

```c
while(1)
{
}
```

Press:

```
Ctrl+C
```

Kernel sends:

```
SIGINT
```

Program exits.

---

# `SIGTERM`

This is the standard way to politely ask a process to stop.

Example:

```bash
kill PID
```

By default:

```
kill

↓

SIGTERM
```

The process may:

- Clean up resources
- Save data
- Exit gracefully

---

# `SIGKILL`

Unlike most signals:

```
SIGKILL

↓

Cannot Be Ignored

Cannot Be Blocked

Cannot Be Handled
```

Kernel immediately destroys the process.

---

# Visualization

```
SIGTERM

↓

Program Can Cleanup

-------------------------

SIGKILL

↓

Immediate Death
```

---

# `SIGSEGV`

Occurs when:

```
Invalid Memory Access
```

Example:

```c
int *p = NULL;

*p = 10;
```

Flow:

```
CPU Exception

↓

Kernel

↓

SIGSEGV

↓

Program Terminates
```

---

# Signal Handler

Instead of using the default behavior,

a program may install a handler.

Example:

```c
void handler(int sig)
{
    printf("Signal Received\n");
}
```

Now the program can respond when the signal arrives.

---

# Using `signal()`

Example:

```c
#include <signal.h>

signal(
SIGINT,
handler);
```

Now:

```
Ctrl+C

↓

handler()

↓

Continue

(or exit manually)
```

---

# Complete Example

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int sig)
{
    printf("\nCaught SIGINT\n");
}

int main()
{
    signal(SIGINT, handler);

    while(1)
    {
        sleep(1);
    }
}
```

Press:

```
Ctrl+C
```

Output:

```
Caught SIGINT
```

Instead of immediate termination.

---

# Why `signal()` Isn't Preferred

Historically,

different Unix systems implemented `signal()` differently.

Modern Unix/Linux programs usually prefer:

```
sigaction()
```

because it provides:

- More predictable behavior
- Better control
- Additional options

---

# `sigaction()`

Conceptually:

```
Register Handler

↓

Signal Arrives

↓

Kernel

↓

Handler

↓

Return
```

Professional software typically uses `sigaction()` instead of `signal()`.

---

# Asynchronous Nature

Signals can arrive:

```
Any Time
```

Example:

```
Computing

↓

Signal Arrives

↓

Handler Executes

↓

Resume
```

This makes signal programming more challenging than ordinary function calls.

---

# Safe Signal Handlers

Because handlers interrupt normal execution,

they should do as little work as possible.

Common practice:

```
Signal

↓

Set Flag

↓

Return
```

The main program later checks the flag.

---

# Example

```c
volatile sig_atomic_t stop = 0;

void handler(int sig)
{
    stop = 1;
}
```

Main loop:

```c
while(!stop)
{
    /* work */
}
```

This is much safer than performing complex operations inside the handler.

---

# Sending Signals

Command:

```bash
kill -TERM PID
```

or:

```bash
kill -KILL PID
```

Another process requests that the kernel deliver the specified signal.

---

# Process Communication

Process A:

```
kill()

↓

Kernel

↓

SIGTERM

↓

Process B
```

Signals are a simple form of inter-process communication.

---

# Real-World Example — Shell

Press:

```
Ctrl+C
```

Shell:

```
Terminal

↓

Kernel

↓

SIGINT

↓

Foreground Process
```

The shell itself usually remains running.

---

# Real-World Example — Docker

Stopping a container:

```
docker stop

↓

SIGTERM

↓

Wait

↓

SIGKILL (if necessary)
```

This allows applications time to shut down cleanly.

---

# Real-World Example — Nginx

Reload configuration:

```
Signal

↓

Workers Reload

↓

No Downtime
```

Signals are commonly used for administrative control.

---

# Real-World Example — Database

Database server:

```
SIGTERM

↓

Flush Data

↓

Close Connections

↓

Exit Safely
```

Graceful termination protects data integrity.

---

# Common Mistakes

---

## Using `SIGKILL` First

Prefer:

```
SIGTERM
```

Allow the process to clean up before forcing termination.

---

## Doing Too Much Inside Handlers

Handlers should remain small and simple.

Avoid complex logic.

---

## Ignoring `SIGCHLD`

Programs that create child processes often need to handle or wait for them to avoid zombies.

---

## Assuming Signals Arrive at Predictable Times

Signals are asynchronous.

They may interrupt execution almost anywhere.

---

# Complete Example

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

volatile sig_atomic_t stop = 0;

void handler(int sig)
{
    stop = 1;
}

int main()
{
    signal(SIGINT, handler);

    while(!stop)
    {
        printf("Running...\n");
        sleep(1);
    }

    printf("Exiting...\n");

    return 0;
}
```

Possible output:

```
Running...
Running...
Running...

Ctrl+C

Exiting...
```

---

# Hands-on Labs

## Lab 1

Install a `SIGINT` handler.

Press:

```
Ctrl+C
```

Observe the handler execution.

---

## Lab 2

Send:

```bash
kill -TERM PID
```

Observe graceful termination.

---

## Lab 3

Compare:

```bash
kill PID
```

and

```bash
kill -9 PID
```

Observe the differences.

---

## Lab 4

Replace `signal()` with `sigaction()`.

Study the additional configuration options.

---

## Lab 5

Create a program that exits cleanly after receiving `SIGTERM`.

---

# Interview Questions

### What is a signal?

An asynchronous notification delivered to a process to indicate that an event has occurred.

---

### What is the difference between `SIGTERM` and `SIGKILL`?

`SIGTERM` requests graceful termination.

`SIGKILL` immediately terminates the process and cannot be caught or ignored.

---

### What causes `SIGSEGV`?

An invalid memory access, such as dereferencing an invalid pointer.

---

### Why is `sigaction()` preferred over `signal()`?

It provides more reliable, portable, and configurable signal handling.

---

### Why should signal handlers remain simple?

Because they execute asynchronously and may interrupt the program at almost any point.

---

# Summary

```
Program Running

↓

Signal

↓

Kernel

↓

Signal Handler

↓

Continue

or

Terminate
```

Common Signals:

| Signal | Purpose |
|---------|---------|
| `SIGINT` | Ctrl+C |
| `SIGTERM` | Graceful stop |
| `SIGKILL` | Immediate termination |
| `SIGSEGV` | Invalid memory access |
| `SIGCHLD` | Child process status change |

## Key Takeaways

- Signals provide asynchronous notifications to running processes.
- The kernel delivers signals in response to events such as keyboard interrupts, timers, or hardware exceptions.
- `SIGTERM` allows graceful shutdown; `SIGKILL` forces immediate termination.
- Signal handlers allow programs to customize their response to many signals.
- `sigaction()` is the preferred modern interface for installing signal handlers.
- Signals are a fundamental building block of Unix process management and inter-process communication.

---

# Next Chapter

## Chapter 117 — Pipes (`pipe()`) — How Two Processes Talk to Each Other

You'll learn:

- What pipes are
- Anonymous pipes
- `pipe()`
- Reading and writing through pipes
- Parent-child communication
- File descriptors in pipes
- How the Unix shell connects commands like:

```bash
ls | grep txt
```

using pipes