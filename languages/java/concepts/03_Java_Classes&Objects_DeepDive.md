# Java Classes & Objects Deep Dive - Interview Revision Notes

> This section covers advanced Class & Object concepts frequently asked in Java interviews.
>
> Topics Covered:
>
> - Object Lifecycle
> - Immutable Classes
> - POJO
> - JavaBean
> - Static vs Instance Members

---

# What is an Object?

An object is an instance of a class.

Example:

```java
class User {

    String name;
}
```

Object Creation:

```java
User u = new User();
```

Here:

```java
User
```

is a class.

```java
u
```

is an object.

---

# Object Lifecycle

One of the most important interview topics.

Every object goes through multiple phases.

```text
Creation
    ↓
Initialization
    ↓
Usage
    ↓
Eligible For GC
    ↓
Destroyed By GC
```

---

# Step 1: Object Creation

Object is created using:

```java
new
```

Example:

```java
User u = new User();
```

What happens internally?

### JVM Actions

```text
1. Memory allocated in Heap
2. Constructor called
3. Object initialized
4. Reference returned
```

---

# Step 2: Object Initialization

Constructor initializes object.

Example:

```java
class User {

    String name;

    User() {

        name = "Rahul";
    }
}
```

Usage:

```java
User u = new User();

System.out.println(u.name);
```

Output:

```text
Rahul
```

---

# Step 3: Object Usage

Object methods and variables are used.

Example:

```java
class User {

    void login() {

        System.out.println("Login Success");
    }
}
```

Usage:

```java
User u = new User();

u.login();
```

Output:

```text
Login Success
```

---

# Step 4: Object Becomes Eligible For Garbage Collection

Example:

```java
User u = new User();

u = null;
```

Now:

```text
No reference points to object.
```

Object becomes eligible for GC.

---

# Step 5: Garbage Collection

JVM automatically removes unused objects.

Example:

```java
System.gc();
```

Important:

```text
GC can be requested
but not guaranteed.
```

---

# Object Lifecycle Diagram

```text
User u = new User()
        ↓
Object Created
        ↓
Constructor Executes
        ↓
Object Used
        ↓
Reference Removed
        ↓
Eligible For GC
        ↓
Memory Released
```

---

# Immutable Class

## Definition

An Immutable Class is a class whose objects
cannot be modified after creation.

Once object is created:

```text
State Cannot Change
```

---

# Example

String class is immutable.

```java
String name = "Java";

name.concat(" Programming");

System.out.println(name);
```

Output:

```text
Java
```

Original object unchanged.

---

# How To Create Immutable Class?

## Step 1

Make class final.

```java
final class Employee
```

---

## Step 2

Make variables private and final.

```java
private final String name;
```

---

## Step 3

Initialize using constructor.

---

## Step 4

Provide getters only.

No setters.

---

# Complete Example

```java
final class Employee {

    private final int id;
    private final String name;

    public Employee(int id,String name) {

        this.id=id;
        this.name=name;
    }

    public int getId() {

        return id;
    }

    public String getName() {

        return name;
    }
}
```

Usage:

```java
Employee e =
        new Employee(101,"Rahul");

System.out.println(e.getName());
```

Output:

```text
Rahul
```

Cannot modify object.

---

# Why Immutable Classes?

Benefits:

```text
Thread Safe
Secure
Easy To Understand
No Data Corruption
```

---

# Interview Definition

```text
Immutable class is a class whose
state cannot be changed after object creation.
```

---

# POJO

## Full Form

```text
Plain Old Java Object
```

---

# What is POJO?

A simple Java class that contains:

```text
Private Variables
Constructors
Getter Methods
Setter Methods
```

No special restrictions.

---

# Example

```java
class Student {

    private int id;
    private String name;

    public Student() {

    }

    public Student(int id,String name) {

        this.id=id;
        this.name=name;
    }

    public int getId() {

        return id;
    }

    public void setId(int id) {

        this.id=id;
    }

    public String getName() {

        return name;
    }

    public void setName(String name) {

        this.name=name;
    }
}
```

---

# Why Use POJO?

```text
Simple
Reusable
Easy Maintenance
Framework Friendly
```

---

# Interview Definition

```text
POJO is a simple Java object
that does not depend on any special framework.
```

---

# JavaBean

## Definition

JavaBean is a special type of POJO.

It follows specific rules.

---

# Rules Of JavaBean

