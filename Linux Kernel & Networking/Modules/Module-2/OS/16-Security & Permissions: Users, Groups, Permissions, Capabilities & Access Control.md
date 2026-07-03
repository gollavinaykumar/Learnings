Most developers know Linux has commands like:

```bash
chmod

chown

sudo
```

They know files have permissions like:

```
-rwxr-xr--
```

But very few understand:

- Why permissions exist
- How Linux prevents users from accessing each other's files
- Why Docker containers don't always run as `root`
- What `sudo` actually does
- What Linux Capabilities are
- Why Kubernetes recommends running containers as non-root

The answer is:

**Linux Security.**

Without security:

- Any user could delete the operating system.
- Any application could read your passwords.
- Malware could take complete control of your computer.
- Containers would not provide meaningful isolation.

After this chapter, you'll understand the Linux permission model and the foundations of Operating System security.

---

# 👑 Linux Kernel & Networking Mastery

# Module 2 — Operating Systems

# Chapter 16 — Security & Permissions: Users, Groups, Permissions, Capabilities & Access Control

> **Learning Objectives**
>
> By the end of this chapter, you will understand:
>
> - Why Operating Systems need security
> - Users
> - Groups
> - User IDs (UID)
> - Group IDs (GID)
> - Root User
> - File Ownership
> - Linux File Permissions
> - Read, Write & Execute
> - `chmod`
> - `chown`
> - `sudo`
> - SUID
> - SGID
> - Sticky Bit
> - Access Control Lists (ACL)
> - Linux Capabilities
> - Authentication vs Authorization
> - Principle of Least Privilege

---

# 📖 Why Do We Need Security?

Imagine two people share one computer.

```
Alice

Bob
```

Alice stores:

```
salary.xlsx
```

Bob stores:

```
personal_notes.txt
```

Should Bob be able to open Alice's salary file?

No.

Should Alice be able to delete the Operating System?

Also no.

The Operating System must decide:

```
Who Can Access What?
```

---

# Security Goals

A secure Operating System should provide:

✔ Privacy

✔ Authentication

✔ Authorization

✔ Integrity

✔ Isolation

✔ Accountability

---

# Authentication vs Authorization

These two terms are often confused.

---

## Authentication

Authentication answers:

```
Who are you?
```

Example:

```
Username

+

Password
```

or

```
SSH Key

Fingerprint

Face ID
```

---

## Authorization

Authorization answers:

```
What are you allowed to do?
```

Example:

```
Read File

✔ Allowed
```

```
Delete System File

❌ Denied
```

---

# Linux Users

Linux is a multi-user Operating System.

Example:

```
alice

↓

UID 1000
```

```
bob

↓

UID 1001
```

Each user has:

- Username
- User ID (UID)
- Home Directory
- Groups

---

# User ID (UID)

Linux internally identifies users using numbers.

Example:

```
alice

↓

UID 1000
```

```
bob

↓

UID 1001
```

The kernel uses UIDs,

not usernames,

for permission checks.

---

# Root User

Linux has one special user.

```
root

↓

UID 0
```

Root has almost unlimited privileges.

Examples:

✔ Install software

✔ Kill any process

✔ Read any file

✔ Modify system configuration

✔ Mount disks

Because root is so powerful,

it should be used carefully.

---

# Groups

Suppose five developers work on one project.

Instead of granting permissions individually,

Linux creates a group.

Example:

```
developers

↓

Alice

Bob

Charlie
```

Now one permission change affects the entire team.

---

# Group ID (GID)

Like users,

groups also have IDs.

Example:

```
developers

↓

GID 100
```

Linux checks:

```
UID

+

GID
```

before granting access.

---

# File Ownership

Every file has:

```
Owner

↓

Group
```

Example:

```
report.pdf

↓

Owner:

alice

↓

Group:

developers
```

---

# Linux Permission Model

Every file stores permissions for:

```
Owner

↓

Group

↓

Others
```

Three permission sets.

---

# Read (r)

Read means:

```
Can View Data
```

Example:

```
cat file.txt
```

Requires:

```
Read Permission
```

---

# Write (w)

Write means:

```
Can Modify Data
```

Examples:

```
Edit

Delete

Append
```

---

# Execute (x)

Execute means:

```
Can Run Program
```

Example:

```bash
./app
```

Without execute permission,

the program cannot run.

---

# Permission Structure

Example:

```
-rwxr-xr--
```

Breakdown:

```
-

↓

File Type

rwx

↓

Owner

r-x

↓

Group

r--

↓

Others
```

---

# Understanding rwx

```
r

↓

Read
```

```
w

↓

Write
```

```
x

↓

Execute
```

```
-

↓

Permission Missing
```

---

# Example

```
-rw-r-----
```

Owner:

```
Read

Write
```

Group:

```
Read
```

Others:

```
No Access
```

---

# Numeric Permissions

Linux also represents permissions numerically.

```
Read

=

4
```

```
Write

=

2
```

```
Execute

=

1
```

Examples:

```
7

↓

4+2+1

↓

rwx
```

```
6

↓

rw-
```

```
5

↓

r-x
```

```
4

↓

r--
```

---

# Common Permission Values

| Number | Permission |
|---------|------------|
| 7 | rwx |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |
| 0 | --- |

---

# chmod

Change file permissions.

Example:

```bash
chmod 755 app
```

Result:

```
Owner

rwx
```

```
Group

r-x
```

```
Others

r-x
```

---

# chown

Change file owner.

Example:

```bash
chown alice report.txt
```

Now:

```
Owner

↓

alice
```

---

# sudo

Suppose Alice needs to install software.

Normally:

