# 👑 C Mastery Roadmap

# Module 4 --- Arrays & Strings

# Chapter 23 --- Character Arrays

------------------------------------------------------------------------

## Most beginners think:

> "A character is just a letter."

To a computer, a character is simply a **number**.

The letter `'A'` is stored as **65** (ASCII).

A character array is therefore just an array of numbers that represent
text.

Understanding this is the foundation for learning strings in C.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a character is
-   ASCII encoding
-   Character arrays
-   Declaring and initializing character arrays
-   Memory layout
-   Null terminator preview
-   Character array vs integer array
-   Common mistakes

------------------------------------------------------------------------

# What is a Character?

A `char` stores one byte.

``` c
char grade = 'A';
```

Internally:

``` text
Character    ASCII Value

'A'          65
'B'          66
'C'          67
```

------------------------------------------------------------------------

# What is a Character Array?

A character array stores multiple characters.

``` c
char name[5];
```

Each element stores one character.

``` text
Index

0   1   2   3   4

Value

H   e   l   l   o
```

------------------------------------------------------------------------

# Declaring a Character Array

``` c
char word[6];
```

Space is reserved for six characters.

------------------------------------------------------------------------

# Initializing

``` c
char word[6] = {'H','e','l','l','o','\0'};
```

Each character occupies one byte.

------------------------------------------------------------------------

# Memory Layout

``` text
Address     Value

0x1000      H
0x1001      e
0x1002      l
0x1003      l
0x1004      o
0x1005      \0
```

Characters are stored contiguously just like integers.

------------------------------------------------------------------------

# Character Array vs Integer Array

``` c
char text[5];

int numbers[5];
```

  Character Array     Integer Array
  ------------------- -------------------------
  1 byte/element      Usually 4 bytes/element
  Stores characters   Stores integers

------------------------------------------------------------------------

# Accessing Characters

``` c
char word[] = "Hello";

printf("%c\n", word[0]);
printf("%c\n", word[4]);
```

Output:

``` text
H
o
```

------------------------------------------------------------------------

# Traversing a Character Array

``` c
char word[] = "Hello";

for(int i=0;i<5;i++)
{
    printf("%c ", word[i]);
}
```

Output:

``` text
H e l l o
```

------------------------------------------------------------------------

# The Null Terminator

C strings end with a special character:

``` text
'\0'
```

It marks the end of the text.

Without it, functions cannot determine where the string ends.

We'll study this in detail in the next chapter.

------------------------------------------------------------------------

# Real-World Uses

Character arrays are used for:

-   Names
-   File paths
-   URLs
-   Network messages
-   Commands
-   Passwords
-   Configuration values

------------------------------------------------------------------------

# Common Mistakes

## Forgetting Space for `'\0'`

Wrong:

``` c
char name[5] = "Hello";
```

Needs six bytes.

Correct:

``` c
char name[6] = "Hello";
```

------------------------------------------------------------------------

## Treating Characters as Strings

Wrong:

``` c
char c = "A";
```

Correct:

``` c
char c = 'A';
```

Single quotes are for one character.

Double quotes create a string.

------------------------------------------------------------------------

# Hands-on Labs

1.  Create a character array manually.
2.  Print each character using a loop.
3.  Print ASCII values using `%d`.
4.  Modify one character.
5.  Compare memory usage of `char` and `int` arrays with `sizeof`.

------------------------------------------------------------------------

# Interview Questions

### What is a character array?

An array whose elements are of type `char`.

### How many bytes does a `char` usually occupy?

One byte.

### What does `'\0'` represent?

The null terminator marking the end of a C string.

### Why are character arrays stored contiguously?

Because arrays always occupy consecutive memory locations.

------------------------------------------------------------------------

# Summary

``` text
Character
     │
     ▼
ASCII Number
     │
     ▼
Character Array
     │
     ▼
Contiguous Memory
     │
     ▼
Text Representation
```

Key Takeaways:

-   Characters are stored as numeric codes.
-   Character arrays store text one byte at a time.
-   Arrays remain contiguous in memory.
-   C strings rely on the null terminator.
-   Understanding character arrays prepares you for string manipulation.

------------------------------------------------------------------------

# Next Chapter

## Chapter 24 --- Strings

You'll learn how C strings work internally, why they end with `'\0'`,
and how the standard library manipulates text.
