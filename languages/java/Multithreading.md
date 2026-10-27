# Java Multithreading: Basic to Advanced

### Visual study notes • Practical Java examples • Interview revision

> **Goal:** Understand Java concurrency from the ground up, learn why
> common problems happen, choose the right tool, and apply the concepts
> safely in backend applications.
>
> **Version baseline:** Most examples use modern Java syntax and APIs
> available in Java 17 or Java 21+. Virtual threads require Java 21+.
> Structured concurrency is version-sensitive and is covered separately.
> Check the JDK version used by your project before copying
> version-specific code.

------------------------------------------------------------------------

## How to use these notes

- **Learn in order** the first time; use the tables and checklists for
  revision.
- Run examples in a small Java project. Concurrency output order can
  vary between runs.
- Do not use a microbenchmark made with `System.currentTimeMillis()` to
  decide which concurrency primitive is faster. Use JMH for serious
  comparisons.
- A thread-safe Java object does not automatically make a multi-step
  business workflow or a database transaction safe.

## Table of contents

1.  [Concurrency fundamentals](#1-concurrency-fundamentals)
2.  [Creating threads and thread
    lifecycle](#2-creating-threads-and-thread-lifecycle)
3.  [Shared state and race
    conditions](#3-shared-state-and-race-conditions)
4.  [Intrinsic locks and
    synchronized](#4-intrinsic-locks-and-synchronized)
5.  [volatile and the Java Memory
    Model](#5-volatile-and-the-java-memory-model)
6.  [Atomic variables and CAS](#6-atomic-variables-and-cas)
7.  [wait, notify, sleep, join and
    interruption](#7-wait-notify-sleep-join-and-interruption)
8.  [Runnable, Callable, Future and
    ExecutorService](#8-runnable-callable-future-and-executorservice)
9.  [CompletableFuture](#9-completablefuture)
10. [Concurrent collections and
    producer-consumer](#10-concurrent-collections-and-producer-consumer)
11. [Locks, semaphores and
    coordination](#11-locks-semaphores-and-coordination)
12. [Deadlock, starvation and
    livelock](#12-deadlock-starvation-and-livelock)
13. [Fork/Join, work stealing and parallel
    streams](#13-forkjoin-work-stealing-and-parallel-streams)
14. [Virtual threads](#14-virtual-threads)
15. [ThreadLocal and context
    propagation](#15-threadlocal-and-context-propagation)
16. [Structured concurrency](#16-structured-concurrency)
17. [Advanced internals: ABA, LongAdder, false
    sharing](#17-advanced-internals-aba-longadder-false-sharing)
18. [Production patterns and system
    design](#18-production-patterns-and-system-design)
19. [Debugging and performance
    tools](#19-debugging-and-performance-tools)
20. [Common mistakes and best
    practices](#20-common-mistakes-and-best-practices)
21. [Interview revision questions](#21-interview-revision-questions)
22. [Quick revision cheat sheet](#22-quick-revision-cheat-sheet)

------------------------------------------------------------------------

# 1. Concurrency fundamentals

## 1.1 What is a process?

A **process** is an executing program with its own address space and
operating-system resources.

Example: running a Java application starts a JVM process.

## 1.2 What is a thread?

A **thread is an execution path within a process**. A Java process can
have many threads executing different tasks.

Threads in one process generally share heap objects, while each thread
has its own call stack and execution state.

### Visual model

``` text
Java application / JVM process
|
+-- Heap (objects shared when references are shared)
|   +-- Booking object
|   +-- Wallet object
|   +-- ConcurrentHashMap
|
+-- Platform thread A
|   +-- Its own call stack
|
+-- Platform thread B
|   +-- Its own call stack
|
+-- Virtual threads (Java 21+)
    +-- Many lightweight Java threads scheduled over carrier platform threads
```

## 1.3 Concurrency vs parallelism

- **Concurrency:** Multiple tasks make progress during overlapping
  periods. They may take turns on one CPU core.
- **Parallelism:** Multiple tasks literally execute at the same time,
  for example on separate CPU cores.

Concurrency is about managing overlapping work; parallelism is about
simultaneous execution.

## 1.4 Why use multiple threads?

Examples in a backend application:

- Process requests while other requests wait for I/O.
- Call independent services concurrently.
- Process background jobs.
- Consume queue messages.
- Run CPU-heavy computations in parallel when the work can be divided.

More threads do **not** automatically mean better performance. Threads
consume resources, compete for CPU, and may contend for shared data or
downstream services.

## 1.5 CPU-bound vs I/O-bound

| Workload  | Main characteristic                                      | Typical starting point                                                |
|-----------|----------------------------------------------------------|-----------------------------------------------------------------------|
| CPU-bound | Most time is spent computing                             | Bounded platform-thread pool or Fork/Join, sized around available CPU |
| I/O-bound | Much time is spent waiting on network, database or files | Virtual threads or an appropriately designed asynchronous I/O model   |
| Mixed     | Some computation and some waiting                        | Measure; isolate CPU-heavy work and control I/O concurrency           |

**Remember:** Virtual threads improve scalability for many waiting
tasks; they do not make CPU instructions faster or increase database
capacity.

------------------------------------------------------------------------

# 2. Creating threads and thread lifecycle

## 2.1 Extending `Thread`

``` java
class GreetingThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from " +
                Thread.currentThread().getName());
    }
}

public class Main {
    public static void main(String[] args) {
        Thread thread = new GreetingThread();
        thread.start();
    }
}
```

`start()` asks the JVM to start a new thread of execution. The JVM then
invokes that thread’s `run()` method.

## 2.2 Implementing `Runnable`

``` java
class BookingTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Processing booking");
    }
}

public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(new BookingTask());
        thread.start();
    }
}
```

Implementing `Runnable` separates the task from the thread that executes
it and leaves the class free to extend another class. For most simple
tasks, a lambda is concise:

``` java
Thread thread = new Thread(() ->
        System.out.println("Processing booking"));
thread.start();
```

## 2.3 `start()` vs `run()`

| Call             | Meaning                                                                           |
|------------------|-----------------------------------------------------------------------------------|
| `thread.start()` | Starts a separate thread of execution                                             |
| `thread.run()`   | Ordinary method call on the current thread; does not start a new thread by itself |

A `Thread` object can be started only once. Starting it a second time
throws `IllegalThreadStateException`.

## 2.4 Thread lifecycle

Java’s `Thread.State` has six states:

| State           | Meaning                                      |
|-----------------|----------------------------------------------|
| `NEW`           | Created but not started                      |
| `RUNNABLE`      | Ready to run or currently running            |
| `BLOCKED`       | Waiting to acquire an intrinsic monitor lock |
| `WAITING`       | Waiting indefinitely for another action      |
| `TIMED_WAITING` | Waiting for a specified duration             |
| `TERMINATED`    | Execution has completed                      |

The Java state `RUNNABLE` includes both ready-to-run and running. It is
not a direct mapping to every operating-system scheduling state.

``` java
Thread t = new Thread(() -> System.out.println("work"));
System.out.println(t.getState()); // NEW
t.start();
// State may be RUNNABLE or TERMINATED depending on timing.
```

## 2.5 Useful thread methods

- `Thread.currentThread()` — returns the current thread.
- `Thread.sleep(...)` — pauses the current thread for a duration; does
  not release monitors it holds.
- `thread.join()` — waits for a thread to finish.
- `thread.interrupt()` — requests interruption; it does not forcibly
  kill the thread.
- `thread.isAlive()` — checks whether a thread has started and not yet
  terminated.

Avoid using deprecated unsafe methods such as `Thread.stop()`.

------------------------------------------------------------------------

# 3. Shared state and race conditions

## 3.1 What is shared state?

Shared state is data that multiple threads can access, for example a
shared counter, wallet, cache or booking object.

## 3.2 What is a race condition?

A **race condition** occurs when the correctness of a result depends on
the timing or interleaving of concurrent actions.

Consider:

``` java
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}
```

`count++` is not one indivisible operation. Conceptually it reads the
value, calculates a new value, and writes it back. Two threads can read
the same old value and overwrite each other’s increments.

For example:

``` text
Initial count = 5

Thread A reads 5
Thread B reads 5
Thread A writes 6
Thread B writes 6

Expected after two increments: 7
Actual result:                 6
```

This is a lost update.

## 3.3 A shared wallet example

``` java
class Wallet {
    private int balance = 1_000;

    boolean withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }
}
```

Two concurrent withdrawals of ₹800 could both pass the check if they
read the same initial balance. The check and update must be protected as
one operation.

**Important:** In production, money should use an appropriate exact
representation such as integer minor units or `BigDecimal`, and
persistent wallet changes need database transactions and concurrency
controls. A Java lock alone cannot protect the same account across
multiple application instances.

------------------------------------------------------------------------

# 4. Intrinsic locks and `synchronized`

## 4.1 What does `synchronized` do?

A `synchronized` method or block uses an object’s **intrinsic monitor
lock**.

It provides two key guarantees when competing threads use the same
monitor:

1.  **Mutual exclusion:** At most one thread at a time owns that
    monitor.
2.  **Visibility and ordering:** An unlock happens-before a subsequent
    lock of the same monitor.

It is not accurate to say every contended `synchronized` call always
requires an operating-system context switch. JVMs can optimize
uncontended synchronization; the actual implementation depends on the
runtime and workload.

## 4.2 Thread-safe counter

``` java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

Both methods synchronize on the same object (`this`), so concurrent
calls coordinate through the same monitor.

If `getCount()` were unsynchronized and there were no other visibility
mechanism, the getter would not receive the same monitor-based
visibility guarantee.

## 4.3 Synchronized block

``` java
class Counter {
    private int count = 0;
    private final Object lock = new Object();

    public void increment() {
        synchronized (lock) {
            count++;
        }
    }
}
```

A block lets you protect only the critical section. All code that needs
the same protection must coordinate using the same lock object.

Do not synchronize on publicly accessible objects, string literals, or
objects that unrelated code may also lock.

## 4.4 Instance lock vs class lock

``` java
public synchronized void update() {
    // Locks this object
}

public static synchronized void updateGlobal() {
    // Locks MyClass.class
}
```

| Instance synchronized                                  | Static synchronized                                                     |
|--------------------------------------------------------|-------------------------------------------------------------------------|
| Locks `this`                                           | Locks the class’s `Class` object                                        |
| Different instances use different instance monitors    | All calls to that static synchronized method use the same class monitor |
| Coordinates shared state only if the same lock is used | Does not automatically coordinate with instance-synchronized methods    |

A static synchronized method and an instance synchronized method use
different locks.

## 4.5 When is `synchronized` a good choice?

Use it for straightforward critical sections, especially when a small
set of operations must update related in-memory state together.

Do not assume it makes every method in the class thread-safe. Every
access to the protected state must follow a consistent synchronization
strategy.

------------------------------------------------------------------------

# 5. `volatile` and the Java Memory Model

## 5.1 What is visibility?

Visibility means one thread can observe another thread’s writes under
the guarantees of the Java Memory Model (JMM).

The JMM defines the legal ordering and visibility of memory actions
between threads. CPU registers, caches, compiler optimization and
processor ordering help explain why synchronization matters, but the
JMM—not a simplified picture of “flushing everything to main
memory”—defines Java’s guarantees.

## 5.2 A stop flag

Unsafe version:

``` java
class Worker {
    private boolean running = true;

    void work() {
        while (running) {
            // Do work
        }
    }

    void stop() {
        running = false;
    }
}
```

If one thread runs the loop while another calls `stop()`, the
unsynchronized shared access is a data race. The loop is not guaranteed
to observe the update promptly or at all.

Correct visibility for a simple flag:

``` java
class Worker {
    private volatile boolean running = true;

    void work() {
        while (running) {
            // Do work
        }
    }

    void stop() {
        running = false;
    }
}
```

A volatile write happens-before a subsequent volatile read that observes
that write. This gives the flag the required visibility and ordering
behavior.

## 5.3 What `volatile` does and does not do

| Property                                   | `volatile` |
|--------------------------------------------|------------|
| Visibility of updates to that variable     | Yes        |
| Ordering guarantees around volatile access | Yes        |
| Makes `count++` atomic                     | No         |
| Protects a multi-variable invariant        | No         |
| Replaces every use of locks                | No         |

Example:

``` java
private volatile int count = 0;

void increment() {
    count++; // Still not atomic
}
```

`volatile` does not make the read-modify-write sequence atomic.

## 5.4 Happens-before

**Happens-before** is a formal relationship that guarantees visibility
and ordering.

Important examples:

- Within a thread, program order establishes happens-before between
  earlier and later actions.
- Unlocking a monitor happens-before a subsequent lock of the same
  monitor.
- A volatile write happens-before a subsequent read of that variable
  that observes the write.
- Calling `Thread.start()` happens-before actions in the started thread.
- Actions in a thread happen-before another thread successfully returns
  from `join()` on that completed thread.

The relationship is transitive: if A happens-before B, and B
happens-before C, then A happens-before C.

### Example: safe publication through a volatile flag

``` java
class MessageBox {
    private int message = 0;
    private volatile boolean ready = false;

    void writer() {
        message = 100;
        ready = true;
    }

    void reader() {
        if (ready) {
            System.out.println(message); // 100 if ready was observed true
        }
    }
}
```

If the reader observes `ready == true` through the volatile read, the
earlier write to `message` is visible, assuming no other writes
interfere.

## 5.5 `final` and safe publication

A correctly constructed immutable object with `final` fields has special
initialization-safety guarantees. But `final` does not make arbitrary
mutable state thread-safe, nor does it replace every safe-publication
requirement.

Useful safe-publication techniques include volatile references,
consistent lock-based publication, static initialization, and
thread-safe queues or collections.

------------------------------------------------------------------------

# 6. Atomic variables and CAS

## 6.1 `AtomicInteger`

``` java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private final AtomicInteger count = new AtomicInteger();

    void increment() {
        count.incrementAndGet();
    }

    int getCount() {
        return count.get();
    }
}
```

`incrementAndGet()` performs an atomic increment. Multiple threads can
call it without losing increments.

## 6.2 Compare-and-set (CAS)

CAS means compare-and-set or compare-and-swap. It atomically compares
the current value with an expected value and changes it only if they
match.

``` text
Expected value = 5
New value      = 6

Current value still 5?
  Yes → set to 6
  No  → fail; retry if appropriate
```

CAS is commonly implemented using hardware-supported atomic operations,
as exposed through JVM mechanisms. Some algorithms retry when a
competing update wins.

CAS avoids conventional lock acquisition for that atomic update, but it
is not automatically faster in every workload. Heavy contention can
cause repeated retries and CPU use.

## 6.3 Atomic types

| Type                        | Common use                                                             |
|-----------------------------|------------------------------------------------------------------------|
| `AtomicInteger`             | Atomic integer state                                                   |
| `AtomicLong`                | Atomic long state                                                      |
| `AtomicBoolean`             | Atomic boolean state                                                   |
| `AtomicReference<T>`        | Atomically update a reference                                          |
| `AtomicStampedReference<T>` | Reference plus integer stamp, useful for some ABA-sensitive algorithms |
| `LongAdder`                 | High-contention statistical counters                                   |

## 6.4 AtomicReference

``` java
import java.util.concurrent.atomic.AtomicReference;

record VehicleStatus(String location, int battery) {}

AtomicReference<VehicleStatus> status =
        new AtomicReference<>(new VehicleStatus("Vijayawada", 80));

status.set(new VehicleStatus("Guntur", 75));
VehicleStatus current = status.get();
```

The reference can be atomically replaced. `AtomicReference` does not
automatically make a mutable object stored in that reference
thread-safe.

## 6.5 `updateAndGet` and side effects

``` java
AtomicInteger count = new AtomicInteger(0);
count.updateAndGet(oldValue -> oldValue + 1);
```

Update functions can be retried under contention. Keep them free from
side effects. Do not send a payment, publish an event, or send a
notification inside a function that might be evaluated more than once.

## 6.6 The ABA problem

A thread reads A. Another thread changes A → B → A. The first thread’s
CAS sees A and succeeds, unaware of the intermediate change.

This matters when the history or structure behind a value matters, such
as certain lock-free stack algorithms.

`AtomicStampedReference` can pair a reference with a stamp/version.
Updates must consistently change the stamp, and the finite stamp can
eventually wrap around.

## 6.7 `AtomicLong` vs `LongAdder`

| Feature                            | `AtomicLong`                             | `LongAdder`                                  |
|------------------------------------|------------------------------------------|----------------------------------------------|
| Best use                           | Atomic state and conditional updates     | High-contention statistics                   |
| Update model                       | One atomic value                         | Distributes updates across cells when needed |
| Read                               | `get()` returns the current atomic value | `sum()` combines cells                       |
| Snapshot during concurrent updates | Atomic value read                        | `sum()` is not an atomic snapshot            |
| Memory                             | Usually lower                            | May use more memory                          |

Use `LongAdder` for metrics such as request counts when high update
throughput matters. Use `AtomicLong` when you need a single atomic value
for conditional logic.

## 6.8 `synchronized` vs `AtomicInteger`

| Feature                                    | `synchronized`                               | `AtomicInteger`                                |
|--------------------------------------------|----------------------------------------------|------------------------------------------------|
| Mechanism                                  | Intrinsic monitor                            | Atomic operations such as CAS                  |
| Protects multiple related actions together | Yes, if guarded by the same monitor          | Not automatically                              |
| Simple counter                             | Works                                        | Often convenient                               |
| Performance                                | Can be excellent; JVM optimizes common cases | Can be excellent; contention may cause retries |
| Always faster than the other?              | No                                           | No                                             |

**Rule of thumb:** Use `AtomicInteger` for simple atomic state
transitions; use `synchronized` or a lock when several operations or
variables must remain consistent together.

------------------------------------------------------------------------

# 7. `wait`, `notify`, `sleep`, `join` and interruption

## 7.1 `sleep()` vs `wait()`

| `Thread.sleep()`                             | `Object.wait()`                                   |
|----------------------------------------------|---------------------------------------------------|
| Pauses the current thread for a duration     | Waits for notification, interruption, or timeout  |
| Does not release monitors held by the thread | Releases the monitor on which `wait()` is called  |
| Static method on `Thread`                    | Method on `Object`                                |
| Does not require owning a monitor            | Must be called while owning that object’s monitor |

## 7.2 Correct wait/notify pattern

``` java
class Signal {
    private boolean ready = false;

    public synchronized void awaitReady()
            throws InterruptedException {
        while (!ready) {
            wait();
        }
    }

    public synchronized void signalReady() {
        ready = true;
        notifyAll();
    }
}
```

Call `wait()` inside a loop because a thread can wake spuriously, or the
condition may no longer hold when it reacquires the monitor. The
condition must be checked while holding the same monitor used by the
notifying thread.

`notify()` wakes one waiting thread; `notifyAll()` wakes all threads
waiting on that monitor. Woken threads still need to reacquire the
monitor before continuing.

For many producer-consumer problems, a `BlockingQueue` is simpler than
manually coordinating `wait()` and `notify()`.

## 7.3 `join()`

``` java
Thread worker = new Thread(() -> {
    System.out.println("Working");
});

worker.start();
worker.join();
System.out.println("Worker completed");
```

`join()` waits for the worker to terminate. It also provides a
happens-before relationship from the worker’s actions to the successful
return from `join()`.

## 7.4 Interruption

Interruption is a cooperative cancellation signal, not forced
termination.

``` java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    return;
}
```

If you cannot propagate `InterruptedException`, restoring the interrupt
flag is often appropriate when you are stopping or handing cancellation
to an outer layer.

Tasks should respond to interruption and avoid swallowing it silently.
Methods such as `BlockingQueue.take()`, `Thread.sleep()`, and many
blocking operations are interruptible.

------------------------------------------------------------------------

# 8. Runnable, Callable, Future and ExecutorService

## 8.1 Why use an executor?

Creating a new platform thread for every task can become costly. An
executor separates task submission from the details of execution.

## 8.2 Runnable vs Callable

| Feature            | `Runnable`                                | `Callable<V>`                              |
|--------------------|-------------------------------------------|--------------------------------------------|
| Result             | None                                      | Returns `V`                                |
| Method             | `run()`                                   | `call()`                                   |
| Checked exceptions | `run()` cannot declare checked exceptions | `call()` can declare checked exceptions    |
| Common use         | Logging or notifications                  | Calculations and tasks that return results |

## 8.3 ExecutorService

``` java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        try {
            for (int i = 1; i <= 6; i++) {
                int taskId = i;
                executor.submit(() ->
                        System.out.println("Task " + taskId));
            }
        } finally {
            executor.shutdown();
        }
    }
}
```

`shutdown()` stops accepting new tasks and allows submitted tasks to
finish. It does not mean every task has completed at the instant the
method returns.

## 8.4 Common executor factories

| Factory                     | Behavior                                | Caution                                        |
|-----------------------------|-----------------------------------------|------------------------------------------------|
| `newFixedThreadPool(n)`     | Fixed worker count                      | Uses an unbounded queue                        |
| `newSingleThreadExecutor()` | One worker processes tasks sequentially | Queue can grow without bound                   |
| `newCachedThreadPool()`     | Reuses idle workers and may create more | Thread count can grow substantially            |
| `newScheduledThreadPool(n)` | Delayed and periodic work               | Consider task duration and scheduling behavior |
| `newWorkStealingPool()`     | Work-stealing executor                  | Intended mainly for suitable parallel tasks    |

## 8.5 Bounded ThreadPoolExecutor

``` java
import java.util.concurrent.*;

ExecutorService executor = new ThreadPoolExecutor(
        4,                         // core workers
        8,                         // maximum workers
        60, TimeUnit.SECONDS,      // idle timeout
        new ArrayBlockingQueue<>(100),
        new ThreadPoolExecutor.AbortPolicy()
);
```

This example uses a bounded queue. When the executor cannot accept more
tasks, `AbortPolicy` rejects submissions with
`RejectedExecutionException`.

Choose worker counts, queue sizes, and rejection behavior based on
workload and capacity. A bounded queue is useful only when the
application has an explicit overload policy.

## 8.6 Future

``` java
ExecutorService executor = Executors.newFixedThreadPool(2);

try {
    Future<Integer> future = executor.submit(() -> 5000);
    Integer result = future.get();
    System.out.println(result);
} finally {
    executor.shutdown();
}
```

`Future.get()` waits for the result. `get(timeout, unit)` limits how
long the caller waits but does not automatically stop the underlying
task. `cancel(true)` requests cancellation and may interrupt a running
task; cancellation remains cooperative.

Useful methods: `get()`, timed `get()`, `isDone()`, `cancel(...)`,
`isCancelled()`.

## 8.7 Shutdown pattern

For controlled shutdown, an application may call `shutdown()`, wait with
`awaitTermination(...)`, and use `shutdownNow()` if the timeout expires.
`shutdownNow()` requests interruption of active tasks and returns tasks
that never commenced; it cannot guarantee that running tasks stop.

------------------------------------------------------------------------

# 9. CompletableFuture

## 9.1 What problem does it solve?

`CompletableFuture` composes asynchronous stages, transforms results,
combines independent tasks, and handles failures.

## 9.2 Basic example

``` java
import java.util.concurrent.CompletableFuture;

public class Main {
    public static void main(String[] args) {
        CompletableFuture<Integer> future =
                CompletableFuture.supplyAsync(() -> 5000)
                        .thenApply(price -> price + 900)
                        .thenApply(total -> total * 2);

        System.out.println(future.join()); // 11800
    }
}
```

`join()` waits for completion and reports exceptional completion through
an unchecked `CompletionException`.

## 9.3 Important methods

| Method            | Purpose                                       |
|-------------------|-----------------------------------------------|
| `supplyAsync()`   | Run a task that returns a result              |
| `runAsync()`      | Run a task with no result                     |
| `thenApply()`     | Transform a result                            |
| `thenAccept()`    | Consume a result                              |
| `thenCompose()`   | Chain an operation that returns another stage |
| `thenCombine()`   | Combine two independent results               |
| `allOf()`         | Complete after all supplied futures complete  |
| `exceptionally()` | Recover from a failure                        |
| `whenComplete()`  | Observe completion or failure                 |

By default, async methods without an explicit executor commonly use the
common Fork/Join pool. Choose an executor deliberately when workload
isolation or blocking behavior matters.

Non-async continuations may run on the thread that completes the
previous stage, or the thread attaching the continuation if the prior
stage is already complete. Use async variants when execution placement
matters.

## 9.4 `thenApply()` vs `thenCompose()`

- `thenApply`: transforms `T` into `U`.
- `thenCompose`: transforms `T` into a `CompletionStage<U>` and flattens
  the result instead of nesting futures.

Use `thenCompose()` for a flow such as “get booking ID, then
asynchronously fetch matching carriers.”

## 9.5 Parallel independent tasks

``` java
CompletableFuture<String> carrier =
        CompletableFuture.supplyAsync(() -> "Carrier details");

CompletableFuture<Integer> price =
        CompletableFuture.supplyAsync(() -> 5000);

CompletableFuture<String> combined =
        carrier.thenCombine(price,
                (c, p) -> c + ", price = " + p);

System.out.println(combined.join());
```

For production code, consider timeouts, cancellation, executor choice,
and what should happen if one operation fails.

------------------------------------------------------------------------

# 10. Concurrent collections and producer-consumer

## 10.1 ConcurrentHashMap

``` java
import java.util.concurrent.ConcurrentHashMap;

ConcurrentHashMap<String, String> locations =
        new ConcurrentHashMap<>();

locations.put("TRUCK-101", "Vijayawada");
locations.putIfAbsent("TRUCK-102", "Unknown");
String current = locations.get("TRUCK-101");
```

`ConcurrentHashMap` supports concurrent reads and updates. Methods such
as `putIfAbsent()` and `compute()` provide atomic operations for a key.

It does not make an arbitrary sequence of separate map operations
atomic, and it does not coordinate state across multiple application
instances.

## 10.2 Other concurrent collections

| Collection              | Use                                                 |
|-------------------------|-----------------------------------------------------|
| `ConcurrentHashMap`     | Concurrent key-value access                         |
| `CopyOnWriteArrayList`  | Read-heavy lists with rare changes                  |
| `BlockingQueue`         | Producer-consumer transfer with blocking operations |
| `ConcurrentLinkedQueue` | Non-blocking concurrent queue                       |
| `ConcurrentSkipListMap` | Concurrent sorted map                               |

## 10.3 Producer-consumer

``` java
import java.util.concurrent.*;

BlockingQueue<String> queue = new ArrayBlockingQueue<>(100);

ExecutorService executor = Executors.newFixedThreadPool(2);

executor.submit(() -> {
    try {
        queue.put("GPS update");
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

executor.submit(() -> {
    try {
        String update = queue.take();
        System.out.println("Processing " + update);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

executor.shutdown();
```

A bounded queue provides a limit on queued items. A complete production
design also needs shutdown coordination, failure handling, retry rules
and, where required, durable storage.

------------------------------------------------------------------------

# 11. Locks, semaphores and coordination

## 11.1 ReentrantLock

``` java
import java.util.concurrent.locks.ReentrantLock;

class Wallet {
    private int balance = 1000;
    private final ReentrantLock lock = new ReentrantLock();

    boolean withdraw(int amount) {
        lock.lock();
        try {
            if (balance >= amount) {
                balance -= amount;
                return true;
            }
            return false;
        } finally {
            lock.unlock();
        }
    }
}
```

Always release an acquired lock in `finally`. Otherwise, an exception
could leave the lock held.

`ReentrantLock` supports `tryLock()`, timed acquisition, interruptible
acquisition and optional fairness. Use it when those features help;
`synchronized` is often simpler.

## 11.2 ReadWriteLock

`ReentrantReadWriteLock` allows multiple readers to hold the read lock
concurrently, while the write lock is exclusive.

Useful when reads greatly outnumber writes and the work inside the lock
is substantial enough to justify the overhead.

## 11.3 StampedLock

`StampedLock` offers read, write and optimistic-read modes. An
optimistic read must be validated before its result is trusted. It is
not reentrant, so it requires careful design.

## 11.4 Semaphore

A semaphore limits concurrent access to a resource.

``` java
import java.util.concurrent.Semaphore;

class CarrierClient {
    private final Semaphore permits = new Semaphore(20);

    String call() throws InterruptedException {
        permits.acquire();
        try {
            // Call the external service
            return "OK";
        } finally {
            permits.release();
        }
    }
}
```

This limits concurrent calls within this semaphore’s scope. Twenty
permits per application instance do not create a global limit of twenty
across multiple instances. Use a distributed coordination mechanism or
downstream enforcement if a global cap is required.

## 11.5 CountDownLatch, CyclicBarrier and Phaser

- `CountDownLatch`: one or more threads wait until a countdown reaches
  zero; it cannot be reset.
- `CyclicBarrier`: a fixed group of parties waits until all reach a
  barrier; it can be reused.
- `Phaser`: flexible coordination for tasks arriving across multiple
  phases.

Choose the simplest coordination primitive that fits the workflow.

------------------------------------------------------------------------

# 12. Deadlock, starvation and livelock

## 12.1 Deadlock

Deadlock occurs when threads wait in a cycle for resources held by one
another.

``` text
Thread A holds lock A, waits for lock B
Thread B holds lock B, waits for lock A
              |
              v
         Deadlock possible
```

Prevention strategies:

- Acquire multiple locks in a consistent global order.
- Avoid unnecessary nested locks.
- Use timed or interruptible lock acquisition when appropriate.
- Inspect thread dumps during an incident.

## 12.2 Starvation

A thread is starved when it repeatedly fails to obtain needed CPU time
or resources. Other threads may continue making progress.

## 12.3 Livelock

Threads remain active and respond to each other, but the system makes no
useful progress. Retry loops that continually back off or yield to each
other can cause livelock.

## 12.4 Deadlock vs starvation vs livelock

| Problem    | What happens                                                      |
|------------|-------------------------------------------------------------------|
| Deadlock   | A dependency cycle prevents involved threads from progressing     |
| Starvation | One thread repeatedly fails to obtain resources or execution time |
| Livelock   | Threads keep reacting but make no useful progress                 |

------------------------------------------------------------------------

# 13. Fork/Join, work stealing and parallel streams

## 13.1 Fork/Join

The Fork/Join framework is designed for tasks that can be split into
smaller tasks and whose results can be combined.

- `RecursiveTask<V>` returns a result.
- `RecursiveAction` performs work without returning a result.
- `ForkJoinPool` schedules tasks and supports work stealing.

## 13.2 Divide and conquer

``` text
Large task
  |
  +-- Split into left task
  |      +-- Split again if needed
  |
  +-- Split into right task
         +-- Split again if needed

Small results
  |
  +-- Combine partial results
```

## 13.3 Example: sum an array

``` java
import java.util.concurrent.*;

class SumTask extends RecursiveTask<Long> {
    private final int[] numbers;
    private final int start;
    private final int end;
    private static final int THRESHOLD = 1000;

    SumTask(int[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;

        if (length <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += numbers[i];
            }
            return sum;
        }

        int middle = start + length / 2;
        SumTask left = new SumTask(numbers, start, middle);
        SumTask right = new SumTask(numbers, middle, end);

        left.fork();
        long rightResult = right.compute();
        long leftResult = left.join();

        return leftResult + rightResult;
    }
}
```

A sensible threshold avoids creating so many tiny tasks that scheduling
overhead dominates the computation.

## 13.4 Work stealing

An idle Fork/Join worker can steal suitable queued tasks from another
worker. This helps balance uneven workloads.

## 13.5 Parallel streams

``` java
long total = numbers.parallelStream()
        .mapToLong(Integer::longValue)
        .sum();
```

Parallel streams can help for large, independent, CPU-intensive
operations. They may be slower for small collections, cheap operations,
blocking calls, shared-state mutation or contended workloads.

Unsafe:

``` java
List<Integer> result = new ArrayList<>();
numbers.parallelStream().forEach(n -> result.add(n));
```

Multiple workers modify one `ArrayList` without coordination.

Safer:

``` java
List<Integer> result = numbers.parallelStream()
        .map(n -> n * 2)
        .toList();
```

------------------------------------------------------------------------

# 14. Virtual threads

## 14.1 What is a virtual thread?

Virtual threads became a permanent Java feature in **Java 21**. They are
lightweight Java threads scheduled by the JVM over platform threads,
known as carrier threads.

They are designed to make thread-per-task programming scalable,
especially for workloads with many blocking I/O operations.

## 14.2 Platform vs virtual threads

| Platform thread                                              | Virtual thread                                                        |
|--------------------------------------------------------------|-----------------------------------------------------------------------|
| Corresponds to an OS thread                                  | Many virtual threads can share carrier threads                        |
| Relatively expensive to create in large quantities           | Relatively inexpensive to create                                      |
| Useful for conventional execution and CPU-sized worker pools | Useful for high-concurrency blocking workflows                        |
| More platform-thread resources per concurrent thread         | Can suspend during supported blocking operations and free the carrier |

Virtual threads are not inherently faster for CPU-heavy calculations.

## 14.3 Create a virtual thread

``` java
Thread thread = Thread.ofVirtual().start(() -> {
    System.out.println(Thread.currentThread());
});

thread.join();
```

## 14.4 Virtual-thread-per-task executor

``` java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        try (ExecutorService executor =
                     Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 1000; i++) {
                int taskId = i;
                executor.submit(() ->
                        System.out.println("Task " + taskId));
            }
        }
    }
}
```

This executor assigns each task its own virtual thread. Closing the
executor initiates shutdown and waits for submitted tasks to complete.

Do not pool virtual threads just to reuse them. Instead, limit the
actual scarce resource—database connections, external API concurrency,
memory or queued work.

## 14.5 Virtual threads and pinning

In Java 21 and Java 22, some blocking while inside `synchronized` code
could pin a virtual thread to its carrier. **JEP 491 in Java 24**
changed monitor implementation to eliminate this form of pinning caused
by `synchronized` monitors.

Other operations, including some native or foreign-function
interactions, can still constrain carrier availability.

Do not mechanically replace `synchronized` with `ReentrantLock`; choose
based on the actual blocking behavior and runtime version.

## 14.6 Production cautions

- Configure HTTP and database timeouts.
- Keep database connection pool limits in mind.
- Bound outstanding work when input can exceed capacity.
- Handle interruption and cancellation.
- Avoid storing huge objects in per-thread state for very large numbers
  of virtual threads.
- Monitor downstream latency and errors.

------------------------------------------------------------------------

# 15. ThreadLocal and context propagation

## 15.1 What is ThreadLocal?

`ThreadLocal<T>` associates a separate value with each thread that
accesses it.

``` java
private static final ThreadLocal<String> requestId =
        new ThreadLocal<>();

static void handle(String id) {
    try {
        requestId.set(id);
        System.out.println(requestId.get());
    } finally {
        requestId.remove();
    }
}
```

Useful for some legacy logging or request-context designs, but it is not
a universal replacement for explicit parameters.

## 15.2 Why cleanup matters

Platform threads in a pool are reused. A thread-local value can remain
on the worker after one task finishes and be observed by a later task if
it is not reset or removed.

Use `try/finally` to clean up values in reusable thread pools.

## 15.3 ThreadLocal vs volatile vs synchronized

| Tool           | Main purpose                                      |
|----------------|---------------------------------------------------|
| `ThreadLocal`  | Per-thread value                                  |
| `volatile`     | Visibility and ordering for a shared variable     |
| `synchronized` | Mutual exclusion and visibility through a monitor |

ThreadLocal does not make other shared objects thread-safe.

## 15.4 ThreadLocal and virtual threads

Virtual threads support ThreadLocal. But if every virtual thread stores
large objects, total memory usage can become significant.

Modern Java also provides `ScopedValue` as a newer mechanism for certain
one-way context-sharing patterns. Its availability and status depend on
JDK version; check the target JDK before adopting it.

------------------------------------------------------------------------

# 16. Structured concurrency

## 16.1 What is it?

Structured concurrency treats related concurrent tasks as a unit with a
defined lifetime. A parent operation coordinates child tasks, their
completion, and cancellation.

Example: a booking-details endpoint needs carrier information, pricing,
and tracking status.

``` text
Booking request
  |
  +-- Fetch carrier
  +-- Calculate price
  +-- Fetch tracking
  |
  +-- Coordinate completion/failure
  +-- Combine results
```

This can make cancellation and error handling easier to reason about
than manually coordinating unrelated futures.

## 16.2 Version warning

Structured concurrency APIs have evolved across Java releases and have
been preview features in multiple releases. Do not assume one code
sample compiles unchanged on every JDK.

Check the official JDK documentation and the exact version used by the
application. Preview APIs require the corresponding preview compiler and
runtime flags.

## 16.3 When is it useful?

Use structured concurrency when one request launches multiple related
tasks and their lifetimes should be coordinated. `CompletableFuture`
remains useful for composing asynchronous stages and reusable
asynchronous pipelines; the two approaches address overlapping but
different needs.

------------------------------------------------------------------------

# 17. Advanced internals: ABA, LongAdder, false sharing

## 17.1 False sharing

False sharing occurs when independent variables modified by different
CPU cores happen to occupy the same cache line. Cache-coherence traffic
can hurt performance even though the variables are logically
independent.

Possible responses:

- Measure first.
- Avoid unnecessary shared writes.
- Consider `LongAdder` for highly contended statistics.
- Use padding or data-layout strategies only when evidence supports
  them.

False sharing is a performance problem, not necessarily a Java data
race.

## 17.2 Lock-free vs wait-free

| Term             | Progress guarantee                                                    |
|------------------|-----------------------------------------------------------------------|
| Lock-based       | A thread may wait for a lock owner                                    |
| Obstruction-free | An operation can complete if it eventually runs alone for long enough |
| Lock-free        | System-wide progress continues; individual threads may starve         |
| Wait-free        | Every operation completes within a bounded number of its own steps    |

Using CAS does not automatically make an entire algorithm lock-free or
wait-free. The full algorithm must satisfy the relevant progress
guarantee.

## 17.3 Benchmarking

Use [JMH](https://openjdk.org/projects/code-tools/jmh/) for JVM
microbenchmarks. Proper benchmarks account for warm-up, JIT compilation,
dead-code elimination, workload setup and measurement methodology.

Avoid assuming that `AtomicInteger` is always faster than
`synchronized`, or that virtual threads make every application faster.

------------------------------------------------------------------------

# 18. Production patterns and system design

## 18.1 Prevent duplicate booking assignment

A Java `synchronized` method protects only threads using the same
monitor in the same JVM. It cannot by itself coordinate multiple
application instances.

Use a conditional database update or equivalent transaction/constraint:

``` sql
UPDATE bookings
SET status = 'ASSIGNED',
    carrier_id = :carrierId
WHERE id = :bookingId
  AND status = 'AVAILABLE';
```

Check the affected row count:

- `1`: this request won the assignment.
- `0`: the booking was no longer available or the condition did not
  match.

Add the appropriate transaction boundaries and business-rule validation.

## 18.2 Limit external API concurrency

Use a semaphore or a bounded executor to limit concurrent calls. For a
multi-instance deployment, local semaphores create per-instance limits;
they do not automatically enforce a global cap.

## 18.3 Backpressure

When tasks arrive faster than the system can process them, an unbounded
queue can grow until memory and latency become unacceptable.

Use a bounded queue and define an overload policy: block producers,
reject requests, shed replaceable work, or persist important work for
later processing.

For payments and booking changes, use durable processing, idempotency
and retry rules rather than silently dropping work.

## 18.4 Production decision table

| Requirement                             | Starting point                                        |
|-----------------------------------------|-------------------------------------------------------|
| Shared counter                          | `AtomicLong`; `LongAdder` for high-contention metrics |
| Several related in-memory updates       | `synchronized` or `Lock`                              |
| Concurrent cache/map access             | `ConcurrentHashMap`                                   |
| Background tasks                        | `ExecutorService`                                     |
| Many blocking I/O tasks                 | Virtual threads plus resource limits                  |
| Large CPU divide-and-conquer task       | Fork/Join                                             |
| Limit simultaneous API calls            | Semaphore                                             |
| Producer-consumer flow                  | Bounded `BlockingQueue`                               |
| Duplicate persistent booking assignment | Database transaction/conditional update               |
| Thread contention diagnosis             | Thread dump and JFR                                   |

## 18.5 Reliability checklist

Before shipping a concurrent workflow, ask:

- [ ] Which state is shared?
- [ ] What makes each state transition atomic?
- [ ] How do readers observe updates?
- [ ] Can the operation execute twice?
- [ ] What happens if the thread is interrupted?
- [ ] Are all queues and resource pools appropriately bounded?
- [ ] What is the timeout and cancellation policy?
- [ ] Can multiple application instances race?
- [ ] Are retries idempotent?
- [ ] Can we observe queue depth, latency, rejection and errors?

------------------------------------------------------------------------

# 19. Debugging and performance tools

## 19.1 Thread dump

``` bash
jcmd <pid> Thread.print
```

Or:

``` bash
jstack <pid>
```

Replace `<pid>` with the Java process ID.

A thread dump helps inspect thread states, stack traces, monitor
ownership and some deadlock situations. A blocked thread alone does not
prove a deadlock.

## 19.2 Java Flight Recorder

JFR can help analyze thread activity, contention, latency, allocation
and other runtime events. Java Mission Control can help inspect
recordings.

## 19.3 Useful symptoms

| Symptom                           | Possible investigation                                                |
|-----------------------------------|-----------------------------------------------------------------------|
| High CPU                          | Hot loops, too much parallel work, contention, inefficient algorithms |
| Low CPU but requests stall        | Blocking I/O, deadlock, lock waits, downstream latency                |
| Queue grows continually           | Arrival rate exceeds processing capacity                              |
| High rejection rate               | Executor saturated or configured limits reached                       |
| Lost counter increments           | Race condition / non-atomic update                                    |
| Requests see another request’s ID | Shared mutable context or uncleared ThreadLocal                       |
| Duplicate booking assignment      | Missing persistent concurrency control or idempotency                 |

Use evidence from thread dumps, metrics and traces before changing
synchronization code.

------------------------------------------------------------------------

# 20. Common mistakes and best practices

1.  **Calling `run()` instead of `start()`** when you intend to start a
    new thread.
2.  **Assuming `volatile` makes `count++` atomic.**
3.  **Assuming a synchronized method protects different object
    instances.**
4.  **Assuming a Java lock protects a database row across multiple
    application instances.**
5.  **Forgetting to release a `ReentrantLock` in `finally`.**
6.  **Swallowing `InterruptedException` without restoring interruption
    or handling cancellation.**
7.  **Using unbounded queues without an overload policy.**
8.  **Blocking common-pool workers with long-running or blocking
    operations without considering the impact.**
9.  **Assuming virtual threads remove database connection limits.**
10. **Mutating shared collections from parallel-stream operations.**
11. **Performing external side effects inside retryable CAS update
    functions.**
12. **Optimizing based on assumptions rather than measurements.**

### Core principle

> First establish correctness. Then measure. Then optimize the actual
> bottleneck.

------------------------------------------------------------------------

# 21. Interview revision questions

## Fundamentals

**1. What is a thread?**  
An independent execution path inside a process.

**2. What is the difference between concurrency and parallelism?**  
Concurrency means tasks make progress during overlapping periods;
parallelism means tasks execute simultaneously.

**3. Why is `count++` not atomic?**  
It is a read-modify-write operation, so concurrent updates can be lost.

## Synchronization and memory

**4. What does `synchronized` guarantee?**  
Mutual exclusion and visibility/order guarantees through the same
monitor.

**5. What does `volatile` guarantee?**  
Visibility and ordering for volatile accesses; it does not make compound
operations atomic.

**6. What is happens-before?**  
A formal ordering relationship that guarantees visibility between
actions.

**7. Can a static synchronized method block an instance synchronized
method?**  
Not through their intrinsic locks: they lock different objects (`Class`
object vs instance).

## Atomic operations

**8. What is CAS?**  
An atomic compare-and-update operation that succeeds only if the current
value matches the expected value.

**9. What is ABA?**  
A value changes A → B → A, so a CAS that checks only the current value
may miss the intermediate change.

**10. When is LongAdder useful?**  
For high-contention statistics where throughput matters more than an
atomic snapshot of the combined sum.

## Executors and async work

**11. Runnable vs Callable?**  
Runnable does not return a result; Callable returns a result and can
throw checked exceptions.

**12. What does Future.get() do?**  
Waits for and retrieves the result, or reports failure/cancellation.

**13. thenApply vs thenCompose?**  
thenApply transforms a value; thenCompose chains a stage-producing
operation.

**14. Why use a bounded queue?**  
To limit pending work and provide a basis for an overload policy.

## Advanced concurrency

**15. What is deadlock?**  
A circular resource dependency prevents involved threads from
progressing.

**16. What is work stealing?**  
An idle worker takes queued work from another worker to help balance
execution.

**17. When are virtual threads useful?**  
For large numbers of concurrent tasks that spend substantial time in
supported blocking operations.

**18. Do virtual threads make CPU-bound code faster?**  
No, not inherently.

**19. What is ThreadLocal?**  
A mechanism that associates a separate value with each accessing thread.

**20. Why is a local Java lock insufficient for multi-instance booking
assignment?**  
Each JVM has its own lock; persistent state needs database-level
concurrency control.

**21. Lock-free vs wait-free?**  
Lock-free guarantees system-wide progress; wait-free guarantees bounded
completion for each operation.

**22. How do you investigate a suspected deadlock?**  
Collect thread dumps using `jcmd` or `jstack`, inspect lock ownership
and waiting relationships, and correlate with application metrics.

------------------------------------------------------------------------

# 22. Quick revision cheat sheet

``` text
start()                 → start a new thread
run()                   → ordinary method call if invoked directly
synchronized            → monitor-based mutual exclusion + visibility
volatile                → visibility/order, not compound atomicity
AtomicInteger           → atomic integer updates
AtomicReference         → atomic reference updates
LongAdder               → high-contention statistical counters
wait()                  → wait and release the object's monitor
sleep()                 → pause; does not release held monitors
join()                  → wait for thread completion
Runnable                → task with no result
Callable<V>             → task that returns V
Future<V>               → retrieve or cancel an asynchronous result
ExecutorService         → task execution lifecycle and worker management
CompletableFuture       → compose asynchronous stages
ConcurrentHashMap       → concurrent key-value operations
BlockingQueue           → producer-consumer coordination
ReentrantLock           → explicit lock, timed/interruptible options
Semaphore               → limit concurrent permit holders
ForkJoinPool            → divide-and-conquer and work stealing
Virtual threads         → lightweight threads for high-concurrency workloads
ThreadLocal             → per-thread associated values
Structured concurrency  → coordinate related task lifetimes
jcmd / jstack           → inspect thread dumps
JFR                     → analyze runtime behavior and contention
```

## Final learning path

For strong practical mastery, study in this order:

1.  Understand race conditions and `synchronized`.
2.  Master `volatile`, happens-before and the Java Memory Model.
3.  Practice `AtomicInteger`, `AtomicReference` and CAS.
4.  Learn executor sizing, queue capacity, rejection and shutdown.
5.  Build workflows with `CompletableFuture`.
6.  Practice concurrent collections and producer-consumer.
7.  Learn deadlock diagnosis and interruption.
8.  Compare Fork/Join, parallel streams and virtual threads.
9.  Understand resource limits, database concurrency and idempotency.
10. Benchmark and troubleshoot using JMH, thread dumps and JFR.

**The most important takeaway:** Multithreading is not just about
executing tasks simultaneously. It is about ensuring correctness,
controlling resource usage, coordinating failures, and preserving
business invariants while tasks execute concurrently.