### Rule 1

Class must be public.

```java
public class Employee
```

---

### Rule 2

Variables must be private.

```java
private int id;
```

---

### Rule 3

Must provide getters/setters.

```java
getId()
setId()
```

---

### Rule 4

Must have no-argument constructor.

```java
public Employee() {

}
```

---

### Rule 5

Should implement Serializable.

```java
implements Serializable
```

(Optional but preferred)

---

# Example

```java
import java.io.Serializable;

public class Employee
implements Serializable {

    private int id;
    private String name;

    public Employee() {

    }

    public int getId() {

        return id;
    }

    public void setId(int id) {

        this.id=id;
    }

    public String getName() {

        return name;
    }

    public void setName(String name) {

        this.name=name;
    }
}
```

---

# POJO vs JavaBean

| Feature | POJO | JavaBean |
|----------|----------|----------|
| Private Variables | Optional | Required |
| Getter/Setter | Optional | Required |
| No Arg Constructor | Optional | Required |
| Serializable | Optional | Usually Yes |
| Restrictions | Very Few | More Rules |

---

# Static vs Instance Members

Very Important Interview Topic.

---

# Static Variable

Static variable belongs to:

```text
Class
```

Only one copy exists.

Shared among all objects.

---

# Example

```java
class Student {

    static String college =
            "ABC College";
}
```

Usage:

```java
Student s1 = new Student();
Student s2 = new Student();

System.out.println(Student.college);
```

Output:

```text
ABC College
```

---

# Static Memory Diagram

```text
Student Class
    ↓
college
    ↓
Single Copy
```

All objects share same value.

---

# Instance Variable

Instance variable belongs to:

```text
Object
```

Each object gets separate copy.

---

# Example

```java
class Student {

    String name;
}
```

Usage:

```java
Student s1 = new Student();
Student s2 = new Student();

s1.name = "Rahul";
s2.name = "Amit";
```

Memory:

```text
s1 -> Rahul

s2 -> Amit
```

Separate copies.

---

# Static Method

Belongs to class.

Can be called without object.

Example:

```java
class MathUtil {

    static void show() {

        System.out.println("Static");
    }
}
```

Usage:

```java
MathUtil.show();
```

---

# Instance Method

Belongs to object.

Needs object.

Example:

```java
class User {

    void login() {

        System.out.println("Login");
    }
}
```

Usage:

```java
User u = new User();

u.login();
```

---

# Static Block

Used to initialize static variables.

Runs only once.

Example:

```java
class Test {

    static {

        System.out.println("Static Block");
    }
}
```

---

# Static vs Instance

| Feature | Static | Instance |
|----------|----------|----------|
| Belongs To | Class | Object |
| Memory | One Copy | Multiple Copies |
| Access | Class Name | Object |
| Created When | Class Loaded | Object Created |
| Shared | Yes | No |

---

# Interview Questions

## Q1. What is Object Lifecycle?

```text
Creation
Initialization
Usage
Eligible For GC
Destroyed
```

---

## Q2. What is Immutable Class?

```text
A class whose object state
cannot be changed after creation.
```

---

## Q3. How To Create Immutable Class?

```text
1. Make class final
2. Make variables private final
3. Initialize through constructor
4. Provide getters only
5. No setters
```

---

## Q4. What is POJO?

```text
Plain Old Java Object
```

Simple Java class without framework dependency.

---

## Q5. What is JavaBean?

```text
Special POJO
```

Must have:

```text
Private Variables
Public Getters/Setters
No-Arg Constructor
Serializable
```

---

## Q6. Difference Between POJO And JavaBean?

```text
JavaBean follows strict rules.

POJO does not.
```

---

## Q7. Difference Between Static And Instance Variable?

Static:

```text
Belongs To Class
One Copy
Shared
```

Instance:

```text
Belongs To Object
Multiple Copies
```

---

# One-Minute Revision

```text
Object Lifecycle
-----------------
Create
Initialize
Use
GC Eligible
Destroyed

Immutable Class
-----------------
final class
private final variables
No setters

POJO
-----------------
Plain Old Java Object

JavaBean
-----------------
POJO + Rules

Static
-----------------
Class Level
One Copy
Shared

Instance
-----------------
Object Level
Multiple Copies

Static Method
-----------------
ClassName.method()

Instance Method
-----------------
object.method()
```

# End of Classes & Objects Deep Dive Notes