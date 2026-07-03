Most developers think a file is simply:

```
photo.jpg

document.pdf

app.js

README.md
```

But the Operating System doesn't see files this way.

When you save:

```text
hello.txt
```

The Operating System must answer many questions:

- Where should the file be stored?
- Which disk blocks should be used?
- How can the file be found later?
- What happens if the computer suddenly loses power?
- How does Linux find a file among millions of others?

The answer is:

**The File System.**

A File System is one of the most important components of an Operating System.

Without it:

- Databases couldn't store data.
- Docker couldn't store images.
- Linux couldn't boot.
- Photos and videos couldn't be saved.
- Every reboot would lose your files.

After this chapter, you'll understand how Linux stores, finds, and protects data on disk.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 14 — File Systems: How Linux Stores and Retrieves Data

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why File Systems exist
> - Files vs Directories
> - Storage Devices
> - Disk Blocks
> - Inodes
> - Superblock
> - Directory Structure
> - File Metadata
> - Journaling
> - Mount Points
> - Virtual File System (VFS)
> - ext4
> - XFS
> - Btrfs
> - How Linux finds a file
> - Complete File Read & Write Flow

---

# 📖 Why Do We Need a File System?

Imagine your SSD has:

```
1 TB
```

Without a File System,

it's simply:

```
Raw Bytes
```

Like this:

```
00010101

11001010

10101011

...
```

There are no:

- Files
- Folders
- Names
- Permissions

The Operating System needs a way to organize this data.

That organization is called a **File System**.

---

# Without a File System

Imagine a huge warehouse.

Everything is thrown randomly onto the floor.

```
Laptop

Book

Phone

Chair

Bottle
```

Finding one item would be nearly impossible.

---

# With a File System

Now imagine the warehouse has:

- Shelves
- Labels
- Boxes
- Categories

```
Electronics

↓

Phones

↓

Laptop
```

Finding items becomes easy.

This is exactly what a File System does.

---

# What is a File?

A File is simply:

```
Data

+

Metadata
```

Example:

```
hello.txt
```

Contains:

```
Hello World
```

But Linux also stores:

- Owner
- Permissions
- Creation Time
- Modification Time
- File Size

---

# What is a Directory?

A Directory is **not** a special container.

It is actually another file.

Instead of storing text,

it stores:

```
File Name

↓

Inode Number
```

Example:

```
Documents

↓

resume.pdf

↓

photo.jpg

↓

notes.txt
```

Directories organize files.

---

# Linux Directory Structure

Everything starts from:

```
/
```

Called:

```
Root Directory
```

Example:

```
/

├── bin

├── boot

├── dev

├── etc

├── home

├── lib

├── proc

├── sys

├── tmp

├── usr

└── var
```

Every file belongs somewhere under `/`.

---

# Storage Blocks

Disks are divided into equal-sized blocks.

Example:

```
Block 1

↓

Block 2

↓

Block 3

↓

Block 4
```

Typical block sizes:

```
4 KB

8 KB
```

Files occupy one or more blocks.

---

# Example

Suppose:

```
File Size

10 KB
```

Disk block size:

```
4 KB
```

Storage:

```
Block 10

↓

Block 11

↓

Block 12
```

The last block is partially used.

---

# File Metadata

Every file has metadata.

Examples:

```
File Name

Size

Owner

Permissions

Creation Time

Modification Time

Location
```

Linux stores this information separately from the actual file data.

---

# What is an Inode?

One of the most important Linux concepts.

Every file has an:

```
Inode
```

An Inode stores metadata,

but **not the file name**.

---

# Inode Contains

An Inode stores:

✔ File Size

✔ Owner

✔ Permissions

✔ Timestamps

✔ Disk Block Locations

✔ File Type

It does **not** store:

```
File Name
```

---

# File Name Storage

Where is the file name?

Inside the directory.

Example:

```
Directory

↓

hello.txt

↓

Inode 245
```

Then:

```
Inode 245

↓

Block 101

↓

Block 102
```

The filename points to the inode.

The inode points to the data.

---

# Complete File Lookup

Suppose you open:

```
notes.txt
```

Linux performs:

```
Directory

↓

Find Filename

↓

Get Inode

↓

Locate Disk Blocks

↓

Read Data

↓

Return File
```

The filename itself is never used to locate data directly.

---

# Visual Flow

```
notes.txt

↓

Directory Entry

↓

Inode

↓

Disk Blocks

↓

Actual Data
```

---

# Why Inodes Are Brilliant

Suppose you rename:

```
notes.txt

↓

my_notes.txt
```

Does Linux move the file?

No.

Only the directory entry changes.

The inode and data blocks remain the same.

Renaming is therefore very fast.

---

# Hard Links

Because filenames point to inodes,

multiple filenames can point to the same inode.

Example:

```
report.txt

↓

Inode 300
```

```
backup.txt

↓

Inode 300
```

Both names reference the same data.

These are called:

```
Hard Links
```

---

# Symbolic Links (Soft Links)

Unlike Hard Links,

a Symbolic Link points to another filename.

Example:

```
shortcut

↓

report.txt
```

It behaves like a shortcut.

If the original file disappears,

the symbolic link becomes invalid.

---

# Superblock

Every File System has a special structure called the:

```
Superblock
```

Think of it as the file system's identity card.

It stores:

- File System Type
- Block Size
- Total Blocks
- Free Blocks
- Total Inodes
- Free Inodes

