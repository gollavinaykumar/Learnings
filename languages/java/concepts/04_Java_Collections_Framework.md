# Java Collections Framework - Complete Interview Revision Notes

> One of the most important topics for Java interviews.
>
> Collections Framework provides ready-made data structures to store and manipulate data efficiently.
>
> Topics Covered:
>
> - List
> - Set
> - Map
> - Queue
> - ArrayList
> - LinkedList
> - HashSet
> - TreeSet
> - HashMap
> - ConcurrentHashMap
> - Hashing
> - equals() & hashCode()
> - Internal Resizing
> - Time Complexity

---

# What is Collections Framework?

Collection Framework is a set of classes and interfaces used to store and manipulate groups of objects.

Without Collections:

```java
int a = 10;
int b = 20;
int c = 30;
```

With Collections:

```java
List<Integer> numbers =
        new ArrayList<>();
```

Benefits:

```text
Dynamic Size
Ready-Made Data Structures
Easy Searching
Easy Sorting
Better Performance
```

---

# Collection Hierarchy

```text
                Iterable
                    |
               Collection
            /      |      \
         List     Set    Queue

Map (Separate Interface)
```

---

# Core Interfaces

## 1. List

### Definition

List stores:

```text
Ordered Data
Duplicates Allowed
Index Based
```

Example:

```java
List<String> names =
        new ArrayList<>();

names.add("Rahul");
names.add("Amit");
names.add("Rahul");

System.out.println(names);
```

Output:

```text
[Rahul, Amit, Rahul]
```

Duplicates allowed.

---

# 2. Set

### Definition

Set stores:

```text
Unique Elements
No Duplicates
```

Example:

```java
Set<String> names =
        new HashSet<>();

names.add("Rahul");
names.add("Amit");
names.add("Rahul");

System.out.println(names);
```

Output:

```text
[Amit, Rahul]
```

Duplicate removed.

---

# 3. Map

### Definition

Stores data as:

```text
Key → Value
```

Keys must be unique.

---

Example:

```java
Map<Integer,String> map =
        new HashMap<>();

map.put(1,"Rahul");
map.put(2,"Amit");

System.out.println(map);
```

Output:

```text
{1=Rahul, 2=Amit}
```

---

# 4. Queue

### Definition

Queue follows:

```text
FIFO
First In First Out
```

Example:

```java
Queue<Integer> q =
        new LinkedList<>();

q.add(10);
q.add(20);
q.add(30);

System.out.println(q.poll());
```

Output:

```text
10
```

First element removed.

---

# ArrayList

## Definition

ArrayList uses a dynamic array internally.

Features:

```text
Ordered
Duplicates Allowed
Index Based
Fast Retrieval
```

---

# Example

```java
ArrayList<String> list =
        new ArrayList<>();

list.add("Java");
list.add("Spring");
list.add("SQL");

System.out.println(list);
```

Output:

```text
[Java, Spring, SQL]
```

---

# Access By Index

```java
System.out.println(list.get(1));
```

Output:

```text
Spring
```

---

# Internal Working

ArrayList internally uses:

```java
Object[]
```

Example:

```text
[10][20][30][40]
```

Stored in continuous memory.

---

# Advantages

```text
Fast Random Access
Easy Traversal
```

---

# Disadvantages

```text
Slow Insertions
Slow Deletions
```

because shifting is required.

---

# Time Complexity

| Operation | Complexity |
|------------|------------|
| get() | O(1) |
| add() end | O(1) |
| insert middle | O(n) |
| delete middle | O(n) |

---

# LinkedList

## Definition

LinkedList uses nodes.

Each node contains:

```text
Data
Pointer To Next Node
```

---

# Structure

```text
[10] -> [20] -> [30] -> NULL
```

---

# Example

```java
LinkedList<String> list =
        new LinkedList<>();

list.add("Java");
list.add("Spring");
list.add("SQL");

System.out.println(list);
```

---

# Advantages

