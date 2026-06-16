# Java OOP (Object-Oriented Programming) - Complete Interview Revision Notes

> Based on standard Java interview topics and aligned with your notes.
> Covers:
>
> - Class & Object
> - Encapsulation
> - Inheritance
> - Polymorphism
> - Abstraction
> - Method Overloading
> - Method Overriding
> - Constructor
> - Constructor Overloading
> - Constructor Chaining
> - this Keyword
> - Access Modifiers
> - IS-A Relationship
> - HAS-A Relationship (Composition & Aggregation)

---

# What is OOP?

OOP (Object-Oriented Programming) is a programming style that uses **objects** to organize software design.

Benefits:

- Code Reusability
- Scalability
- Easy Maintenance
- Easy Debugging
- Better Software Structure

---

# Class and Object

## Class

A class is a blueprint/template used to create objects.

A class contains:

- Variables (Data Members)
- Methods (Member Functions)

Example:

```java
class User {
    String name;
}
```

Here:

```java
User
```

is a class.

---

## Object

An object is an instance of a class.

Objects represent actual data.

Syntax:

```java
ClassName objectName = new ClassName();
```

Example:

```java
class User {

    String name;
}

public class Main {

    public static void main(String[] args) {

        User u1 = new User();

        u1.name = "Rahul";

        System.out.println(u1.name);
    }
}
```

Output:

```text
Rahul
```

---

# Four Pillars of OOP

```text
1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction
```

---

# 1. Encapsulation

## Definition

Encapsulation means:

> Binding data members and methods into a single unit and protecting data from misuse.

Also called:

```text
Data Hiding
```

---

## Why Encapsulation?

Without Encapsulation:

```java
class User {

    public String password;
}
```

Anyone can modify password.

```java
user.password = "123";
```

Not secure.

---

# Steps To Encapsulate Program

## Step 1

Make variables private.

```java
private String password;
```

---

## Step 2

Provide public methods.

```java
public void setPassword()
```

```java
public String getPassword()
```

---

# Complete Example

```java
class User {

    private String password;

    public void setPassword(String password) {

        if(password.length() >= 8) {
            this.password = password;
        }
    }

    public String getPassword() {

        return password;
    }
}
```

Usage:

```java
public class Main {

    public static void main(String[] args) {

        User u = new User();

        u.setPassword("admin123");

        System.out.println(u.getPassword());
    }
}
```

Output:

```text
admin123
```

---

# Advantages

- Security
- Data Protection
- Controlled Access
- Easy Maintenance

---

# Interview Definition

```text
Encapsulation is the process of hiding data
using private variables and providing
controlled access through public methods.
```

---

# 2. Inheritance

## Definition

Inheritance means:

> One class acquires properties and methods of another class.

Keyword:

```java
extends
```

---

# IS-A Relationship

Inheritance creates an

```text
IS-A Relationship
```

Examples:

```text
Dog IS-A Animal

Car IS-A Vehicle

Manager IS-A Employee
```

---

# Example

```java
class Animal {

    void eat() {

        System.out.println("Eating");
    }
}
```

```java
class Dog extends Animal {

}
```

Usage:

```java
Dog d = new Dog();

d.eat();
```

Output:

```text
Eating
```

---

# Why Inheritance?

- Code Reusability
- Avoid Duplicate Code
- Easy Maintenance

---

# Types of Inheritance

## Single Level

```text
A → B
```

---

## Multilevel

```text
A → B → C
```

Example:

```java
class Animal {}

class Dog extends Animal {}

class Puppy extends Dog {}
```

---

## Hierarchical

```text
        Animal
        /    \
      Dog    Cat
```

---

## Multiple Inheritance

Not supported with classes.

❌

```java
class C extends A,B
```

---

# Interview Definition

```text
Inheritance allows one class to acquire
properties and methods of another class
using extends keyword.
```

---

# 3. Polymorphism

## Definition

Polymorphism means:

```text
One Thing → Many Forms
```

Example:

```text
Person

Student
Employee
Customer
```

Same person, multiple forms.

---

# Types of Polymorphism

## Compile Time Polymorphism

Achieved using:

```text
Method Overloading
```

---

## Runtime Polymorphism

Achieved using:

```text
Method Overriding
```

---

