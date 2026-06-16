# Java IO & NIO - Complete Interview Revision Notes

> ⭐ IMPORTANT FOR JAVA BACKEND & SPRING BOOT INTERVIEWS
>
> IO is used whenever data needs to be:
>
> - Read from files
> - Written to files
> - Read from network
> - Written to network
> - Process logs
> - Upload/Download files
>
> Topics Covered:
>
> - What is IO?
> - Traditional IO
> - FileReader
> - FileWriter
> - NIO
> - Buffers
> - Channels
> - Selectors
> - Blocking vs Non-Blocking IO
> - Real World Examples
> - Performance Comparison

---

# What is IO?

IO stands for:

```text
Input / Output
```

Input:

```text
Reading Data
```

Output:

```text
Writing Data
```

---

# Real World Examples

Input:

```text
Read File
Read User Input
Read Network Request
Read Database Export
```

---

Output:

```text
Write File
Generate Report
Send Response
Store Logs
```

---

# Traditional IO (java.io)

Java originally introduced:

```java
java.io
```

Package.

Known as:

```text
Traditional IO
Blocking IO
```

---

# Why Called Blocking IO?

When reading data:

```java
read()
```

thread waits until operation completes.

Example:

```text
Read Large File
      ↓
Thread Waits
      ↓
Continue Execution
```

---

# IO Architecture

```text
Application
      ↓
   Stream
      ↓
     File
```

---

# Streams

Traditional IO is stream based.

Types:

```text
Input Stream
Output Stream
Reader
Writer
```

---

# Byte Streams

Used for:

```text
Images
Videos
PDFs
Binary Data
```

Examples:

```java
FileInputStream
FileOutputStream
```

---

# Character Streams

Used for:

```text
Text Files
CSV
JSON
XML
```

Examples:

```java
FileReader
FileWriter
```

---

# FileReader

## Definition

Used to read text files.

---

# Example

test.txt

```text
Hello Java
```

Program:

```java
import java.io.*;

public class Main {

    public static void main(String[] args)
            throws Exception {

        FileReader reader =
                new FileReader("test.txt");

        int ch;

        while((ch = reader.read()) != -1){

            System.out.print((char) ch);
        }

        reader.close();
    }
}
```

Output:

```text
Hello Java
```

---

# How It Works

```java
reader.read()
```

Reads one character at a time.

---

# Real World Use Cases

```text
Read Config Files
Read CSV
Read JSON
Read XML
```

---

# FileWriter

## Definition

Used to write text files.

---

# Example

```java
import java.io.*;

public class Main {

    public static void main(String[] args)
            throws Exception {

        FileWriter writer =
                new FileWriter("test.txt");

        writer.write("Hello Java");

        writer.close();
    }
}
```

File Content:

```text
Hello Java
```

---

# Append Mode

Default:

```text
Overwrite Existing File
```

---

Append:

```java
FileWriter writer =
        new FileWriter(
                "test.txt",
                true);
```

---

# Example

```java
writer.write("\nSpring Boot");
```

Output:

```text
Hello Java
Spring Boot
```

---

# Traditional IO Problems

Suppose:

```text
1 GB File
```

Reading:

```java
reader.read()
```

one character at a time.

---

Problems:

```text
Slow
Many System Calls
Thread Blocking
High Memory Usage
```

---

# Solution → NIO

Java 1.4 introduced:

```java
java.nio
```

NIO means:

```text
New IO
```

---

# Why NIO?

Designed for:

```text
High Performance
Large Files
Network Servers
Scalable Applications
```

---

# Traditional IO vs NIO

| Feature | IO | NIO |
|----------|----------|----------|
| Data Flow | Stream | Buffer |
| Blocking | Yes | Can Be Non-Blocking |
| Performance | Lower | Higher |
| Large Files | Slower | Faster |
| Multiple Connections | Difficult | Easy |

---

# NIO Architecture

```text
Application
     ↓
 Buffer
     ↓
 Channel
     ↓
 File / Socket
```

---

# Core NIO Components

```text
Buffer
Channel
Selector
```

---

# Buffer

Most Important NIO Concept.

---

# What is Buffer?

Buffer is a temporary memory area used to store data.

Think:

```text
Bucket Of Data
```

instead of:

```text
One Character At A Time
```

---

# Example

```java
ByteBuffer buffer =
        ByteBuffer.allocate(1024);
```

Creates:

```text
1 KB Buffer
```

---

# Buffer Working

```text
Read Data
     ↓
Store In Buffer
     ↓
Process Data
```

---

# Real Life Example

Traditional IO:

```text
Read
Read
Read
Read
```

---

Buffer:

```text
Read Large Chunk
Process Chunk
```

Faster.

---

# Important Buffer Methods

## put()

Insert data.

```java
buffer.put((byte)65);
```

---

## flip()

Switch write mode to read mode.

```java
buffer.flip();
```

---

## get()

Read data.

```java
byte b = buffer.get();
```

---

## clear()

Reset buffer.

```java
buffer.clear();
```

---

# Example

```java
ByteBuffer buffer =
        ByteBuffer.allocate(10);

buffer.put((byte)65);

buffer.flip();

System.out.println(
        (char)buffer.get());
```

