# Java Copying, Pass By Value, Shallow Copy & Deep Copy - Complete Interview Revision Notes

> One of the most confusing topics for Java developers coming from JavaScript.
>
> JS Concepts:
>
> - Pass By Value
> - Pass By Reference
> - Shallow Copy
> - Deep Copy
> - Structured Clone
>
> Java has similar concepts but implemented differently.

---

# Most Important Interview Fact

## Is Java Pass By Value or Pass By Reference?

✅ Java is ALWAYS Pass By Value.

There is NO Pass By Reference in Java.

---

# Primitive Types

Primitive Types:

```java
byte
short
int
long
float
double
char
boolean
```

Store actual values.

---

# Example

```java
public class Main {

    static void change(int x) {

        x = 100;
    }

    public static void main(String[] args) {

        int a = 10;

        change(a);

        System.out.println(a);
    }
}
```

Output:

```text
10
```

---

# Memory

Before:

```text
a = 10
```

Method Call:

```java
change(a);
```

Memory:

```text
a -> 10

x -> 10
```

Different copies.

Changing:

```java
x = 100;
```

does not affect:

```java
a
```

---

# Primitive Summary

```text
Actual Value Copied
```

Example:

```java
int
long
double
boolean
```

---

# Objects

Objects behave differently.

---

# Example

```java
class User {

    String name;
}
```

```java
public class Main {

    static void update(User user) {

        user.name = "Vinay";
    }

    public static void main(String[] args) {

        User u = new User();

        u.name = "Java";

        update(u);

        System.out.println(u.name);
    }
}
```

Output:

```text
Vinay
```

---

# Why Did It Change?

Java passed:

```text
Copy Of Reference
```

not:

```text
Object Itself
```

---

# Memory Diagram

Heap:

```text
User Object

name = Java
```

Stack:

```text
u ------> User Object
```

Method Call:

```java
update(u);
```

Creates:

```text
user ------> User Object
```

Both references point to same object.

```text
u --------\
           \
            User Object
           /
user -----/
```

---

# Modifying Object

```java
user.name = "Vinay";
```

Changes same object.

Result:

```text
Vinay
```

---

# Reassigning Object

Example:

```java
static void update(User user) {

    user = new User();

    user.name = "Spring";
}
```

---

```java
User u = new User();

u.name = "Java";

update(u);

System.out.println(u.name);
```

Output:

```text
Java
```

---

# Why?

Method got:

```text
Copy Of Reference
```

Changing:

```java
user = new User();
```

changes only local reference.

Original:

```java
u
```

unchanged.

---

# Memory

Before:

```text
u ------> User1

user ---> User1
```

After:

```text
u ------> User1

user ---> User2
```

Different objects.

---

# Interview Answer

```text
Java is always Pass By Value.

Primitive Types:
Copy of actual value.

Objects:
Copy of reference value.
```

---

# Java Copying Concepts

Java does not have:

```text
structuredClone()
```

like JavaScript.

We implement copies manually.

---

# Original Object

```java
class Address {

    String city;

    Address(String city){

        this.city = city;
    }
}
```

```java
class User {

    String name;

    Address address;

    User(String name,
         Address address){

        this.name = name;
        this.address = address;
    }
}
```

---

# Shallow Copy

Most Asked Interview Question.

---

# What is Shallow Copy?

Creates:

```text
New Parent Object
```

but nested objects are shared.

---

# Example

```java
Address addr =
        new Address("Hyderabad");
```

```java
User u1 =
        new User(
                "Vinay",
                addr);
```

---

Shallow Copy:

```java
User u2 =
        new User(
                u1.name,
                u1.address);
```

---

# Memory

```text
u1 --------\
            \
             Address
            /
u2 --------/
```

Same Address object.

---

# Verify

```java
u2.address.city =
        "Vijayawada";
```

---

Print:

```java
System.out.println(
        u1.address.city);
```

Output:

```text
Vijayawada
```

---

# Why?

Because:

```text
Address Reference Shared
```

---