# Interview Definition

```text
Polymorphism allows the same method
to behave differently in different situations.
```

---

# Method Overloading

## Definition

Same method name

Different parameters

---

# Example

```java
class Calculator {

    int add(int a,int b) {

        return a+b;
    }

    int add(int a,int b,int c) {

        return a+b+c;
    }

    double add(double a,double b) {

        return a+b;
    }
}
```

Usage:

```java
Calculator c = new Calculator();

System.out.println(c.add(10,20));

System.out.println(c.add(10,20,30));

System.out.println(c.add(10.5,20.5));
```

Output:

```text
30
60
31.0
```

---

# Method Overriding

## Definition

Child class provides its own implementation of parent method.

---

# Example

```java
class Animal {

    void sound() {

        System.out.println("Animal Sound");
    }
}
```

```java
class Dog extends Animal {

    @Override
    void sound() {

        System.out.println("Bark");
    }
}
```

Usage:

```java
Animal a = new Dog();

a.sound();
```

Output:

```text
Bark
```

---

# Difference Between Overloading and Overriding

| Feature | Overloading | Overriding |
|----------|------------|------------|
| Class | Same Class | Parent + Child |
| Parameters | Different | Same |
| Binding | Compile Time | Runtime |
| Polymorphism | Compile Time | Runtime |

---

# Method Hiding

## Definition

Method Hiding occurs when a child class defines a static method
with the same signature as a static method in the parent class.

Unlike method overriding:

```text
Static Methods → Hidden
Instance Methods → Overridden
```

---

## Example

```java
class Parent {

    static void display() {

        System.out.println("Parent Display");
    }
}
```

```java
class Child extends Parent {

    static void display() {

        System.out.println("Child Display");
    }
}
```

Usage:

```java
public class Main {

    public static void main(String[] args) {

        Parent p = new Child();

        p.display();
    }
}
```

Output:

```text
Parent Display
```

---

## Why?

Because static methods belong to class,
not to objects.

Method call depends on:

```java
Parent p
```

not

```java
new Child()
```

---

## Difference Between Method Hiding and Method Overriding

| Feature | Method Hiding | Method Overriding |
|----------|----------|----------|
| Method Type | Static | Non-Static |
| Binding | Compile Time | Runtime |
| Polymorphism | No | Yes |
| Keyword | static | @Override |

---

# Type Casting in Objects

Object casting means converting references
between parent and child classes.

Two Types:

```text
1. Upcasting
2. Downcasting
```

---

# Upcasting

## Definition

Converting Child Object Reference
to Parent Reference.

Upcasting is automatic.

---

## Syntax

```java
Parent p = new Child();
```

---

## Example

```java
class Animal {

    void eat() {

        System.out.println("Eating");
    }
}
```

```java
class Dog extends Animal {

    void bark() {

        System.out.println("Barking");
    }
}
```

Usage:

```java
Animal a = new Dog();

a.eat();
```

Output:

```text
Eating
```

---

## What Happens?

Object Created:

```java
new Dog()
```

Reference Type:

```java
Animal
```

Parent reference points to child object.

This is Upcasting.

---

## Why Use Upcasting?

Used to achieve:

```text
Runtime Polymorphism
```

Example:

```java
Animal a = new Dog();
```

```java
Animal a = new Cat();
```

Same code works for different objects.

---

# Downcasting

## Definition

Converting Parent Reference
back into Child Reference.

Downcasting is manual.

---

## Syntax

```java
Child c = (Child) parentReference;
```

---

## Example

```java
class Animal {

}
```

```java
class Dog extends Animal {

    void bark() {

        System.out.println("Bark");
    }
}
```

Usage:

```java
Animal a = new Dog();

Dog d = (Dog)a;

d.bark();
```

Output:

```text
Bark
```

---

## Why Needed?

Parent reference cannot access child-specific methods.

Example:

```java
Animal a = new Dog();

a.bark();
```

Compile Error

Because:

```java
Animal
```

doesn't contain:

```java
bark()
```

So:

```java
Dog d = (Dog)a;
```

Then:

```java
d.bark();
```

works.

---

# Invalid Downcasting

Example:

```java
Animal a = new Animal();

Dog d = (Dog)a;
```

Output:

```text
ClassCastException
```

