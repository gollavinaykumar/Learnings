# Java Multithreading & Concurrency - Complete Interview Revision Notes

> ⭐ VERY IMPORTANT FOR INTERVIEWS
>
> Asked in:
>
> - Java Developer
> - Backend Developer
> - Spring Boot Developer
> - Microservices
> - Senior Java Interviews
>
> Topics Covered:
>
> - Thread Basics
> - Thread Lifecycle
> - Thread Class
> - Runnable Interface
> - Lambda Threads
> - Race Condition
> - Synchronization
> - Locks
> - Deadlock
> - ExecutorService
> - ForkJoinPool
> - CompletableFuture
> - Parallel Streams
> - Real World Use Cases

---

# What is Multithreading?

Multithreading means:

```text
Executing Multiple Threads Concurrently
```

Instead of doing one task at a time:

```text
Task 1
Task 2
Task 3
```

CPU can execute:

```text
Task 1 + Task 2 + Task 3
```

simultaneously (or seemingly simultaneously).

---

# Real Life Example

Food Delivery App

Without Threads:

```text
Receive Order
↓
Process Payment
↓
Assign Delivery Boy
↓
Send Notification
```

Everything waits.

---

With Threads:

```text
Thread 1 -> Payment

Thread 2 -> Notification

Thread 3 -> Delivery Assignment
```

Faster execution.

---

# What is a Thread?

A thread is the smallest unit of execution inside a process.

Example:

```text
Chrome Browser

Process
 ├── Tab 1 Thread
 ├── Tab 2 Thread
 ├── Download Thread
 └── Rendering Thread
```

---

# Creating Threads

Two common ways:

```text
1. Thread Class
2. Runnable Interface
```

---

# Using Thread Class

## Step 1

Extend Thread

```java
class MyThread extends Thread {

    @Override
    public void run() {

        System.out.println(
                "Thread Running");
    }
}
```

---

## Step 2

Start Thread

```java
public class Main {

    public static void main(String[] args) {

        MyThread t = new MyThread();

        t.start();
    }
}
```

Output:

```text
Thread Running
```

---

# Why start() Not run()?

Wrong:

```java
t.run();
```

Executes like normal method.

---

Correct:

```java
t.start();
```

Creates separate thread.

---

# Runnable Interface

Preferred approach.

---

## Step 1

Implement Runnable

```java
class MyTask implements Runnable {

    @Override
    public void run() {

        System.out.println(
                "Task Executed");
    }
}
```

---

## Step 2

Pass Runnable To Thread

```java
public class Main {

    public static void main(String[] args) {

        Runnable task =
                new MyTask();

        Thread t =
                new Thread(task);

        t.start();
    }
}
```

Output:

```text
Task Executed
```

---

# Why Runnable Is Preferred?

Java supports:

```text
Single Inheritance
```

If class already extends another class:

```java
class Employee extends Person
```

Cannot extend Thread.

Runnable solves this problem.

---

# Lambda Thread (Java 8)

Most Common Modern Syntax

```java
new Thread(() -> {

    System.out.println(
            "Thread Running");

}).start();
```

Output:

```text
Thread Running
```

---

# Real World Example

Sending Email

```java
new Thread(() -> {

    sendEmail();

}).start();
```

User doesn't wait.

Email runs in background.

---

# Thread Lifecycle

Interview Favorite

---

# States

```text
NEW
 ↓
RUNNABLE
 ↓
RUNNING
 ↓
WAITING/BLOCKED
 ↓
TERMINATED
```

---

# Example

```java
Thread t =
        new Thread(() -> {

            System.out.println(
                    "Running");
        });
```

State:

```text
NEW
```

---

After:

```java
t.start();
```

State:

```text
RUNNABLE
```

---

After completion:

```text
TERMINATED
```

---

# Thread Lifecycle Diagram

```text
NEW
 ↓
start()
 ↓
RUNNABLE
 ↓
CPU Assigned
 ↓
RUNNING
 ↓
sleep()/wait()
 ↓
WAITING
 ↓
Resume
 ↓
RUNNING
 ↓
Completed
 ↓
TERMINATED
```

