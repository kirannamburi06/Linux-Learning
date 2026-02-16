# Day-08: Linux Boot Process

Hello everyone.

These are the topics that are going to be covered in this post:

- Overview of the boot process
- BIOS/UEFI firmware initialization
- Boot loader stage (GRUB)
- Kernel initialization
- Init system and systemd
- User space initialization

---

## What Happens When You Press the Power Button?

When you press the power button on your computer, a complex sequence of events takes place before you see the login screen or desktop environment. The entire process can be divided into several distinct stages, each with its own responsibility.

The boot process is not instantaneous. It involves hardware initialization, loading critical software, starting system services, and finally preparing the user environment. Understanding this process helps in troubleshooting boot issues and optimizing system performance.

---

## Stage 1: BIOS/UEFI Firmware

The first thing that happens after pressing the power button is the execution of firmware code stored in a chip on the motherboard. This firmware is either BIOS (Basic Input/Output System) or UEFI (Unified Extensible Firmware Interface).

### BIOS (Legacy System)

BIOS is the older firmware standard that has been around since the 1980s. When the system powers on, the CPU is hardwired to execute code from a specific memory address where the BIOS resides.

The BIOS performs POST (Power-On Self-Test), which checks if all the hardware components like RAM, keyboard, storage devices, and other peripherals are working properly. If any hardware fails during POST, the BIOS will either show an error message or emit a series of beep codes indicating the problem.

After POST, the BIOS looks for a bootable device based on the boot order configured in the BIOS settings. It checks devices like hard drives, SSDs, USB drives, CD/DVD drives, or network interfaces.

Once a bootable device is found, the BIOS reads the first sector of that device, which is 512 bytes in size and called the MBR (Master Boot Record). The MBR contains two critical pieces of information:

- Boot loader code (446 bytes)
- Partition table (64 bytes)

The boot loader code in the MBR is extremely limited in size, so it cannot do much. Its job is just to load the next stage of the boot loader from a known location on the disk. This is called chain loading.

### UEFI (Modern System)

UEFI is the modern replacement for BIOS and addresses many of its limitations. Unlike BIOS, which operates in 16-bit real mode, UEFI runs in 32-bit or 64-bit mode and provides a much richer pre-boot environment.

UEFI does not rely on the MBR. Instead, it uses GPT (GUID Partition Table), which supports larger disks (over 2TB), more partitions (up to 128), and includes redundancy with backup partition tables.

UEFI firmware can directly read filesystems. It looks for an EFI System Partition (ESP), which is a special FAT32 partition that contains boot loader files. These files have the .efi extension and are located in directories like `/EFI/BOOT/` or `/EFI/ubuntu/`.

UEFI maintains a list of boot entries in its NVRAM, which stores information about where to find boot loaders. You can manage these entries using tools like `efibootmgr` in Linux.

UEFI also supports Secure Boot, which ensures that only cryptographically signed boot loaders and kernels can execute. This prevents rootkits and bootkits from compromising the system before the OS loads.

The main advantage of UEFI over BIOS is flexibility. UEFI boot loaders can be much larger and more sophisticated because they are not constrained by the 512-byte MBR limitation.

---

## Stage 2: Boot Loader (GRUB)

The boot loader is responsible for loading the Linux kernel into memory and passing control to it. The most common boot loader on Linux systems is GRUB2 (Grand Unified Bootloader version 2).

### How GRUB Works

When GRUB starts, it presents a menu showing available operating systems and kernel versions. You can select which one to boot, or GRUB will automatically boot the default entry after a timeout (usually 5-10 seconds).

GRUB is modular and consists of several components:

- `boot.img` - The very first piece loaded from the MBR or ESP
- `core.img` - Contains filesystem drivers and other essential modules
- `/boot/grub/` - Directory containing configuration files, modules, fonts, and themes

The GRUB configuration file is located at `/boot/grub/grub.cfg`. This file is auto-generated and should not be edited directly. Instead, you modify files in `/etc/default/grub` and `/etc/grub.d/`, then run `update-grub` or `grub-mkconfig` to regenerate the configuration.

Each menu entry in GRUB specifies:

- Which kernel to load (e.g., `/boot/vmlinuz-5.15.0-43-generic`)
- Initial ramdisk to use (e.g., `/boot/initrd.img-5.15.0-43-generic`)
- Kernel parameters to pass (e.g., `root=/dev/sda1 ro quiet splash`)

### Kernel Parameters

Kernel parameters are arguments passed to the kernel at boot time. Common parameters include:

`root=/dev/sda1` - Specifies which partition contains the root filesystem

`ro` - Mounts the root filesystem as read-only initially (it will be remounted as read-write later)

