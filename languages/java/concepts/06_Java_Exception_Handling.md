# Java Exception Handling - Complete Interview Revision Notes

> Exception Handling is one of the most important Java interview topics.
>
> It helps programs handle runtime errors gracefully instead of crashing.
>
> Topics Covered:
>
> - What is Exception?
> - Exception Hierarchy
> - try
> - catch
> - finally
> - Checked Exceptions
> - Unchecked Exceptions
> - throw
> - throws
> - Custom Exceptions
> - Frequently Asked Interview Questions

---

# What is Exception?

An Exception is an unexpected event that occurs during program execution and disrupts the normal flow of the program.

Example:

```java
int a = 10;
int b = 0;

System.out.println(a / b);
```

Output:

```text
Exception in thread "main"
java.lang.ArithmeticException
```

Program crashes.

---

# Why Exception Handling?

Without Exception Handling:

```text
Program Terminates
Application Crashes
Poor User Experience
```

With Exception Handling:

```text
Program Continues
Error Managed Properly
Better Reliability
```

---

# Exception Hierarchy

```text
                Object
                   |
               Throwable
               /       \
          Error      Exception
                        |
            ----------------------
            |                    |
 RuntimeException      Checked Exceptions
```

---

# Example Exceptions

## Checked Exceptions

```text
IOException
SQLException
FileNotFoundException
ClassNotFoundException
```

---

## Unchecked Exceptions

```text
ArithmeticException
NullPointerException
ArrayIndexOutOfBoundsException
NumberFormatException
```

---

# try Block

## Definition

Code that may cause exception is written inside try block.

Example:

```java
try {

    int result = 10 / 0;

}
catch(Exception e) {

    System.out.println("Exception Occurred");
}
```

---

# Why try Block?

Without try:

```java
10 / 0
```

causes program termination.

With try:

```text
Exception is handled gracefully.
```

---

# catch Block

## Definition

Used to catch and handle exceptions.

Syntax:

```java
try {

}
catch(Exception e) {

}
```

---

# Example

```java
try {

    int result = 10 / 0;

}
catch(ArithmeticException e) {

    System.out.println("Cannot Divide By Zero");
}
```

Output:

```text
Cannot Divide By Zero
```

---

# Exception Object

```java
catch(Exception e)
```

Here:

```java
e
```

contains exception details.

---

# Useful Methods

## getMessage()

```java
System.out.println(e.getMessage());
```

Output:

```text
/ by zero
```

---

## printStackTrace()

```java
e.printStackTrace();
```

Displays complete error details.

---

# Multiple catch Blocks

Example:

```java
try {

    String str = null;

    System.out.println(str.length());

}
catch(ArithmeticException e) {

    System.out.println("Arithmetic Error");
}
catch(NullPointerException e) {

    System.out.println("Null Error");
}
```

Output:

```text
Null Error
```

---

# finally Block

## Definition

finally block executes whether exception occurs or not.

---

# Syntax

```java
try {

}
catch(Exception e) {

}
finally {

}
```

---

# Example

```java
try {

    int result = 10 / 2;

}
catch(Exception e) {

    System.out.println("Error");
}
finally {

    System.out.println("Finally Executed");
}
```

Output:

```text
Finally Executed
```

---

# Why finally?

Used for resource cleanup.

Examples:

```text
Close File
Close Database Connection
Close Network Connection
```

---

# Example

```java
FileInputStream file = null;

try {

    file =
    new FileInputStream("test.txt");

}
catch(Exception e) {

}
finally {

    System.out.println("Closing File");
}
```

---

# Execution Flow

## No Exception

```text
try
finally
```

---

## Exception Handled

```text
try
catch
finally
```

---

## Exception Not Handled

```text
try
finally
Program Terminates
```

---

# Checked Exceptions

Most Asked Interview Topic.

---

# Definition

Exceptions checked at compile time.

Compiler forces handling.

---

# Example

```java
FileInputStream file =
new FileInputStream("test.txt");
```

Compile Error:

```text
Unhandled exception
FileNotFoundException
```

---

# Correct Handling

```java
try {

    FileInputStream file =
    new FileInputStream("test.txt");

}
catch(FileNotFoundException e) {

    System.out.println("File Not Found");
}
```

---

# Examples

```text
IOException
SQLException
ClassNotFoundException
FileNotFoundException
```

---

# Why Called Checked?

Compiler checks them before execution.

---

# Unchecked Exceptions

## Definition

Exceptions checked at runtime.

Compiler does NOT force handling.

---

# Example

```java
int result = 10 / 0;
```

Output:

```text
ArithmeticException
```

---

# Examples

```text
ArithmeticException
NullPointerException
NumberFormatException
ArrayIndexOutOfBoundsException
```

---

# Example

```java
String str = null;

System.out.println(str.length());
```

Output:

```text
NullPointerException
```

---

# Checked vs Unchecked Exceptions

| Feature | Checked | Unchecked |
|----------|----------|----------|
| Checked By | Compiler | JVM |
| Time | Compile Time | Runtime |
| Handling Mandatory | Yes | No |
| Parent Class | Exception | RuntimeException |

---

# throw Keyword

## Definition

Used to explicitly create and throw an exception.

---

# Syntax

```java
throw new ExceptionType();
```

---

