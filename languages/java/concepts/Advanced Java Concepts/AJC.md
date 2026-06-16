# Advanced Java Concepts - Complete Interview Revision Notes

> 🔴 VERY IMPORTANT FOR SENIOR JAVA INTERVIEWS
>
> Frequently Asked In:
>
> - Java Backend Interviews
> - Spring Boot Interviews
> - Framework Development
> - Hibernate Internals
> - Spring Internals
> - JVM Interviews
>
> Topics Covered:
>
> - Reflection
> - Annotations
> - Dynamic Proxies
> - Class Loaders
> - Unsafe Operations
> - Real World Examples
> - Spring & Hibernate Internals

---

# Why Advanced Java?

Frameworks like:

```text
Spring

Spring Boot

Hibernate

JUnit

Mockito

Lombok
```

heavily use:

```text
Reflection
Annotations
Dynamic Proxies
Class Loaders
```

---

# Reflection

Most Asked Advanced Java Topic.

---

# What is Reflection?

Reflection allows Java programs to:

```text
Inspect Classes

Inspect Methods

Inspect Fields

Create Objects

Invoke Methods
```

at Runtime.

---

# Normal Method Call

```java
User user = new User();

user.login();
```

Compile time knowledge required.

---

# Reflection Method Call

```java
Class<?> clazz =
        User.class;
```

```java
Method method =
        clazz.getMethod("login");
```

```java
method.invoke(
        new User());
```

Method called dynamically.

---

# Real World Example

Spring Framework

When Spring starts:

```text
Scan Classes

Find Annotations

Create Beans
```

using Reflection.

---

# Getting Class Information

Example

```java
class User {

    private String name;

    public void login(){

    }
}
```

---

Get Class

```java
Class<?> clazz =
        User.class;
```

---

Get Class Name

```java
System.out.println(
        clazz.getName());
```

Output:

```text
User
```

---

Get Methods

```java
Method[] methods =
        clazz.getDeclaredMethods();
```

---

Get Fields

```java
Field[] fields =
        clazz.getDeclaredFields();
```

---

# Creating Object Using Reflection

Normal:

```java
User user =
        new User();
```

Reflection:

```java
Class<?> clazz =
        User.class;
```

```java
Object obj =
        clazz.getDeclaredConstructor()
             .newInstance();
```

---

# Access Private Field

Example

```java
class User {

    private String name =
            "Vinay";
}
```

---

Reflection

```java
Field field =
        clazz.getDeclaredField(
                "name");
```

```java
field.setAccessible(true);
```

```java
System.out.println(
        field.get(user));
```

Output:

```text
Vinay
```

---

# Why Reflection Is Powerful?

Can access:

```text
Private Fields

Private Methods

Constructors

Metadata
```

---

# Reflection Drawbacks

```text
Slower

Breaks Encapsulation

Security Risks

Harder To Debug
```

---

# Annotations

Most Important Spring Topic.

---

# What is Annotation?

Annotation provides metadata to classes and methods.

---

Example

```java
@Override
public String toString(){

}
```

---

Here:

```java
@Override
```

is annotation.

---

# Why Annotations?

Without annotations:

```text
XML Configuration
Large Configuration Files
```

---

With annotations:

```java
@Service

@Repository

@RestController
```

Cleaner code.

---

# Common Java Annotations

```java
@Override

@Deprecated

@SuppressWarnings
```

---

# Example

```java
class Parent {

    void show(){

    }
}
```

```java
class Child extends Parent {

    @Override
    void show(){

    }
}
```

Compiler validates override.

---

# Custom Annotation

Step 1

```java
@interface Author {

    String name();
}
```

---

Step 2

Use Annotation

```java
@Author(name="Vinay")
class User {

}
```

---

# Annotation Retention Policies

Interview Question.

---

## SOURCE

Available only during compilation.

Example:

```java
@Override
```

---

## CLASS

Stored in class file.

Not available at runtime.

---

## RUNTIME

Available through reflection.

Example:

```java
@Service

@Entity
```

---

# Real World Example

Spring

```java
@Service
public class UserService {

}
```

Spring uses reflection to detect:

```java
@Service
```

and create bean.

---

# Dynamic Proxies

Very Important Spring AOP Topic.

---

# What is Proxy?

Proxy sits between:

```text
Caller

Actual Object
```

---

Example

```text
Client
   ↓
Proxy
   ↓
Service
```

---

# Why Use Proxy?

Add extra behavior.

Examples:

```text
Logging

Security

Transactions

Caching
```

without modifying original code.

---

# Real World Example

Spring @Transactional

```java
@Transactional
public void saveUser(){

}
```

Spring creates proxy around method.

---

Before method:

```text
Start Transaction
```

After method:

```text
Commit Transaction
```

---

# Dynamic Proxy Example

Interface

```java
interface PaymentService {

    void pay();
}
```

---

Implementation

```java
class PaymentServiceImpl
implements PaymentService {

    public void pay(){

        System.out.println(
                "Payment Done");
    }
}
```

---

Proxy

