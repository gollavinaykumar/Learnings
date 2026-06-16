# Java Collections + Streams Performance - Complete Interview Revision Notes

> ⭐ VERY IMPORTANT FOR JAVA 8+ INTERVIEWS
>
> Most developers know Streams API,
> but interviews often focus on:
>
> - Lazy Evaluation
> - Intermediate Operations
> - Terminal Operations
> - Stream Performance
> - Parallel Streams
> - Risks of Parallel Processing
>
> Topics Covered:
>
> - Stream Execution Flow
> - Lazy Evaluation
> - Intermediate Operations
> - Terminal Operations
> - Stream Optimization
> - Parallel Streams
> - Parallel Stream Risks
> - Real World Examples
> - Performance Best Practices

---

# Why Streams Were Introduced?

Traditional Collection Processing:

```java
List<Integer> nums =
        Arrays.asList(1,2,3,4,5);

for(Integer n : nums){

    if(n > 2){

        System.out.println(n);
    }
}
```

Java 8 Streams:

```java
nums.stream()
    .filter(n -> n > 2)
    .forEach(System.out::println);
```

Benefits:

```text
Cleaner Code
Less Boilerplate
Functional Style
Parallel Processing Support
```

---

# Stream Processing Pipeline

Every Stream follows:

```text
Source
   ↓
Intermediate Operations
   ↓
Terminal Operation
```

Example:

```java
nums.stream()
    .filter(n -> n > 10)
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

---

# Pipeline Breakdown

Source:

```java
nums
```

Intermediate Operations:

```java
filter()
map()
```

Terminal Operation:

```java
collect()
```

---

# Lazy Evaluation

Most Asked Stream Interview Question.

---

# What is Lazy Evaluation?

Streams do NOT execute intermediate operations immediately.

Execution starts only when a terminal operation is encountered.

---

# Example

```java
List<Integer> nums =
        Arrays.asList(1,2,3,4,5);

nums.stream()
    .filter(n -> {

        System.out.println(
                "Filtering " + n);

        return n > 2;
    });
```

Output:

```text
No Output
```

Why?

Because:

```text
No Terminal Operation
```

Stream never executes.

---

# Add Terminal Operation

```java
nums.stream()
    .filter(n -> {

        System.out.println(
                "Filtering " + n);

        return n > 2;
    })
    .collect(Collectors.toList());
```

Output:

```text
Filtering 1
Filtering 2
Filtering 3
Filtering 4
Filtering 5
```

Now stream executes.

---

# Why Lazy Evaluation?

Improves performance.

Avoids unnecessary work.

---

# Example

Without lazy evaluation:

```text
Filter All Data
Map All Data
Sort All Data
```

Even if result not required.

---

With lazy evaluation:

```text
Execute Only When Needed
```

Less CPU usage.

---

# Stream Optimization Example

Suppose:

```java
List<Integer> nums =
        Arrays.asList(
                1,2,3,4,5,6,7,8,9,10);
```

Find first even number.

---

Code:

```java
nums.stream()
    .filter(n -> {

        System.out.println(
                "Checking " + n);

        return n % 2 == 0;
    })
    .findFirst();
```

Output:

```text
Checking 1
Checking 2
```

Stops immediately.

Does NOT check:

```text
3
4
5
...
10
```

---

# Why?

Because of lazy evaluation.

Stream processes only what is necessary.

---

# Intermediate Operations

## Definition

Operations that transform streams.

Characteristics:

```text
Return Stream
Lazy
Can Be Chained
```

---

# Common Intermediate Operations

```java
filter()
map()
sorted()
distinct()
limit()
skip()
peek()
```

---

# filter()

Used for filtering data.

Example:

```java
List<Integer> result =
        nums.stream()
            .filter(n -> n > 5)
            .collect(Collectors.toList());
```

Output:

```text
[6,7,8,9,10]
```

---

# map()

Used for transformation.

Example:

```java
List<Integer> result =
        nums.stream()
            .map(n -> n * 2)
            .collect(Collectors.toList());
