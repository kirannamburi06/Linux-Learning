# Day 4: Shell and Terminal – What They Are and What Actually Happens When You Run a Command

Hello everyone.

In this post, these topics are covered:

- What a terminal is
- What a shell is
- The difference between shell and terminal
- What exactly happens internally when we run a command
- How fork, exec, and processes come into play

Most beginners use the words _terminal_, _shell_, and _command line_ interchangeably, but internally they are **very different components**, each having a specific role.

---

## What Is a Terminal?

A **terminal** is simply an **interface for input and output**.

Historically, terminals were physical devices:

- keyboard for input
- screen or printer for output

They were connected to a powerful central computer.

Today, the _terminal_ is usually a **terminal emulator**, which is just a program that:

- takes your keyboard input
- displays text output
- sends input to another program (the shell)

Examples of terminal emulators:

- GNOME Terminal
- Konsole
- xterm
- Alacritty

> The terminal **does not understand commands**.  
> It only displays text and forwards input.

So when you type:

```bash
ls
```

The terminal is **not** executing `ls`.

---

## What Is a Shell?

A **shell** is a **command interpreter**.

It is a normal user-space program whose job is to:

- read commands from input
- parse them
- ask the kernel to execute programs

Examples of shells:

- `bash`
- `sh`
- `zsh`
- `fish`

---

## Relationship Between Terminal and Shell

```
Keyboard → Terminal → Shell → Kernel
Kernel → Shell → Terminal → Screen
```

- Terminal: input/output
- Shell: logic and command execution
- Kernel: actual execution and resource management

---

## What Actually Happens When You Run a Command?

If you type:

```bash
ls
```

### Step 1: Terminal Sends Input

- Terminal reads keystrokes
- Sends the string `ls` to the shell

---

### Step 2: Shell Parses the Command

The shell:

- tokenizes input (`ls`) means, it divides commands, options and input.
- checks whether it is a built-in command or external program. If it is a builtin, forking doesnot happen.
- Then it searches for the binary in `$PATH`

If found, the shell prepares to execute it.

---

### Step 3: Shell Calls `fork()`

The shell **creates a child process** using `fork()`.

Important facts about `fork()`:

- Child is an exact copy of the shell
- Same memory (initially, via copy-on-write)
- Same environment
- Different PID

Now there are **two processes**:

- Parent → shell
- Child → future command

---

### Step 4: Child Calls `exec()`

The child process calls an `exec()` family function.

What `exec()` does:

- replaces the child’s memory image
- loads the command’s binary (`/bin/ls`)
- execution starts from the program’s entry point

> `exec()` does **not** create a new process.  
> It **replaces** the current one.

So now:

- the child is no longer a shell
- it has become `ls`

---

### Step 5: Kernel Executes the Program

The kernel:

- loads the binary
- sets up memory
- maps libraries
- switches CPU to user mode
- starts executing the program

The program runs, produces output, and exits.

---

### Step 6: Child Exits and Parent Waits

When `ls` finishes:

- child process exits
- kernel sends exit status to parent

The shell:

- waits using `wait()`
- regains control
- prints a new prompt

This is why the shell **does not disappear** after running a command.

---

## Why fork the process to create a child?

- shell must survive after command execution
- shell must manage multiple commands
- shell can do redirection, pipes, background jobs

Example:

```bash
ls | grep txt
```

Here:

- multiple child processes
- pipes between them
- shell coordinates everything

This design is one of the core strengths of Unix-like systems.

---

## Built-in Commands vs External Commands

Not all commands behave the same.

### Built-in Commands

Some commands are **built into the shell itself**.

Examples:

- `cd`
- `exit`
- `export`
- `alias`

These **do not create a new process**.

Reason:
`cd` must change the shell’s own working directory.
If it ran in a child process, the change would be lost.

---

### External Commands

Most commands are **external programs**.

Examples:

- `ls`
- `cat`
- `grep`
- `touch`
- `ps`

These are actual binaries located in directories like:

- `/bin`
- `/usr/bin`

These **do create new processes**.

---