---

# Race Condition

Very Important

---

# What is Race Condition?

Occurs when multiple threads modify shared data simultaneously.

Result becomes unpredictable.

---

# Example

Bank Account

Balance:

```text
1000
```

Thread A:

```text
Withdraw 500
```

Thread B:

```text
Withdraw 700
```

Both execute simultaneously.

Incorrect balance may occur.

---

# Example Code

```java
class Counter {

    int count = 0;

    void increment() {

        count++;
    }
}
```

Two threads:

```java
Counter c = new Counter();

Thread t1 =
        new Thread(() -> {

            for(int i=0;i<1000;i++)
                c.increment();
        });

Thread t2 =
        new Thread(() -> {

            for(int i=0;i<1000;i++)
                c.increment();
        });

t1.start();
t2.start();
```

Expected:

```text
2000
```

Actual:

```text
1987
1992
1998
```

Random.

---

# Why?

Because:

```java
count++
```

is not atomic.

Internally:

```text
Read
Modify
Write
```

Multiple steps.

---

# Synchronization

Solution to Race Condition.

---

# What is Synchronization?

Allows only one thread at a time
to access critical section.

---

# Example

```java
class Counter {

    int count = 0;

    synchronized void increment() {

        count++;
    }
}
```

Now:

```text
One Thread At A Time
```

Output:

```text
2000
```

Correct.

---

# Real World Example

ATM Machine

Only one transaction should update account balance at a time.

---

# Synchronization Block

Instead of whole method:

```java
synchronized(this) {

    count++;
}
```

Locks only critical section.

Better performance.

---

# Locks

Advanced Synchronization

Package:

```java
java.util.concurrent.locks
```

---

# ReentrantLock Example

```java
import java.util.concurrent.locks.*;

class Counter {

    Lock lock =
            new ReentrantLock();

    int count=0;

    void increment() {

        lock.lock();

        try {

            count++;

        } finally {

            lock.unlock();
        }
    }
}
```

---

# Why Locks?

More control than synchronized.

Supports:

```text
Try Lock
Timeout
Fairness
Interruptible Lock
```

---

# Deadlock

Most Asked Interview Question.

---

# What is Deadlock?

Two threads waiting forever for each other.

---

# Real Life Example

Person A:

```text
Needs Resource B
```

Person B:

```text
Needs Resource A
```

Both waiting forever.

---

# Deadlock Code

```java
Object lock1 =
        new Object();

Object lock2 =
        new Object();
```

Thread 1:

```java
synchronized(lock1) {

    synchronized(lock2) {

    }
}
```

Thread 2:

```java
synchronized(lock2) {

    synchronized(lock1) {

    }
}
```

Possible:

```text
Deadlock
```

---

# Deadlock Prevention

Always acquire locks in same order.

Good:

```text
Lock A
↓
Lock B
```

for all threads.

---

# ExecutorService

Very Important For Modern Java.

---

# Problem

Creating threads manually:

```java
new Thread(...)
```

for every task is expensive.

---

# Solution

ExecutorService

Manages thread pool.

---

# Example

```java
import java.util.concurrent.*;

ExecutorService executor =
        Executors.newFixedThreadPool(3);

executor.submit(() -> {

    System.out.println(
            Thread.currentThread()
                  .getName());
});

executor.shutdown();
```

---

# Real World Example

E-commerce Website

```text
Order Processing
Payment Processing
Inventory Update
```

Handled by thread pool.

---

# Benefits

```text
Thread Reuse
Better Performance
Easy Management
```

---

# ForkJoinPool

Used for:

```text
Divide And Conquer Tasks
```

Large tasks split into smaller tasks.

---

# Real World Example

Sum:

```text
1 to 1,000,000
```

Split into:

```text
1-250000
250001-500000
500001-750000
750001-1000000
```

Process in parallel.

---

# Example

