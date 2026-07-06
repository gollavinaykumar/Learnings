# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 136 — `inode` Deep Dive — The Most Important Data Structure in Linux Filesystems

---

# Most Linux users think:

```
notes.txt

↓

The File
```

Actually...

```
notes.txt

↓

Not The File
```

The filename is only a label.

The real file is the:

```
inode
```

This is one of the biggest mindset shifts in Linux.

If you understand **inodes**, you'll understand:

- Hard links
- Symbolic links
- Permissions
- Ownership
- File size
- Disk blocks
- How Linux finds every byte of a file

---

# Learning Objectives

After this chapter you will understand:

- What an inode is
- inode number
- File metadata
- Hard links
- Symbolic links
- Block pointers
- Direct blocks
- Indirect blocks
- Double indirect blocks
- Triple indirect blocks

---

# The Big Misconception

Many people think:

```
notes.txt

↓

Contains Data
```

Reality:

```
notes.txt

↓

Directory Entry

↓

inode

↓

Disk Blocks
```

The filename only helps Linux locate the inode.

---

# What is an inode?

Definition:

> **An inode is a data structure that stores everything about a file except its filename.**

Think of it as the file's identity card.

---

# Visualization

```
Filename

↓

inode

↓

Actual Data
```

---

# What Does an inode Store?

Typical metadata:

```
Owner

↓

Group

↓

Permissions

↓

File Size

↓

Creation Time

↓

Modification Time

↓

Access Time

↓

Disk Block Addresses

↓

Link Count
```

---

# What Does NOT Exist Inside an inode?

Surprisingly:

```
Filename

↓

NOT Stored
```

Instead,

filenames live inside directories.

---

# Directory Entry

A directory contains:

```
Filename

↓

inode Number
```

Example:

```
notes.txt

↓

inode 15021
```

---

# Visualization

```
Directory

↓

notes.txt

↓

15021

↓

inode

↓

Blocks
```

---

# inode Number

Every inode has a unique number **within a filesystem**.

Example:

```bash
ls -i
```

Output:

```
15021 notes.txt
```

Here:

```
15021

↓

inode Number
```

---

# Why inode Numbers Matter

Linux identifies files internally using inode numbers,

not filenames.

---

# Opening a File

Program:

```c
open("notes.txt");
```

Linux performs:

```
Directory Lookup

↓

inode Number

↓

inode

↓

Open File
```

---

# Example

Directory:

```
notes.txt

↓

15021
```

inode:

```
15021

↓

Metadata

↓

Blocks
```

---

# Disk Blocks

An inode stores pointers to where the file's data lives.

Example:

```
inode

↓

Block 25

↓

Block 26

↓

Block 27
```

Actual bytes exist inside those disk blocks.

---

# Visualization

```
inode

↓

Block Pointer

↓

Disk Block

↓

Data
```

---

# Small File

Example:

```
Hello World
```

One block.

```
inode

↓

Block 105

↓

Data
```

Easy.

---

# Large File

Example:

```
5 GB Video
```

Needs thousands of blocks.

How can one inode store all those pointers?

Linux uses multiple levels of indirection.

---

# Direct Blocks

Traditional Unix inode:

```
inode

↓

Pointer

↓

Block

↓

Pointer

↓

Block

↓

Pointer

↓

Block
```

These are called:

```
Direct Block Pointers
```

Fast.

---

# Indirect Block

Eventually,

direct pointers become full.

Linux stores:

```
Pointer

↓

Indirect Block

↓

More Pointers

↓

More Data Blocks
```

---

# Visualization

```
inode

↓

Indirect Block

↓

Pointer

↓

Pointer

↓

Pointer

↓

Data Blocks
```

---

# Double Indirect Block

Even larger file:

```
inode

↓

Double Indirect

↓

Indirect Blocks

↓

Data Blocks
```

One extra level.

---

# Triple Indirect Block

Huge files:

```
inode

↓

Triple Indirect

↓

Double

↓

Indirect

↓

Blocks
```

Allows extremely large files.

---

# Visualization

```
inode

↓

Direct

↓

Indirect

↓

Double

↓

Triple
```

Each level increases maximum file size.

---

# Why This Design?

Small files:

```
Fast

↓

Direct Pointers
```

Large files:

```
Expandable

↓

Indirect Pointers
```

Efficient for both cases.

---

# Hard Links

Suppose:

```bash
ln notes.txt copy.txt
```

Now:

```
notes.txt

↓

inode 15021

↑

copy.txt
```

