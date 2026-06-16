# Node.js Internals – Interview Revision Guide 🚀

⚠️ **Node.js Internals is one of the most important backend interview topics.**

Many companies ask:

- How Node.js handles millions of requests?
- Is Node.js single-threaded?
- What is libuv?
- What is the Thread Pool?
- How do Streams work?
- What are Buffers?

Understanding Node.js internals helps you explain **why Node.js is fast**.

---

# Table of Contents

1. What is Node.js?
2. Node.js Architecture
3. Is Node.js Single-Threaded?
4. Event-Driven Architecture
5. Non-Blocking I/O
6. libuv
7. Thread Pool
8. Streams
9. Buffers
10. Common Interview Questions
11. Interview Quick Revision

---

# 1. What is Node.js?

Node.js is a JavaScript runtime built on:

```text
Google V8 Engine
```

---

It allows JavaScript to run outside the browser.

Examples:

```text
Web Servers
APIs
Microservices
CLI Tools
Real-time Apps
```

---

Example

```js
console.log("Hello Node");
```

Run:

```bash
node app.js
```

---

# Why Node.js Became Popular?

Because it can handle:

```text
Thousands of Concurrent Requests
```

without creating thousands of threads.

---

# 2. Node.js Architecture

High-Level Architecture:

```text
JavaScript
      ↓
V8 Engine
      ↓
Node.js APIs
      ↓
libuv
      ↓
Operating System
```

---

### Visualization

```text
Application
      ↓
Event Loop
      ↓
libuv
      ↓
OS
```

---

# Core Components

```text
V8 Engine
Event Loop
libuv
Thread Pool
OS
```

---

# 3. Is Node.js Single-Threaded?

⚠️ Most Asked Interview Question.

---

## Short Answer

```text
JavaScript execution is single-threaded.
```

---

Meaning:

```text
One Call Stack
```

---

Example

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```text
A
B
C
```

---

Only one JavaScript task runs at a time.

---

# Then How Does Node Handle Many Requests?

Because Node uses:

```text
Event Loop
+
libuv
+
Thread Pool
```

behind the scenes.

---

# Important Interview Statement

```text
Node.js is single-threaded for JavaScript execution,
but not single-threaded internally.
```

---

# 4. Event-Driven Architecture

Node.js follows:

```text
Event-Driven Architecture
```

---

Instead of waiting:

```text
Request
 ↓
Wait
 ↓
Response
```

Node registers events.

---

Example

```js
server.on(
  "request",
  () => {
    console.log("Request");
  }
);
```

---

When event occurs:

```text
Request Arrives
```

↓

```text
Handler Executes
```

---

# Real World Example

```js
fs.readFile(
  "file.txt",
  callback
);
```

---

Node registers:

```text
Read File Event
```

and continues.

---

When file finishes:

```text
Callback Executes
```

---

# Benefits

✅ Fast

✅ Scalable

✅ Efficient

---

# 5. Non-Blocking I/O

⚠️ Core Node.js concept.

---

## What is I/O?

Input/Output operations.

Examples:

```text
Database
File System
Network
API Calls
```

---

# Blocking Example

```js
const data =
fs.readFileSync(
  "file.txt"
);

console.log(data);
```

---

Node waits.

```text
Read File
 ↓
Continue
```

---

Thread is blocked.

---

# Non-Blocking Example

```js
fs.readFile(
  "file.txt",
  (err, data) => {

    console.log(data);

  }
);

console.log("Done");
```

Output:

```text
Done
File Content
```

---

Node doesn't wait.

---

### Flow

```text
Read File
    ↓
Continue Execution
    ↓
File Completes
    ↓
Callback Executes
```

---

# Why Important?

Allows handling thousands of requests efficiently.

---

# 6. libuv

⚠️ One of the MOST IMPORTANT Node.js interview topics.

---

## What is libuv?

A C library used by Node.js.

---

libuv provides:

```text
Event Loop
Thread Pool
Async I/O
Timers
Networking
```

---

Without libuv:

```text
Node.js Async Features
Would Not Exist
```

---

# Responsibilities of libuv

### Event Loop

```text
Handles Async Tasks
```

---

### Thread Pool

```text
Runs Expensive Operations
```

---

### File System

```text
fs.readFile()
```

---

### Networking

```text
HTTP
TCP
Sockets
```

---

# Visualization

```text
JavaScript
      ↓
Node APIs
      ↓
libuv
      ↓
OS
```

---

# Important Interview Point

V8 executes JavaScript.

libuv handles asynchronous operations.

---

# 7. Thread Pool

⚠️ Very Important.

---

## What is Thread Pool?

A set of worker threads managed by libuv.

Default size:

```text
4 Threads
```

---

Can be changed:

```bash
UV_THREADPOOL_SIZE=8
```

---

# Why Needed?

Some tasks are expensive.

Examples:

```text
File System
DNS
Crypto
Compression
```

---

Node offloads them.

---

# Example

```js
fs.readFile(
  "big-file.txt",
  callback
);
```

---

Flow

```text
JavaScript
      ↓
Thread Pool
      ↓
File Read
      ↓
Callback
```

---

# Tasks Using Thread Pool

### File System

```js
fs.readFile()
```

---

### Crypto

```js
crypto.pbkdf2()
```