```

Output:

```text
[2,4,6,8,10]
```

---

# sorted()

Example:

```java
nums.stream()
    .sorted()
    .forEach(System.out::println);
```

---

# distinct()

Removes duplicates.

Example:

```java
Arrays.asList(
        1,1,2,2,3)
        .stream()
        .distinct()
        .forEach(System.out::println);
```

Output:

```text
1
2
3
```

---

# limit()

Example:

```java
nums.stream()
    .limit(3)
    .forEach(System.out::println);
```

Output:

```text
1
2
3
```

---

# skip()

Example:

```java
nums.stream()
    .skip(3)
    .forEach(System.out::println);
```

Output:

```text
4
5
6
...
```

---

# peek()

Used for debugging.

Example:

```java
nums.stream()
    .peek(System.out::println)
    .collect(Collectors.toList());
```

---

# Terminal Operations

## Definition

Operations that trigger execution.

Characteristics:

```text
End Stream Pipeline
Return Result
Execute Stream
```

---

# Common Terminal Operations

```java
collect()
forEach()
count()
findFirst()
findAny()
reduce()
anyMatch()
allMatch()
```

---

# collect()

Most common terminal operation.

Example:

```java
List<Integer> result =
        nums.stream()
            .filter(n -> n > 5)
            .collect(Collectors.toList());
```

---

# forEach()

Example:

```java
nums.stream()
    .forEach(System.out::println);
```

---

# count()

Example:

```java
long total =
        nums.stream()
            .count();
```

---

# findFirst()

Example:

```java
Optional<Integer> first =
        nums.stream()
            .findFirst();
```

---

# reduce()

Used for aggregation.

Example:

```java
int sum =
        nums.stream()
            .reduce(
                0,
                Integer::sum);
```

Output:

```text
15
```

---

# Intermediate vs Terminal Operations

| Feature | Intermediate | Terminal |
|----------|----------|----------|
| Returns | Stream | Value/Result |
| Lazy | Yes | No |
| Can Chain | Yes | No |
| Executes Stream | No | Yes |

---

# Example

```java
nums.stream()
    .filter(n -> n > 5)
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

Intermediate:

```java
filter()
map()
```

Terminal:

```java
collect()
```

---

# Parallel Streams

Very Important Interview Topic.

---

# What is Parallel Stream?

Parallel Stream splits data across multiple CPU cores.

---

# Normal Stream

```java
nums.stream()
```

Processing:

```text
Single Thread
```

---

# Parallel Stream

```java
nums.parallelStream()
```

Processing:

```text
Multiple Threads
```

---

# Example

```java
List<Integer> nums =
        Arrays.asList(
                1,2,3,4,5,6,7,8);

nums.parallelStream()
    .forEach(System.out::println);
```

Output Order:

```text
May Be Random
```

Example:

```text
5
2
8
1
3
```

---

# Why?

Because multiple threads process data simultaneously.

---

# Real World Example

Process:

```text
10 Million Records
```

Sequential:

```text
One CPU Core
```

Parallel:

```text
All CPU Cores
```

Much faster.

---

# Performance Example

Sequential:

```java
long count =
        nums.stream()
            .filter(x -> x > 100)
            .count();
```

---

Parallel:

```java
long count =
        nums.parallelStream()
            .filter(x -> x > 100)
            .count();
```

Large datasets may be significantly faster.

---

# Parallel Streams Risks

Most Asked Senior-Level Interview Topic.

---

# Risk 1: Unpredictable Ordering

Example:

```java
nums.parallelStream()
    .forEach(System.out::println);
```

Output order:

```text
Random
```

---

# Solution

```java
forEachOrdered()
```

Example:

```java
nums.parallelStream()
    .forEachOrdered(
            System.out::println);
```

---

# Risk 2: Shared Mutable State

Bad Example:

```java
List<Integer> result =
        new ArrayList<>();
```

```java
nums.parallelStream()
    .forEach(result::add);
```

Problem:

```text
Multiple Threads
Modify Same List
```

May cause:

```text
Data Corruption
Missing Values
```

