# 👑 C Mastery Roadmap

# Module 4 --- Arrays & Strings

# Chapter 22 --- Multidimensional Arrays

------------------------------------------------------------------------

## Most beginners think:

> "A two-dimensional array is an array inside another array."

That's a helpful mental model, but internally it's one continuous block
of memory.

Understanding that memory layout is essential for graphics, image
processing, game development, scientific computing, and the Linux
kernel.

------------------------------------------------------------------------

# Learning Objectives

After this chapter you will understand:

-   What a multidimensional array is
-   2D and 3D arrays
-   Declaration and initialization
-   Row and column indexing
-   Memory layout
-   Row-major order
-   Nested loops
-   Common mistakes

------------------------------------------------------------------------

# Why Do We Need Multidimensional Arrays?

Suppose you want to store marks for 3 students in 4 subjects.

Without a 2D array:

``` c
int s1[4];
int s2[4];
int s3[4];
```

With a 2D array:

``` c
int marks[3][4];
```

One variable represents the whole table.

------------------------------------------------------------------------

# What is a 2D Array?

Think of it as a table.

``` text
        Subject
      0   1   2   3

S0   80  75  92  88
S1   70  82  90  78
S2   95  89  84  91
```

Access:

``` c
marks[1][2];
```

Row 1, Column 2.

------------------------------------------------------------------------

# Declaring a 2D Array

``` c
int matrix[3][4];
```

This creates:

-   3 rows
-   4 columns

Total elements:

``` text
3 × 4 = 12
```

------------------------------------------------------------------------

# Initializing

``` c
int matrix[2][3] = {
    {1,2,3},
    {4,5,6}
};
```

------------------------------------------------------------------------

# Memory Layout

Although it looks like rows and columns, memory is linear.

``` text
Addresses

0x1000 → 1
0x1004 → 2
0x1008 → 3
0x100C → 4
0x1010 → 5
0x1014 → 6
```

This is called **row-major order**.

Rows are stored one after another.

------------------------------------------------------------------------

# Row-Major Order

``` text
Matrix

1 2 3
4 5 6

Stored as

1 2 3 4 5 6
```

C stores complete rows before moving to the next row.

------------------------------------------------------------------------

# Accessing Elements

``` c
printf("%d\n", matrix[0][1]);
printf("%d\n", matrix[1][2]);
```

------------------------------------------------------------------------

# Traversing a 2D Array

``` c
for(int i=0;i<2;i++)
{
    for(int j=0;j<3;j++)
    {
        printf("%d ", matrix[i][j]);
    }
    printf("\n");
}
```

Nested loops naturally match rows and columns.

------------------------------------------------------------------------

# 3D Arrays

C also supports higher dimensions.

``` c
int cube[2][3][4];
```

Useful for:

-   RGB images
-   Scientific simulations
-   Volumetric data

------------------------------------------------------------------------

# Real-World Uses

Multidimensional arrays are used for:

-   Images
-   Matrices
-   Chess boards
-   Sudoku
-   Heat maps
-   Pixel buffers
-   Neural network tensors (basic representation)

------------------------------------------------------------------------

# Common Mistakes

## Accessing Invalid Columns

``` c
matrix[0][5];
```

Undefined behavior.

------------------------------------------------------------------------

## Swapping Rows and Columns

Wrong:

``` c
matrix[column][row];
```

Remember:

``` text
matrix[row][column]
```

------------------------------------------------------------------------

# Hands-on Labs

1.  Read and print a 3×3 matrix.
2.  Calculate row sums.
3.  Calculate column sums.
4.  Find the largest element.
5.  Compute the transpose of a matrix.

------------------------------------------------------------------------

# Interview Questions

### What is a multidimensional array?

An array whose elements are themselves arrays.

### How are 2D arrays stored in C?

In row-major order.

### Why are nested loops commonly used?

Because one loop processes rows and the inner loop processes columns.

### Can C support 3D arrays?

Yes. C supports arrays of any number of dimensions.

------------------------------------------------------------------------

# Summary

``` text
2D Array
     │
     ▼
Rows
     │
     ▼
Columns
     │
     ▼
Row-Major Memory
     │
     ▼
Contiguous Storage
```

Key Takeaways:

-   Multidimensional arrays represent tables and grids.
-   C stores them in row-major order.
-   Memory remains contiguous.
-   Nested loops are the standard way to process them.
-   Understanding memory layout prepares you for pointer arithmetic.

------------------------------------------------------------------------

# Next Chapter

## Chapter 23 --- Character Arrays

You'll learn how characters are stored, how text differs from numbers,
and why strings in C are simply character arrays terminated by a null
character.
