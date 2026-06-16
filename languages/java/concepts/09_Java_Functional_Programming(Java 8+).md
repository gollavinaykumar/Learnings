# Java Functional Programming (Java 8+) - Complete Interview Revision Notes

> ⭐ VERY IMPORTANT FOR MODERN JAVA INTERVIEWS
>
> Java 8 introduced Functional Programming features that changed how Java code is written.
>
> Used heavily in:
>
> - Spring Boot
> - Microservices
> - Stream Processing
> - Data Transformation
> - Parallel Processing
>
> Topics Covered:
>
> - Functional Programming Basics
> - Lambda Expressions
> - Functional Interfaces
> - Streams API
> - Method References
> - Stream Operations
> - Real World Examples
> - Frequently Asked Interview Questions

---

# What is Functional Programming?

Functional Programming is a programming style where:

```text
Functions are treated as data.
```

Instead of:

```java
Write loops manually
```

we write:

```java
What To Do
```

rather than

```java
How To Do It
```

---

# Traditional Java Example

Find even numbers.

```java
List<Integer> nums =
        Arrays.asList(1,2,3,4,5,6);

List<Integer> result =
        new ArrayList<>();

for(Integer n : nums) {

    if(n % 2 == 0) {

        result.add(n);
    }
}

System.out.println(result);
```

Output:

```text
[2, 4, 6]
```

---

# Functional Style

```java
List<Integer> result =
        nums.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
```

Much cleaner.

---

# Lambda Expressions

Most Asked Java 8 Topic.

---

# What is Lambda Expression?

Lambda Expression is an anonymous function.

It has:

```text
No Name
No Return Type Declaration
No Access Modifier
```

---

# Syntax

```java
(parameters) -> expression
```

---

# Example

Traditional:

```java
public int add(int a,int b){

    return a+b;
}
```

Lambda:

```java
(a,b) -> a+b
```

---

# Simple Example

```java
Runnable r = () -> {

    System.out.println("Hello");
};

r.run();
```

Output:

```text
Hello
```

---

# Lambda Components

```java
(x,y) -> x+y
```

| Part | Meaning |
|--------|--------|
| (x,y) | Parameters |
| -> | Lambda Operator |
| x+y | Body |

---

# Real World Example

Sort employees.

Traditional:

```java
Collections.sort(
        employees,
        new Comparator<Employee>() {

            @Override
            public int compare(
                    Employee e1,
                    Employee e2) {

                return e1.getId()
                        - e2.getId();
            }
        });
```

---

Using Lambda:

```java
employees.sort(
        (e1,e2) ->
        e1.getId()-e2.getId());
```

Cleaner.

---

# Functional Interfaces

Very Important Interview Topic.

---

# Definition

Functional Interface contains:

```text
Exactly One Abstract Method
```

---

# Examples

```java
Runnable
Callable
Comparator
Consumer
Supplier
Predicate
Function
```

---

# Example

```java
@FunctionalInterface
interface Calculator {

    int add(int a,int b);
}
```

Lambda:

```java
Calculator c =
        (a,b) -> a+b;

System.out.println(
        c.add(10,20));
```

Output:

```text
30
```

---

# Why @FunctionalInterface?

Compiler verifies:

```text
Only One Abstract Method Exists
```

---

# Built-In Functional Interfaces

Most Asked Interview Topic.

---

# Predicate

Used for:

```text
Filtering
Condition Checking
```

Method:

```java
test()
```

---

Example

```java
Predicate<Integer> p =
        x -> x > 10;

System.out.println(
        p.test(15));
```

Output:

```text
true
```

---

# Function

Used for:

```text
Transformation
```

Method:

```java
apply()
```

---

Example

```java
Function<String,Integer> f =
        str -> str.length();

System.out.println(
        f.apply("Java"));
```

Output:

```text
4
```

---

# Consumer

Used for:

```text
Consuming Data
```

Method:

```java
accept()
```

---

Example

```java
Consumer<String> c =
        name -> System.out.println(name);

c.accept("Rahul");
```

Output:

```text
Rahul
```

---

# Supplier

Used for:

```text
Providing Data
```

Method:

```java
get()
```

---

Example

```java
Supplier<String> s =
        () -> "Java";

System.out.println(
        s.get());
```

Output:

```text
Java
```

---

# Streams API

Most Important Java 8 Feature.

---

# What is Stream?

A Stream is a sequence of data used for processing collections.

---

Without Stream

```java
for(Integer n : nums){

}
```

---

With Stream

```java
nums.stream()
```

---

# Stream Pipeline

```text
Source
   ↓
Intermediate Operations
   ↓
Terminal Operation
```

---

# Example

```java
nums.stream()
    .filter(n -> n > 10)
    .collect(Collectors.toList());
```

---

# Source

```java
nums
```

---

# Intermediate Operation

```java
filter()
```

---

# Terminal Operation

```java
collect()
```

---

# filter()

Used for filtering data.

---

Example

```java
List<Integer> nums =
        Arrays.asList(
                5,10,15,20,25);
```

```java
List<Integer> result =
        nums.stream()
            .filter(n -> n > 10)
            .collect(Collectors.toList());
```

