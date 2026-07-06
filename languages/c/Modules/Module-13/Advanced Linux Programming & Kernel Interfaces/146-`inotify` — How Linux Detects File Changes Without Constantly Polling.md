# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 146 — `inotify` — How Linux Detects File Changes Without Constantly Polling

---

# Most beginners write programs like this:

```c
while (1)
{
    stat("config.json");

    sleep(1);
}
```

Every second:

```
Check File

↓

Nothing Changed

↓

Check Again

↓

Nothing Changed

↓

Repeat Forever
```

This is called:

```
Polling
```

It wastes:

- CPU
- System calls
- Battery
- Time

Linux has a much better solution:

```
inotify
```

---

# Learning Objectives

After this chapter you will understand:

- What `inotify` is
- Polling vs event-driven design
- Watching files
- Watching directories
- File events
- Event queue
- Recursive watching limitations
- Real-world applications

---

# The Problem

Suppose you're building:

```
VS Code
```

User edits:

```
main.c
```

Question:

How does VS Code instantly know?

Does it check:

```
100 Times Per Second?
```

That would waste enormous CPU resources.

---

# Another Example

Dropbox.

You save:

```
report.pdf
```

Upload begins immediately.

How?

---

# Linux Solution

Instead of asking:

```
Did Anything Change?

↓

Did Anything Change?

↓

Did Anything Change?
```

Linux says:

```
I'll Notify You

↓

When Something Changes
```

Hence the name:

```
inotify

↓

inode notification
```

---

# What is `inotify`?

Definition:

> **`inotify` is a Linux kernel API that notifies applications when files or directories change.**

It is:

```
Event Driven
```

Instead of:

```
Polling
```

---

# Polling vs Event Driven

Polling:

```
Program

↓

Check

↓

Check

↓

Check

↓

Check
```

---

# Event Driven

```
Program

↓

Sleep

↓

Kernel Detects Change

↓

Wake Program
```

Much more efficient.

---

# Think About a Doorbell

Polling:

```
Open Door

↓

Anyone?

↓

Close Door

↓

Repeat
```

Event-driven:

```
Press Doorbell

↓

Notification
```

`inotify` is Linux's doorbell.

---

# Architecture

```
Application

↓

inotify

↓

Kernel

↓

Filesystem

↓

Events
```

---

# Creating an `inotify` Instance

Program creates:

```
inotify Instance
```

Kernel returns:

```
File Descriptor
```

Just like:

```
socket()

↓

pipe()

↓

eventfd()
```

Everything in Linux becomes a file descriptor.

---

# Watching a File

Example:

```
config.json
```

Program asks:

```
Notify Me

↓

If Modified
```

Kernel remembers the watch.

---

# Watching a Directory

Even more common.

Example:

```
src/

↓

Watch Entire Directory
```

Kernel reports events for files inside that directory.

---

# Common Events

Linux can notify about:

```
File Created

↓

File Deleted

↓

File Modified

↓

File Moved

↓

File Opened

↓

File Closed

↓

Attribute Changed
```

---

# Visualization

```
User Saves File

↓

Filesystem

↓

Kernel

↓

inotify Event

↓

Application
```

---

# Event Queue

Kernel stores events in a queue.

Program reads:

```
Event

↓

Event

↓

Event
```

No polling required.

---

# Example Flow

Editor saves:

```
main.c
```

Kernel generates:

```
MODIFY
```

Application reads:

```
Event Queue

↓

main.c Modified
```

---

# File Descriptor

Remember:

`inotify` returns:

```
FD
```

Program waits using:

```
read()

↓

epoll()

↓

select()

↓

poll()
```

It integrates perfectly with Linux event loops.

---

# Relationship with `epoll`

Large applications:

```
Sockets

↓

Timers

↓

inotify

↓

eventfd

↓

epoll
```

One event loop handles everything.

---

# Recursive Watching

Important limitation.

Suppose:

```
project/

↓

src/

↓

components/

↓

Button/
```

Watching:

```
project/
```

does **not automatically** watch every new subdirectory recursively.

Applications often add watches for directories individually.

---

# Why?

Linux keeps the API simple and efficient.

Recursive directory management is typically handled in user space.

---

# File Rename

Suppose:

```
main.c

↓

app.c
```