```text
Fast Insertions
Fast Deletions
```

---

# Disadvantages

```text
Slow Searching
Slow Random Access
```

---

# Time Complexity

| Operation | Complexity |
|------------|------------|
| get(index) | O(n) |
| add() | O(1) |
| delete() | O(1) |
| search() | O(n) |

---

# ArrayList vs LinkedList

| Feature | ArrayList | LinkedList |
|----------|----------|----------|
| Internal Structure | Dynamic Array | Doubly Linked List |
| Access | Fast | Slow |
| Insert | Slow | Fast |
| Delete | Slow | Fast |
| Memory | Less | More |

---

# HashSet

## Definition

HashSet stores:

```text
Unique Elements
Unordered Data
```

Uses:

```text
HashMap Internally
```

---

# Example

```java
HashSet<Integer> set =
        new HashSet<>();

set.add(10);
set.add(20);
set.add(10);

System.out.println(set);
```

Output:

```text
[10, 20]
```

---

# Features

```text
No Duplicates
Fast Search
No Ordering
```

---

# Time Complexity

| Operation | Complexity |
|------------|------------|
| add() | O(1) |
| remove() | O(1) |
| contains() | O(1) |

---

# TreeSet

## Definition

TreeSet stores:

```text
Unique Elements
Sorted Order
```

Internally uses:

```text
Red Black Tree
```

---

# Example

```java
TreeSet<Integer> set =
        new TreeSet<>();

set.add(50);
set.add(10);
set.add(30);

System.out.println(set);
```

Output:

```text
[10, 30, 50]
```

Automatically sorted.

---

# Time Complexity

| Operation | Complexity |
|------------|------------|
| add() | O(log n) |
| remove() | O(log n) |
| search() | O(log n) |

---

# HashMap

Most Important Interview Topic.

---

# Definition

Stores:

```text
Key → Value
```

Features:

```text
Unique Keys
Duplicate Values Allowed
Unordered
```

---

# Example

```java
HashMap<Integer,String> map =
        new HashMap<>();

map.put(101,"Rahul");
map.put(102,"Amit");

System.out.println(map);
```

Output:

```text
{101=Rahul, 102=Amit}
```

---

# Internal Structure

Java 8+

```text
Bucket Array
     ↓
Linked List
     ↓
Red Black Tree
```

---

# How HashMap Works

Step 1

```java
hashCode()
```

calculates bucket.

---

Step 2

Object placed into bucket.

---

Step 3

Collision handled using:

```text
Linked List
```

or

```text
Red Black Tree
```

(Java 8+)

---

# HashMap Example

```java
map.put(101,"Rahul");
```

HashMap internally:

```text
hashCode()
      ↓
Bucket Index
      ↓
Store Entry
```

---

# Time Complexity

| Operation | Complexity |
|------------|------------|
| put() | O(1) |
| get() | O(1) |
| remove() | O(1) |

Worst Case:

```text
O(n)
```

---

# ConcurrentHashMap

## Definition

Thread-safe version of HashMap.

Used in:

```text
Multithreaded Applications
```

---

# Example

```java
ConcurrentHashMap<Integer,String> map =
        new ConcurrentHashMap<>();

map.put(1,"Java");
map.put(2,"Spring");
```

---

# Why Not HashMap?

HashMap is:

```text
Not Thread Safe
```

Multiple threads may corrupt data.

---

# ConcurrentHashMap Benefits

```text
Thread Safe
High Performance
Concurrent Access
```

---

# Hashing

Very Important Interview Topic.

---

# Definition

Hashing converts data into a fixed integer value.

Example:

```java
String name = "Java";
```

Hash Value:

```java
name.hashCode();
```

Output:

```text
2301506
```

---

# Why Hashing?

Used for:

```text
Fast Searching
Fast Insertion
Fast Lookup
```

HashMap and HashSet rely heavily on hashing.

---

# equals() and hashCode()

Most Asked Interview Question.