---

### Compression

```js
zlib
```

---

### DNS

```js
dns.lookup()
```

---

# Interview Question

Does every async operation use thread pool?

Answer:

❌ No.

---

Networking usually uses OS async capabilities.

---

Thread pool mainly handles:

```text
File System
Crypto
DNS
Compression
```

---

# 8. Streams

⚠️ One of the most important backend topics.

---

## What is a Stream?

A way to process data piece by piece.

---

Instead of:

```text
Load Entire File
```

Node processes:

```text
Chunk
Chunk
Chunk
Chunk
```

---

# Example

Large file:

```text
5 GB
```

---

Bad:

```js
fs.readFile()
```

Loads everything into memory.

---

Better:

```js
fs.createReadStream()
```

---

# Example

```js
const stream =
fs.createReadStream(
  "big.txt"
);

stream.on(
  "data",
  chunk => {

    console.log(chunk);

  }
);
```

---

# Stream Benefits

✅ Less memory

✅ Faster

✅ Scalable

---

# Types of Streams

---

## Readable Stream

Read data.

```js
fs.createReadStream()
```

---

## Writable Stream

Write data.

```js
fs.createWriteStream()
```

---

## Duplex Stream

Read + Write.

Example:

```text
Socket
```

---

## Transform Stream

Modify data.

Example:

```text
Compression
Encryption
```

---

# Stream Pipeline

```text
File
 ↓
Stream
 ↓
Process
 ↓
Output
```

---

# 9. Buffers

⚠️ Frequently asked with Streams.

---

## What is a Buffer?

Temporary memory area for binary data.

---

JavaScript normally works with:

```text
Strings
Objects
Arrays
```

---

But files and network data are:

```text
Binary Data
```

---

Buffers store that data.

---

# Create Buffer

```js
const buffer =
Buffer.from("Hello");
```

---

Output

```text
<Buffer 48 65 6c 6c 6f>
```

---

# Convert Buffer to String

```js
buffer.toString();
```

Output:

```text
Hello
```

---

# Why Buffers Exist?

Used for:

```text
Files
Images
Videos
Streams
Network Packets
```

---

# Stream + Buffer Relationship

When reading a file:

```text
File
 ↓
Chunks
 ↓
Buffers
```

---

Each chunk is usually a Buffer.

---

# Common Interview Questions

---

## Q1: What is Node.js?

A JavaScript runtime built on V8.

---

## Q2: Is Node.js Single-Threaded?

JavaScript execution is single-threaded.

Internally Node uses:

```text
libuv
Thread Pool
```

---

## Q3: What is Event-Driven Architecture?

Applications react to events instead of blocking execution.

---

## Q4: What is Non-Blocking I/O?

Operations that don't stop the main thread while waiting.

---

## Q5: What is libuv?

A C library that provides:

```text
Event Loop
Thread Pool
Async I/O
```

---

## Q6: What is Thread Pool?

A set of worker threads used for expensive operations.

---

## Q7: Default Thread Pool Size?

```text
4
```

---

## Q8: What is a Stream?

Processing data chunk by chunk instead of loading everything.

---

## Q9: What is a Buffer?

Temporary memory used to store binary data.

---

## Q10: Why Are Streams Better For Large Files?

Lower memory consumption.

---

# Interview Quick Revision 🚀

## Node.js

```text
JavaScript Runtime
```

---

## V8

```text
Executes JavaScript
```

---

## libuv

```text
Event Loop
Thread Pool
Async I/O
```

---

## Non-Blocking I/O

```text
Continue Execution
While Waiting
```

---

## Thread Pool

Default:

```text
4 Threads
```

---

## Streams

```js
createReadStream()
```

Process chunks.

---

## Buffers

```js
Buffer.from()
```

Store binary data.

---

# Most Important Interview Diagram ⭐

### Node.js Architecture

```text
Application
      ↓
JavaScript
      ↓
V8 Engine
      ↓
Node APIs
      ↓
libuv
      ↓
Operating System
```

---

### Async File Read

```text
fs.readFile()
       ↓
Thread Pool
       ↓
File Read
       ↓
Callback Queue
       ↓
Event Loop
       ↓
Callback Executes
```

---

### Stream Processing

```text
Large File
     ↓
Chunk
Chunk
Chunk
Chunk
     ↓
Process
```

---

### Buffer Flow

```text
File
 ↓
Buffer
 ↓
Stream
 ↓
Application
```

---

# Golden Interview Tips ⭐

1. JavaScript execution in Node.js is single-threaded.
2. Node.js internally uses libuv and worker threads.
3. libuv powers the Event Loop and asynchronous behavior.
4. Non-blocking I/O is a key reason Node.js scales well.
5. The default libuv thread pool size is 4.
6. Not all async operations use the thread pool.
7. Streams process data in chunks and save memory.
8. Buffers store binary data.
9. Know the difference between:
   ```text
   V8 → Executes JS

   libuv → Handles Async Operations
   ```
10. If asked:
   > "Why is Node.js fast?"
   
   Answer:

```text
Event Loop
+
Non-Blocking I/O
+
libuv
+
Efficient Resource Usage
```

🚀 Master Node.js Internals and you'll be able to explain how real-world backend systems handle massive numbers of concurrent requests.