Kernel generates move-related events,

allowing applications to update their internal state.

---

# File Delete

Suppose:

```
config.json

↓

Deleted
```

Kernel sends:

```
DELETE
```

Application reacts immediately.

---

# File Create

Suppose:

```
notes.txt

↓

Created
```

Kernel sends:

```
CREATE
```

---

# Overflow

Very busy systems may generate events faster than applications consume them.

If the event queue overflows,

the application receives an overflow notification and may need to resynchronize.

---

# Relationship with VFS

Flow:

```
Filesystem

↓

VFS

↓

inotify

↓

Application
```

`inotify` hooks into filesystem activity inside the kernel.

---

# Real-World Example — VS Code

```
Save File

↓

inotify

↓

Reload Explorer

↓

Rebuild Language Index
```

---

# Real-World Example — webpack

Developer saves:

```
app.js
```

`inotify` notifies webpack.

Webpack recompiles automatically.

---

# Real-World Example — Dropbox

Save:

```
photo.jpg
```

Immediately:

```
Upload Begins
```

No continuous polling required.

---

# Real-World Example — Git GUI

GUI applications update their file tree automatically when files change.

`inotify` provides those notifications.

---

# Real-World Example — Kubernetes

Configuration reloaders and operators often watch configuration directories for changes using filesystem notification mechanisms.

---

# Polling Comparison

Polling:

```
1000 Files

↓

1000 stat()

↓

Every Second
```

Large CPU overhead.

---

# `inotify`

```
1000 Files

↓

Sleep

↓

Only Wake

↓

On Change
```

Far more efficient.

---

# Common Mistakes

---

## Thinking `inotify` Watches Entire Directory Trees

Recursive watching is not automatic.

Applications usually manage watches themselves.

---

## Ignoring Queue Overflow

Applications should handle overflow events gracefully.

---

## Assuming It Works Across Every Filesystem Identically

Most local Linux filesystems support `inotify` well, but behavior may differ for some network or special filesystems.

---

## Using Polling Instead of Events

For change detection,

event-driven designs are usually more efficient.

---

# Hands-on Labs

## Lab 1

Install tools such as:

```bash
inotifywait
```

Watch a directory while creating and deleting files.

---

## Lab 2

Observe modification events:

```bash
touch test.txt
```

---

## Lab 3

Rename files and observe move-related events.

---

## Lab 4

Delete files and observe delete notifications.

---

## Lab 5

Write a small C program using `inotify` to monitor a directory for file changes.

---

# Interview Questions

### What is `inotify`?

A Linux kernel API that notifies applications about filesystem changes.

---

### Why is `inotify` better than polling?

It avoids repeated checks and wakes applications only when relevant events occur.

---

### Can `inotify` automatically watch every subdirectory recursively?

No.

Applications usually add watches for directories individually.

---

### How are `inotify` events delivered?

Through a file descriptor that applications read from or integrate into an event loop.

---

### Which applications commonly use `inotify`?

Editors, IDEs, build tools, synchronization software, and many file monitoring services.

---

# Summary

Architecture:

```
Filesystem

↓

Kernel

↓

inotify

↓

Application
```

Typical Events:

| Event | Meaning |
|--------|----------|
| CREATE | File created |
| DELETE | File deleted |
| MODIFY | File contents changed |
| MOVED_FROM | File moved out |
| MOVED_TO | File moved in |
| ATTRIB | Metadata changed |
| OPEN | File opened |
| CLOSE | File closed |

Polling vs Events:

```
Polling

↓

Repeated Checks

------------------------

inotify

↓

Wake On Change
```

## Key Takeaways

- `inotify` provides efficient filesystem change notifications.
- It replaces inefficient polling for many use cases.
- Applications receive events through a file descriptor.
- It integrates naturally with `epoll` and event-driven architectures.
- Editors, build systems, and synchronization tools rely heavily on `inotify`.
- Understanding `inotify` is essential for building responsive Linux applications.

---

# Next Chapter

## Chapter 147 — `eventfd` — The Simplest Way for Linux Processes and Threads to Signal Events

You'll learn:

- What `eventfd` is
- Counter-based event notification
- Thread synchronization
- Process synchronization
- Integration with `epoll`
- Why high-performance servers prefer `eventfd` over pipes for simple notifications