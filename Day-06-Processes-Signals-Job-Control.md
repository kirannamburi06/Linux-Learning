# Day 06: Processes, Process Lifecycle, Signals, and Job Control

Hello everyone.

In this part, we are going to explore **processes**, how they are created, how they live, how they die, and how Linux lets us control them.  
Everything is a process in linux.

---

## What Is a Process?

A **process** is a running instance of a program.

A program is just a file on disk.  
A process is that program **loaded into memory and executed by the CPU**.

Linux treats every running activity as a process, whether it is a user command, a background service, or a system daemon.

A program resides on Hard disk or any other secondary storage, but a process resides in main memory.
A process will be allocated memory, cpu time and other resources that are essential for the process to run, and when it completes execution, the allocated resources will be freed, so that they can be allocated for other processes.

---

## Process Identity

Linux identifies each process using numbers, not names.

Every process has:

- a **PID** (Process ID)
- a **PPID** (Parent Process ID)
- an owning **UID** and **GID**

These identifiers allow the kernel to track relationships, permissions, and resource usage.

---

## Parent and Child Processes

Linux processes follow a **parent–child model**.

When a process creates another process, the original process becomes the **parent**, and the new one becomes the **child**.

The shell acts as a parent process most of the time.  
Every external command you run becomes a child of the shell.

---

## Process Creation: fork and exec

Linux creates processes in two distinct steps.

First, the parent calls `fork()`.  
This creates a new process that is an almost exact copy of the parent.

Then, the child usually calls `exec()`.  
This replaces the child’s memory with a new program.

Since `exec()` replaces the binaries of process, creating another process is important, otherwise the original process will be replaced into another and we lose our original process.
The shell needs to survive after commands finish.  
The child needs to transform into a different program.

This design allows redirection, pipes, background execution, and job control to work cleanly.

---

## Process Memory View

Each process sees its own **virtual address space**.

The kernel isolates processes from each other.  
One process cannot directly access another process’s memory.

This isolation is the foundation of system stability and security.

---

## Process States (High-Level View)

A process does not always run.

At any moment, a process can be:

- waiting for CPU time inside the ready queue
- running on a CPU
- waiting for I/O
- sleeping
- stopped
- terminated

The kernel scheduler constantly moves processes between these states.

---

## Common Linux Process States

Linux represents process states using letters.

- **R** means running or runnable
- **S** means sleeping
- **D** means uninterruptible sleep (usually I/O)
- **T** means stopped
- **Z** means zombie

These states appear in tools like `ps` and `top`.

---

## Zombie Processes

A **zombie process** has finished execution but still exists in the process table.

This happens when:

- the child exits
- the parent does not collect the exit status

Zombies do not consume CPU or memory, but they occupy process table entries.
At some point, it exhausts the process ids, so that a new process has no id left to be allocated.

---

## Orphan Processes

An **orphan process** is a running process whose parent has exited.

Here, since the parent left, there is no process to collect its exit status or manage its lifecycle. So the kernel reassigns orphan processes to PID 1.  
PID 1 ensures proper cleanup and lifecycle management.
Just like the name, PID 1 is the first process ever to be started when linux boots.
It is like the parent for all processes.
PID 1 is called **systemd**

---

## Process Termination

A process can terminate in multiple ways.

It may:

- finish normally
- receive a signal
- crash due to an error
- be killed by another process

The kernel records an exit status for every terminated process.

---

## What Are Signals?

A **signal** is an asynchronous notification sent to a process.

Signals allow the kernel or another process to interrupt execution and request an action.

Signals form one of the oldest and most important IPC mechanisms in Unix.

---

## Why Signals Exist

Processes cannot constantly poll for events.

Signals allow Linux to:

- notify processes instantly
- handle errors
- manage process control

Without signals, process coordination would be inefficient and complex.

---

## Common Signals You Should Know

Some signals appear constantly in daily Linux usage.

- `SIGTERM` asks a process to terminate gracefully
- `SIGKILL` forces immediate termination
- `SIGINT` interrupts a process (Ctrl+C)
- `SIGHUP` requests configuration reload
- `SIGSTOP` stops a process
- `SIGCONT` resumes a stopped process

Each signal carries a specific intention.

---

## Signal Handling

Processes can **handle**, **ignore**, or **block** most signals.

The kernel delivers the signal.  
The process decides how to respond.

`SIGKILL` and `SIGSTOP` are exceptions.  
The kernel enforces them unconditionally.

---

## Foreground and Background Processes

The shell introduces the concept of **job control**.

A foreground process:

- owns the terminal
- receives keyboard input

A background process:

- runs without terminal control
- cannot read input from the terminal

The shell manages this distinction, not the kernel.

---

## Jobs vs Processes

A **job** is a shell-level concept.

A job may contain:

- one process
- or a pipeline of multiple processes

The shell assigns job IDs.  
The kernel only tracks processes.

---

## Starting Background Jobs

You can start a background job using `&`.

The shell:

- launches the process
- immediately returns control to the user

The process continues execution independently of the prompt.

---

## Stopping and Resuming Jobs

The shell allows interactive control over jobs.

- `Ctrl+Z` stops the foreground job
- `bg` resumes a job in the background
- `fg` brings a job to the foreground

These commands manipulate process states using signals.

---

## How Job Control Works Internally

When you press `Ctrl+Z`, the terminal sends `SIGTSTP`.

The shell:

- marks the job as stopped
- updates its job table

When you run `fg`, the shell sends `SIGCONT` and restores terminal control.

---

## Process Groups and Job Control

Linux groups related processes into **process groups**.

Each job runs in its own process group.  
Signals from the terminal target the entire group.

This design ensures pipelines behave as a single controllable unit.

---

So, if we summarize,

A process represents **execution**.  
A signal represents **control**.  
A job represents **shell-level management**.

Together, they form Linux multitasking.

---