Two filenames.

One inode.

One file.

---

# Visualization

```
notes.txt

↓

15021

↑

copy.txt
```

Delete one filename:

```
copy.txt

↓

Still Exists
```

The inode remains until its link count reaches zero.

---

# Link Count

inode contains:

```
Links

↓

2
```

Delete one name:

```
Links

↓

1
```

Delete last name:

```
Links

↓

0

↓

Free inode
```

---

# Symbolic Link

Different.

```
shortcut.txt

↓

Contains Path

↓

notes.txt
```

A symbolic link points to a pathname,

not directly to the target inode.

---

# Visualization

Hard Link

```
Name

↓

inode
```

Symbolic Link

```
Name

↓

Path

↓

Another Name

↓

inode
```

---

# File Deletion

Suppose:

```bash
rm notes.txt
```

Linux removes:

```
Directory Entry
```

Not necessarily the data.

If:

```
Link Count > 0
```

inode remains.

---

# When Is Disk Space Freed?

Only after:

```
Link Count = 0

AND

No Process Uses File
```

Then:

```
inode

↓

Blocks Freed
```

---

# Why Can Linux Delete an Open File?

Suppose:

```
rm logfile
```

Program still writing.

Works because:

```
Directory Entry

↓

Removed

↓

inode

↓

Still Referenced

↓

Still Valid
```

---

# File Metadata

Command:

```bash
stat notes.txt
```

Shows:

- inode number
- permissions
- owner
- timestamps
- size
- links

---

# Real-World Example — Git

Git tracks files by content,

but Linux still identifies them through inodes while they're stored on disk.

---

# Real-World Example — PostgreSQL

Database file:

```
Directory

↓

inode

↓

Blocks
```

Every database file ultimately uses inodes.

---

# Real-World Example — Docker

Container layers consist of ordinary Linux files,

each backed by inodes in the underlying filesystem.

---

# Common Mistakes

---

## Thinking Filename Is the File

The inode represents the file.

The filename is merely a directory entry.

---

## Confusing Hard Links and Symbolic Links

Hard links reference the same inode.

Symbolic links reference a pathname.

---

## Thinking `rm` Immediately Erases Data

Usually it removes the directory entry first.

Blocks are reclaimed only when the inode is no longer referenced.

---

## Assuming inode Stores File Contents

It stores metadata and pointers.

Actual bytes live in data blocks.

---

# Hands-on Labs

## Lab 1

Run:

```bash
ls -i
```

Observe inode numbers.

---

## Lab 2

Create a hard link:

```bash
ln file copy
```

Compare inode numbers.

---

## Lab 3

Create a symbolic link:

```bash
ln -s file shortcut
```

Compare inode numbers.

---

## Lab 4

Run:

```bash
stat file
```

Inspect metadata.

---

## Lab 5

Open a file in one terminal.

Delete it from another.

Observe that the first process can still use the file.

---

# Interview Questions

### What is an inode?

A filesystem data structure storing file metadata and pointers to the file's data blocks.

---

### Does an inode contain the filename?

No.

Filenames are stored in directory entries.

---

### What is the difference between a hard link and a symbolic link?

A hard link references the same inode.

A symbolic link stores a pathname pointing to another file.

---

### When is disk space actually freed?

After the inode's link count reaches zero and no process still has the file open.

---

### Why does Linux use indirect blocks?

To efficiently support very large files without making every inode excessively large.

---

# Summary

File Lookup:

```
Filename

↓

Directory Entry

↓

inode

↓

Block Pointers

↓

Disk Blocks
```

Link Types:

```
Hard Link

↓

Same inode

----------------------

Symbolic Link

↓

Path

↓

Target
```

Block Structure:

```
inode

↓

Direct

↓

Indirect

↓

Double Indirect

↓

Triple Indirect
```

## Key Takeaways

- An inode is the true identity of a file in Linux.
- Filenames are stored separately in directory entries.
- Inodes store metadata and pointers to file data.
- Hard links share the same inode; symbolic links point to a pathname.
- Linux frees disk space only when the inode is no longer referenced.
- Understanding inodes explains many seemingly unusual Linux behaviors.

---

# Next Chapter

## Chapter 137 — The Linux Page Cache — Why Reading the Same File Twice Is Much Faster

You'll learn:

- What the page cache is
- Cache hits vs cache misses
- Dirty pages
- Write-back caching
- `fsync()`
- `sync()`
- Read-ahead
- Memory pressure
- How Linux dramatically accelerates file I/O using RAM