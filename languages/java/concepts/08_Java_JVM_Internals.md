# Java JVM Internals - Complete Interview Revision Notes

> ⭐ VERY IMPORTANT FOR INTERVIEWS
>
> JVM Internals are frequently asked in:
>
> - Java Developer Interviews
> - Spring Boot Interviews
> - Backend Developer Interviews
> - Senior Java Interviews
>
> Topics Covered:
>
> - JVM Architecture
> - Class Loading Mechanism
> - Heap vs Stack Memory
> - Garbage Collection
> - GC Types
> - Memory Tuning
> - JVM Flags
> - Real World Examples
> - Frequently Asked Interview Questions

---

# What is JVM?

JVM stands for:

```text
Java Virtual Machine
```

It is the engine responsible for:

```text
Loading Classes
Executing Bytecode
Memory Management
Garbage Collection
```

---

# Java Execution Flow

```text
Java Source Code
      ↓
   javac
      ↓
Bytecode (.class)
      ↓
     JVM
      ↓
Machine Code
      ↓
    Output
```

---

# JVM Architecture

```text
               JVM
                |
      --------------------
      |                  |
  Class Loader      Runtime Memory
                          |
      ----------------------------------
      |         |         |            |
    Heap      Stack    Method Area   PC Register
```

---

# Class Loading Mechanism

Most Asked JVM Interview Question.

---

# What is Class Loading?

Class Loading means:

```text
Loading .class files into memory
before execution.
```

Example:

```java
User user = new User();
```

Before object creation:

```text
User.class
```

must be loaded into JVM.

---

# Phases of Class Loading

```text
Loading
   ↓
Linking
   ↓
Initialization
```

---

# Step 1: Loading

Class Loader loads:

```text
User.class
```

into JVM memory.

---

# Step 2: Linking

Three sub-phases:

```text
Verification
Preparation
Resolution
```

---

## Verification

Checks:

```text
Bytecode Valid?
Security Rules?
Java Format Correct?
```

---

## Preparation

Allocates memory for:

```java
static variables
```

Example:

```java
class Test {

    static int count = 10;
}
```

Memory reserved.

---

## Resolution

Converts symbolic references into actual references.

Example:

```java
String
Object
List
```

resolved.

---

# Step 3: Initialization

Executes:

```text
Static Variables
Static Blocks
```

Example:

```java
class Test {

    static {

        System.out.println(
                "Static Block");
    }
}
```

Output:

```text
Static Block
```

---

# Class Loader Types

Interview Favorite.

---

# 1. Bootstrap ClassLoader

Loads:

```text
Core Java Classes
```

Examples:

```java
String
Object
Integer
```

Location:

```text
JAVA_HOME/lib
```

---

# 2. Extension ClassLoader

Loads:

```text
Extension Libraries
```

---

# 3. Application ClassLoader

Loads:

```text
Application Classes
```

Examples:

```java
Employee.class
User.class
Product.class
```

---

# Class Loader Hierarchy

```text
Bootstrap
    ↑
Extension
    ↑
Application
```

---

# Parent Delegation Model

Important Interview Question.

---

# How It Works

When JVM needs a class:

```text
Application Loader
      ↓
Extension Loader
      ↓
Bootstrap Loader
```

Search starts from parent.

---

# Why?

Security.

Prevents malicious replacement of:

```java
java.lang.String
```

---

# Runtime Memory Areas

JVM memory divided into multiple areas.

---

# Heap Memory

Most Important.

---

# What is Heap?

Heap stores:

```text
Objects
Arrays
Instance Variables
```

---

# Example

```java
User user =
        new User();
```

Object stored in:

```text
Heap
```

---

# Characteristics

```text
Shared By All Threads
Managed By GC
```

---

# Real World Example

```java
new Employee()

new ArrayList()

new HashMap()
```

All stored in Heap.

---

# Stack Memory

Stores:

```text
Method Calls
Local Variables
References
```

---

# Example

```java
public void show() {

    int age = 25;
}
```

Stored in:

```text
Stack
```

---

# Stack Example

```java
public static void main(String[] args) {

    show();
}
```

```java
static void show() {

    int x = 10;
}
```

Memory:

```text
Stack

main()
show()
x=10
```

Method finishes:

```text
Stack Frame Removed
```

---

# Heap vs Stack

Most Asked Interview Question.

| Feature | Heap | Stack |
|----------|----------|----------|
| Stores | Objects | Local Variables |
| Shared | Yes | No |
| Managed By | GC | JVM |
| Size | Larger | Smaller |
| Access Speed | Slower | Faster |

---

# Memory Example

```java
User user =
        new User();
```

Memory:

```text
Stack
-----
user reference

Heap
-----
User Object
```

---

# JVM Memory Diagram

```text
Stack
------
user

Heap
------
User Object

Method Area
------
Class Metadata
Static Variables
```

---

# Method Area

Stores:

```text
Class Metadata
Static Variables
Runtime Constant Pool
```

Example:

```java
static int count = 100;
```

Stored in:

```text
Method Area
```

---

# Program Counter Register

Stores:

```text
Current Instruction Address
```

Each thread has its own PC Register.

---

# Native Method Stack

Used for:

```text
JNI Calls
Native Methods
```

Example:

```java
C++
Operating System Calls
```

---

# Garbage Collection

Most Important JVM Topic.

---

# What is Garbage Collection?

Automatic process of removing unused objects.

---

# Example

```java
User user =
        new User();

user = null;
```

Object has no reference.

Eligible for:

```text
Garbage Collection
```

---

# Why GC?

Without GC:

```text
Memory Leak
Application Crash
OutOfMemoryError
```

---

# Garbage Collector Flow

