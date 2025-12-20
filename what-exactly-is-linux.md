# Day 1 – What Exactly Is Linux?

## Introduction

Hello everyone.  
I am going to tell you what I know and learnt about Linux, and by the end of this,  
I hope everyone will get a clear idea of what Linux actually is.

When you hear the word *Linux*, the first thing that usually comes to mind is an **operating system**.  
Let me clear this common misconception.

Most people think Linux is an operating system, but actually, **Linux is a kernel, not a full OS**.

Before going deeper, let us first understand what an OS and a kernel are.

---

## What Is an Operating System?

An operating system is a software — or more accurately, a **collection of programs** — that sits between your computer hardware and you.

Yes!

It is responsible for managing hardware and providing services like:
- Process management
- File management
- I/O management
- User interface
- Memory management

In simple words, the OS makes the hardware usable for applications and users.

---

## What Is a Kernel?

The kernel is a **subset of the operating system** and the **most important part** of it.

When a system boots, the kernel is loaded into memory and runs in a **privileged mode**.  
This means it has full access to the CPU, memory, and hardware.

You can think of the kernel as the *core authority* of the system — almost like a god.

The kernel is responsible for things like:
- CPU scheduling
- Memory allocation and deallocation
- Context switching
- Communicating with hardware devices

User programs do **not** have direct access to hardware.  
Only the kernel does.

---

## A Simple Example

Let us say you are running an application that needs input from the keyboard.

The application **cannot directly read your keystrokes**.  
It does not have that level of permission.

So what does it do?

It makes a **system call**.

The kernel receives that request and communicates with the hardware —  
in this case, the keyboard — and then passes the data back to the application.

This is how applications safely interact with hardware.

---

## Then What Is Linux?

Linux is **the kernel** used by many operating systems.

You may have heard of Linux distributions like:
- Ubuntu
- Fedora
- Linux Mint
- Kali Linux
- Arch Linux

All of them use **the same Linux kernel** — that is the similarity between them.

What makes them different is everything **around** the kernel:
- Tools
- Package managers
- Desktop environments
- Security configurations
- Default software choices

---

## What’s Next?

In the next post, I will talk about:
- The history of Linux
- How Linux is similar to and different from Unix
- Different Linux distribution families

