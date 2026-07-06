# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 118 — Named Pipes (FIFOs) — Communicating Between Unrelated Processes

---

# Most beginners think:

> "Pipes only work between a parent and child process."

That is true for **anonymous pipes**.

Example:

```
Parent

↓

fork()

↓

Child
```

Both inherit the pipe.

But what if:

```
Terminal 1

↓

Program A

------------------------

Terminal 2

↓

Program B
```

These programs have **no relationship**.

How can they communicate?

Linux provides:

```
FIFO

(Named Pipe)
```

---

# Learning Objectives

After this chapter you will understand:

- What a FIFO is
- Anonymous pipe vs named pipe
- `mkfifo()`
- Reading from FIFOs
- Writing to FIFOs
- Blocking behavior
- Real-world FIFO usage

---

# What is a FIFO?

FIFO stands for:

```
First In

↓

First Out
```

Data leaves in the same order it entered.

Example:

```
Write:

A

B

C

↓

Read:

A

B

C
```

Exactly like a queue.

---

# Why Is It Called a Named Pipe?

Anonymous pipe:

```
Exists

↓

Only In Memory
```

FIFO:

```
Exists

↓

Filesystem
```

It has a pathname.

Example:

```
/tmp/chatpipe
```

---

# Visualization

Anonymous Pipe

```
Parent

↓

Pipe

↓

Child
```

Named Pipe

```
Program A

↓

/tmp/chatpipe

↓

Program B
```

No parent-child relationship required.

---

# Creating a FIFO

Prototype:

```c
#include <sys/stat.h>

int mkfifo(
const char *path,
mode_t mode);
```

Example:

```c
mkfifo(
"/tmp/chat",
0666);
```

Creates:

```
/tmp/chat
```

as a special FIFO file.

---

# Does It Store Data on Disk?

Many beginners think:

```
FIFO

↓

File

↓

Disk Storage
```

Wrong.

The filename exists in the filesystem,

but the data flows through memory,

just like an anonymous pipe.

---

# Checking the FIFO

Linux command:

```bash
ls -l /tmp/chat
```

Example output:

```
prw-r--r--
```

Notice:

```
p
```

Means:

```
Pipe
```

Not:

```
Regular File
```

---

# Writing

Program:

```c
write(
fd,
"Hello",
5);
```

Data enters:

```
Kernel FIFO Buffer
```

---

# Reading

Program:

```c
read(
fd,
buffer,
5);
```

Receives:

```
Hello
```

---

# Visualization

```
Writer

↓

FIFO Buffer

↓

Reader
```

Exactly like an anonymous pipe.

---

# Blocking Behavior

Suppose:

Reader opens FIFO.

No writer exists.

Result:

```
Block

↓

Wait
```

---

# Another Case

Writer opens FIFO.

No reader exists.

Result:

```
Block

↓

Wait
```

Both sides wait until the other side connects.

---

# Complete Example

Writer:

```c
int fd =
open(
"/tmp/chat",
O_WRONLY);

write(
fd,
"Linux",
5);
```

Reader:

```c
int fd =
open(
"/tmp/chat",
O_RDONLY);

read(
fd,
buffer,
5);
```

Output:

```
Linux
```

---

# FIFO Lifetime

Anonymous Pipe:

```
Process Ends

↓

Pipe Gone
```

FIFO:

```
Filesystem Object

↓

Still Exists
```

Until removed:

```bash
rm /tmp/chat
```

---

# Anonymous Pipe vs FIFO

| Anonymous Pipe | FIFO |
|----------------|------|
| Parent-child only | Any processes |
| Created with `pipe()` | Created with `mkfifo()` |
| No filename | Has filename |
| Exists only in memory | Appears in filesystem |

---

# Why FIFO Exists

Suppose:

```
Logger

↓

FIFO

↓

Monitoring Tool
```

Two completely separate applications can exchange data without sockets or shared memory.

---

# Kernel Buffer

Like anonymous pipes,

the kernel keeps an internal FIFO buffer.