---

# Correct Solution

```java
List<Integer> result =
        nums.parallelStream()
            .collect(
                Collectors.toList());
```

Thread-safe approach.

---

# Risk 3: Small Collections

Example:

```java
Arrays.asList(1,2,3,4)
```

Using:

```java
parallelStream()
```

may be slower.

---

# Why?

Parallel processing has overhead:

```text
Thread Creation
Task Splitting
Task Merging
```

---

# Rule

Small Data:

```java
stream()
```

Large Data:

```java
parallelStream()
```

---

# Risk 4: Database Calls

Bad Example

```java
users.parallelStream()
     .forEach(
         user ->
         repository.findById(
             user.getId()));
```

---

Problem

```text
Database Connection Exhaustion
```

Too many parallel DB calls.

---

# Risk 5: Blocking Operations

Bad Example

```java
parallelStream()
```

with:

```text
HTTP Calls
File Reading
Database Queries
```

---

Why?

Parallel streams are designed mainly for:

```text
CPU Intensive Tasks
```

Not:

```text
I/O Intensive Tasks
```

---

# Good Use Cases For Parallel Streams

✅

```text
Large Data Processing
Mathematical Computation
Image Processing
Report Generation
Aggregation
```

---

# Avoid Parallel Streams For

❌

```text
Small Collections
Database Calls
HTTP Calls
Shared Mutable Objects
Ordered Processing
```

---

# Stream Performance Best Practices

## Use Streams For Readability

Good:

```java
employees.stream()
         .filter(
             e -> e.getSalary() > 50000)
         .collect(Collectors.toList());
```

---

## Avoid Complex Nested Streams

Bad:

```java
stream()
.stream()
.stream()
```

Hard to maintain.

---

## Prefer Primitive Streams

Instead of:

```java
Stream<Integer>
```

Use:

```java
IntStream
```

Example:

```java
IntStream.range(1,100)
         .sum();
```

Better performance.

---

## Use Parallel Streams Carefully

Only for:

```text
Large CPU Bound Tasks
```

---

# Frequently Asked Interview Questions

## Q1. What is Lazy Evaluation?

```text
Intermediate operations execute
only when terminal operation is called.
```

---

## Q2. Difference Between Intermediate and Terminal Operations?

Intermediate:

```text
Return Stream
Lazy
```

Terminal:

```text
Return Result
Trigger Execution
```

---

## Q3. Name Common Intermediate Operations.

```text
filter()
map()
sorted()
distinct()
limit()
skip()
peek()
```

---

## Q4. Name Common Terminal Operations.

```text
collect()
count()
forEach()
reduce()
findFirst()
```

---

## Q5. What is Parallel Stream?

```text
Stream Processing Using
Multiple CPU Cores
```

---

## Q6. Is Parallel Stream Always Faster?

```text
No
```

For small collections it may be slower.

---

## Q7. Why Is Ordering Lost In Parallel Streams?

```text
Multiple Threads Process
Data Independently
```

---

## Q8. When Should We Use Parallel Streams?

```text
Large Collections
CPU Intensive Work
```

---

## Q9. Why Avoid Parallel Streams With Database Calls?

```text
Can Exhaust DB Connections
And Reduce Performance
```

---

# One-Minute Revision

```text
Stream Pipeline
----------------
Source
↓
Intermediate
↓
Terminal

Lazy Evaluation
----------------
No Terminal Operation
→ No Execution

Intermediate Operations
----------------
filter()
map()
sorted()
distinct()
limit()
skip()
peek()

Terminal Operations
----------------
collect()
count()
reduce()
forEach()
findFirst()

Parallel Stream
----------------
Multiple CPU Cores

Good For
----------------
Large Data
CPU Intensive Tasks

Avoid For
----------------
Small Data
DB Calls
HTTP Calls
Shared Mutable State

Interview Favorites
----------------
Lazy Evaluation
Intermediate vs Terminal
Parallel Stream Risks
forEach vs forEachOrdered
```

# End of Collections + Streams Performance Notes