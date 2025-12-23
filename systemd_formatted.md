# What is systemd?

Systemd is a system and service manager that has PID 1, meaning it is the first process to be run when the system boots.

It is responsible for:
- starting the system
- starting and stopping services, and restarting failed services
- tracking dependencies
- managing logs
- reaping zombies, and orphan processes become children of systemd

---

## Before systemd

Before systemd, Linux was using SystemV init, which is a copied version of the init system of SystemV Unix.

But there are a lot of disadvantages in the SystemV init.

SystemV init uses runlevels from 0–6 and shell scripts. The starting of services is sequential, and all other services will be waiting until then.

For example, in runlevel 03:

```
/etc/rc3.d/
   S10network
   S20syslog
   S30sshd
   S40nginx
```

Here the S<number> decides the order of startup.

Here there is no global dependency graph, and the init system does not know which service depends on which service, so it is not able to load services in parallel, and the runlevels are predefined and the services load according to that order. This is very slow.

---

## How systemd is different

But systemd is different. It does not use numbered scripts. It uses unit files, dependencies, and dependency graphs.

That means, instead of:

```
start service A
then service B
then service C
```

systemd uses:

```
Service B requires A
Service C wants B
```

So here, systemd creates a dependency graph, and it can know which service depends on which service, and it is able to start services in parallel that do not depend on any other services to load beforehand. This makes systemd fast and less error-prone.

### Example

```
network.service
db.service
logging.service
```

These do not depend on each other, so they are started in parallel.

---

## On-demand activation

Systemd can also delay starting services until they are needed. This is called on-demand activation of daemons.

For example, the SSH service `sshd` does not start at boot; rather, systemd listens on port 22 and only starts `sshd` if there is a connection request.

---

## Backward compatibility

Systemd is backward compatible with SysV init, meaning those old script files of SysV init can still run on systemd, so that it adds flexibility to move from SysV init to systemd without stopping the system.

---

## Unit files

Everything that systemd manages is a unit. A unit file is a configuration file.

You can determine the type of unit file by seeing its extension.

Unit files reside at three main paths:
- /etc/systemd/system/
- /run/systemd/system/
- /lib/systemd/system/

These are ordered from highest priority to lowest. Systemd starts looking for units from top to bottom in these file paths.

When you do edits on those config files, the changes are saved in the `/etc/systemd/system` path.

The command to do so is:
```bash
sudo systemctl edit <unit file name>
```

---

## Structure of a unit file

```
[Unit]
Description=My Demo Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/demo/app.py
Restart=always
User=demo
Group=demo

[Install]
WantedBy=multi-user.target
```

### Unit section

Contains metadata and dependencies.

Metadata include description (summarizes what this unit does) and documentation (information about the unit, commonly man pages).

Dependencies include:
- After: Ordering. Start after something. Example = After=network.target.
- Before: Ordering. Start before something.
- Requires: Hard dependency. Example = Requires=network.target. If the network.target fails, this also fails.
- Wants: Soft dependency.
- Conflicts: Cannot run together.

---

### Service section

What the service does. Unique for each unit type like service, mount, path, etc.

---

### Install section

This is used when a service should start automatically after boot.

Example:
```
[Install]
WantedBy=multi-user.target
```

When we run this command:
```bash
systemctl enable myunit.service
```

systemd will create a symbolic link that points to the `multi-user.target`, and whenever the target is achieved, the service will run.

---

## Unit file types

### 1) .service (or) Daemons

This unit type represents a daemon, which is a background process that runs without user interaction and does system tasks, or an application that systemd should manage.

It includes how to start, stop, and reload the program, defines its dependencies on other units, and whether it should start or not at boot.

Examples:
- sshd.service
- docker.service
- httpd.service

---

### 2) .target

A target is a special type of systemd unit that groups other units to achieve a specific system state or goal. A target is also called a synchronization point.

Examples:
- graphical.target
- multi-user.target
- rescue.target

Targets are the replacements for runlevels in SysV init.

View default target:
```bash
systemctl get-default
```

View current active targets:
```bash
systemctl list-units --type target
```

View all targets, even the ones that are not active:
```bash
systemctl list-units --type target --all
```

Change default target:
```bash
systemctl set-default name.target
```

Change the current target:
```bash
systemctl isolate name.target
```

---

### 3) .timer

A .timer unit schedules the activation of another unit, usually a service at a specific time or interval.

The file name of the .timer unit should be the same as the file name of the .service unit file, so that when the time comes, it triggers the service unit.

Systemd supports two different timing models.

(i) Based on calendar times:
```
OnCalendar=Mon *-*-* 02:00:00
```
This means trigger every Monday at 2 AM.

(ii) Based on monotonic timers, meaning event-based:
```
OnBootSec=5min
OnUnitActiveSec=1h
```
This means schedule the service 5 minutes after boot, and 1 hour after the service last ran.

Timer is a replacement for cron in SysV init.

If the system is shut down before a scheduled service should happen, then it is automatically triggered on the next boot. But cron cannot do that, which results in loss of services or jobs.

---

### 4) .socket

A socket unit tells systemd to listen on a communication endpoint and start the service only when activity occurs on that endpoint.

So the services are started on-demand. Without sockets, all services should be started at boot, so boot time is slow, and if they are not used, CPU time and resources will be wasted.

---

### 5) .mount

A mount unit tells systemd how to mount a filesystem at a specific mount point.

It defines what to mount, where to mount, when to mount, and the type of filesystem (like ext4), etc.

This is a native (modern) way of mounting filesystems. The old way is to use `/etc/fstab`.

File name convention:
```
/         → -.mount
/home     → home.mount
/var/log  → var-log.mount
/mnt/data → mnt-data.mount
```

Slashes `/` become dashes `-`.

---

### 6) .path

Runs a service whenever a change happens in the path specified.

This is done by the inotify API, which watches over the path and triggers the service.

---

## Managing services

1) Starting a service:
```bash
systemctl start name.service
```

2) Stopping a service:
```bash
systemctl stop name.service
```

3) Restarting a service:
```bash
systemctl restart name.service
```
It actually stops and starts the service.

Here systemd terminates the process, memory is freed, and when started again, it gets a new PID with new memory and reads configs again from scratch.

4) Reload a service:
```bash
systemctl reload name.service
```
It will not stop the process; rather, it only re-reads the config files.

5) Enable a service:
```bash
systemctl enable name.service
```
Every time you boot, it starts.

6) Disable a service:
```bash
systemctl disable name.service
```

7) Check status:
```bash
systemctl status name.service
```

---

## Service types

### 1) Type=simple
The process stays in the foreground.
No forking.
Modern programs use this.

### 2) Type=forking
Service forks itself into the background, and the parent exits.
Systemd becomes the parent for it.
Used by legacy services.

### 3) Type=notify
Systemd waits for a readiness signal, and the service notifies it using sd_notify().

### 4) Type=idle
The service starts after all system startup tasks are completed.
Used for non-critical tasks.
