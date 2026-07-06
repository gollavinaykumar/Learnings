# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 119 — Unix Domain Sockets (Fast Local IPC Without TCP/IP)

---

# Most beginners think:

> "Sockets are only for networking."

When most developers hear the word:

```
Socket
```

they immediately think:

```
Internet

↓

TCP

↓

IP Address

↓

Port
```

Example:

```
Browser

↓

Internet

↓

Google
```

But Linux also provides another type of socket:

```
Unix Domain Socket
```

These sockets **never leave your computer**.

They are designed for:

```
Local Process

↓

Local Process
```

communication.

---

# Learning Objectives

After this chapter you will understand:

- What Unix Domain Sockets are
- Why they exist
- Stream sockets
- Datagram sockets
- Local IPC
- `socket()`
- `bind()`
- `listen()`
- `accept()`
- `connect()`

---

# The Problem

Suppose two programs are running:

```
PostgreSQL

↓

???

↓

psql
```

They are on:

```
Same Computer
```

Using:

```
TCP/IP
```

would work,

but it's unnecessary.

Linux provides something faster:

```
Unix Domain Socket
```

---

# Think About an Office

Suppose two employees sit:

```
Same Room
```

Do they send letters through:

```
Postal Service?
```

No.

They simply talk directly.

Unix Domain Sockets are like direct communication inside one computer.

---

# What is a Unix Domain Socket?

It is a socket used **only for communication between processes on the same machine**.

Instead of:

```
IP Address

↓

Port
```

it uses:

```
Filesystem Path
```

Example:

```
/tmp/server.sock
```

---

# Visualization

TCP Socket

```
Client

↓

Internet

↓

Server
```

Unix Socket

```
Client

↓

/tmp/server.sock

↓

Server
```

No network stack required.

---

# Creating a Socket

Example:

```c
socket(
AF_UNIX,
SOCK_STREAM,
0);
```

Notice:

```
AF_UNIX
```

instead of:

```
AF_INET
```

---

# Socket Types

Unix sockets support:

```
SOCK_STREAM

↓

Reliable Stream

-------------------

SOCK_DGRAM

↓

Datagrams
```

Just like TCP and UDP,

but locally.

---

# Filesystem Representation

Server creates:

```
/tmp/server.sock
```

Linux command:

```bash
ls -l
```

Shows:

```
srwxrwxrwx
```

Notice:

```
s
```

means:

```
Socket
```

---

# Basic Workflow

Server:

```
socket()

↓

bind()

↓

listen()

↓

accept()
```

Client:

```
socket()

↓

connect()
```

Very similar to TCP programming.

---

# `bind()`

Example:

```c
bind(
fd,
address,
...);
```

For Unix sockets,

the address contains:

```
Filesystem Path
```

instead of an IP address.

---

# `listen()`

Server says:

```
I'm Ready

↓

Waiting For Clients
```

Kernel begins accepting incoming connection requests.

---

# `accept()`

When a client connects:

```
Kernel

↓

accept()

↓

New Connected Socket
```

The original listening socket remains available for future clients.

---

# Client

Client:

```c
connect(
fd,
address,
...);
```

Kernel:

```
Locate

↓

/tmp/server.sock

↓

Connect
```

---

# Communication

After connection:

```
read()

↓

write()
```

Exactly like:

- TCP sockets
- Pipes

Everything is still based on file descriptors.

---

# Visualization

```
Client

↓

Socket FD

↓

Unix Socket

↓

Socket FD

↓

Server
```

---

# Why Faster Than TCP?

TCP requires:

```
IP

↓

Routing

↓

Checksums

↓

Network Stack
```

Unix sockets skip most networking overhead.

Everything stays inside the kernel.

---

# Why Use a Filesystem Path?

Example:

```
/tmp/database.sock
```

Advantages:

- Easy discovery
- File permissions
- Local-only communication

---

# File Permissions

Socket file:

```
srwx------
```

Only authorized users can connect.

The filesystem provides access control.

---

# Stream Socket

Behaves similarly to TCP.

```
Client

↓

Reliable Bytes

↓

Server
```

Maintains order.