```java
ForkJoinPool pool =
        new ForkJoinPool();

pool.submit(() -> {

    System.out.println(
            "Fork Join Task");
});
```

---

# CompletableFuture

Most Important Java 8+ Topic.

---

# What is CompletableFuture?

Allows asynchronous programming.

Run task without blocking main thread.

---

# Example

```java
CompletableFuture.runAsync(() -> {

    System.out.println(
            "Background Task");
});
```

Output:

```text
Background Task
```

---

# Real World Example

Food Delivery App

While order is processing:

```text
Payment
Notification
Tracking
```

run asynchronously.

---

# Returning Value

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(
                () -> "Order Ready");

System.out.println(
        future.join());
```

Output:

```text
Order Ready
```

---

# Chaining

```java
CompletableFuture
        .supplyAsync(() -> "Java")
        .thenApply(s -> s + " Spring")
        .thenAccept(System.out::println);
```

Output:

```text
Java Spring
```

---

# Parallel Streams

Java 8 Feature

---

# Normal Stream

```java
list.stream()
```

Single thread.

---

# Parallel Stream

```java
list.parallelStream()
```

Multiple threads.

---

# Example

```java
List<Integer> nums =
        Arrays.asList(
                1,2,3,4,5,6);

nums.parallelStream()
        .forEach(System.out::println);
```

Multiple threads process data.

---

# Real World Example

Processing:

```text
10 Lakh Records
```

Can be split across CPU cores.

---

# When To Use?

Good For:

```text
Large Data Processing
CPU Intensive Work
```

Avoid For:

```text
Small Collections
Database Calls
```

---

# ExecutorService vs CompletableFuture

| Feature | ExecutorService | CompletableFuture |
|----------|----------|----------|
| Thread Pool | Yes | Uses Pool |
| Async Tasks | Basic | Advanced |
| Chaining | No | Yes |
| Error Handling | Manual | Built-In |
| Modern Java | Good | Best |

---

# synchronized vs Lock

| Feature | synchronized | Lock |
|----------|----------|----------|
| Simpler | Yes | No |
| Timeout | No | Yes |
| Fairness | No | Yes |
| Manual Unlock | No | Yes |

---

# Frequently Asked Interview Questions

## Q1. Difference Between Thread and Runnable?

Thread:

```text
Class
```

Runnable:

```text
Interface
Preferred Approach
```

---

## Q2. What is Race Condition?

```text
Multiple Threads
Access Shared Data
Simultaneously
```

Produces inconsistent results.

---

## Q3. What is Synchronization?

```text
Allows Only One Thread
To Access Critical Section
At A Time
```

---

## Q4. What is Deadlock?

```text
Two Threads Waiting Forever
For Each Other's Lock
```

---

## Q5. Why ExecutorService?

```text
Thread Pool Management
Thread Reuse
Better Performance
```

---

## Q6. What is CompletableFuture?

```text
Asynchronous Programming API
```

---

## Q7. Difference Between StringBuffer And Synchronization?

Both are thread-safe because synchronization is used internally.

---

## Q8. Difference Between Parallel Stream and Stream?

Stream:

```text
Single Thread
```

Parallel Stream:

```text
Multiple Threads
```

---

# One-Minute Revision

```text
Thread Creation
-------------
Thread Class
Runnable Interface
Lambda Thread

Thread Lifecycle
-------------
NEW
RUNNABLE
RUNNING
WAITING
TERMINATED

Race Condition
-------------
Shared Data Problem

Synchronization
-------------
One Thread At A Time

Lock
-------------
Advanced Synchronization

Deadlock
-------------
Threads Waiting Forever

ExecutorService
-------------
Thread Pool

ForkJoinPool
-------------
Divide And Conquer

CompletableFuture
-------------
Async Programming

Parallel Stream
-------------
Multi-Core Processing

Interview Favorites
-------------
Race Condition
Synchronization
Deadlock
ExecutorService
CompletableFuture
```

# End of Java Multithreading Notes