```text
Object Created
      ↓
Used
      ↓
Reference Removed
      ↓
Eligible For GC
      ↓
Memory Reclaimed
```

---

# How GC Identifies Objects?

Uses:

```text
Reachability Analysis
```

---

# Reachable Object

```java
User user =
        new User();
```

Reference exists.

GC won't remove.

---

# Unreachable Object

```java
user = null;
```

No reference.

GC may remove.

---

# Garbage Collection Types

Interview Favorite.

---

# 1. Serial GC

Single thread performs GC.

JVM Flag:

```bash
-XX:+UseSerialGC
```

Suitable:

```text
Small Applications
```

---

# 2. Parallel GC

Multiple threads perform GC.

Default in many JVMs.

JVM Flag:

```bash
-XX:+UseParallelGC
```

Suitable:

```text
High Throughput Applications
```

---

# 3. G1 GC

Most Common Modern GC.

G1 =

```text
Garbage First
```

JVM Flag:

```bash
-XX:+UseG1GC
```

Suitable:

```text
Large Heap Applications
```

---

# Features

```text
Low Pause Time
High Performance
Efficient Memory Usage
```

---

# 4. ZGC

Ultra-low latency GC.

JVM Flag:

```bash
-XX:+UseZGC
```

Suitable:

```text
Huge Applications
```

Pause time:

```text
Less Than 10 ms
```

---

# 5. Shenandoah GC

Low latency collector.

JVM Flag:

```bash
-XX:+UseShenandoahGC
```

---

# GC Comparison

| GC | Threads | Performance |
|------|------|------|
| Serial | Single | Small Apps |
| Parallel | Multiple | Throughput |
| G1 | Multiple | Balanced |
| ZGC | Multiple | Ultra Low Latency |
| Shenandoah | Multiple | Low Pause |

---

# Memory Tuning

Very Common in Production Interviews.

---

# Why Tune JVM?

Problems:

```text
High Memory Usage
Frequent GC
Slow Response Time
OutOfMemoryError
```

---

# Heap Size Settings

## Initial Heap

```bash
-Xms512m
```

Means:

```text
Start Heap = 512 MB
```

---

## Maximum Heap

```bash
-Xmx2g
```

Means:

```text
Maximum Heap = 2 GB
```

---

# Example

```bash
java -Xms512m -Xmx2g App
```

---

# Young Generation Size

```bash
-XX:NewSize=256m
```

---

# Survivor Ratio

```bash
-XX:SurvivorRatio=8
```

---

# Why Increase Heap?

Scenario:

```text
OutOfMemoryError
```

Solution:

```bash
-Xmx4g
```

Increase heap memory.

---

# Common JVM Flags

Most Asked DevOps/Backend Question.

---

# Heap Configuration

```bash
-Xms512m
```

Initial Heap.

---

```bash
-Xmx2g
```

Maximum Heap.

---

# GC Selection

```bash
-XX:+UseG1GC
```

Enable G1 GC.

---

```bash
-XX:+UseZGC
```

Enable ZGC.

---

# GC Logging

```bash
-Xlog:gc
```

Logs GC activity.

---

# Heap Dump

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

Generates heap dump.

---

# Heap Dump Location

```bash
-XX:HeapDumpPath=/logs
```

---

# Print JVM Settings

```bash
-XX:+PrintFlagsFinal
```

---

# Production Example

Spring Boot Application

```bash
java \
-Xms1g \
-Xmx4g \
-XX:+UseG1GC \
-Xlog:gc \
jar app.jar
```

---

# Real World Example

E-commerce Application

Problems:

```text
Slow Performance
Frequent GC
High Memory Usage
```

Solution:

```bash
Increase Heap
Use G1GC
Analyze Heap Dump
```

---

# Frequently Asked Interview Questions

## Q1. Difference Between Heap and Stack?

Heap:

```text
Objects
Shared
GC Managed
```

Stack:

```text
Local Variables
Method Calls
Thread Specific
```

---

## Q2. What is Class Loading?

```text
Loading .class files
into JVM memory.
```

---

## Q3. What are Class Loader Types?

```text
Bootstrap
Extension
Application
```

---

## Q4. What is Parent Delegation Model?

```text
Child asks Parent first
before loading class.
```

---

## Q5. What is Garbage Collection?

```text
Automatic removal
of unused objects.
```

---

## Q6. Which GC is Most Used Today?

```text
G1GC
```

---

## Q7. Difference Between Serial GC and G1GC?

Serial:

```text
Single Thread
```

G1:

```text
Multiple Threads
Low Pause Time
```

---

## Q8. What is Xms?

```text
Initial Heap Size
```

---

## Q9. What is Xmx?

```text
Maximum Heap Size
```

---

## Q10. What causes OutOfMemoryError?

```text
Heap Full
Memory Leak
Insufficient Heap
```

---

# One-Minute Revision

```text
JVM
-------------
Executes Bytecode

Class Loading
-------------
Loading
Linking
Initialization

Class Loaders
-------------
Bootstrap
Extension
Application

Heap
-------------
Objects
Arrays
Shared

Stack
-------------
Local Variables
Method Calls

Method Area
-------------
Class Metadata
Static Variables

Garbage Collection
-------------
Removes Unused Objects

GC Types
-------------
Serial
Parallel
G1GC
ZGC
Shenandoah

Memory Tuning
-------------
-Xms
-Xmx

Important JVM Flags
-------------
-Xms512m
-Xmx2g
-XX:+UseG1GC
-Xlog:gc

Interview Favorites
-------------
Heap vs Stack
Class Loading
Parent Delegation
GC Types
Memory Tuning
JVM Flags
```

# End of JVM Internals Notes