```java
PaymentService proxy =
    (PaymentService)
    Proxy.newProxyInstance(
        PaymentService.class
            .getClassLoader(),
        new Class[]{
            PaymentService.class
        },
        (obj,method,args)->{

            System.out.println(
                    "Before");

            Object result =
                    method.invoke(
                            service,
                            args);

            System.out.println(
                    "After");

            return result;
        });
```

---

Output

```text
Before

Payment Done

After
```

---

# Real Framework Usage

Spring AOP

```java
@Transactional

@Cacheable

@Async
```

implemented using proxies.

---

# Class Loaders

Very Important JVM Topic.

---

# What is Class Loader?

Loads:

```text
.class Files
```

into JVM memory.

---

Example

```java
User.class
```

↓

```text
Class Loader
```

↓

```text
JVM Memory
```

---

# Class Loader Types

---

# Bootstrap ClassLoader

Loads:

```java
String
Object
Integer
```

Core Java classes.

---

# Extension ClassLoader

Loads:

```text
External Libraries
```

---

# Application ClassLoader

Loads:

```text
Your Classes
```

Examples:

```java
User

Employee

Product
```

---

# Hierarchy

```text
Bootstrap

   ↑

Extension

   ↑

Application
```

---

# Parent Delegation Model

Interview Favorite.

---

Application Loader asks:

```text
Parent First
```

before loading class.

---

Why?

Security.

Prevents replacement of:

```java
java.lang.String
```

---

# Example

```java
ClassLoader loader =
        User.class
            .getClassLoader();

System.out.println(loader);
```

Output:

```text
ApplicationClassLoader
```

---

# Unsafe Operations

Senior-Level JVM Topic.

---

# What is Unsafe?

Internal JVM API.

Class:

```java
sun.misc.Unsafe
```

---

# Why Called Unsafe?

Can bypass JVM safety rules.

---

Can:

```text
Allocate Memory

Modify Memory Directly

Create Objects Without Constructor

CAS Operations
```

---

# Example

Normal Object Creation

```java
User user =
        new User();
```

Constructor executes.

---

Unsafe

```java
Unsafe unsafe = ...
```

```java
User user =
    (User) unsafe.allocateInstance(
            User.class);
```

Constructor NOT executed.

---

# Why Use Unsafe?

Used internally by:

```text
Netty

Kafka

Concurrent Libraries

JVM Internals
```

for high performance.

---

# Example

Normal Memory

```java
int[] arr =
        new int[100];
```

Managed by JVM.

---

Unsafe Memory

```java
long address =
        unsafe.allocateMemory(
                100);
```

Direct memory allocation.

---

# Risks

```text
Memory Corruption

JVM Crash

Security Issues

Hard Debugging
```

---

# Modern Alternative

Instead of Unsafe:

```text
VarHandle

ByteBuffer

Foreign Memory API
```

preferred.

---

# How Spring Uses These Concepts

---

# Reflection

```java
@Service
```

Bean creation.

---

# Annotations

```java
@RestController

@Service

@Repository
```

Configuration.

---

# Dynamic Proxies

```java
@Transactional

@Cacheable

@Async
```

AOP features.

---

# Class Loaders

Load application classes.

---

# Unsafe

Used indirectly by some libraries.

---

# Frequently Asked Interview Questions

## Q1. What is Reflection?

```text
Ability To Inspect And
Manipulate Classes At Runtime.
```

---

## Q2. Why Reflection Is Used?

```text
Framework Development

Dependency Injection

ORM Frameworks
```

---

## Q3. What is Annotation?

```text
Metadata Attached To Code.
```

---

## Q4. Why Use Annotations?

```text
Provide Configuration
Without XML
```

---

## Q5. What is Dynamic Proxy?

```text
Object That Adds Behavior
Before/After Method Calls.
```

---

## Q6. Where Does Spring Use Dynamic Proxies?

```text
@Transactional

@Cacheable

@Async
```

---

## Q7. What Are Class Loader Types?

```text
Bootstrap

Extension

Application
```

---

## Q8. What is Parent Delegation Model?

```text
Class Loader Asks Parent First
Before Loading Class.
```

---

## Q9. What is Unsafe?

```text
Low-Level JVM API
For Direct Memory Access.
```

---

## Q10. Why Avoid Unsafe?

```text
Can Crash JVM

Bypasses Safety Checks
```

---

# One-Minute Revision

```text
Reflection
----------------
Inspect Classes At Runtime

Used For
----------------
Spring
Hibernate
JUnit

Annotations
----------------
Metadata

Examples
----------------
@Override
@Service
@Entity

Dynamic Proxy
----------------
Adds Extra Behavior

Used By
----------------
@Transactional
@Cacheable
@Async

Class Loaders
----------------
Bootstrap
Extension
Application

Parent Delegation
----------------
Ask Parent First

Unsafe
----------------
Direct Memory Access

Risks
----------------
Memory Corruption
JVM Crash

Interview Favorites
----------------
Reflection
Annotations
Dynamic Proxy
Class Loader
Unsafe
Spring Internals
```

# End of Advanced Java Concepts Notes