`quiet` - Suppresses most boot messages

`splash` - Shows a graphical splash screen instead of text messages

`init=/bin/bash` - Boots directly into a bash shell (useful for recovery)

`systemd.unit=multi-user.target` - Boots into a specific systemd target (runlevel)

You can temporarily edit kernel parameters at boot time by pressing 'e' in the GRUB menu, modifying the line starting with `linux`, and pressing Ctrl+X or F10 to boot. This is useful for troubleshooting or booting into single-user mode.

### Initial Ramdisk (initrd/initramfs)

Before the kernel can mount the root filesystem, it needs drivers for the storage device and filesystem type. However, these drivers might be compiled as kernel modules rather than built into the kernel itself.

This creates a chicken-and-egg problem: the kernel needs to mount the root filesystem to access modules, but it needs modules to mount the root filesystem.

The solution is the initial ramdisk. GRUB loads both the kernel and a compressed filesystem image (initramfs) into memory. The kernel extracts this image into a temporary root filesystem in RAM.

The initramfs contains essential drivers, utilities like `busybox`, and an init script that:

- Loads necessary kernel modules for storage and filesystem
- Detects hardware and initializes devices
- Assembles RAID arrays or decrypts LUKS volumes if needed
- Mounts the actual root filesystem
- Switches from the temporary ramdisk root to the actual root (pivot_root)

You can see what's inside an initramfs by copying it and extracting it:

```
cp /boot/initrd.img-$(uname -r) /tmp/initrd.img.gz
cd /tmp
gunzip initrd.img.gz
cpio -idv < initrd.img
```

Once the real root filesystem is mounted, the initramfs is no longer needed and is freed from memory.

---

## Stage 3: Kernel Initialization

After GRUB loads the kernel and initramfs into memory, it transfers control to the kernel. The kernel initialization process involves several phases.

### Kernel Decompression

The kernel image (vmlinuz) is actually compressed. The first thing that happens is decompression. The kernel contains a small decompressor stub at the beginning that uncompresses the rest of the kernel.

### Hardware Detection and Driver Loading

The kernel starts by initializing the CPU and memory management subsystems. It sets up paging, enables virtual memory, and initializes the memory allocator.

Next, it detects and initializes hardware devices. The kernel uses device trees, ACPI tables, or PCI enumeration to discover hardware. For each device, it loads the appropriate driver.

Drivers can be compiled into the kernel (built-in) or as loadable kernel modules. Built-in drivers are initialized during boot, while modules are loaded on-demand or by the init system.

### Mounting the Root Filesystem

Using the information from the `root=` kernel parameter and the drivers from initramfs, the kernel mounts the root filesystem. Initially, it is mounted read-only to check for filesystem errors.

The kernel runs `fsck` (filesystem check) if needed, then remounts the root filesystem as read-write so that files can be modified.

### Starting the Init Process

The final step of kernel initialization is starting the init process. The kernel looks for an executable to run as PID 1 (Process ID 1) in the following order:

- `/sbin/init`
- `/etc/init`
- `/bin/init`
- `/bin/sh`

On modern Linux systems, `/sbin/init` is usually a symbolic link to systemd.

The init process becomes the parent of all other processes. It never exits, and if it crashes, the kernel panics.

At this point, the kernel's job is mostly done. It continues to manage hardware, schedule processes, and handle system calls, but the initialization of user space is now the responsibility of the init system.

---

## Stage 4: Init System (systemd)

The init system is the first user-space process and is responsible for bringing the system to a usable state. On most modern Linux distributions, the init system is systemd, though older systems may use SysVinit or Upstart.

### What is systemd?

systemd is a system and service manager that provides a standard way to manage services, handle dependencies, and parallelize startup tasks.

Unlike older init systems that use shell scripts and run tasks sequentially, systemd uses unit files and starts services in parallel when possible, which significantly reduces boot time.

### systemd Units

Everything in systemd is represented as a unit. There are different types of units:

- `.service` - Represents a system service (e.g., `sshd.service`)
- `.target` - Groups other units, similar to runlevels (e.g., `multi-user.target`)
- `.mount` - Represents a mount point (e.g., `/home`)
- `.socket` - Socket-based activation for services
- `.timer` - Similar to cron jobs
- `.device` - Represents hardware devices

Unit files are located in:

- `/lib/systemd/system/` - System-provided units
- `/etc/systemd/system/` - Administrator-created or modified units
- `/run/systemd/system/` - Runtime units

You can view a unit file with `systemctl cat <unit-name>`.

### systemd Targets

Targets are similar to runlevels in SysVinit. They define what state the system should be in. Common targets include:

