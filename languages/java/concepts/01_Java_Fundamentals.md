# Java Fundamentals - Interview Revision Notes

> Quick and simple revision guide for Java interviews.
> Covers basics, JVM/JRE/JDK, variables, data types, operators, type casting,
> control flow, compilation & execution, bytecode, and garbage collection.

---

# 1. What is Java?

Java is a **high-level, object-oriented programming language** developed by Sun Microsystems (now Oracle).

### Why Java is popular?
- Platform Independent ("Write Once, Run Anywhere")
- Secure
- Robust
- Object-Oriented
- Automatic Memory Management (Garbage Collection)

Example:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

Output:

```
Hello Java
```

---

# 2. JDK, JRE, JVM

This is one of the most common interview questions.

## Simple Analogy

Imagine you want to watch a movie.

- Movie = Java Program
- DVD Player = JVM
- Home Theater Setup = JRE
- Movie Studio + Editing Tools + DVD Player = JDK

---

## JVM (Java Virtual Machine)

JVM is the engine that runs Java programs.

Responsibilities:
- Executes bytecode
- Manages memory
- Performs garbage collection

Diagram:

```
Java Code
    ↓
Bytecode
    ↓
JVM
    ↓
Machine Code
```

Example:

When you run:

```java
java Main
```

JVM loads and executes Main.class.

---

## JRE (Java Runtime Environment)

JRE provides everything needed to run Java applications.

Contains:
- JVM
- Libraries
- Runtime files

Use JRE when:
- You only want to run Java applications.

```
JRE = JVM + Libraries
```

---

## JDK (Java Development Kit)

JDK is used to develop Java applications.

Contains:
- JRE
- Compiler (javac)
- Debugging tools
- Development tools

Use JDK when:
- Writing code
- Compiling code
- Running code

```
JDK = JRE + Development Tools
```

---

## Difference Table

| Feature | JVM | JRE | JDK |
|----------|----------|----------|----------|
| Runs Java Program | Yes | Yes | Yes |
| Contains JVM | No | Yes | Yes |
| Contains Compiler | No | No | Yes |
| Used for Development | No | No | Yes |

Interview Answer:

```
JVM executes Java bytecode.
JRE provides runtime environment.
JDK provides tools for development and contains JRE.
```

---

# 3. Variables

A variable is a container that stores data.

Example:

```java
int age = 25;
```

Here:

- int → data type
- age → variable name
- 25 → value

---

## Rules for Variable Names

Valid:

```java
int age;
int studentAge;
int _count;
int $price;
```

Invalid:

```java
int 1age;
int class;
```

---

# 4. Data Types

Data types specify what kind of value a variable can store.

---

## Primitive Data Types

Java has 8 primitive data types.

| Type | Size | Example |
|--------|--------|--------|
| byte | 1 byte | byte a = 10; |
| short | 2 bytes | short b = 100; |
| int | 4 bytes | int c = 1000; |
| long | 8 bytes | long d = 10000L; |
| float | 4 bytes | float e = 10.5f; |
| double | 8 bytes | double f = 10.5; |
| char | 2 bytes | char g = 'A'; |
| boolean | JVM dependent | boolean h = true; |

Example:

```java
int age = 22;
double salary = 45000.75;
char grade = 'A';
boolean passed = true;
```

---

## Non-Primitive Data Types

Examples:

```java
String name = "John";
int[] numbers = {1,2,3};
```

Types:
- String
- Arrays
- Classes
- Interfaces

---

# 5. Operators

Operators perform operations on values.

---

## Arithmetic Operators

| Operator | Meaning |
|-----------|-----------|
| + | Addition |
| - | Subtraction |
| * | Multiplication |
| / | Division |
| % | Modulus |

Example:

```java
int a = 10;
int b = 3;

System.out.println(a + b); //13
System.out.println(a - b); //7
System.out.println(a * b); //30
System.out.println(a / b); //3
System.out.println(a % b); //1
```

---

## Relational Operators

Used for comparison.

| Operator | Meaning |
|-----------|-----------|
| == | Equal |
| != | Not Equal |
| > | Greater Than |
| < | Less Than |
| >= | Greater or Equal |
| <= | Less or Equal |

Example:

```java
int age = 20;

System.out.println(age > 18);
```

Output:

```
true
```

---

## Logical Operators

| Operator | Meaning |
|-----------|-----------|
| && | AND |
| \|\| | OR |
| ! | NOT |

Example:

```java
int age = 20;
boolean citizen = true;

System.out.println(age >= 18 && citizen);
```

Output:

```
true
```

---

## Assignment Operators

Example:

```java
int x = 10;

x += 5;
```

Equivalent to:

```java
x = x + 5;
```

Other operators:

```java
x -= 2;
x *= 3;
x /= 2;
```

---

## Increment and Decrement

```java
int a = 5;

a++;
a--;
```

Equivalent:

```java
a = a + 1;
a = a - 1;
```

---

# 6. Type Casting

Type casting means converting one data type into another.

---

## Implicit Type Casting (Widening)

Small → Large

Automatic conversion.

```java
int a = 10;

double b = a;

System.out.println(b);
```