---

# equals()

Used to compare:

```text
Object Content
```

Example:

```java
String a = new String("Java");

String b = new String("Java");

System.out.println(a.equals(b));
```

Output:

```text
true
```

---

# hashCode()

Returns integer hash value.

Example:

```java
System.out.println(a.hashCode());
```

---

# Why Override Both?

Custom Class:

```java
class Employee {

    int id;

    Employee(int id) {

        this.id=id;
    }
}
```

Without override:

```java
Employee e1 =
        new Employee(1);

Employee e2 =
        new Employee(1);
```

HashSet treats them as different.

---

# Correct Implementation

```java
class Employee {

    int id;

    Employee(int id) {

        this.id=id;
    }

    @Override
    public boolean equals(Object obj) {

        Employee e=(Employee)obj;

        return this.id==e.id;
    }

    @Override
    public int hashCode() {

        return id;
    }
}
```

---

# Contract Rule

If:

```java
a.equals(b) == true
```

then

```java
a.hashCode() == b.hashCode()
```

must also be true.

---

# Internal Resizing

Important HashMap Interview Topic.

---

# Default Capacity

```java
16
```

---

# Load Factor

```java
0.75
```

---

# Resize Condition

```text
capacity × loadFactor
```

Example:

```text
16 × 0.75 = 12
```

After 12 elements:

```text
Resize Happens
```

---

# New Capacity

```text
16 → 32
32 → 64
64 → 128
```

Doubles each time.

---

# Why Resizing?

To reduce:

```text
Collisions
```

and improve performance.

---

# Time Complexity Summary

| Collection | Search | Insert | Delete |
|------------|---------|---------|---------|
| ArrayList | O(1) | O(1) | O(n) |
| LinkedList | O(n) | O(1) | O(1) |
| HashSet | O(1) | O(1) | O(1) |
| TreeSet | O(log n) | O(log n) | O(log n) |
| HashMap | O(1) | O(1) | O(1) |
| ConcurrentHashMap | O(1) | O(1) | O(1) |

---

# Frequently Asked Interview Questions

## Q1. Difference Between List and Set?

List:

```text
Duplicates Allowed
Ordered
```

Set:

```text
Duplicates Not Allowed
```

---

## Q2. Difference Between ArrayList and LinkedList?

ArrayList:

```text
Fast Read
Slow Insert/Delete
```

LinkedList:

```text
Slow Read
Fast Insert/Delete
```

---

## Q3. How HashMap Works?

```text
hashCode()
     ↓
Bucket
     ↓
Store Entry
     ↓
Collision Handling
```

---

## Q4. Why Override equals() and hashCode() Together?

Because collections use both methods for comparison and storage.

---

## Q5. Default Capacity of HashMap?

```text
16
```

---

## Q6. Default Load Factor?

```text
0.75
```

---

## Q7. Difference Between HashSet and TreeSet?

HashSet:

```text
Unordered
O(1)
```

TreeSet:

```text
Sorted
O(log n)
```

---

# One-Minute Revision

```text
List
-------------
Ordered
Duplicates Allowed

Set
-------------
Unique Elements

Map
-------------
Key Value Pair

Queue
-------------
FIFO

ArrayList
-------------
Dynamic Array
Fast Read

LinkedList
-------------
Node Based
Fast Insert/Delete

HashSet
-------------
Unique
O(1)

TreeSet
-------------
Sorted
O(log n)

HashMap
-------------
Key Value
Uses Hashing

ConcurrentHashMap
-------------
Thread Safe HashMap

Hashing
-------------
Fast Lookup

equals()
-------------
Compare Content

hashCode()
-------------
Generate Hash Value

HashMap Resize
-------------
Default Capacity = 16
Load Factor = 0.75

Time Complexity
-------------
HashMap = O(1)
HashSet = O(1)
TreeSet = O(log n)
ArrayList get = O(1)
LinkedList search = O(n)
```

# End of Collections Framework Notes