Runtime Error

Because Animal object is not actually a Dog.

---

# Safe Downcasting Using instanceof

Example:

```java
Animal a = new Dog();

if(a instanceof Dog) {

    Dog d = (Dog)a;

    d.bark();
}
```

Output:

```text
Bark
```

---

# Upcasting vs Downcasting

| Feature | Upcasting | Downcasting |
|----------|----------|----------|
| Direction | Child → Parent | Parent → Child |
| Conversion | Automatic | Manual |
| Safe | Yes | May Throw Exception |
| Type Casting Required | No | Yes |
| Usage | Polymorphism | Access Child Features |

---

# Interview Questions

## Q1. What is Upcasting?

```text
Converting Child Object Reference
into Parent Reference.

Example:

Animal a = new Dog();
```

---

## Q2. What is Downcasting?

```text
Converting Parent Reference
back into Child Reference.

Example:

Dog d = (Dog)a;
```

---

## Q3. Why is Upcasting Used?

```text
To achieve Runtime Polymorphism.
```

---

## Q4. Can Static Methods be Overridden?

```text
No

Static methods are hidden,
not overridden.
```

---

## Q5. Difference Between Overriding and Hiding?

Overriding:

```text
Runtime
Non-Static Method
```

Method Hiding:

```text
Compile Time
Static Method
```

# 4. Abstraction

## Definition

Abstraction means:

> Showing only essential features and hiding implementation details.

---

# Real Life Example

ATM Machine

Visible:

```text
Withdraw
Deposit
Balance Check
```

Hidden:

```text
Database
Network
Server Logic
```

---

# How Real Applications Use Abstraction

Amazon App

Visible:

```java
placeOrder();
cancelOrder();
trackOrder();
```

Hidden:

```text
Database Logic
Inventory Logic
Payment Gateway
Delivery Service
```

User only sees required functionality.

---

# Steps To Implement Abstraction

## Step 1

Create Abstract Class

```java
abstract class Vehicle {

    abstract void start();
}
```

---

## Step 2

Provide Implementation

```java
class Car extends Vehicle {

    @Override
    void start() {

        System.out.println("Car Started");
    }
}
```

---

## Step 3

Create Object

```java
public class Main {

    public static void main(String[] args) {

        Vehicle v = new Car();

        v.start();
    }
}
```

Output:

```text
Car Started
```

---

# Advantages

- Hides Complexity
- Better Security
- Cleaner Design
- Easy Maintenance

---

# Interview Definition

```text
Abstraction hides implementation details
and shows only essential functionality.
```

---

# Constructor

## Definition

Constructor is a special method executed during object creation.

Characteristics:

- Same Name As Class
- No Return Type
- Called Automatically

---

# Example

```java
class User {

    User() {

        System.out.println("Constructor Called");
    }
}
```

Usage:

```java
User u = new User();
```

Output:

```text
Constructor Called
```

---

# Types of Constructors

## Default Constructor

```java
class User {

}
```

Java provides one automatically.

---

## Parameterized Constructor

```java
class User {

    String name;

    User(String name) {

        this.name = name;
    }
}
```

Usage:

```java
User u = new User("Rahul");
```

---

# Constructor Overloading

Multiple constructors in same class.

Example:

```java
class User {

    User() {

    }

    User(String name) {

    }

    User(int age) {

    }
}
```

Condition:

```text
Parameter list must be different.
```

---

# Constructor Chaining

## Definition

Calling one constructor from another constructor of same class.

Keyword:

```java
this()
```

---

# Example

```java
class Employee {

    int id;
    String name;

    Employee() {

        this(101,"Rahul");

        System.out.println("Default Constructor");
    }

    Employee(int id,String name) {

        this.id=id;
        this.name=name;

        System.out.println(id+" "+name);
    }
}
```

Usage:

```java
new Employee();
```

Output:

```text
101 Rahul
Default Constructor
```

---

# Rules

## Rule 1

```java
this()
```

must be first statement.

✅

```java
Employee() {

    this(10);
}
```

---

## Rule 2

Only one constructor can be called.

---

# Parent Constructor Chaining

When inheritance is used JVM automatically places:

```java
super();
```

---

Example

```java
class Parent {

    Parent() {

        System.out.println("Parent");
    }
}
```