Without the Superblock,

the Operating System cannot understand the file system.

---

# Journaling

Suppose you're saving a file.

Suddenly:

```
Power Failure
```

Without protection,

the file system could become corrupted.

Linux solves this using:

```
Journal
```

---

# How Journaling Works

Before changing the disk,

Linux records the operation.

```
Write Intent

↓

Journal

↓

Update Disk

↓

Mark Complete
```

If power fails,

Linux replays the journal.

This greatly improves reliability.

---

# Mount Points

Linux doesn't assign drive letters like Windows.

Instead,

every storage device becomes part of one directory tree.

Example:

```
/

↓

mnt

↓

usb
```

USB drive:

```
Mounted Here
```

Now files appear under:

```
/mnt/usb
```

---

# Virtual File System (VFS)

Linux supports many file systems.

Examples:

- ext4
- XFS
- Btrfs
- FAT32
- NTFS

Applications shouldn't care which one is used.

Linux provides:

```
Virtual File System

(VFS)
```

---

# VFS Architecture

```
Application

↓

VFS

↓

ext4

or

XFS

or

Btrfs
```

Applications use one interface.

VFS translates requests.

---

# ext4

The most common Linux File System.

Advantages:

✔ Stable

✔ Fast

✔ Journaling

✔ Widely Supported

Used by many Linux servers.

---

# XFS

Designed for:

- Large Files
- High Throughput
- Enterprise Servers

Often used for:

- Databases
- Storage Servers

---

# Btrfs

Modern Copy-on-Write File System.

Features:

✔ Snapshots

✔ Compression

✔ Checksums

✔ RAID Support

Widely used in container environments.

---

# File Read Flow

Suppose:

```bash
cat notes.txt
```

Linux performs:

```
Application

↓

VFS

↓

Directory Lookup

↓

Inode

↓

Disk Blocks

↓

SSD

↓

RAM

↓

Application
```

---

# File Write Flow

Suppose:

```bash
echo "Hello" > notes.txt
```

Linux performs:

```
Application

↓

System Call

↓

VFS

↓

Allocate Blocks

↓

Update Inode

↓

Journal

↓

Write SSD
```

---

# Real Example — Docker Image

Docker image layers are simply:

```
Files

↓

Directories

↓

Inodes

↓

Disk Blocks
```

Later,

you'll learn how Docker builds on Linux file systems using:

```
OverlayFS
```

---

# Why File Systems Matter

Every modern application depends on file systems.

Examples:

- PostgreSQL stores database files.
- Docker stores image layers.
- Kubernetes stores volumes.
- Nginx serves static files.
- Browsers cache downloaded content.

Without file systems,

none of these would work.

---

# Hands-on Lab

## Lab 1 — View Mounted File Systems

```bash
mount
```

---

## Lab 2 — View Disk Usage

```bash
df -h
```

---

## Lab 3 — View Inode Usage

```bash
df -i
```

Observe inode statistics.

---

## Lab 4 — Find File Inode

```bash
ls -i README.md
```

Observe the inode number.

---

## Lab 5 — Create a Hard Link

```bash
ln file.txt file2.txt
```

View inode numbers:

```bash
ls -li
```

Notice both files share the same inode.

---

## Lab 6 — Create a Symbolic Link

```bash
ln -s file.txt shortcut.txt
```

View:

```bash
ls -l
```

Observe the symbolic link.

---

# Interview Questions

## Why do Operating Systems need a File System?

A File System organizes raw storage into files and directories, making data easy to store and retrieve.

---

## What is an Inode?

An Inode stores a file's metadata and the locations of its data blocks but does not store the filename.

---

## Where is the filename stored?

The filename is stored in the directory entry, which maps the name to an inode.

---

## What is the purpose of the Superblock?

The Superblock stores essential metadata about the file system, such as block size, inode count, and free space.

---

## What is Journaling?

Journaling records pending filesystem changes before applying them, helping recover safely after crashes.

---

## What is the Virtual File System (VFS)?

VFS provides a common interface so applications can work with different file systems without knowing their implementation details.

---

## What is the difference between a Hard Link and a Symbolic Link?

A Hard Link points directly to an inode.

A Symbolic Link points to another file's pathname.

---

# Summary

The File System organizes raw storage into a structured hierarchy of files and directories.

```
Application

↓

VFS

↓

Directory

↓

Inode

↓

Disk Blocks

↓

SSD
```

Key concepts:

- File Systems organize storage.
- Files consist of data and metadata.
- Directories map filenames to inodes.
- Inodes store metadata and block locations.
- Superblocks describe the entire file system.
- Journaling protects against crashes.
- Mount Points integrate storage into one directory tree.
- VFS allows Linux to support multiple file systems.
- ext4, XFS, and Btrfs serve different workloads.

Understanding file systems is essential before learning Linux internals, Docker storage, OverlayFS, databases, and container technologies.

---

# Next Chapter

## Chapter 15 — Input/Output (I/O): How Linux Talks to Disks, Keyboards, Screens & Network Cards

We'll explore:

- What is I/O
- Blocking vs Non-Blocking I/O
- Synchronous vs Asynchronous I/O
- Device Drivers
- Character Devices
- Block Devices
- DMA (Direct Memory Access)
- Interrupts
- Polling
- epoll
- io_uring
- High-performance I/O in Linux