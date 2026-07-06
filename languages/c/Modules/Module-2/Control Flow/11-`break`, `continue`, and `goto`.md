# 👑 C Mastery Roadmap

# Module 2 --- Control Flow

# Chapter 11 --- `break`, `continue`, and `goto`

------------------------------------------------------------------------

## Most beginners think:

> "Once a loop starts, it must finish naturally."

But real programs often need to change the normal flow.

Examples:

-   Stop searching after finding a result.
-   Skip invalid input.
-   Exit nested logic immediately.

C provides special control statements for these situations.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What `break` does
-   What `continue` does
-   What `goto` does
-   Differences between them
-   When to use each
-   Common mistakes

------------------------------------------------------------------------

# Why Flow Control Matters

``` text
Loop
 │
 ▼
Condition
 │
 ▼
Body
 │
 ├── break
 ├── continue
 └── goto
```

These statements change the normal execution path.

------------------------------------------------------------------------

# `break`

`break` immediately exits the nearest loop or `switch`.

``` c
for (int i = 1; i <= 10; i++)
{
    if (i == 5)
        break;

    printf("%d\n", i);
}
```

Output:

``` text
1
2
3
4
```

Execution stops when `i` becomes 5.

------------------------------------------------------------------------

# Real Example

Searching an array:

``` c
for (int i = 0; i < n; i++)
{
    if (arr[i] == target)
    {
        printf("Found!\n");
        break;
    }
}
```

Once the value is found, there is no need to continue searching.

------------------------------------------------------------------------

# `continue`

`continue` skips the remaining statements in the current iteration and
starts the next iteration.

``` c
for (int i = 1; i <= 5; i++)
{
    if (i == 3)
        continue;

    printf("%d\n", i);
}
```

Output:

``` text
1
2
4
5
```

The iteration where `i == 3` is skipped.

------------------------------------------------------------------------

# `break` vs `continue`

  `break`      `continue`
  ------------ -------------------------
  Exits loop   Skips current iteration
  Loop ends    Loop continues

------------------------------------------------------------------------

# `goto`

`goto` jumps to a labeled statement.

``` c
#include <stdio.h>

int main(void)
{
    printf("Start\n");

    goto end;

    printf("This never runs\n");

end:
    printf("Finished\n");
}
```

Output:

``` text
Start
Finished
```

------------------------------------------------------------------------

# Why `goto` Exists

Some low-level code uses `goto` for cleanup.

Example:

``` c
FILE *fp = fopen("data.txt", "r");

if (!fp)
    goto error;

/* Work with file */

fclose(fp);
return 0;

error:
printf("Failed to open file\n");
return 1;
```

The Linux kernel also uses carefully structured `goto` statements for
resource cleanup.

------------------------------------------------------------------------

# Why Overusing `goto` Is Dangerous

Too many jumps make programs difficult to understand.

``` text
Start
 │
 ▼
goto A
 │
 ▼
goto B
 │
 ▼
goto C
```

This style is often called **spaghetti code**.

Modern C code usually prefers functions, loops, and structured control
flow.

------------------------------------------------------------------------

# Common Mistakes

## Breaking the Wrong Loop

`break` exits only the **nearest** loop.

Nested loops require additional logic if you want to exit multiple
levels.

------------------------------------------------------------------------

## Misusing `continue`

Remember:

``` text
continue
```

does **not** exit the loop.

It only skips the rest of the current iteration.

------------------------------------------------------------------------

## Excessive `goto`

Avoid replacing normal control structures with `goto`.

Use it only when it makes the code clearer, such as centralized cleanup.

------------------------------------------------------------------------

# Hands-on Labs

1.  Search an array and stop using `break`.
2.  Print odd numbers using `continue`.
3.  Create a menu that exits with `break`.
4.  Write a small program using `goto` for cleanup after an error.
5.  Observe how `break` behaves inside nested loops.

------------------------------------------------------------------------

# Interview Questions

### What does `break` do?

It immediately exits the nearest loop or `switch`.

### What does `continue` do?

It skips the remainder of the current loop iteration and begins the next
one.

### Does `continue` end the loop?

No. It only skips one iteration.

### What is `goto`?

A statement that transfers execution directly to a labeled statement.

### Why is `goto` discouraged?

Because excessive use makes programs difficult to understand and
maintain.

------------------------------------------------------------------------

# Summary

``` text
Loop
 │
 ├── break ─────► Exit Loop
 │
 ├── continue ─► Next Iteration
 │
 └── goto ─────► Jump to Label
```

Key Takeaways:

-   `break` exits a loop immediately.
-   `continue` skips the current iteration.
-   `goto` jumps to a label.
-   Use `goto` sparingly and primarily for structured cleanup in
    low-level C.
-   Understanding these statements helps you write efficient and
    readable control flow.

------------------------------------------------------------------------

# Next Chapter

## Chapter 12 --- Recursion

You'll learn:

-   What recursion is
-   Base case
-   Recursive case
-   Call stack
-   Stack frames
-   Recursion vs iteration
-   Common recursive algorithms
