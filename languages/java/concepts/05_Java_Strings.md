# Java Strings - Complete Interview Revision Notes

> Strings are one of the most frequently asked topics in Java interviews.
>
> Topics Covered:
>
> - What is String?
> - String Pool
> - String Immutability
> - StringBuilder
> - StringBuffer
> - StringBuilder vs StringBuffer
> - String Comparison
> - Frequently Asked Interview Questions

---

# What is a String?

A String is a sequence of characters.

Example:

```java
String name = "Java";
```

Here:

```text
J
a
v
a
```

is a sequence of characters stored as a String object.

---

# Ways to Create String

## Method 1: String Literal

```java
String s1 = "Java";
```

Most common way.

---

## Method 2: Using new Keyword

```java
String s2 = new String("Java");
```

Creates a new object in Heap Memory.

---

# String Memory Diagram

```java
String s1 = "Java";

String s2 = "Java";
```

Memory:

```text
String Pool
-----------------
"Java"
```

Both references point to same object.

```text
s1 ----\
         > "Java"
s2 ----/
```

---

# String Pool

Very Important Interview Topic.

---

# What is String Pool?

String Pool is a special memory area inside Heap Memory where JVM stores String literals.

Purpose:

```text
Save Memory
Improve Performance
Avoid Duplicate Objects
```

---

# Example 1

```java
String s1 = "Java";

String s2 = "Java";
```

Memory:

```text
String Pool
-------------
"Java"
```

Only one object created.

Both references point to same object.

---

# Comparison

```java
System.out.println(s1 == s2);
```

Output:

```text
true
```

Because both references point to same object.

---

# Example 2

```java
String s1 = new String("Java");

String s2 = new String("Java");
```

Memory:

```text
Heap
--------
Object1

Object2
```

Two different objects created.

---

Comparison:

```java
System.out.println(s1 == s2);
```

Output:

```text
false
```

Different memory locations.

---

# intern() Method

Used to move String into String Pool.

Example:

```java
String s1 = new String("Java");

String s2 = s1.intern();
```

Now:

```java
s2
```

points to pooled object.

---

# String Pool Diagram

```text
String s1 = "Java"
String s2 = "Java"

        String Pool
        ----------
           Java
         /      \
       s1        s2
```

---

# Why String Pool?

Without Pool:

```java
String s1="Java";
String s2="Java";
String s3="Java";
String s4="Java";
```

Four objects created.

More memory used.

---

With Pool:

```text
Only One Object
```

Memory optimized.

---

# String Immutability

Most Asked Interview Question.

---

# What is Immutability?

Immutable means:

```text
Cannot Be Modified
```

Once String object is created:

```text
Its value cannot change.
```

---

# Example

```java
String name = "Java";

name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java
```

Not:

```text
Java Programming
```

---

# Why?

Because:

```java
concat()
```

creates a new object.

Original object remains unchanged.

---

# Correct Way

```java
String name = "Java";

name = name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java Programming
```

---

# Memory Example

```java
String s = "Java";
```

Memory:

```text
Java
```

After:

```java
s = s.concat(" Spring");
```

New Object:

```text
Java Spring
```

Old object remains unchanged.

---

# Why String is Immutable?

Several reasons:

---

## 1. Security

Used in:

```text
Database URL
Username
Password
Network Connections
File Paths
```

If String were mutable:

```text
Data could be changed accidentally.
```

---

## 2. String Pool Safety

Many references share same object.

Example:

```java
String s1 = "Java";

String s2 = "Java";
```

If one modifies:

```java
s1
```

then

```java
s2
```

would also change.

Not acceptable.

---

## 3. Thread Safety

Immutable objects are automatically thread-safe.

Multiple threads can safely use same String object.

---

## Interview Definition

```text
String is immutable because once created,
its state cannot be modified.
Any modification creates a new object.
```

---

# String Comparison

Important Interview Topic.

---

# == Operator

Compares:

```text
Memory Address
Reference
```

Example:

```java
String s1 = "Java";

String s2 = "Java";

System.out.println(s1 == s2);
```

Output:

```text
true
```

Same pooled object.

---

Example:

```java
String s1 = new String("Java");

String s2 = new String("Java");
```

```java
System.out.println(s1 == s2);
```

Output:

```text
false
```

Different objects.

---

# equals() Method

