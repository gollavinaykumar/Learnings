# 👑 C Mastery Roadmap

# Module 12 — C Meets the Operating System

# Chapter 115 — Environment Variables (`env`, `PATH`, `HOME`, `USER`, `execve()`)

---

# Most beginners think:

> "`PATH` is just something Linux uses."

Actually,

every process in Linux carries around a collection of:

```
Environment Variables
```

These variables tell programs things like:

- Who the current user is
- Where the home directory is
- Which directories contain executable programs
- What language to use
- Which terminal is running
- Hundreds of other configuration values

Without environment variables,

many Linux commands would stop working correctly.

---

# Learning Objectives

After this chapter you will understand:

- What environment variables are
- How they are stored
- Process inheritance
- `PATH`
- `HOME`
- `USER`
- `getenv()`
- `setenv()`
- `execve()`
- Shell environment

---

# What is an Environment Variable?

An environment variable is simply:

```
Name

↓

Value
```

Example:

```
HOME

↓

/home/vinay
```

Another:

```
USER

↓

vinay
```

Programs read these values whenever they need configuration.

---

# Think of Them Like Settings

Imagine a game.

Instead of hardcoding:

```
Language

↓

English
```

Store:

```
LANG

↓

en_US.UTF-8
```

Now every program automatically knows which language to use.

---

# Viewing Environment Variables

Linux command:

```bash
env
```

or:

```bash
printenv
```

Example output:

```
HOME=/home/vinay

USER=vinay

PATH=/usr/bin:/bin

SHELL=/bin/bash
```

---

# Where Are They Stored?

When Linux creates a process,

its virtual memory contains:

```
Text

↓

Data

↓

Heap

↓

Stack

↓

Arguments

↓

Environment Variables
```

Remember from the previous chapter:

Environment variables are part of the process memory.

---

# `main()` Can Access Them

Most programs use:

```c
int main(
int argc,
char *argv[])
```

But another valid form is:

```c
int main(
int argc,
char *argv[],
char *envp[])
```

Where:

```
envp

↓

Environment
```

---

# Visualization

```
argv

↓

Program Arguments

---------------------

envp

↓

Environment Variables
```

Different purposes.

---

# Example

Program:

```bash
./app hello world
```

Memory contains:

```
argv[0]

↓

./app

argv[1]

↓

hello

argv[2]

↓

world
```

Environment:

```
HOME

↓

/home/vinay

PATH

↓

...
```

---

# `getenv()`

Prototype:

```c
#include <stdlib.h>

char *getenv(
const char *name);
```

Example:

```c
char *home =
getenv("HOME");
```

Returns:

```
/home/vinay
```

or whatever the current process inherited.

---

# Example Program

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("%s\n",
           getenv("HOME"));

    return 0;
}
```

Possible output:

```
/home/vinay
```

---

# What is `PATH`?

Probably the most important environment variable.

Example:

```
PATH=

/usr/local/bin

:

/usr/bin

:

/bin
```

It is a list of directories,

separated by:

```
:
```

on Unix-like systems.

---

# Why `PATH` Exists

Suppose you type:

```bash
ls
```

Shell asks:

```
Where Is

ls
```

Search order:

```
/usr/local/bin

↓

/usr/bin

↓

/bin
```

Finds:

```
/bin/ls
```

Then executes it.

---

# Without `PATH`

You would have to type:

```bash
/bin/ls
```

every time.

`PATH` makes command execution convenient.

---

# `HOME`

Example:

```
HOME

↓

/home/vinay
```

Programs use this for:

- Configuration files
- Downloads
- Documents
- User data

Instead of hardcoding paths.

---

# `USER`

Example:

```
USER

↓

vinay
```

Allows applications to know who is currently logged in.

---

# `PWD`

Current working directory.

Example:

```
PWD

↓

/home/vinay/projects
```

Many tools rely on this information.

---

# Inheritance

Suppose:

```
Shell

↓

Environment

↓

fork()

↓

Child

↓

exec()
```

Question:

Does the child receive the environment?

Yes.

Children inherit a copy of the parent's environment by default.

---

# Visualization

```
Shell

↓

PATH

↓

HOME

↓

USER

↓

fork()

↓

Child

↓

Same Variables
```

Initially,

the child has its own copy.

---

# `setenv()`

Prototype:

```c
setenv(
"name",
"value",
1);
```

Example:

```c
setenv(
"COLOR",
"blue",
1);
```

Adds or updates an environment variable for the current process.

---

# `unsetenv()`

Example:

```c
unsetenv("COLOR");
```

Removes the variable from the process environment.

---

# Important

Changing an environment variable affects:

```
Current Process

