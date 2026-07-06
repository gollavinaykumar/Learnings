# 👑 C Mastery Roadmap

# Module 1 --- C Fundamentals

# Chapter 8 --- Input & Output

------------------------------------------------------------------------

## Most beginners think:

> "A program just prints something on the screen."

But a useful program must communicate with the outside world.

It must:

-   Read user input
-   Display results
-   Read files
-   Write files
-   Communicate with devices

The C Standard Library provides functions for this communication.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What Input and Output (I/O) is
-   Standard streams
-   `printf()`
-   `scanf()`
-   `getchar()`
-   `putchar()`
-   `puts()`
-   `fgets()`
-   Format specifiers
-   Safe user input
-   Common mistakes

------------------------------------------------------------------------

# What is I/O?

I/O stands for **Input / Output**.

``` text
Keyboard
    │
    ▼
 Program
    │
    ▼
Monitor
```

Input enters the program.

Output leaves the program.

------------------------------------------------------------------------

# Standard Streams

Every C program starts with three standard streams.

``` text
stdin   → Keyboard (Input)

stdout  → Screen (Normal Output)

stderr  → Error Messages
```

These streams are managed by the operating system.

------------------------------------------------------------------------

# printf()

`printf()` prints formatted output to `stdout`.

``` c
#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");
}
```

Output:

``` text
Hello, World!
```

------------------------------------------------------------------------

# Format Specifiers

``` c
int age = 25;
float price = 99.5f;
char grade = 'A';

printf("%d\n", age);
printf("%.2f\n", price);
printf("%c\n", grade);
```

Common specifiers:

  Specifier   Meaning
  ----------- --------------
  `%d`        int
  `%u`        unsigned int
  `%c`        char
  `%f`        float/double
  `%s`        string
  `%p`        pointer
  `%x`        hexadecimal

------------------------------------------------------------------------

# scanf()

`scanf()` reads formatted input from `stdin`.

``` c
int age;

scanf("%d", &age);

printf("%d\n", age);
```

Notice the `&`.

`scanf()` needs the **address** where it should store the value.

------------------------------------------------------------------------

# Why & is Required

``` text
Variable

age

Address

0x1000

scanf()

      │

Stores value at

0x1000
```

We'll learn pointers in detail later, but for now remember:

`&` means **address of**.

------------------------------------------------------------------------

# getchar()

Reads one character.

``` c
char ch = getchar();

printf("%c\n", ch);
```

Useful for simple keyboard input.

------------------------------------------------------------------------

# putchar()

Prints one character.

``` c
putchar('A');
```

Output:

``` text
A
```

------------------------------------------------------------------------

# puts()

Prints an entire string followed by a newline.

``` c
puts("Welcome to C");
```

Equivalent to:

``` c
printf("Welcome to C\n");
```

------------------------------------------------------------------------

# fgets()

`fgets()` safely reads an entire line.

``` c
char name[50];

fgets(name, sizeof(name), stdin);
```

Unlike older functions, `fgets()` helps prevent buffer overflow.

It is generally preferred over unsafe input functions.

------------------------------------------------------------------------

# Example Program

``` c
#include <stdio.h>

int main(void)
{
    char name[30];
    int age;

    printf("Enter your name: ");
    fgets(name, sizeof(name), stdin);

    printf("Enter your age: ");
    scanf("%d", &age);

    printf("\nHello %s", name);
    printf("Age: %d\n", age);

    return 0;
}
```

------------------------------------------------------------------------

# Common Mistakes

## Forgetting '&'

Wrong:

``` c
scanf("%d", age);
```

Correct:

``` c
scanf("%d", &age);
```

------------------------------------------------------------------------

## Buffer Overflow

Avoid reading more characters than your array can hold.

Use:

``` c
fgets()
```

instead of unsafe approaches.

------------------------------------------------------------------------

## Wrong Format Specifier

Wrong:

``` c
float price;

printf("%d", price);
```

Correct:

``` c
printf("%f", price);
```

------------------------------------------------------------------------

# Hands-on Labs

## Lab 1

Read two integers using `scanf()`.

Print their sum.

------------------------------------------------------------------------

## Lab 2

Read one character using `getchar()`.

Print it using `putchar()`.

------------------------------------------------------------------------

## Lab 3

Read a full name using `fgets()`.

Display it using `puts()`.

------------------------------------------------------------------------

## Lab 4

Print different data types using the correct format specifiers.

------------------------------------------------------------------------

# Interview Questions

### What is `printf()`?

A function that prints formatted output to `stdout`.

### Why is `&` used with `scanf()`?

Because `scanf()` needs the memory address where it should store the
input.

### What is the difference between `getchar()` and `putchar()`?

`getchar()` reads one character.

`putchar()` writes one character.

### Why is `fgets()` preferred?

Because it limits the number of characters read and helps prevent buffer
overflow.

### What are the three standard streams?

-   `stdin`
-   `stdout`
-   `stderr`

------------------------------------------------------------------------

# Summary

``` text
Keyboard
   │
   ▼
 stdin
   │
   ▼
 Program
   │
   ├────────► stdout ───────► Screen
   │
   └────────► stderr ───────► Error Output
```

Key Takeaways:

-   I/O allows programs to communicate with users.
-   `printf()` displays formatted output.
-   `scanf()` reads formatted input.
-   `getchar()` and `putchar()` handle single characters.
-   `puts()` prints strings.
-   `fgets()` is the preferred way to read lines safely.
-   Always use the correct format specifier.

------------------------------------------------------------------------

# Module 1 Complete ✅

You have completed the C Fundamentals module.

Next you'll begin:

# Module 2 --- Control Flow

You'll learn:

-   if
-   else
-   switch
-   for
-   while
-   do-while
-   break
-   continue
-   goto
-   recursion