Compares:

```text
Content
```

Example:

```java
String s1 = new String("Java");

String s2 = new String("Java");

System.out.println(s1.equals(s2));
```

Output:

```text
true
```

Content same.

---

# StringBuilder

## Definition

StringBuilder is a mutable sequence of characters.

Mutable means:

```text
Can Be Modified
```

without creating new object.

---

# Example

```java
StringBuilder sb =
        new StringBuilder("Java");

sb.append(" Spring");

System.out.println(sb);
```

Output:

```text
Java Spring
```

---

# Memory

```java
StringBuilder
```

modifies same object.

No new object created.

---

# Common Methods

## append()

```java
sb.append(" SQL");
```

---

## insert()

```java
sb.insert(0,"Hello ");
```

---

## delete()

```java
sb.delete(0,5);
```

---

## reverse()

```java
sb.reverse();
```

---

# Example

```java
StringBuilder sb =
        new StringBuilder("Java");

sb.reverse();

System.out.println(sb);
```

Output:

```text
avaJ
```

---

# StringBuffer

## Definition

StringBuffer is also mutable.

Same functionality as StringBuilder.

But:

```text
Thread Safe
```

---

# Example

```java
StringBuffer sb =
        new StringBuffer("Java");

sb.append(" Spring");

System.out.println(sb);
```

Output:

```text
Java Spring
```

---

# Why Thread Safe?

Methods are synchronized.

Example:

```java
public synchronized StringBuffer append()
```

Multiple threads can safely use it.

---

# StringBuilder vs StringBuffer

Most Asked Interview Question.

---

| Feature | StringBuilder | StringBuffer |
|----------|----------|----------|
| Mutable | Yes | Yes |
| Thread Safe | No | Yes |
| Synchronization | No | Yes |
| Performance | Faster | Slower |
| Introduced | Java 5 | Java 1.0 |

---

# When To Use?

## StringBuilder

Use when:

```text
Single Thread Application
```

Examples:

```text
Data Processing
Algorithms
String Manipulation
```

---

## StringBuffer

Use when:

```text
Multiple Threads
```

Examples:

```text
Concurrent Applications
Multithreaded Systems
```

---

# String vs StringBuilder

| Feature | String | StringBuilder |
|----------|----------|----------|
| Mutable | No | Yes |
| New Object Created | Yes | No |
| Memory Usage | More | Less |
| Performance | Slower | Faster |

---

# Example

## Using String

```java
String s = "Java";

s = s + " Spring";

s = s + " SQL";
```

Creates multiple objects.

---

## Using StringBuilder

```java
StringBuilder sb =
        new StringBuilder("Java");

sb.append(" Spring");

sb.append(" SQL");
```

Uses same object.

Better performance.

---

# Frequently Asked Interview Questions

## Q1. What is String Pool?

```text
Special memory area in Heap
used to store String literals.
```

---

## Q2. Why String is Immutable?

```text
Security
Thread Safety
String Pool Optimization
```

---

## Q3. Difference Between == and equals()?

==

```text
Compare References
```

equals()

```text
Compare Content
```

---

## Q4. Difference Between String and StringBuilder?

String:

```text
Immutable
```

StringBuilder:

```text
Mutable
```

---

## Q5. Difference Between StringBuilder and StringBuffer?

StringBuilder:

```text
Not Thread Safe
Fast
```

StringBuffer:

```text
Thread Safe
Slow
```

---

## Q6. Which Is Faster?

```text
StringBuilder
```

Because no synchronization.

---

## Q7. Does concat() Modify Original String?

```text
No
```

Creates a new String object.

---

# One-Minute Revision

```text
String
-------------
Immutable

String Pool
-------------
Stores String Literals
Saves Memory

String Creation
-------------
String s = "Java";

String s =
new String("Java");

== Operator
-------------
Compare References

equals()
-------------
Compare Content

String Immutability
-------------
Cannot Change
New Object Created

StringBuilder
-------------
Mutable
Fast
Not Thread Safe

StringBuffer
-------------
Mutable
Thread Safe
Synchronized

StringBuilder vs StringBuffer
-------------
Fast vs Safe

Interview Favorite
-------------
Why String Immutable?
What is String Pool?
Difference:
== vs equals()
StringBuilder vs StringBuffer
```

# End of Java Strings Notes