↓

Future Children
```

It does **not** modify the parent process.

---

# Why?

Suppose:

```
Shell

↓

fork()

↓

Program
```

The child cannot change the parent's memory.

Each process has its own address space.

---

# `execve()`

Prototype:

```c
execve(
path,
argv,
envp);
```

Unlike other `exec` variants,

`execve()` allows you to provide an entirely new environment.

---

# Visualization

```
Program

↓

Arguments

↓

Environment

↓

execve()

↓

New Program
```

This is how programs can launch children with customized environments.

---

# Example

Parent:

```
HOME

↓

/home/vinay
```

Calls:

```
execve()

↓

New Environment

↓

HOME=/tmp
```

The new program starts with:

```
HOME

↓

/tmp
```

---

# Shell Example

Command:

```bash
export NAME=Vinay
```

Shell:

```
Environment

↓

NAME

↓

Vinay
```

Every child process inherits:

```
NAME=Vinay
```

until changed.

---

# Real-World Example — Python

Python reads:

```
PYTHONPATH
```

to locate additional modules.

---

# Real-World Example — Java

Java uses:

```
JAVA_HOME
```

to locate the Java installation.

---

# Real-World Example — Docker

Containers often receive configuration through environment variables:

```
DATABASE_URL

↓

API_KEY

↓

PORT
```

These are passed into the container process when it starts.

---

# Real-World Example — Git

Git uses variables such as:

```
GIT_EDITOR

↓

GIT_AUTHOR_NAME
```

to customize behavior.

---

# Common Mistakes

---

## Thinking Environment Variables Are Global

They belong to individual processes.

---

## Expecting a Child to Modify the Parent

Impossible.

Each process has its own environment.

---

## Hardcoding Paths

Prefer:

```
HOME

↓

PATH

↓

USER
```

instead of fixed filesystem paths.

---

## Assuming `PATH` Executes Programs

`PATH` only tells the shell where to search.

The shell still uses system calls such as `execve()` to start programs.

---

# Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char *user = getenv("USER");

    if(user)
    {
        printf("Hello %s\n", user);
    }

    return 0;
}
```

Possible output:

```
Hello vinay
```

---

# Hands-on Labs

## Lab 1

Run:

```bash
env
```

Inspect your current environment.

---

## Lab 2

Write a program using:

```c
getenv()
```

Read:

- `HOME`
- `USER`
- `PATH`

---

## Lab 3

Use:

```c
setenv()
```

Create a custom environment variable.

Verify it with:

```c
getenv()
```

---

## Lab 4

Export a variable:

```bash
export MYNAME=Vinay
```

Run your program.

Read it using:

```c
getenv("MYNAME");
```

---

## Lab 5

Research `execve()` and explain how it differs from `execl()` and `execvp()` with respect to environment handling.

---

# Interview Questions

### What is an environment variable?

A name-value pair stored in a process that provides configuration information.

---

### What does `PATH` do?

It specifies the directories the shell searches when locating executable programs.

---

### What does `getenv()` return?

The value of an environment variable, or `NULL` if it does not exist.

---

### Do child processes inherit environment variables?

Yes.

Children inherit a copy of the parent's environment by default.

---

### What is special about `execve()`?

It allows the caller to specify a custom environment for the new program.

---

# Summary

```
Shell

↓

Environment

↓

fork()

↓

Child

↓

exec()

↓

Inherited Environment
```

Common Variables:

| Variable | Purpose |
|----------|---------|
| `PATH` | Search path for executables |
| `HOME` | User's home directory |
| `USER` | Current username |
| `PWD` | Current working directory |
| `LANG` | Locale and language settings |

## Key Takeaways

- Environment variables are process-specific configuration values.
- Child processes inherit a copy of their parent's environment.
- `getenv()`, `setenv()`, and `unsetenv()` manage environment variables.
- `PATH` allows shells to locate executables without requiring full paths.
- `execve()` can launch a program with a completely custom environment.
- Environment variables are widely used for configuring applications, containers, shells, and development tools.

---

# Next Chapter

## Chapter 116 — Signals (How Linux Interrupts Running Processes)

You'll learn:

- What signals are
- `SIGINT`
- `SIGTERM`
- `SIGKILL`
- `SIGSEGV`
- Signal handlers
- `signal()`
- `sigaction()`
- How Ctrl+C actually works
- Process interruption and asynchronous events