# 👑 C Mastery Roadmap

# Module 5 --- Pointers

# Chapter 33 --- Function Pointers (Advanced)

------------------------------------------------------------------------

## Most developers think:

> "Function pointers are only used to call another function."

In real systems, function pointers are used to build **entire
architectures**.

They power:

-   Linux kernel operation tables
-   Device drivers
-   Plugin systems
-   Event dispatchers
-   Virtual functions in C
-   High-performance servers

Without advanced function pointers, much of modern systems software
would be impossible.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   Dispatch tables
-   Arrays of function pointers
-   Virtual functions in C
-   Plugin architecture
-   Linux kernel operation tables
-   Callback chains
-   Dynamic behavior at runtime

------------------------------------------------------------------------

# From One Function Pointer...

``` c
int (*op)(int,int);
```

...to many:

``` c
int add(int a,int b){ return a+b; }
int sub(int a,int b){ return a-b; }

int (*ops[])(int,int) = {
    add,
    sub
};
```

Now select behavior at runtime:

``` c
printf("%d\n", ops[0](10,5)); // 15
printf("%d\n", ops[1](10,5)); // 5
```

------------------------------------------------------------------------

# Dispatch Table

Instead of many `if` statements:

``` text
Request
   │
   ▼
Dispatch Table
   │
   ├── read()
   ├── write()
   ├── open()
   └── close()
```

The program chooses the correct function by index or key.

This is faster and easier to extend.

------------------------------------------------------------------------

# Example

``` c
void start(){ puts("Start"); }
void stop(){ puts("Stop"); }

void (*commands[])() = {
    start,
    stop
};

commands[1]();
```

Output:

``` text
Stop
```

------------------------------------------------------------------------

# Virtual Functions in C

C has no classes.

But we can simulate polymorphism.

``` c
struct AnimalOps {
    void (*speak)(void);
};

void dogSpeak(){ puts("Woof"); }
void catSpeak(){ puts("Meow"); }

struct AnimalOps dog = { dogSpeak };
struct AnimalOps cat = { catSpeak };

dog.speak();
cat.speak();
```

The behavior changes based on the function pointer stored in the
structure.

------------------------------------------------------------------------

# Linux Kernel Example

The Linux kernel uses structures full of function pointers.

Conceptually:

``` text
file_operations

 ├── open
 ├── read
 ├── write
 ├── ioctl
 └── release
```

When a process calls `read()`, the kernel invokes the driver's `read`
function through this table.

This allows different drivers to provide different implementations using
the same interface.

------------------------------------------------------------------------

# Plugin Architecture

``` text
Application
      │
      ▼
Plugin Interface
      │
      ├── Plugin A
      ├── Plugin B
      └── Plugin C
```

Each plugin registers its own function pointers.

The application calls whichever plugin is currently loaded.

------------------------------------------------------------------------

# Callback Chains

One callback can invoke another.

``` text
Network Packet
      │
      ▼
Parser
      │
      ▼
Firewall Callback
      │
      ▼
Application Callback
```

This layered design is common in networking software.

------------------------------------------------------------------------

# Common Mistakes

## Wrong Function Signature

Every function stored in a table must match the declared pointer type.

## Calling an Uninitialized Pointer

``` c
void (*fp)(void);

fp();   // Undefined behavior
```

Initialize pointers before use.

------------------------------------------------------------------------

# Real-World Uses

Advanced function pointers are used in:

-   Linux kernel
-   Nginx
-   Redis
-   SQLite
-   Device drivers
-   Event loops
-   GUI frameworks
-   Plugin systems

------------------------------------------------------------------------

# Hands-on Labs

1.  Build a calculator using an array of function pointers.
2.  Replace an `if-else` chain with a dispatch table.
3.  Create a simple plugin interface using a structure of function
    pointers.
4.  Simulate an animal interface with different implementations.

------------------------------------------------------------------------

# Interview Questions

### What is a dispatch table?

A collection of function pointers used to select behavior at runtime.

### Why are function pointers used in the Linux kernel?

To provide interchangeable implementations behind a common interface.

### Can C implement polymorphism?

Yes. By storing function pointers inside structures.

### Why are dispatch tables better than long `if-else` chains?

They improve modularity, extensibility, and often performance.

------------------------------------------------------------------------

# Summary

``` text
Function Pointer
      │
      ▼
Dispatch Table
      │
      ▼
Runtime Selection
      │
      ▼
Execute Correct Function
```

Key Takeaways:

-   Function pointers enable dynamic behavior.
-   Arrays of function pointers create dispatch tables.
-   Structures with function pointers simulate object-oriented behavior.
-   The Linux kernel relies heavily on operation tables.
-   Advanced function pointers are a cornerstone of systems programming.

------------------------------------------------------------------------

# Next Chapter

## Chapter 34 --- Double Pointer

You'll explore double pointers in greater depth, including dynamic
allocation of pointer arrays, 2D dynamic arrays, and APIs that return
allocated memory.