# Shallow Copy Diagram

```text
User1
   |
   ↓
Address

User2
   |
   ↓
Address
```

One Address object.

---

# Deep Copy

Most Important Copying Concept.

---

# What is Deep Copy?

Creates:

```text
New Parent Object
```

AND

```text
New Nested Objects
```

Everything copied.

---

# Example

```java
Address newAddress =
        new Address(
                u1.address.city);
```

```java
User u2 =
        new User(
                u1.name,
                newAddress);
```

---

# Memory

```text
User1
  |
  ↓
Address1
```

```text
User2
  |
  ↓
Address2
```

Completely separate.

---

# Verify

```java
u2.address.city =
        "Vijayawada";
```

---

Print:

```java
System.out.println(
        u1.address.city);
```

Output:

```text
Hyderabad
```

Original unchanged.

---

# Deep Copy Diagram

```text
User1
  |
  ↓
Address1

User2
  |
  ↓
Address2
```

No shared references.

---

# Copy Constructor

Common Deep Copy Technique.

---

# Example

```java
class Address {

    String city;

    Address(Address other){

        this.city =
                other.city;
    }
}
```

---

```java
class User {

    String name;

    Address address;

    User(User other){

        this.name =
                other.name;

        this.address =
                new Address(
                        other.address);
    }
}
```

Usage:

```java
User copy =
        new User(original);
```

Deep copy.

---

# Cloneable Interface

Older approach.

---

# Example

```java
class User
implements Cloneable {

    String name;

    @Override
    public Object clone()
    throws CloneNotSupportedException {

        return super.clone();
    }
}
```

---

# Problem

```java
super.clone()
```

creates:

```text
Shallow Copy
```

by default.

---

# Deep Clone With clone()

```java
@Override
public Object clone()
throws CloneNotSupportedException {

    User cloned =
            (User) super.clone();

    cloned.address =
            (Address)
            address.clone();

    return cloned;
}
```

---

# Serialization Deep Copy

Production approach sometimes used.

---

# Idea

```text
Object
 ↓
Serialize
 ↓
Deserialize
 ↓
New Object Graph
```

Result:

```text
Deep Copy
```

---

# Java vs JavaScript

| Concept | Java | JavaScript |
|-----------|-----------|-----------|
| Pass By Value | Yes | Yes |
| Pass By Reference | No | No |
| Object Reference Copy | Yes | Yes |
| Shallow Copy | Manual | spread() |
| Deep Copy | Manual | structuredClone() |
| Clone Support | clone() | structuredClone() |

---

# Interview Questions

## Q1. Is Java Pass By Reference?

```text
No.

Java is always Pass By Value.
```

---

## Q2. What Gets Passed For Objects?

```text
Copy Of Reference
```

---

## Q3. Difference Between Shallow Copy and Deep Copy?

Shallow:

```text
Nested Objects Shared
```

Deep:

```text
Nested Objects Copied
```

---

## Q4. Does clone() Create Deep Copy?

```text
No

Default clone() is Shallow Copy.
```

---

## Q5. Equivalent Of JS structuredClone()?

```text
Deep Copy

Using:
Copy Constructor
Serialization
Custom Clone
```

---

# One-Minute Revision

```text
Java Parameter Passing
----------------------
Always Pass By Value

Primitive Types
----------------------
Actual Value Copied

Object Types
----------------------
Reference Value Copied

Shallow Copy
----------------------
Parent Copied

Nested Objects Shared

Example
----------------------
User Copy

Same Address Object

Deep Copy
----------------------
Parent Copied

Nested Objects Copied

Example
----------------------
User Copy

New Address Object

clone()
----------------------
Default = Shallow Copy

Copy Constructor
----------------------
Preferred Deep Copy Method

JS structuredClone()
----------------------
Equivalent To

Java Deep Copy

Interview Favorites
----------------------
Pass By Value

Pass By Reference Myth

Shallow Copy

Deep Copy

clone() vs Copy Constructor
```

# End of Java Copying & Reference Notes