```
Permission Denied
```

Instead:

```bash
sudo apt install nginx
```

`sudo` temporarily executes the command with elevated privileges.

---

# Why Not Always Use Root?

Running everything as root is dangerous.

Suppose Chrome runs as root.

A browser vulnerability could allow an attacker to:

✔ Delete files

✔ Install malware

✔ Modify the kernel configuration

Instead,

applications should use the **least privilege necessary**.

---

# Principle of Least Privilege

Rule:

```
Give Only

The Permissions

Actually Needed
```

Nothing more.

This principle is used everywhere:

- Linux
- Docker
- Kubernetes
- Cloud Platforms

---

# Special Permission — SUID

Suppose:

```
Program Owner

↓

root
```

Normal user runs the program.

With:

```
SUID
```

The program executes using:

```
Owner's Privileges
```

Not the user's.

---

# Example

The `passwd` command updates:

```
/etc/shadow
```

Normal users cannot modify that file.

But `passwd` has:

```
SUID
```

Therefore,

users can safely change their passwords.

---

# SGID

SGID works similarly,

but with groups.

Files created inside an SGID directory inherit the directory's group.

Useful for shared project folders.

---

# Sticky Bit

Suppose many users share:

```
/tmp
```

Without protection,

any user could delete another user's files.

The Sticky Bit changes the rule.

Now:

```
Only Owner

Can Delete Their File
```

Even in a shared directory.

---

# Access Control Lists (ACL)

Traditional permissions allow:

```
Owner

↓

Group

↓

Others
```

Sometimes that's not enough.

Suppose:

```
Alice

Bob

Charlie
```

Need different permissions.

ACL provides fine-grained access control.

---

# Linux Capabilities

Traditionally,

root had:

```
Everything
```

Linux introduced:

```
Capabilities
```

to divide root privileges into smaller pieces.

Examples:

```
CAP_NET_ADMIN

↓

Network Configuration
```

```
CAP_SYS_TIME

↓

Change System Time
```

```
CAP_SYS_ADMIN

↓

Many Administrative Tasks
```

Applications receive only the capabilities they require.

---

# Why Capabilities Matter

Docker containers rarely need:

```
Full Root Access
```

Instead,

Docker grants only specific capabilities.

This significantly improves security.

---

# Real Example

Suppose Nginx needs to listen on:

```
Port 80
```

Traditionally:

```
Run as Root
```

Modern Linux:

```
Grant

CAP_NET_BIND_SERVICE
```

Now Nginx binds to port 80

without having unlimited root privileges.

---

# Why Security Matters

Everything relies on Linux permissions.

Examples:

- Docker Containers
- Kubernetes Pods
- Databases
- SSH
- Cloud Servers
- Web Servers

Without proper permissions,

systems become vulnerable.

---

# Hands-on Lab

## Lab 1 — View File Permissions

```bash
ls -l
```

Observe:

```
-rwxr-xr-x
```

---

## Lab 2 — Change Permissions

```bash
chmod 755 app
```

View again:

```bash
ls -l
```

---

## Lab 3 — Change Owner

```bash
sudo chown $USER file.txt
```

---

## Lab 4 — View Current User

```bash
whoami
```

---

## Lab 5 — View User ID

```bash
id
```

Observe:

```
UID

GID

Groups
```

---

## Lab 6 — View Capabilities

Linux:

```bash
getcap /usr/bin/ping
```

Observe capabilities assigned to programs.

---

# Interview Questions

## What is the difference between Authentication and Authorization?

Authentication verifies identity.

Authorization determines what actions that identity is allowed to perform.

---

## What is the purpose of UID?

The User ID uniquely identifies a user inside the Linux kernel for permission checks.

---

## Why is the root user special?

The root user (UID 0) has almost unrestricted access to system resources.

---

## What do Read, Write, and Execute permissions mean?

Read allows viewing data.

Write allows modifying data.

Execute allows running a file as a program.

---

## What is the purpose of `sudo`?

`sudo` allows a user to execute specific commands with elevated privileges without logging in as root.

---

## What is SUID?

SUID allows an executable to run with the permissions of its owner instead of the user executing it.

---

## Why were Linux Capabilities introduced?

Capabilities split root privileges into smaller, independent privileges, improving system security.

---

## What is the Principle of Least Privilege?

Grant only the minimum permissions required to perform a task.

---

# Summary

Linux security is built on users, groups, permissions, and privilege management.

```
User

↓

Authentication

↓

Authorization

↓

Permission Check

↓

Kernel

↓

Resource Access
```

Key concepts:

- Linux is a multi-user Operating System.
- Every user has a unique UID.
- Root (UID 0) has unrestricted privileges.
- Files have owners, groups, and permissions.
- `r`, `w`, and `x` define access rights.
- `chmod` changes permissions.
- `chown` changes ownership.
- `sudo` provides temporary elevated privileges.
- SUID, SGID, and Sticky Bit add specialized behaviors.
- Linux Capabilities divide root privileges into fine-grained permissions.
- The Principle of Least Privilege is fundamental to secure system design.

Understanding Linux security is essential before studying namespaces, containers, Docker, Kubernetes, and Linux kernel security mechanisms.

---

# Next Chapter

## Chapter 17 — Operating System Architecture: Monolithic Kernel, Microkernel & Hybrid Kernel

We'll explore:

- Kernel Architecture
- Monolithic Kernel
- Microkernel
- Hybrid Kernel
- Modular Kernel
- Why Linux uses a Monolithic Kernel
- Windows Kernel Architecture
- macOS XNU Kernel
- Performance Trade-offs
- How kernel architecture affects modern operating systems