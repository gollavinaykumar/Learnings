# 👑 C Mastery Roadmap

# Module 13 — Advanced Linux Programming & Kernel Interfaces

# Chapter 135 — The Virtual File System (VFS) — Why Every Filesystem Looks the Same to Linux

---

# Most developers think:

> "Linux only has one filesystem."

Actually, Linux supports hundreds of filesystem types.

Examples:

```
ext4

XFS

Btrfs

FAT32

NTFS

NFS

tmpfs

procfs

sysfs
```

Yet all of these work with exactly the same code:

```c
open()

read()

write()

close()
```

How?

The answer is:

```
VFS

(Virtual File System)
```

---

# Learning Objectives

After this chapter you will understand:

- Why VFS exists
- Filesystem abstraction
- `inode`
- `dentry`
- `superblock`
- File operations
- Mount points
- Filesystem drivers

---

# The Problem

Suppose Linux supported:

```
ext4
```

only.

Easy.

But Linux also supports:

```
FAT32

↓

NTFS

↓

NFS

↓

Btrfs

↓

XFS
```

Each filesystem stores data differently.

Without VFS,

every application would need to understand every filesystem.

Impossible.

---

# The Solution

Linux creates one common interface.

Application:

```
read()

↓

VFS

↓

Filesystem Driver
```

Applications never know which filesystem is underneath.

---

# Think About USB

Suppose you plug in:

```
Keyboard

↓

Mouse

↓

Printer

↓

Camera
```

Every manufacturer is different.

Yet applications use the same USB interface.

VFS provides a similar abstraction for filesystems.

---

# What is VFS?

Definition:

> **The Virtual File System is a kernel layer that provides a common interface to all filesystem implementations.**

---

# Visualization

```
Application

↓

open()

↓

read()

↓

write()

↓

VFS

↓

ext4

↓

XFS

↓

NFS

↓

FAT
```

---

# Why Applications Love VFS

Program:

```c
open("notes.txt");
```

Doesn't care whether the file lives on:

```
SSD

↓

USB

↓

Network

↓

RAM
```

VFS handles the differences.

---

# Filesystem Drivers

Each filesystem provides its own driver.

Example:

```
ext4 Driver

↓

Understands ext4 Metadata

-------------------------

XFS Driver

↓

Understands XFS Metadata
```

The application doesn't know or care.

---

# VFS Objects

The Linux VFS uses several important kernel structures.

Major ones:

```
Superblock

↓

inode

↓

dentry

↓

file
```

Let's understand each.

---

# Superblock

Think of it as:

```
Filesystem Information
```

Contains:

- Filesystem type
- Block size
- Total blocks
- Free blocks
- Root inode

Every mounted filesystem has one superblock.

---

# Visualization

```
Filesystem

↓

Superblock

↓

Metadata
```

---

# inode

One of Linux's most important concepts.

Definition:

> **An inode stores metadata about a file, but not its filename.**

Contains:

- Owner
- Permissions
- Size
- Timestamps
- Block locations

---

# Example

File:

```
notes.txt
```

inode:

```
Size

↓

Owner

↓

Permissions

↓

Disk Blocks
```

Notice:

```
Filename

↓

NOT Stored Here
```

---

# Why?

Because multiple filenames may refer to the same inode.

Example:

```
Hard Link
```

We'll learn more later.

---

# dentry (Directory Entry)

If inode doesn't store filenames,

where are they?

Inside:

```
Directory Entries

↓

dentry
```

A dentry connects:

```
Filename

↓

inode
```

---

# Visualization

```
notes.txt

↓

dentry

↓

inode
```

---

# file Object

When you call:

```c
open()
```

Kernel creates:

```
struct file
```

Contains:

- Current file offset
- Access mode
- Pointer to inode
- File operations

---

# Visualization

```
FD

↓

file

↓

inode
```

---

# Relationship

```
Superblock

↓

inode

↑

dentry

↓

file
```

These four structures form the core of the VFS.

---

# File Operations

Every filesystem implements functions like:

```
open()

↓

read()

↓

write()

↓

close()
```

VFS simply calls the correct implementation.

---

# Visualization

```
Application

↓

read()

↓

VFS

↓

ext4_read()

or

xfs_read()

or

nfs_read()
```

The application never changes.

---

# Mount Points

Suppose:

```
SSD

↓

ext4

↓

Mounted At

↓

/
```

USB:

```
FAT32

↓

Mounted At

↓

/media/usb
```

Network:

```
NFS

↓

Mounted At

↓

/mnt/server
```

All appear as one unified directory tree.

---

# Visualization

```
/

↓

home

↓

media

↓

mnt
```

Behind each directory,

a different filesystem may exist.