```
Writer

↓

AAAA

BBBB

CCCC

↓

FIFO Buffer

↓

Reader
```

---

# Multiple Writers

Possible:

```
Program A

↓

FIFO

↑

Program B

↓

Reader
```

The kernel serializes writes,

though applications must still consider message boundaries and synchronization.

---

# Multiple Readers

Possible,

but the data is generally consumed by whichever reader receives it.

A byte read by one process is not automatically delivered to another.

---

# FIFO Permissions

Like regular files,

FIFOs have permissions.

Example:

```
0666
```

Meaning:

```
Read

Write

Everyone
```

Permissions control who can access the FIFO.

---

# Real-World Example — Logging

Application:

```
Writes Logs

↓

FIFO

↓

Log Collector
```

Streaming without creating temporary files.

---

# Real-World Example — Printer Queue

Application:

```
Print Job

↓

FIFO

↓

Printer Daemon
```

Historically,

Unix printing systems often relied on FIFO-like queueing concepts.

---

# Real-World Example — Media Streaming

Producer:

```
Frames

↓

FIFO

↓

Encoder
```

Allows continuous streaming between programs.

---

# Real-World Example — Monitoring

One process generates metrics.

Another reads and displays them in real time through a FIFO.

---

# Common Mistakes

---

## Thinking FIFO Stores Data Permanently

The filename exists,

but the data is transient.

Once read,

it is gone.

---

## Forgetting Blocking Behavior

Opening a FIFO may block until another process opens the opposite end.

---

## Confusing FIFO with Regular Files

You cannot randomly seek through FIFO data.

It is a stream,

not ordinary file storage.

---

## Expecting Broadcast Behavior

Reading removes data from the FIFO.

Other readers do not automatically receive the same bytes.

---

# Complete Example

Create FIFO:

```bash
mkfifo /tmp/chat
```

Terminal 1:

```bash
echo "Hello Linux" > /tmp/chat
```

Terminal 2:

```bash
cat /tmp/chat
```

Output:

```
Hello Linux
```

No parent-child relationship required.

---

# Hands-on Labs

## Lab 1

Create a FIFO using:

```bash
mkfifo /tmp/chat
```

Communicate between two terminals.

---

## Lab 2

Write a C program that writes messages to the FIFO.

---

## Lab 3

Write another C program that continuously reads from the FIFO.

---

## Lab 4

Observe blocking behavior when only one side opens the FIFO.

---

## Lab 5

Inspect the FIFO using:

```bash
ls -l
```

Notice the leading:

```
p
```

file type.

---

# Interview Questions

### What is a FIFO?

A named pipe that allows unrelated processes to communicate through a special filesystem object.

---

### What is the difference between an anonymous pipe and a FIFO?

Anonymous pipes are typically used between related processes.

FIFOs allow unrelated processes to communicate using a pathname.

---

### Which function creates a FIFO?

```c
mkfifo()
```

---

### Does a FIFO permanently store data?

No.

It provides a streaming communication channel.

---

### Why are FIFOs useful?

They enable simple inter-process communication without requiring sockets or shared memory.

---

# Summary

Anonymous Pipe:

```
Parent

↓

pipe()

↓

Child
```

Named Pipe:

```
Program A

↓

FIFO

↓

Program B
```

Creation:

```
mkfifo()

↓

Filesystem Object

↓

Kernel Buffer

↓

Communication
```

## Key Takeaways

- FIFOs are named pipes represented as special filesystem objects.
- They enable communication between unrelated processes.
- `mkfifo()` creates a FIFO.
- Data flows in FIFO order (First In, First Out).
- FIFOs behave like streams, not regular files.
- They are useful for lightweight inter-process communication on the same machine.

---

# Next Chapter

## Chapter 119 — Unix Domain Sockets (Local IPC Faster Than TCP)

You'll learn:

- What Unix Domain Sockets are
- How they differ from FIFOs
- Stream vs datagram Unix sockets
- Local client-server communication
- `socket()`, `bind()`, `connect()`
- Why Docker, PostgreSQL, and system services often use Unix sockets instead of TCP