Output:

```
10.0
```

Why safe?

Because double can store larger values than int.

---

## Explicit Type Casting (Narrowing)

Large → Small

Manual conversion needed.

```java
double price = 99.99;

int amount = (int) price;

System.out.println(amount);
```

Output:

```
99
```

Decimal part is lost.

---

## Interview Tip

```
Widening → Automatic
Narrowing → Manual
```

---

# 7. Control Flow Statements

Control flow decides the order of execution.

---

# if Statement

```java
int age = 20;

if(age >= 18){
    System.out.println("Eligible");
}
```

Output:

```
Eligible
```

---

# if-else

```java
int age = 16;

if(age >= 18){
    System.out.println("Adult");
}
else{
    System.out.println("Minor");
}
```

Output:

```
Minor
```

---

# else-if Ladder

```java
int marks = 85;

if(marks >= 90){
    System.out.println("A");
}
else if(marks >= 75){
    System.out.println("B");
}
else{
    System.out.println("C");
}
```

Output:

```
B
```

---

# switch Statement

Used when checking many values.

```java
int day = 2;

switch(day){
    case 1:
        System.out.println("Monday");
        break;

    case 2:
        System.out.println("Tuesday");
        break;

    default:
        System.out.println("Invalid");
}
```

Output:

```
Tuesday
```

---

# Loops

Used to repeat code.

---

## for Loop

```java
for(int i=1; i<=5; i++){
    System.out.println(i);
}
```

Output:

```
1
2
3
4
5
```

Use when number of iterations is known.

---

## while Loop

```java
int i = 1;

while(i <= 5){
    System.out.println(i);
    i++;
}
```

Use when condition is known.

---

## do-while Loop

Runs at least once.

```java
int i = 1;

do{
    System.out.println(i);
    i++;
}
while(i <= 5);
```

---

## break Statement

Stops loop immediately.

```java
for(int i=1; i<=10; i++){

    if(i==5)
        break;

    System.out.println(i);
}
```

Output:

```
1
2
3
4
```

---

## continue Statement

Skips current iteration.

```java
for(int i=1; i<=5; i++){

    if(i==3)
        continue;

    System.out.println(i);
}
```

Output:

```
1
2
4
5
```

---

# 8. Compilation vs Execution

Very important interview topic.

---

## Step 1: Write Source Code

File:

```
Main.java
```

Example:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

---

## Step 2: Compilation

Command:

```bash
javac Main.java
```

Compiler:

```
javac
```

Output:

```
Main.class
```

The compiler converts Java code into bytecode.

---

## Step 3: Execution

Command:

```bash
java Main
```

JVM loads bytecode and executes it.

Output:

```
Hello
```

---

## Flow

```
Main.java
   ↓
javac
   ↓
Main.class (Bytecode)
   ↓
JVM
   ↓
Machine Code
   ↓
Output
```

---

# 9. Bytecode

Bytecode is the intermediate code generated by Java compiler.

Example:

```bash
javac Main.java
```

Produces:

```bash
Main.class
```

This .class file contains bytecode.

---

## Why Bytecode?

Because it makes Java platform independent.

Same bytecode can run on:

- Windows
- Linux
- Mac

As long as JVM exists.

Interview Answer:

```
Bytecode is platform-independent code generated by the Java compiler and executed by JVM.
```

---

# 10. Garbage Collection (GC)

Java automatically removes unused objects from memory.

This process is called Garbage Collection.

---

## Why Needed?

Without GC:

```
Memory keeps increasing
Application becomes slow
May crash
```

GC frees unused memory automatically.

---

## Example

```java
class Student {
}

public class Main {

    public static void main(String[] args) {

        Student s = new Student();

        s = null;
    }
}
```

After:

```java
s = null;
```

The object has no reference.

GC may remove it later.

---

## Important Points

- Automatic memory management
- Managed by JVM
- Removes unused objects
- Improves memory usage

---

## Can We Force Garbage Collection?

We can request:

```java
System.gc();
```

But JVM may ignore it.

So GC is:

```
Requested, not guaranteed.
```

---

# Frequently Asked Interview Questions

### Q1. Why is Java platform independent?

Because Java code is converted into bytecode, and bytecode runs on any JVM.

---

### Q2. Difference between JDK, JRE, and JVM?

```
JDK = JRE + Development Tools
JRE = JVM + Libraries
JVM = Executes Bytecode
```

---

### Q3. What is Bytecode?

Platform-independent code generated by the Java compiler.

---

### Q4. What is Garbage Collection?

Automatic process of removing unused objects from memory.

---

### Q5. Difference between Compilation and Execution?

Compilation:
```
.java → .class
```

Execution:
```
.class → Output
```

---

# One-Minute Revision

```
Java Source Code (.java)
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

JDK = JRE + Tools
JRE = JVM + Libraries

Primitive Types:
byte, short, int, long,
float, double, char, boolean

Type Casting:
Widening = Automatic
Narrowing = Manual

Control Flow:
if, else, switch
for, while, do-while

GC:
Removes unused objects automatically.
```

# End of Java Fundamentals Revision Notes