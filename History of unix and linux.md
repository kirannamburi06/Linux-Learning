# Day 2: History of UNIX and LINUX

Hello everyone.

These are the topics that are going to be covered in this post:
- History of Unix and how it inspired Linux  
- History of Linux and various distributions of Linux  

---

## How It All Started and the Birth of Unix?

Most people use the terms Linux and Unix interchangeably, but there is a difference between them.  
This goes back to the time when Unix was invented.

In the year 1969, at AT&T (American Telephone and Telegraph), which is a telecom company, there was a research facility called Bell Labs, which later became known as the legendary research arm responsible for world-changing inventions like the transistor, laser, Unix, C, etc.

A project named MULTICS (Multiplexed Information and Computing Service), which aimed to develop a multi-user operating system, failed, even though AT&T invested a lot of money in it.  
This led to the discontinuation of the project, but a researcher named Ken Thompson continued experimenting with operating system designs.

They had a PDP-7 minicomputer with a very fast disk drive at that time.  
Since the disk was fast, the available software at that time could not use the disk efficiently, so Ken experimented by writing low-level code to schedule reads/writes, reduce the time for performing operations, and control the disk.

That means he actually made a disk driver and an interface.  
This is basically what the kernel does now, but the term *kernel* had not appeared yet at that time.

At this point, Ken realised that the code he wrote was halfway to a complete OS, because the code already did hardware management, provided a structured way for programs to use the disk, and controlled access to resources.

He realised that he needed three more weeks to develop it into a complete OS prototype, as there were three main components:
- Kernel  
- Assembler  
- Editor  

The kernel is for controlling hardware.  
This was built on his existing work on the disk interface.

The assembler is a program that converts assembly code into low-level machine language.

The editor provides an interface to write and modify code.

He developed each component in one week, built a prototype, and named it UNIX.

But since the code was written in assembly language, and assembly language is machine-dependent (unique for each microprocessor), it could not be used on computers other than the one on which it was written.

At the same time, Dennis Ritchie, another researcher at Bell Labs, developed the C language, which is portable and machine-independent.  
Unix was rewritten in C, and this became a major step in the evolution of Unix.

After that, different editions of Unix were released, but it was never made open source.

Unix introduced commands that we use in Linux today, such as:  
`cat`, `chdir` (like `cd`), `chmod`, `chown`, `cmp`, `cp`, `date`, `df`, `du`, `echo`, `find`, `ln`, `ls`, `man`, `mkdir`, `mount`, `mv`, `rm`, `rmdir`, `sort`, `umount`, `wc`, and `who`.

---

## Birth of Linux

In the year 1983, Richard Stallman started the GNU project (GNU’s Not Unix — a recursive acronym), which aimed to develop a Unix-like OS composed of free and open-source software, meaning the freedom to study, create, modify, and distribute software.

They created and gathered all the required components for their system, such as compilers, libraries, shell, and editors, but the only thing they lacked was a kernel, which is the crucial component for the OS.

In the year 1991, a student from Finland named Linus Torvalds, who was using MINIX (a Unix-like OS with a restricted license), wanted to write his own kernel and started a personal project aiming to develop a free and open-source Unix-like kernel.

He developed a kernel, named it Linux, and made it open source, which was a major step in the evolution of Linux.  
Many developers worldwide contributed to it, and it became a community project.

What actually differentiated Linux from Unix is that Linux is a monolithic kernel and Unix is a microkernel.

A monolithic kernel means all the OS services run together inside the kernel in the same memory space as a single process.  
So there is less context switching and IPC, unlike a microkernel, where most of the services reside in user space and continuously need context switching and IPC, which has a great computational cost.

This made Linux faster and increased its performance.

Later, GNU and Linux were merged, because both are open source, and formed a complete open-source operating system that is technically known as GNU/Linux.

When you run the `uname -o` command on a Linux machine, you can see the output as GNU/Linux, where `-o` means operating system name.

---

## Linux Distributions

Early GNU/Linux was CLI-focused.  
Later, different distributions came into existence.

They use the same Linux kernel and GNU components, but vary significantly in system software, libraries, package management, and configuration tools.  
They also added interactive GUIs, and user experience improved.

There are basically two main families of distributions: Debian and Red Hat.

Debian is community-driven, known for stability and huge package support, and uses APT (Advanced Packaging Tool).  
Red Hat has enterprise-level support, stability, consistency, and security (SELinux), and uses the YUM package manager, which was later updated to DNF (Dandified Yum).

### Debian Family
- Ubuntu  
  - Linux Mint  
  - Pop!_OS  
- Kali Linux  
- MX Linux  
- Deepin  

### Red Hat Family
Red Hat Enterprise Linux (RHEL) is the main product and has children like:
- Fedora (serves as the main testing ground for new features to be introduced into RHEL)  
- Rocky Linux  
- Oracle Linux  
- AlmaLinux  

### Other Linux Families
- Arch Linux (uses the `pacman` package manager)  
  - Garuda  
  - EndeavourOS  
  - Manjaro  
- SUSE family  
  - openSUSE  
- Alpine Linux  
  - used by Docker containers, Kubernetes, and cloud-native environments  
- Void Linux  
  - uses the `runit` service manager  
  - used by advanced users  
