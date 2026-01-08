# Day 5: Users, Groups, and Permissions – How Linux Controls Access

Hello everyone.

In this post, we are going to deeply understand **users, groups, and permissions** in Linux.  
This is one of the most fundamental topics, because **everything in Linux is built around access control**.

It is foundation for concepts like sudo, setuid, ACLs, containers, and security models.

---

## Why Linux Needs Users and Permissions

Linux is a **multi-user operating system** by design.

This means:

- many users can exist at the same time
- users can run programs simultaneously
- users must not interfere with each other

So Linux needs a strict way to answer one question:

> Who is allowed to do what, on which resource?

That answer comes from **users, groups, and permissions**.

---

## What Is a User?

A **user** is an identity in the system.

Internally, Linux does not care about usernames.  
It only cares about a number called **UID (User ID)**.

Every user has:

- a username
- a UID (machine-friendly)
- a home directory
- a default shell

User information is stored in:

```
/etc/passwd
```

---

## Types of Users in Linux

Linux generally has three categories of users.

**Root user**

- UID = 0
- unlimited privileges
- can do anything on the system

**System users**

- used by services and daemons
- usually UID < 1000
- no login shell

**Normal users**

- real people using the system
- UID ≥ 1000 (on most distros)

---

## What Is a Group?

A **group** is a collection of users.

Groups exist to:

- share access to files
- simplify permission management
- avoid giving permissions user-by-user

Just like users have UIDs, groups have **GIDs (Group IDs)**.

Group information is stored in:

```
/etc/group
```

---

## Primary Group vs Supplementary Groups

Every user has:

- **one primary group**
- **zero or more supplementary groups**

The **primary group**:

- is assigned at user creation
- becomes the default group for new files

Supplementary groups:

- allow access to shared resources
- do not affect file creation by default

This is why files show **one user and one group**, not multiple groups.

---

## Why Groups Are Important

Without groups:

- every file permission would need per-user control
- administration would be impossible

With groups:

- permissions are applied once
- all group members benefit

Groups are the **scalable permission model** of Unix.

---

## File Ownership in Linux

Every file and directory has:

- one owner (user)
- one group

You can see this using:

```bash
ls -l
```

Example:

```
-rw-r--r-- 1 alice devs file.txt
```

This means:

- owner = alice
- group = devs

Ownership is the foundation of permission checks.

---

## Permission Model: The Three Classes

Linux permissions are divided into **three classes**.

1. **User (owner)**
2. **Group**
3. **Others**

Every access check follows this order:

1. Is the user the owner?
2. Else, is the user in the group?
3. Else, apply others permissions

Only **one class applies**, never multiple.

---

## Permission Types: r, w, x

Linux has exactly **three permission bits**.

- **read (r)**  
  allows reading file contents or listing directory contents

- **write (w)**  
  allows modifying file contents or directory entries

- **execute (x)**  
  allows executing a file or entering a directory

These bits mean different things for files and directories.

---

## Directory Permissions

For directories:

- **read (r)** → list filenames
- **write (w)** → create/delete/rename entries
- **execute (x)** → access files inside

Without execute permission:

> You cannot `cd` into a directory, even if you can read it.

This is why directory permissions matter more than file permissions.

---

## Permission Representation

Permissions can be shown in two ways.

**Symbolic**

```
rwxr-xr--
```

**Numeric (octal)**

```
755
```

Mapping:

- r = 4
- w = 2
- x = 1

So:

- rwx = 7
- r-x = 5
- r-- = 4

---

## How Permission Checks Actually Work

When a process tries to access a file:

1. Kernel checks UID against file owner
2. If matched → user permissions apply
3. Else checks GID against file group
4. If matched → group permissions apply
5. Else → others permissions apply

There is **no combining** of permissions.

---

## Why Root Bypasses Permissions

Root (UID 0):

- bypasses permission checks
- can read/write/execute regardless of bits

This is why root is dangerous and powerful.

---

## Groups and File Creation

When a file is created:

- owner = creating user
- group = user’s **primary group**

Supplementary groups do not affect this unless:

- setgid is used on directories

---
