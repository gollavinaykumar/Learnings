# 👑 C Mastery Roadmap

# Module 4 --- Arrays & Strings

# Chapter 25 --- String Library Functions

------------------------------------------------------------------------

## Most beginners think:

> "Strings are manipulated using operators like numbers."

In C, strings are arrays of characters.

Because arrays cannot be assigned or compared directly, C provides the
**Standard String Library** in `<string.h>`.

These functions power everything from command-line tools to web servers.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Why `<string.h>` exists
-   `strlen()`
-   `strcpy()` and `strncpy()`
-   `strcmp()` and `strncmp()`
-   `strcat()` and `strncat()`
-   `strchr()`
-   `strstr()`
-   Common mistakes

------------------------------------------------------------------------

# Include the Library

``` c
#include <string.h>
```

------------------------------------------------------------------------

# `strlen()` --- String Length

Returns the number of characters **before** `'\0'`.

``` c
char name[] = "Linux";

printf("%zu\n", strlen(name));
```

Output:

``` text
5
```

Memory:

``` text
L i n u x \0
```

The null terminator is **not** counted.

------------------------------------------------------------------------

# `strcpy()` --- Copy a String

``` c
char src[] = "Kernel";
char dest[20];

strcpy(dest, src);
```

After copying:

``` text
dest → "Kernel"
```

⚠️ Ensure the destination array is large enough.

------------------------------------------------------------------------

# `strncpy()`

Copies at most `n` characters.

``` c
strncpy(dest, src, 5);
```

Useful for limiting copied data, but be aware it may not append `'\0'`
if the source is longer than `n`.

------------------------------------------------------------------------

# `strcmp()` --- Compare Strings

``` c
strcmp("abc", "abc");
```

Returns:

``` text
0
```

Meaning:

The strings are equal.

General rules:

-   `< 0` → first string comes before the second
-   `0` → equal
-   `> 0` → first string comes after the second

------------------------------------------------------------------------

# `strncmp()`

Compare only the first `n` characters.

``` c
strncmp("network", "net", 3);
```

Result:

``` text
0
```

The first three characters match.

------------------------------------------------------------------------

# `strcat()` --- Concatenate

``` c
char name[20] = "Hello ";

strcat(name, "World");
```

Result:

``` text
Hello World
```

The destination array must have enough free space.

------------------------------------------------------------------------

# `strncat()`

Append at most `n` characters.

``` c
strncat(name, "Linux", 3);
```

Result:

``` text
Hello WorLin
```

------------------------------------------------------------------------

# `strchr()` --- Find a Character

``` c
char *p = strchr("kernel", 'n');
```

`p` points to:

``` text
nel
```

If the character is not found, it returns `NULL`.

------------------------------------------------------------------------

# `strstr()` --- Find a Substring

``` c
char *p = strstr("OperatingSystem", "System");
```

`p` points to:

``` text
System
```

Useful for searching text.

------------------------------------------------------------------------

# Real-World Uses

These functions are used for:

-   HTTP headers
-   URLs
-   Command parsing
-   Configuration files
-   File paths
-   Log processing
-   Network protocols

------------------------------------------------------------------------

# Common Mistakes

## Comparing with `==`

Wrong:

``` c
if (a == b)
```

Correct:

``` c
if (strcmp(a, b) == 0)
```

------------------------------------------------------------------------

## Buffer Overflow

``` c
char dest[5];

strcpy(dest, "OperatingSystem");
```

Undefined behavior.

Always ensure enough space.

------------------------------------------------------------------------

# Hands-on Labs

1.  Measure string lengths using `strlen()`.
2.  Copy strings with `strcpy()` and `strncpy()`.
3.  Compare user input with `strcmp()`.
4.  Join two strings with `strcat()`.
5.  Search for characters using `strchr()`.

------------------------------------------------------------------------

# Interview Questions

### Which header provides string functions?

`<string.h>`

### Does `strlen()` count `'\0'`?

No.

### Why is `strcmp()` used instead of `==`?

Because `==` compares addresses, not string contents.

### What does `strstr()` return?

A pointer to the first occurrence of a substring, or `NULL` if not
found.

------------------------------------------------------------------------

# Summary

``` text
String
   │
   ▼
<string.h>
   │
   ├── strlen()
   ├── strcpy()
   ├── strcmp()
   ├── strcat()
   ├── strchr()
   └── strstr()
```

Key Takeaways:

-   `<string.h>` provides standard string manipulation functions.
-   Always ensure destination buffers are large enough.
-   Use `strcmp()` for content comparison.
-   String functions rely on the null terminator.

------------------------------------------------------------------------

# Next Chapter

## Chapter 26 --- Memory Representation

You'll learn how arrays and strings are actually laid out in RAM, byte
by byte, preparing you for pointers and memory management.