Output:

```text
A
```

---

# Channel

Most Asked NIO Interview Question.

---

# What is Channel?

Channel is like a pipe that transfers data.

---

Traditional IO:

```text
One-Way Stream
```

---

Channel:

```text
Two-Way Communication
```

---

# Example

```java
FileChannel channel =
        FileChannel.open(
                Path.of("test.txt"));
```

---

# Data Flow

```text
File
  ↕
Channel
  ↕
Buffer
```

---

# Example

```java
FileChannel channel =
        FileChannel.open(
                Path.of("test.txt"));

ByteBuffer buffer =
        ByteBuffer.allocate(1024);

channel.read(buffer);
```

---

# Real World Example

Large File Upload

```text
Disk
 ↕
Channel
 ↕
Buffer
 ↕
Application
```

Much faster than traditional IO.

---

# Channel Types

## FileChannel

```java
FileChannel
```

File Operations.

---

## SocketChannel

```java
SocketChannel
```

Network Communication.

---

## ServerSocketChannel

```java
ServerSocketChannel
```

Server Side Networking.

---

# Selector

Most Important NIO Interview Topic.

---

# What is Selector?

Selector allows one thread to monitor multiple channels.

---

Traditional Server

```text
Connection 1 → Thread 1

Connection 2 → Thread 2

Connection 3 → Thread 3
```

10000 users:

```text
10000 Threads
```

Huge memory.

---

# Using Selector

```text
Connection 1
Connection 2
Connection 3
Connection 4
      ↓
   Selector
      ↓
  Single Thread
```

---

# Why Powerful?

Used by:

```text
Netty
Kafka
Tomcat NIO
High Scale Servers
```

---

# Example

```java
Selector selector =
        Selector.open();
```

Register channel:

```java
channel.register(
        selector,
        SelectionKey.OP_READ);
```

Wait for events:

```java
selector.select();
```

---

# Real World Example

WhatsApp Server

Millions of connections.

Without selector:

```text
Millions Of Threads
```

Impossible.

---

With Selector:

```text
Few Threads
Millions Of Connections
```

---

# Blocking vs Non-Blocking IO

Most Asked Interview Question.

---

# Blocking IO

Example:

```java
reader.read();
```

Thread waits.

```text
Read File
     ↓
Wait
     ↓
Continue
```

---

# Non-Blocking IO

Example:

```java
channel.configureBlocking(false);
```

Thread does not wait.

```text
Request Read
     ↓
Continue Work
     ↓
Data Arrives Later
```

---

# Real World Example

Food Delivery App

Blocking:

```text
Call Restaurant
Wait
Wait
Wait
```

---

Non-Blocking:

```text
Call Restaurant
Continue Other Work
Receive Callback Later
```

---

# IO vs NIO Performance

Suppose:

```text
10000 Client Connections
```

Traditional IO:

```text
10000 Threads
High Memory
Slow
```

---

NIO:

```text
Few Threads
Selector
Fast
Scalable
```

---

# Modern Framework Usage

## Spring Boot

Uses:

```text
Traditional IO
NIO
Netty
```

depending on stack.

---

## Tomcat

Uses:

```text
NIO Connectors
```

---

## Netty

Built completely using:

```text
NIO
Selectors
Channels
```

---

## Kafka

Uses:

```text
NIO + Selector
```

for millions of messages.

---

# Frequently Asked Interview Questions

## Q1. Difference Between FileReader and FileInputStream?

FileReader:

```text
Character Data
```

FileInputStream:

```text
Binary Data
```

---

## Q2. What is NIO?

```text
New IO API
Designed For High Performance
```

---

## Q3. Difference Between IO and NIO?

IO:

```text
Stream Based
Blocking
```

NIO:

```text
Buffer Based
Non-Blocking
```

---

## Q4. What is Buffer?

```text
Temporary Memory Area
Used To Store Data
```

---

## Q5. What is Channel?

```text
Data Transfer Pipe
Between Buffer And Source
```

---

## Q6. What is Selector?

```text
Allows One Thread
To Monitor Multiple Channels
```

---

## Q7. Why NIO Is Faster?

```text
Buffers
Channels
Selectors
Less Thread Usage
```

---

## Q8. Where Is NIO Used?

```text
Kafka
Netty
Tomcat
Spring WebFlux
High Scale Servers
```

---

# One-Minute Revision

```text
Traditional IO
-----------------
Stream Based
Blocking

FileReader
-----------------
Read Text Files

FileWriter
-----------------
Write Text Files

NIO
-----------------
New IO
High Performance

Buffer
-----------------
Temporary Data Storage

Important Methods
-----------------
put()
flip()
get()
clear()

Channel
-----------------
Transfer Data

Types
-----------------
FileChannel
SocketChannel
ServerSocketChannel

Selector
-----------------
Monitor Multiple Channels

Blocking IO
-----------------
Thread Waits

Non-Blocking IO
-----------------
Thread Continues

Interview Favorites
-----------------
IO vs NIO
Buffer
Channel
Selector
Blocking vs Non-Blocking
```

# End of Java IO & NIO Notes