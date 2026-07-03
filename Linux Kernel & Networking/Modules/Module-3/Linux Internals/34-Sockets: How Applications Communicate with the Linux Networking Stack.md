Most developers know this code.

**Node.js**

```javascript
const server = net.createServer();
```

**Java**

```java
ServerSocket server = new ServerSocket(8080);
```

**Go**

```go
ln, _ := net.Listen("tcp", ":8080")
```

The server starts.

Clients connect.

Data flows.

But have you ever wondered:

- What exactly is a socket?
- Is a socket just a port?
- Why is a socket represented as a file descriptor?
- What actually happens during `bind()`?
- What is `listen()` doing?
- What happens when `accept()` returns?
- Where are sockets stored inside Linux?
- Why can `epoll()` handle millions of connections?

The answer is:

**Sockets are the interface between applications and the Linux networking stack.**

Every network application—from Nginx to Docker to PostgreSQL—uses sockets.

After this chapter, you'll understand exactly how applications communicate with the Linux kernel using sockets.

---

# 👑 Linux Kernel & Networking Mastery

# Module 4 — Linux Networking

# Chapter 34 — Sockets: How Applications Communicate with the Linux Networking Stack

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - What is a Socket?
> - Why Sockets exist
> - Socket API
> - Socket Lifecycle
> - Socket File Descriptors
> - `socket()`
> - `bind()`
> - `listen()`
> - `accept()`
> - `connect()`
> - `send()`
> - `recv()`
> - Blocking vs Non-blocking Sockets
> - `select()`
> - `poll()`
> - `epoll()`
> - Linux Socket Internals

---

# 📖 Why Do We Need Sockets?

Suppose Chrome wants to send:

```
HTTP Request

↓

google.com
```

Chrome cannot directly call:

```
TCP

↓

IP

↓

Ethernet
```

Instead,

Linux provides one standard interface.

```
Socket
```

Applications communicate with the kernel through sockets.

---

# What is a Socket?

A socket is a kernel object representing one endpoint of network communication.

```
Application

↓

Socket

↓

Linux Networking Stack
```

Think of a socket as:

```
Network File
```

---

# Why Are Sockets Like Files?

Remember:

Linux says:

```
Everything is a File
```

Sockets follow the same philosophy.

Example:

```
File

↓

read()

↓

write()
```

Socket:

```
Socket

↓

send()

↓

recv()
```

Both are represented by:

```
File Descriptor
```

---

# Socket Lifecycle

Every socket follows roughly the same lifecycle.

```
socket()

↓

bind()

↓

listen()

↓

accept()

↓

send()

↓

recv()

↓

close()
```

---

# Step 1 — `socket()`

Applications first create a socket.

Example:

```c
socket(AF_INET, SOCK_STREAM, 0);
```

Flow:

```
Application

↓

socket()

↓

Kernel Creates Socket

↓

File Descriptor Returned
```

---

# Socket Types

TCP:

```
SOCK_STREAM
```

UDP:

```
SOCK_DGRAM
```

Raw Networking:

```
SOCK_RAW
```

---

# File Descriptor

Suppose Linux returns:

```
FD = 7
```

Now:

```
Socket

↓

File Descriptor 7
```

The application refers to the socket using this descriptor.

---

# Step 2 — `bind()`

The server must choose:

```
IP Address

+

Port
```

Example:

```c
bind(fd, 0.0.0.0:8080)
```

Linux reserves that address.

---

# What Does `bind()` Do?

Suppose:

```
Port 8080
```

Linux records:

```
Socket

↓

8080
```

Now packets arriving on port 8080 can be delivered to this socket.

---

# Step 3 — `listen()`

The socket becomes:

```
Listening Socket
```

It does **not** communicate with clients yet.

Instead:

```
Wait for Connections
```

---

# Listening Queue

Incoming TCP connection requests are placed into a queue.

```
Client A

↓

Queue

↓

Client B

↓

Queue

↓

Client C
```

The server processes them one by one (or in parallel using threads/processes/event loops).

---

# Step 4 — `accept()`

The server calls:

```c
accept()
```

Linux removes one connection from the queue.

```
Listening Socket

↓

accept()

↓

New Socket
```

Important:

The listening socket remains open.

The new socket communicates with one client.

---

# Two Different Sockets

```
Listening Socket

↓

Port 8080
```

```
Client Socket

↓

Client 1
```

```
Client Socket

↓

Client 2
```

One listening socket,

many client sockets.

---

# Step 5 — `connect()`

Clients perform:

```c
connect()
```

Flow:

```
Client

↓

TCP Handshake

↓

Server

↓

Connected
```

---

# Step 6 — `send()`

Application:

```c
send(fd, data)
```

Flow:

```
Application

↓

Socket

↓

TCP

↓

IP

↓

Ethernet

↓

NIC
```

---

# Step 7 — `recv()`

Receiving:

```
NIC

↓

Kernel

↓

Socket Buffer

↓

recv()

↓

Application
```

---

# Socket Buffers

Every socket has buffers.

```
Application

↓

Send Buffer

↓

Kernel

↓

Network
```

Receiving:

```
Network

↓

Receive Buffer

↓

Application
```

These buffers smooth out differences between application speed and network speed.

