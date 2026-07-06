# 👑 C Mastery Roadmap

# Module 4 --- Arrays & Strings

# Chapter 24 --- Strings

------------------------------------------------------------------------

## Most beginners think:

> "A string is a special data type."

In C, there is **no built-in string type**.

A C string is simply a **character array that ends with a null character
(`'\0'`)**.

That single byte tells every string function where the text ends.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a C string is
-   Why `'\0'` exists
-   String literals
-   Declaring and initializing strings
-   Memory layout
-   Reading and printing strings
-   Modifying strings
-   Common mistakes

------------------------------------------------------------------------

# What is a String?

``` c
char name[] = "Vinay";
```

Internally this becomes:

``` text
Index : 0   1   2   3   4   5

Value : V   i   n   a   y  \0
```

The compiler automatically appends the null terminator.

------------------------------------------------------------------------

# Why is `'\0'` Needed?

Imagine memory:

``` text
V i n a y \0 X Y Z ...
```

Functions like `printf("%s", name)` keep reading characters until they
find:

``` text
\0
```

Without it, they would continue reading unrelated memory.

------------------------------------------------------------------------

# String Literal

``` c
char city[] = "Delhi";
```

Equivalent to:

``` c
char city[] = {'D','e','l','h','i','\0'};
```

------------------------------------------------------------------------

# Memory Layout

``` text
Address    Value

0x1000     H
0x1001     e
0x1002     l
0x1003     l
0x1004     o
0x1005     \0
```

Each character occupies one byte.

------------------------------------------------------------------------

# Printing Strings

``` c
char name[] = "Alice";

printf("%s\n", name);
```

`%s` tells `printf()` to print characters until `'\0'`.

------------------------------------------------------------------------

# Reading Strings

Safe approach:

``` c
char name[30];

fgets(name, sizeof(name), stdin);
```

Avoid reading beyond the array size.

------------------------------------------------------------------------

# Modifying Strings

Character arrays can be modified.

``` c
char word[] = "Hello";

word[0] = 'Y';
```

Now:

``` text
Yello
```

------------------------------------------------------------------------

# String Literal vs Character Array

``` c
char a[] = "Hello";
```

Modifiable.

``` c
char *b = "Hello";
```

Points to a string literal.

Attempting to modify it results in undefined behavior.

------------------------------------------------------------------------

# Common Mistakes

## Missing Space for `'\0'`

Wrong:

``` c
char s[5] = "Hello";
```

Correct:

``` c
char s[6] = "Hello";
```

------------------------------------------------------------------------

## Forgetting the Null Terminator

Manually creating a character array without `'\0'` means string
functions cannot determine where the text ends.

------------------------------------------------------------------------

# Real-World Uses

Strings are used for:

-   User names
-   Passwords
-   URLs
-   File paths
-   Commands
-   JSON
-   HTTP requests
-   Network packets

------------------------------------------------------------------------

# Hands-on Labs

1.  Create and print a string.
2.  Print each character using a loop.
3.  Modify one character.
4.  Read a line using `fgets()`.
5.  Print the ASCII value of every character.

------------------------------------------------------------------------

# Interview Questions

### What is a C string?

A character array terminated by `'\0'`.

### Why is the null terminator important?

It marks the end of the string for library functions.

### What format specifier prints a string?

`%s`

### Why is `fgets()` preferred?

Because it limits the number of characters read and helps prevent buffer
overflow.

------------------------------------------------------------------------

# Summary

``` text
Characters
     │
     ▼
Character Array
     │
     ▼
Null Terminator
     │
     ▼
C String
     │
     ▼
String Functions
```

Key Takeaways:

-   C strings are arrays of characters.
-   Every valid C string ends with `'\0'`.
-   `%s` prints until the null terminator.
-   `fgets()` is the preferred way to read strings safely.
-   Understanding string layout is essential before learning string
    library functions.

------------------------------------------------------------------------

# Next Chapter

## Chapter 25 --- String Library Functions

You'll learn:

-   `strlen()`
-   `strcpy()`
-   `strncpy()`
-   `strcmp()`
-   `strncmp()`
-   `strcat()`
-   `strncat()`
-   `strchr()`
-   `strstr()`
