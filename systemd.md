What is systemd?

Systemd is a system and service manager that has the pid 1 meaning it is the first process to be run when the system boots.
It is responsible for:
starting the system
starting and stopping services, and restarting failed services
tracking dependencies
managing logs
reaps zombies and orphans become child to systemd.

Before systemd, linux is using SystemV init, which is a copied version of init system of SystemV unix.But
there are a lot of disadvantages in the SystemV init.
SystemV init uses runlevels from 0-6 and shell scripts. The starting of services is sequential and all other services will be waiting until then.
For example, in runlevel 03,
/etc/rc3.d/
   S10network
   S20syslog
   S30sshd
   S40nginx
Here the S<number> decides the order of startup.
Here there is no global dependency graph and the init system does not know which service depends on which service, so it is not able to load services parallely, and the run levels are pre defined and the services load according to that order. This is very slow.

But systemd is different. It does not use numbered scripts. It uses unit files, dependencies and dependency graphs.
That means, instead of:
start service A
then service B
then service C

systemd uses:
Service B requires A
Service C wants B

So, here the systemd creates a dependency graph, and it can know which service depends on which service, and it is able to start services parallely that does not depend on any other services to load beforehand. This makes systemd fast and less error prone.

Example:
network.service
db.service
logging.service

These dont depend on each other, so they are started parallely.

Systemd can also delay starting services until they are needed. This is called on-demand activation of daemons.
For example the ssh service sshd does not start at boot, rather the systemd listens on port 22 and only starts sshd if there is a connection request.

Systemd is backward compatible with SysV init, meaning, those old script files of SysV init can still run on systemd so that it adds the flexibility to move from sysv init to systemd without stopping the system.

Unit files:

Everything that systemd manages is a unit. A unit file is a configuration file.
You can determine the type of unit file by seeing at its extension.

Unit files reside at three main paths:
/etc/systemd/system/
/run/systemd/system/
/lib/systemd/system/

These are ordered from highest priority to lowest. The systemd starts looking for units from top to bottom in those file paths.
When you do edits on those config files, the changes are saved in the /etc/systemd/system path.
The command to do so is : sudo systemctl edit <unit file name>

Structure of a unit file:

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

Unit section: 
Contains metadata and dependencies.
Metadata include description(summarize what this unit does), documentation(information about the unit, commonly man pages).
Dependencies include:
    After : Ordering. Start after something. Example = After=network.target.
    Before : Ordering. Start before something.
    Requires : Hard Dependency. Example = Requires=network.target. If the network.target fails, this also fails.
    Wants : Soft Dependency. 
    Conflicts : cannot run together.

Service section:
What the service does. Unique for each unit type like service, mount, path etc.

Install section:
This is used when a service should start after boot automatically. 
Example:
[Install]
WantedBy=multi-user.target

When we run this command : systemctl enable myunit.service, the systemd will create a symbolic link that points to the multi-user.target, and whenever the target is achieved, the service will run.

Unit file types:

1) .services (or) Daemons :
    This unit type represents a daemon, which is a background process, that runs without user interaction and does system tasks or an application that systemd should manage.
    It includes how to start, stop, reload the program, defines its dependencies on other units, and whether it should start or not at boot.
    
    Examples: sshd.service, docker.service, httpd.service etc..

2) .target : 
    A target is a special type of systemd unit, that groups other units, to achieve a specific system state or goal. A target is also called a synchronization point.
    Example:
    graphical.target
    multi-user.target
    rescue.target

    targets are the replacements for runlevels in sysV init.

    View default target :
    systemctl get-default
    
    View current active targets :
    systemctl list-units --type target

    View all targets even the ones that are not actice:
    systemctl list-units --type target --all

    Change default target:
    systemctl set-default name.target

    Change the current target:
    systemctl isolate name.target

3) .timer : 
    A .timer unit schedules the activation of another unit, usually a service at a specific time or interval. That means it triggers a service.
    The file name of .timer unit should be same as the file name of .service unit file, so that when the time comes, it triggers the service unit.
    systemd supports two different timing models
    (i) Based on calendar times:
        OnCalendar=Mon *-*-* 02:00:00
        This means trigger every monday at 2AM.
    (ii) Based on monotonic timers meaning event based:
            OnBootSec=5min
            OnUnitActiveSec=1h
            This means, schedule the service after 5 sec after boot, and after 1hour the service last ran.
    
    Timer is a replacement for cron in SysV init. Lets say the system is shutted down before a scheduled service should happen. Then it is automatically triggered on the next boot. But cron cannot do that, which results in loss of services or jobs.

4) .socket : 
    A socket unit tells systemd to listen on a communication endpoint and start the service only when activity occurs on that endpoint. So the services are started on-demand. Without sockets, all services should be started at the boot, so boot time is slow, and if they are not used, cpu time and resources will be wasted.

5) .mount : 
    A mount unit tells systemd how to mount a filesystem at a specific mount point. It defines what to mount, where to mount, when to mount, type of file system(like ext4) etc..
    This is a native(modern) way of mounting file systems. The old way is to use /etc/fstab.
    File name convention:
    Rule:
    / → -.mount
    /home → home.mount
    /var/log → var-log.mount
    /mnt/data → mnt-data.mount

    Slashes / become dashes -.

6) .path : 
    Runs a service whenever a change happens in the path specified. This is done by inotify API, which watches over the path and triggers the service.

Managing services:

1) Starting a service:
    systemctl start name.service
2) Stopping a service:
    systemctl stop name.service
3) Restarting a service:
    systemctl restart name.service
    It actually stops and starts the service.
    Here systemd terminates the process, memory is freed, and when started again, it gets a new pid with new memory and reads configs again from scratch.
4) Reload a service:
    systemctl reload name.service
    It wont stop the process, rather it only re-reads the config files.
5) Enable a service:
    systemctl enable name.service
    Every time you boot, it starts.
6) Disable a service:
    systemctl disable name.service
7) Check status:
    systemctl status name.service

Service types:

1) Type=simple:
    The process stays in foreground.
    No forking.
    Modern programs use this.

2) Type=forking
    Service forks itself into background, and parent exits. The systemd becomes parent for it.
    used by legacy services.

3) Type=notify
    systemd waits for readiness signal and service notifies it by sd_notify().

4) Type=idle
    service starts after all system startup tasks are completed. Used for non critical tasks.

