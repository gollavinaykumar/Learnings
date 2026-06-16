# Java Performance Engineering - Complete Interview Revision Notes

> 🔴 VERY IMPORTANT FOR SENIOR JAVA / SPRING BOOT INTERVIEWS
>
> Performance Engineering focuses on:
>
> - Slow Applications
> - High CPU Usage
> - Memory Issues
> - Thread Problems
> - Garbage Collection Delays
> - Production Troubleshooting
>
> Topics Covered:
>
> - JVM Profiling
> - Memory Leaks
> - Thread Dumps
> - Heap Dumps
> - GC Tuning
> - JVisualVM
> - JProfiler
> - Production Debugging Scenarios

---

# What is Performance Engineering?

Performance Engineering is the process of:

```text
Finding Bottlenecks
Optimizing CPU Usage
Optimizing Memory Usage
Improving Response Time
Improving Throughput
```

---

# Real World Example

Spring Boot API

Expected:

```text
Response Time = 100ms
```

Actual:

```text
Response Time = 5 seconds
```

Performance Engineering helps identify:

```text
Database Issue?
Memory Leak?
CPU Bottleneck?
Thread Blocking?
GC Problem?
```

---

# Common Production Problems

```text
High CPU Usage

OutOfMemoryError

Slow APIs

Frequent Garbage Collection

Thread Deadlocks

Memory Leaks

Application Hangs
```

---

# JVM Profiling

Most Important Performance Topic.

---

# What is Profiling?

Profiling means:

```text
Analyzing Application Behavior
```

to understand:

```text
CPU Usage
Memory Usage
Thread Usage
Object Allocation
Method Execution Time
```

---

# Example

Problem:

```text
API Taking 10 Seconds
```

Profiler helps identify:

```text
Database Query = 9 Seconds
```

or

```text
Loop Consuming CPU
```

---

# What Can Profilers Show?

```text
Hot Methods
Memory Usage
Object Creation
CPU Consumption
Thread Activity
GC Activity
```

---

# Real World Example

```java
for(int i=0;i<100000000;i++){

}
```

Profiler shows:

```text
95% CPU Usage
```

inside loop.

---

# CPU Profiling

Measures:

```text
Which Methods Consume CPU
```

---

# Example

```java
processOrders();
```

Profiler:

```text
processOrders() → 80%

saveOrder() → 10%

sendMail() → 10%
```

---

# Optimization

Focus on:

```java
processOrders()
```

because it consumes most CPU.

---

# Memory Profiling

Measures:

```text
Object Allocation
Heap Usage
Memory Growth
```

---

# Example

```java
new ArrayList<>();
```

created millions of times.

Profiler shows:

```text
ArrayList
```

as top memory consumer.

---

# Memory Leak

Most Asked Production Interview Topic.

---

# What is Memory Leak?

Memory leak occurs when:

```text
Objects Are No Longer Needed
But Still Reachable
```

GC cannot remove them.

---

# Result

```text
Heap Memory Increases

GC Runs Frequently

OutOfMemoryError
```

---

# Example

Bad Code

```java
List<String> cache =
        new ArrayList<>();
```

```java
while(true){

    cache.add("data");
}
```

---

# Problem

```text
List Keeps Growing
```

Objects never released.

---

# Real World Example

Cache System

```java
Map<Long,User> users =
        new HashMap<>();
```

Adding data forever.

```java
users.put(id,user);
```

Never removing.

Memory leak.

---

# Symptoms Of Memory Leak

```text
Memory Usage Keeps Increasing

Frequent GC

Application Slowdown

OutOfMemoryError
```

---

# OutOfMemoryError Example

```text
java.lang.OutOfMemoryError:
Java heap space
```

---

# Common Causes

## Static Collections

```java
static List<User> users =
        new ArrayList<>();
```

Never cleared.

---

## Cache Misuse

```java
Map<String,Object> cache
```

Keeps growing.

---

## Unclosed Resources

```java
File Streams

DB Connections

Sockets
```

---

# Detecting Memory Leaks

Use:

```text
Heap Dump

JVisualVM

JProfiler
```

---

# Heap Dump

Most Important Debugging Tool.

---

# What is Heap Dump?

Heap Dump is:

```text
Snapshot Of Heap Memory
```

at a specific point in time.

---

# Contains

```text
All Objects

Object Count

Memory Usage

References

Object Relationships
```

---

# Why Heap Dump?

Used to identify:

```text
Memory Leaks

Large Objects

Retained Memory
```

---

# Generate Heap Dump

JVM Flag

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

---

# Heap Dump Location

```bash
-XX:HeapDumpPath=/logs
```

---

# Example

```bash
java \
-XX:+HeapDumpOnOutOfMemoryError \
-jar app.jar
```

---

# Heap Dump Analysis

Example Findings:

```text
10 Million ArrayLists

5 Million Strings

Huge Cache Object
```

Problem identified.

---

# Thread Dump

Very Important Interview Topic.

---

# What is Thread Dump?

Snapshot of all JVM threads.

---

# Contains

```text
Thread Names

Thread States

Stack Traces

Deadlocks

Blocked Threads
```

---

# Why Thread Dump?

Used when:

```text
Application Hangs

Slow Response

Deadlocks

High CPU Usage
```

---

# Generate Thread Dump

Linux

```bash
jstack <PID>
```

---

Example

```bash
jstack 12345
```

---

# Sample Thread States

```text
RUNNABLE

WAITING

BLOCKED

TIMED_WAITING
```

---

# Example

```java
synchronized(lock){

}
```

Thread waits.

State:

```text
BLOCKED
```

---

# Thread State Meanings

## RUNNABLE

```text
Thread Running
```

---

## BLOCKED