Output:

```text
[15,20,25]
```

---

# map()

Used for transformation.

---

Example

```java
List<String> names =
        Arrays.asList(
                "java",
                "spring");
```

```java
List<String> result =
        names.stream()
             .map(String::toUpperCase)
             .collect(Collectors.toList());
```

Output:

```text
[JAVA, SPRING]
```

---

# sorted()

Used for sorting.

---

Example

```java
nums.stream()
    .sorted()
    .forEach(System.out::println);
```

---

# distinct()

Removes duplicates.

---

Example

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

Gets first N elements.

---

Example

```java
nums.stream()
    .limit(3)
    .forEach(System.out::println);
```

---

# count()

Returns total count.

---

Example

```java
long count =
        nums.stream()
            .count();
```

---

# findFirst()

Returns first element.

---

Example

```java
Optional<Integer> first =
        nums.stream()
            .findFirst();
```

---

# anyMatch()

Checks condition.

---

Example

```java
boolean result =
        nums.stream()
            .anyMatch(
                    n -> n > 100);
```

---

# reduce()

Used for aggregation.

---

Example

Sum all numbers.

```java
int sum =
        Arrays.asList(
                1,2,3,4,5)
                .stream()
                .reduce(
                        0,
                        Integer::sum);
```

Output:

```text
15
```

---

# collect()

Converts Stream into Collection.

---

Example

```java
List<Integer> result =
        nums.stream()
            .filter(
                    n -> n > 10)
            .collect(
                    Collectors.toList());
```

---

# Complete Stream Example

Question:

Get all even numbers greater than 10.

```java
List<Integer> nums =
        Arrays.asList(
                5,10,15,20,25,30);
```

```java
List<Integer> result =
        nums.stream()
            .filter(
                    n -> n > 10)
            .filter(
                    n -> n % 2 == 0)
            .collect(
                    Collectors.toList());

System.out.println(result);
```

Output:

```text
[20,30]
```

---

# Method References

Most Asked Java 8 Topic.

---

# What is Method Reference?

Shortcut for lambda expressions.

---

# Syntax

```java
ClassName::methodName
```

---

# Example

Lambda:

```java
names.forEach(
        name ->
        System.out.println(name));
```

Method Reference:

```java
names.forEach(
        System.out::println);
```

Same output.

---

# Types of Method References

## Static Method

```java
Math::abs
```

---

## Instance Method

```java
System.out::println
```

---

## Constructor Reference

```java
ArrayList::new
```

---

# Example

```java
Supplier<List<String>> s =
        ArrayList::new;
```

Creates:

```java
new ArrayList<>();
```

---

# Stream vs Collection

| Feature | Collection | Stream |
|----------|----------|----------|
| Stores Data | Yes | No |
| Processes Data | No | Yes |
| Reusable | Yes | No |
| Lazy Evaluation | No | Yes |

---

# Real World Use Cases

---

# Employee Filtering

```java
employees.stream()
         .filter(
            e -> e.getSalary() > 50000)
         .collect(Collectors.toList());
```

---

# Convert Names To Uppercase

```java
names.stream()
     .map(String::toUpperCase)
     .collect(Collectors.toList());
```

---

# Total Salary

```java
double total =
        employees.stream()
                 .map(Employee::getSalary)
                 .reduce(
                    0.0,
                    Double::sum);
```

---

# Sort Employees

```java
employees.stream()
         .sorted(
             Comparator.comparing(
                 Employee::getName))
         .collect(Collectors.toList());
```

---

# Frequently Asked Interview Questions

## Q1. What is Lambda Expression?

```text
Anonymous Function
```

Introduced in Java 8.

---

## Q2. What is Functional Interface?

```text
Interface With Exactly
One Abstract Method.
```

---

## Q3. Name Common Functional Interfaces.

```text
Predicate
Function
Consumer
Supplier
Runnable
Comparator
```

---

## Q4. What is Stream?

```text
Sequence Of Data
Used To Process Collections.
```

---

## Q5. Difference Between map() and filter()?

filter()

```text
Filters Data
```

map()

```text
Transforms Data
```

---

## Q6. What is Method Reference?

```text
Short Form Of Lambda
```

Example:

```java
System.out::println
```

---

## Q7. Are Streams Reusable?

```text
No
```

Once consumed, stream cannot be reused.

---

## Q8. What is Lazy Evaluation?

```text
Stream operations execute only
when terminal operation is called.
```

---

# One-Minute Revision

```text
Functional Programming
------------------
Functions As Data

Lambda
------------------
(a,b) -> a+b

Functional Interface
------------------
One Abstract Method

Common Interfaces
------------------
Predicate
Function
Consumer
Supplier

Streams
------------------
Process Collections

Common Operations
------------------
filter()
map()
sorted()
distinct()
limit()
count()
reduce()
collect()

Method Reference
------------------
System.out::println

Constructor Reference
------------------
ArrayList::new

Interview Favorites
------------------
Lambda
Functional Interface
Predicate
map vs filter
Stream Pipeline
Method References
```

# End of Java Functional Programming Notes