`poweroff.target` - Shutdown state

`rescue.target` - Single-user mode, minimal services

`multi-user.target` - Multi-user mode with networking, no GUI

`graphical.target` - Multi-user mode with GUI

`reboot.target` - Reboot state

The default target is set by a symbolic link at `/etc/systemd/system/default.target`.

When systemd starts, it activates the default target. Each target has dependencies, which are other units that must be started. systemd resolves these dependencies and starts units in the correct order.

Learn more about systemd here : [Day-07-Systemd-units-services](https://github.com/kirannamburi06/Linux-Learning/blob/main/Day-07-Systemd-units-services.md)

---

## Stage 5: User Space Initialization

After systemd starts essential services, the system enters user space initialization, where user-facing services and the login manager are started.

### Essential Services

systemd starts critical system services like:

`systemd-udevd` - Device manager that handles device events and creates device nodes in `/dev`

`systemd-networkd` or `NetworkManager` - Network configuration and management

`systemd-resolved` - DNS resolver

`systemd-logind` - Session management for user logins

`dbus-daemon` - Inter-process communication bus used by many applications

`cron` or `systemd-timers` - Scheduled task execution

`rsyslog` or `systemd-journald` - System logging

### Display Manager (Graphical Login)

If the system is configured to boot into a graphical environment (`graphical.target`), systemd starts a display manager. Common display managers include:

`gdm` (GNOME Display Manager)

`lightdm` (Lightweight display manager)

`sddm` (Simple Desktop Display Manager, used by KDE)

`xdm` (X Display Manager, very basic)

The display manager presents a graphical login screen where users can enter their credentials. After successful authentication, it starts the user's chosen desktop environment or window manager.

### Login Process (Console or Graphical)

For console logins, systemd starts `getty` on virtual terminals (typically tty1 through tty6). getty prompts for a username, then executes `login`, which prompts for a password and authenticates the user.

After authentication, `login` starts the user's shell (specified in `/etc/passwd`), and the shell executes startup scripts:

- `/etc/profile` - System-wide profile
- `~/.bash_profile`, `~/.bash_login`, or `~/.profile` - User-specific login scripts
- `~/.bashrc` - Executed for non-login shells

For graphical logins, the display manager authenticates the user (using PAM - Pluggable Authentication Modules), then starts the user's session by executing their `.xinitrc` or `.xsession` file, or by launching the desktop environment directly.

### Desktop Environment Startup

The desktop environment (GNOME, KDE, XFCE, etc.) is a collection of programs that provide a graphical interface. It includes:

- Window manager (manages window placement, decorations, focus)
- Panel (taskbar showing open windows, system tray)
- File manager
- Settings manager
- Session manager (handles logout, lock screen, etc.)

The desktop environment starts its own set of background services and applications defined in autostart entries located in `/etc/xdg/autostart/` and `~/.config/autostart/`.

At this point, the boot process is complete, and the system is ready for the user to interact with.

---

## Understanding Boot Time

You can analyze your system's boot time using systemd tools.

`systemd-analyze` - Shows total boot time

`systemd-analyze blame` - Lists services by how long they took to start

`systemd-analyze critical-chain` - Shows the critical path of service dependencies

`systemd-analyze plot > boot.svg` - Generates a graphical timeline of the boot process

This helps identify bottlenecks and optimize boot performance.

---

## Troubleshooting Boot Issues

Understanding the boot process is crucial for troubleshooting. Common issues include:

**System doesn't boot past GRUB** - Likely a kernel or initramfs problem. Try booting an older kernel from the GRUB menu.

**System boots but services fail** - Check service status with `systemctl status <service>` and logs with `journalctl -xe`.

**Kernel panic** - Usually due to missing root filesystem or wrong root device. Check kernel parameters.

**Bootloader missing or corrupted** - Reinstall GRUB using a live USB with `grub-install` and `update-grub`.

**Filesystem errors** - Run `fsck` from a live environment. Never run fsck on a mounted filesystem.

You can boot into emergency mode by adding `systemd.unit=emergency.target` to kernel parameters, which gives you a root shell before most services start.

For rescue mode with more services, use `systemd.unit=rescue.target`.

---

So, if we keep it all together,

The Linux boot process involves multiple stages:

1. Firmware (BIOS/UEFI) initializes hardware and loads the boot loader
2. Boot loader (GRUB) loads the kernel and initramfs
3. Kernel initializes hardware, mounts the root filesystem, and starts init
4. Init system (systemd) starts services and brings the system to the target state
5. User space initialization completes with login managers and desktop environments

Each stage depends on the previous one, and understanding this chain helps in diagnosing problems and optimizing system performance.