```text
Waiting For Lock
```

---

## WAITING

```text
Waiting Indefinitely
```

---

## TIMED_WAITING

```text
sleep()
wait(timeout)
```

---

# Deadlock Detection

Thread Dump Example

```text
Thread-1 waiting for lock2

Thread-2 waiting for lock1
```

Deadlock found.

---

# GC Tuning

Most Important Senior-Level Topic.

---

# Why GC Tuning?

Problems:

```text
Frequent GC

Long GC Pause

High Memory Usage

Slow APIs
```

---

# Example

Application:

```text
4 GB Heap
```

GC runs every:

```text
5 Seconds
```

Application becomes slow.

---

# Goal Of GC Tuning

```text
Reduce Pause Time

Reduce GC Frequency

Improve Throughput
```

---

# Common GC Collectors

## Serial GC

```bash
-XX:+UseSerialGC
```

Single Thread.

Small Applications.

---

## Parallel GC

```bash
-XX:+UseParallelGC
```

High Throughput.

---

## G1 GC

Most Common Today.

```bash
-XX:+UseG1GC
```

---

# Benefits

```text
Low Pause Time

Large Heap Support

Good Performance
```

---

## ZGC

Ultra Low Latency.

```bash
-XX:+UseZGC
```

---

Pause:

```text
< 10ms
```

---

# Useful GC Flags

## Enable G1

```bash
-XX:+UseG1GC
```

---

## GC Logging

```bash
-Xlog:gc
```

---

## Heap Size

```bash
-Xms1g
-Xmx4g
```

---

## Pause Time Goal

```bash
-XX:MaxGCPauseMillis=200
```

---

# Example Production JVM

```bash
java \
-Xms2g \
-Xmx4g \
-XX:+UseG1GC \
-Xlog:gc \
-jar app.jar
```

---

# JVisualVM

Most Common Free Tool.

---

# What is JVisualVM?

Free JVM Monitoring Tool.

Comes with JDK.

---

# Features

```text
CPU Profiling

Memory Profiling

Heap Dumps

Thread Dumps

GC Monitoring
```

---

# Use Cases

```text
Find Memory Leaks

Find High CPU Methods

Monitor Heap Usage
```

---

# Example Workflow

```text
Connect To JVM
↓
Monitor Memory
↓
Take Heap Dump
↓
Analyze Objects
```

---

# JProfiler

Most Popular Commercial Tool.

---

# What is JProfiler?

Advanced JVM profiler.

---

# Features

```text
CPU Analysis

Memory Leak Detection

Heap Analysis

Thread Analysis

Database Profiling

JPA Profiling

Spring Profiling
```

---

# Why JProfiler?

Provides more detailed analysis than JVisualVM.

---

# Example

Slow API:

```text
Response = 8 seconds
```

JProfiler shows:

```text
Database Query = 7.5 seconds
```

Problem immediately identified.

---

# Real Production Debugging Scenario

Problem:

```text
Spring Boot App

Memory Increasing

CPU Normal
```

---

# Investigation

Step 1

```text
Take Heap Dump
```

---

Step 2

Analyze:

```text
Large HashMap
```

found.

---

Step 3

Cause:

```java
static Map<Long,User> cache
```

never cleared.

---

Step 4

Fix:

```text
Use Cache Eviction

Use Redis

Remove Old Entries
```

---

# Performance Troubleshooting Flow

```text
Slow Application
       ↓
CPU High?
       ↓
CPU Profiling

Memory High?
       ↓
Heap Dump

App Hanging?
       ↓
Thread Dump

Frequent GC?
       ↓
GC Logs

OutOfMemory?
       ↓
Heap Analysis
```

---

# Frequently Asked Interview Questions

## Q1. What is JVM Profiling?

```text
Analyzing CPU, Memory,
Threads and Object Allocation.
```

---

## Q2. What is Memory Leak?

```text
Objects Not Needed
But Still Reachable
```

GC cannot remove them.

---

## Q3. What is Heap Dump?

```text
Snapshot Of JVM Heap Memory
```

Used to detect memory leaks.

---

## Q4. What is Thread Dump?

```text
Snapshot Of All JVM Threads
```

Used to detect:

```text
Deadlocks
Blocked Threads
Hangs
```

---

## Q5. Difference Between Heap Dump And Thread Dump?

Heap Dump:

```text
Memory Analysis
```

Thread Dump:

```text
Thread Analysis
```

---

## Q6. Which GC Is Most Common Today?

```text
G1GC
```

---

## Q7. How To Detect Memory Leak?

```text
Heap Dump
JVisualVM
JProfiler
```

---

## Q8. What Causes OutOfMemoryError?

```text
Memory Leak

Insufficient Heap

Huge Object Creation
```

---

## Q9. Why Use JVisualVM?

```text
Free Monitoring Tool
```

---

## Q10. Why Use JProfiler?

```text
Advanced Profiling
Detailed Analysis
```

---

# One-Minute Revision

```text
Performance Engineering
--------------------
Find Bottlenecks

JVM Profiling
--------------------
CPU
Memory
Threads

Memory Leak
--------------------
Objects Not Released

Heap Dump
--------------------
Memory Snapshot

Thread Dump
--------------------
Thread Snapshot

GC Tuning
--------------------
Reduce Pause Time
Improve Throughput

Useful JVM Flags
--------------------
-Xms
-Xmx
-Xlog:gc

GC Types
--------------------
Serial
Parallel
G1GC
ZGC

Tools
--------------------
JVisualVM
JProfiler

Interview Favorites
--------------------
Memory Leak
Heap Dump
Thread Dump
GC Tuning
JVisualVM
JProfiler
```

# End of Java Performance Engineering Notes