---

# Datagram Socket

Behaves similarly to UDP.

```
Message

↓

Socket

↓

Receiver
```

Messages are discrete rather than a continuous stream.

---

# Comparison

TCP Socket

```
Network

↓

IP

↓

Port
```

Unix Socket

```
Filesystem

↓

Path
```

---

# Real-World Example — PostgreSQL

Client:

```
psql

↓

/var/run/postgresql/.s.PGSQL.5432

↓

PostgreSQL Server
```

Local clients often use Unix sockets instead of TCP.

---

# Real-World Example — Docker

Docker CLI:

```
docker

↓

/var/run/docker.sock

↓

Docker Daemon
```

This famous socket allows the Docker client to communicate with the Docker daemon.

---

# Real-World Example — Nginx

Nginx:

```
HTTP Request

↓

Unix Socket

↓

PHP-FPM
```

Often faster than communicating over TCP on the same machine.

---

# Real-World Example — System Services

Many Linux services communicate internally through:

```
*.sock
```

files.

Examples include:

- Audio servers
- Display servers
- Databases
- Container runtimes

---

# Common Mistakes

---

## Thinking Unix Sockets Use TCP

They do not.

They use the Unix socket subsystem.

---

## Forgetting to Remove Socket File

After server exits,

you often need:

```bash
rm /tmp/server.sock
```

or programmatically call:

```c
unlink()
```

before binding again.

---

## Confusing Socket File with Data File

The filesystem entry is only an endpoint,

not where communication data is permanently stored.

---

## Assuming Unix Sockets Work Across Machines

They work only on the same computer.

For remote communication,

use TCP/IP.

---

# Complete Server Flow

```
socket()

↓

bind()

↓

listen()

↓

accept()

↓

read()

↓

write()

↓

close()
```

Client Flow:

```
socket()

↓

connect()

↓

write()

↓

read()

↓

close()
```

---

# Hands-on Labs

## Lab 1

Create a Unix Domain Socket server.

Accept one client connection.

---

## Lab 2

Create a client.

Connect using:

```
/tmp/server.sock
```

Exchange messages.

---

## Lab 3

Compare communication latency with:

```
127.0.0.1 TCP
```

Notice that Unix sockets often perform better for local communication.

---

## Lab 4

Inspect the socket:

```bash
ls -l /tmp
```

Observe the socket file.

---

## Lab 5

Delete the socket using:

```bash
rm
```

or `unlink()`.

Restart the server.

---

# Interview Questions

### What is a Unix Domain Socket?

A socket used for communication between processes on the same machine.

---

### What is the main difference between TCP sockets and Unix Domain Sockets?

TCP uses IP addresses and ports.

Unix Domain Sockets use filesystem paths.

---

### Why are Unix sockets usually faster?

They avoid the overhead of the TCP/IP networking stack for local communication.

---

### Where are Unix socket endpoints located?

As special socket entries in the filesystem.

---

### Give examples of software that use Unix sockets.

- Docker
- PostgreSQL
- PHP-FPM
- Nginx (with upstream services)
- Many Linux system services

---

# Summary

TCP:

```
Client

↓

IP Address

↓

Port

↓

Server
```

Unix Socket:

```
Client

↓

Filesystem Path

↓

Server
```

Server Workflow:

```
socket()

↓

bind()

↓

listen()

↓

accept()

↓

read()

↓

write()
```

## Key Takeaways

- Unix Domain Sockets provide fast IPC between processes on the same machine.
- They use filesystem paths instead of IP addresses and ports.
- Their programming model closely resembles TCP sockets.
- They support both stream and datagram communication.
- Many production systems use Unix sockets for efficient local communication.
- Docker, PostgreSQL, Nginx, and many Linux services rely heavily on Unix Domain Sockets.

---

# Next Chapter

## Chapter 120 — Shared Memory (`shm_open()`, `mmap()`) — The Fastest IPC Mechanism

You'll learn:

- What shared memory is
- Why it is faster than pipes and sockets
- `shm_open()`
- `mmap()`
- Shared memory regions
- Synchronization problems
- When shared memory is the best IPC choice