```java
class Child extends Parent {

    Child() {

        System.out.println("Child");
    }
}
```

Usage:

```java
new Child();
```

Output:

```text
Parent
Child
```

---

# this Keyword

## Definition

```java
this
```

represents current object.

---

# Use 1: Access Current Object Variable

```java
class User {

    String name;

    User(String name) {

        this.name = name;
    }
}
```

---

# Use 2: Call Current Class Method

```java
class Demo {

    void m1() {

        this.m2();
    }

    void m2() {

        System.out.println("Method 2");
    }
}
```

---

# Use 3: Constructor Chaining

```java
class Demo {

    Demo() {

        this(10);
    }

    Demo(int x) {

        System.out.println(x);
    }
}
```

---

# HAS-A Relationship

When one class contains another class object.

Two Types:

```text
1. Composition
2. Aggregation
```

---

# Composition

Strong Relationship

Example:

```text
House HAS-A Room
```

If house is destroyed

```text
Room is also destroyed
```

---

# Program

```java
class Room {

}
```

```java
class House {

    private Room room = new Room();
}
```

---

# Aggregation

Weak Relationship

Example:

```text
Department HAS-A Employee
```

Employee can exist independently.

---

# Program

```java
class Employee {

}
```

```java
class Department {

    Employee emp;

    Department(Employee emp) {

        this.emp = emp;
    }
}
```

---

# Access Modifiers

Access modifiers control visibility.

---

# Private

Accessible only inside same class.

```java
private int age;
```

---

# Protected

Accessible:

```text
Same Package
Child Classes
```

Example:

```java
protected int age;
```

---

# Public

Accessible everywhere.

```java
public int age;
```

---

# Default

No keyword.

Accessible only inside package.

```java
int age;
```

---

# Access Modifier Table

| Modifier | Same Class | Same Package | Child Class | Anywhere |
|-----------|-----------|-----------|-----------|-----------|
| private | ✅ | ❌ | ❌ | ❌ |
| default | ✅ | ✅ | ❌ | ❌ |
| protected | ✅ | ✅ | ✅ | ❌ |
| public | ✅ | ✅ | ✅ | ✅ |

---

# Frequently Asked Interview Questions

## Q1. What are the Four Pillars of OOP?

```text
Encapsulation
Inheritance
Polymorphism
Abstraction
```

---

## Q2. Difference Between Encapsulation and Abstraction?

Encapsulation

```text
Hides Data
Uses private variables
```

Abstraction

```text
Hides Implementation
Uses abstract class/interface
```

---

## Q3. Difference Between Overloading and Overriding?

```text
Overloading → Compile Time

Overriding → Runtime
```

---

## Q4. What is Constructor Chaining?

```text
Calling one constructor from another constructor
using this() keyword.
```

---

## Q5. What is this Keyword?

```text
Represents Current Object.
```

---

## Q6. What is IS-A Relationship?

```text
Inheritance
```

Example:

```text
Dog IS-A Animal
```

---

## Q7. What is HAS-A Relationship?

```text
Composition
Aggregation
```

Example:

```text
House HAS-A Room
Department HAS-A Employee
```

---

# One-Minute Revision

```text
Class = Blueprint
Object = Instance

OOP Pillars
-----------------
Encapsulation -> Data Hiding
Inheritance   -> Code Reuse
Polymorphism  -> Many Forms
Abstraction   -> Hide Complexity

Inheritance
-----------------
extends
IS-A Relationship

HAS-A Relationship
-----------------
Composition
Aggregation

Overloading
-----------------
Same Method
Different Parameters
Compile Time

Overriding
-----------------
Same Signature
Runtime

Constructor
-----------------
Initialize Object

Constructor Chaining
-----------------
this()

Parent Constructor
-----------------
super()

this Keyword
-----------------
Current Object
Current Method
Current Constructor

Access Modifiers
-----------------
private
default
protected
public


Method Hiding
-------------
Static Method
Compile Time

Upcasting
-------------
Child → Parent
Automatic

Animal a = new Dog();

Downcasting
-------------
Parent → Child
Manual

Dog d = (Dog)a;

instanceof
-------------
Used for Safe Downcasting
```

# End of Java OOP Interview Revision Notes