---

# Why This Is Amazing

Application:

```c
open("/home/file");
```

or

```c
open("/media/usb/photo.jpg");
```

Uses exactly the same API.

---

# VFS During `read()`

Remember Chapter 134.

Flow:

```
read()

↓

VFS

↓

Filesystem Driver

↓

Page Cache

↓

Disk
```

Now you understand what VFS actually does.

---

# Example — ext4

Application:

```
open()

↓

VFS

↓

ext4 Driver

↓

inode

↓

Blocks
```

---

# Example — NFS

Application:

```
read()

↓

VFS

↓

NFS Driver

↓

Network

↓

Remote Server
```

Exactly the same application code.

---

# Example — tmpfs

Application:

```
write()

↓

VFS

↓

tmpfs

↓

RAM
```

No disk involved.

---

# `/proc`

Linux also treats:

```
/proc
```

as a filesystem.

Example:

```bash
cat /proc/cpuinfo
```

There is no actual file on disk.

The kernel generates the contents dynamically.

Yet:

```
cat

↓

read()

↓

VFS

↓

procfs
```

Still works.

---

# `/sys`

Likewise:

```
/sys
```

uses:

```
sysfs
```

Applications still use:

```
open()

↓

read()

↓

write()
```

---

# Why VFS Is Powerful

One interface supports:

- SSDs
- USB drives
- Network filesystems
- RAM filesystems
- Virtual kernel filesystems

Without changing application code.

---

# Real-World Example — Docker

Container:

```
OverlayFS

↓

VFS

↓

Application
```

Applications don't know OverlayFS is being used.

---

# Real-World Example — Kubernetes

Volumes:

```
Host Path

↓

NFS

↓

Cloud Storage

↓

tmpfs
```

All accessed using ordinary file APIs.

---

# Real-World Example — PostgreSQL

Database files:

```
read()

↓

VFS

↓

ext4
```

No database code needs to understand ext4 internals.

---

# Common Mistakes

---

## Thinking VFS Stores Files

It doesn't.

VFS is an abstraction layer.

The actual filesystem stores the data.

---

## Thinking inode Contains Filename

It stores metadata.

Directory entries map filenames to inodes.

---

## Thinking All Filesystems Work the Same Internally

They expose the same API,

but their internal implementations differ greatly.

---

## Forgetting Mount Points

Linux combines many filesystems into one unified directory tree.

---

# Hands-on Labs

## Lab 1

Run:

```bash
mount
```

Observe all mounted filesystems.

---

## Lab 2

Run:

```bash
df -T
```

See filesystem types.

---

## Lab 3

Inspect:

```bash
ls -i
```

Observe inode numbers.

---

## Lab 4

Read:

```bash
cat /proc/cpuinfo
```

Understand that `/proc` is generated dynamically.

---

## Lab 5

Run:

```bash
stat filename
```

Observe inode metadata.

---

# Interview Questions

### What is the VFS?

A kernel abstraction layer that provides a common interface for all filesystems.

---

### What is an inode?

A kernel data structure storing file metadata, excluding the filename.

---

### What is a dentry?

A directory entry that maps filenames to inodes.

---

### Why does Linux need VFS?

To allow applications to use the same file APIs regardless of the underlying filesystem.

---

### Does VFS store file data?

No.

The underlying filesystem stores the actual data.

---

# Summary

Architecture:

```
Application

↓

open()

↓

read()

↓

write()

↓

VFS

↓

Filesystem Driver

↓

Storage
```

Core VFS Objects:

```
Superblock

↓

inode

↑

dentry

↓

file
```

Filesystem Examples:

| Filesystem | Storage |
|------------|----------|
| ext4 | SSD/HDD |
| XFS | SSD/HDD |
| FAT32 | USB |
| NFS | Network |
| tmpfs | RAM |
| procfs | Kernel Information |
| sysfs | Kernel Objects |

## Key Takeaways

- VFS provides a unified filesystem interface for Linux.
- Applications use the same APIs regardless of the underlying filesystem.
- `inode`, `dentry`, `superblock`, and `file` are the core VFS structures.
- Mount points combine multiple filesystems into one directory tree.
- Virtual filesystems like `/proc` and `/sys` use the same interface as ordinary files.
- VFS is one of the key abstractions that makes Linux flexible and extensible.

---

# Next Chapter

## Chapter 136 — `inode` Deep Dive — The Most Important Data Structure in Linux Filesystems

You'll learn:

- Why filenames are not stored in inodes
- Hard links
- Symbolic links
- Inode numbers
- Block pointers
- Direct, indirect, double indirect, and triple indirect blocks
- File metadata
- How Linux locates every byte of every file on disk