# Example

```java
public class Main {

    public static void main(String[] args) {

        int age = 15;

        if(age < 18) {

            throw new ArithmeticException(
                    "Not Eligible");
        }

        System.out.println("Eligible");
    }
}
```

Output:

```text
Exception in thread "main"
ArithmeticException: Not Eligible
```

---

# Why throw?

Used when business rules fail.

Examples:

```text
Invalid Age
Invalid Account
Invalid Amount
Invalid User
```

---

# Example

```java
double amount = -100;

if(amount < 0) {

    throw new IllegalArgumentException(
            "Amount Cannot Be Negative");
}
```

---

# throws Keyword

## Definition

Used to declare exceptions.

It tells caller:

```text
This method may throw exception.
```

---

# Syntax

```java
returnType method()
throws ExceptionType
```

---

# Example

```java
import java.io.*;

public class Main {

    static void readFile()
    throws IOException {

        FileInputStream file =
        new FileInputStream("test.txt");
    }

    public static void main(String[] args) {

    }
}
```

---

# Why throws?

Instead of handling exception inside method,
responsibility is transferred to caller.

---

# Example

```java
static void readFile()
throws IOException {

}
```

Meaning:

```text
Caller Must Handle Exception.
```

---

# throw vs throws

Most Asked Interview Question.

| Feature | throw | throws |
|----------|----------|----------|
| Purpose | Throw Exception | Declare Exception |
| Used Inside Method | Yes | No |
| Followed By | Object | Class Name |
| Count | One Exception | Multiple Exceptions |

---

# Example

## throw

```java
throw new ArithmeticException();
```

---

## throws

```java
void read()
throws IOException
```

---

# Custom Exception

Very Important For Interviews.

---

# What is Custom Exception?

User-defined exception created by programmer.

Used when built-in exceptions are insufficient.

---

# Example Scenario

Bank Application:

```text
Minimum Balance Required
```

If balance less than minimum:

```text
Custom Exception
```

---

# Step 1 Create Exception Class

```java
class InsufficientBalanceException
extends Exception {

    public InsufficientBalanceException(
            String message) {

        super(message);
    }
}
```

---

# Step 2 Throw Exception

```java
class Bank {

    void withdraw(double amount)
    throws InsufficientBalanceException {

        if(amount > 5000) {

            throw new
            InsufficientBalanceException(
                    "Insufficient Balance");
        }
    }
}
```

---

# Step 3 Handle Exception

```java
public class Main {

    public static void main(String[] args) {

        Bank bank = new Bank();

        try {

            bank.withdraw(10000);

        }
        catch(
        InsufficientBalanceException e) {

            System.out.println(
                    e.getMessage());
        }
    }
}
```

Output:

```text
Insufficient Balance
```

---

# Custom Runtime Exception

Example:

```java
class InvalidAgeException
extends RuntimeException {

    public InvalidAgeException(
            String message) {

        super(message);
    }
}
```

---

# Best Practices

## Catch Specific Exceptions

Good:

```java
catch(FileNotFoundException e)
```

Bad:

```java
catch(Exception e)
```

---

## Never Leave Catch Empty

Bad:

```java
catch(Exception e) {

}
```

---

## Use finally For Cleanup

```java
finally {

    connection.close();
}
```

---

## Create Meaningful Custom Exceptions

Good:

```java
InvalidAgeException
```

Bad:

```java
MyException
```

---

# Frequently Asked Interview Questions

## Q1. What is Exception?

```text
Unexpected event that disrupts
normal program execution.
```

---

## Q2. Difference Between Error and Exception?

Error:

```text
Serious Problem
Usually Cannot Recover
```

Examples:

```text
OutOfMemoryError
StackOverflowError
```

Exception:

```text
Can Be Handled
```

---

## Q3. Difference Between Checked and Unchecked Exception?

Checked:

```text
Compile Time
Mandatory Handling
```

Unchecked:

```text
Runtime
Optional Handling
```

---

## Q4. Difference Between throw and throws?

throw:

```text
Used To Throw Exception
```

throws:

```text
Used To Declare Exception
```

---

## Q5. Does finally Always Execute?

```text
Yes
```

Except:

```text
System.exit()
JVM Crash
Power Failure
```

---

## Q6. Can We Have try Without catch?

Yes.

```java
try {

}
finally {

}
```

Valid.

---

## Q7. Can We Have Multiple catch Blocks?

Yes.

```java
catch(IOException e)

catch(SQLException e)
```

---

## Q8. Why Use Custom Exceptions?

```text
To represent business-specific errors.
```

Example:

```text
InvalidAgeException
InsufficientBalanceException
```

---

# One-Minute Revision

```text
Exception
-------------
Runtime Error

try
-------------
Risky Code

catch
-------------
Handle Exception

finally
-------------
Always Executes

Checked Exception
-------------
Compile Time
Mandatory Handling

Examples:
IOException
SQLException

Unchecked Exception
-------------
Runtime

Examples:
NullPointerException
ArithmeticException

throw
-------------
Create & Throw Exception

throws
-------------
Declare Exception

Custom Exception
-------------
User Defined Exception

Interview Favorite
-------------
Checked vs Unchecked

throw vs throws

finally Block

Custom Exception
```

# End of Java Exception Handling Notes