---

# Blocking Sockets

Default behavior.

```
recv()

↓

Wait

↓

Packet Arrives

↓

Return
```

The thread sleeps until data is available.

---

# Non-blocking Sockets

Instead:

```
recv()

↓

No Data

↓

Return Immediately
```

The application can continue doing other work.

---

# Problem with Blocking

Suppose one server handles:

```
10,000 Clients
```

Blocking threads:

```
10,000 Threads
```

Huge memory usage.

Poor scalability.

---

# `select()`

Older Linux solution.

```
Many Sockets

↓

select()

↓

Ready Socket
```

Works,

but scales poorly for large numbers of sockets.

---

# `poll()`

Improves some limitations of `select()`,

but still checks many sockets repeatedly.

---

# `epoll()`

Linux introduced:

```
epoll
```

Instead of repeatedly checking every socket,

the kernel notifies the application only when events occur.

---

# epoll Flow

```
Sockets

↓

epoll

↓

Kernel Watches

↓

Event Happens

↓

Application Notified
```

No unnecessary scanning.

---

# Why epoll Is Fast

Suppose:

```
1,000,000 Sockets
```

Only:

```
5 Ready
```

`epoll` returns only those 5 ready sockets.

Excellent scalability.

---

# Why Nginx Uses epoll

```
Client

↓

epoll

↓

Ready Socket

↓

Process Request
```

One worker process can efficiently manage thousands of simultaneous connections.

---

# Linux Socket Internals

Inside the kernel,

a socket is represented by kernel data structures.

```
Application

↓

File Descriptor

↓

Socket Object

↓

TCP

↓

IP

↓

Driver
```

The file descriptor points to the kernel socket object.

---

# Real Example

Suppose Chrome loads:

```
https://google.com
```

Flow:

```
socket()

↓

connect()

↓

TCP Handshake

↓

send()

↓

recv()

↓

close()
```

Every web request follows this pattern.

---

# Docker Example

Suppose a container connects to Redis.

```
Application

↓

Socket

↓

Container Network

↓

Host Kernel

↓

Redis Socket
```

Containers use the same Linux socket API.

---

# Why Sockets Matter

Everything depends on sockets.

Examples:

- Nginx
- Apache
- PostgreSQL
- Redis
- Docker
- Kubernetes
- SSH
- Chrome
- Node.js
- Java
- Go

Every network application ultimately communicates through sockets.

---

# Hands-on Lab

## Lab 1 — View Listening Sockets

```bash
ss -l
```

---

## Lab 2 — View TCP Listening Ports

```bash
ss -tl
```

---

## Lab 3 — View UDP Listening Ports

```bash
ss -ul
```

---

## Lab 4 — View Process Owning a Socket

```bash
ss -tulpn
```

Observe:

- PID
- Process Name
- Listening Port

---

## Lab 5 — View Open File Descriptors

```bash
ls -l /proc/$$/fd
```

Observe file descriptors used by the current shell.

---

## Lab 6 — Observe Network Connections

```bash
ss -tan
```

Observe connection states such as:

```
LISTEN

ESTABLISHED

TIME_WAIT
```

---

# Interview Questions

## What is a Socket?

A Socket is a kernel-managed communication endpoint used by applications to exchange data over a network.

---

## Why are sockets represented as file descriptors?

Linux treats sockets like files, allowing applications to use a consistent interface for managing resources.

---

## What is the purpose of `bind()`?

`bind()` associates a socket with a local IP address and port.

---

## What does `listen()` do?

`listen()` marks a TCP socket as ready to accept incoming connection requests.

---

## What is returned by `accept()`?

`accept()` returns a **new socket** dedicated to communicating with a specific client, while the listening socket remains available for additional connections.

---

## What is the difference between blocking and non-blocking sockets?

Blocking sockets wait until an operation completes.

Non-blocking sockets return immediately if no data is available.

---

## Why is `epoll()` faster than `select()` for large-scale servers?

`epoll()` reports only sockets with pending events, avoiding repeated scans of all monitored sockets.

---

# Summary

Sockets provide the bridge between applications and the Linux networking stack.

```
Application

↓

socket()

↓

bind()

↓

listen()

↓

accept()

↓

send()

↓

recv()

↓

close()
```

Key concepts:

- Sockets are kernel-managed communication endpoints.
- Sockets are represented by file descriptors.
- `socket()` creates a socket.
- `bind()` assigns an address and port.
- `listen()` prepares the socket for incoming TCP connections.
- `accept()` creates a new client socket.
- `send()` and `recv()` exchange data.
- `epoll()` enables highly scalable event-driven networking.
- Every major Linux network application depends on sockets.

Understanding sockets prepares you for advanced Linux networking topics such as Netfilter, iptables, Docker networking, Kubernetes networking, eBPF, and high-performance network servers.

---

# Next Chapter

## Chapter 35 — Netfilter & iptables: How Linux Filters and Modifies Network Packets

We'll explore:

- What is Netfilter?
- Why Netfilter exists
- Packet Hooks
- `iptables`
- Tables
- Chains
- Rules
- NAT
- DNAT
- SNAT
- MASQUERADE
- Connection Tracking (`conntrack`)
- How Docker Uses Netfilter