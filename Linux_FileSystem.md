# Linux Filesystem Explained in Simple English

## Introduction

Linux organizes everything into folders (directories).

Think of Linux like a house:

- Programs live in one room.
- Settings live in another room.
- User files live in another room.
- Logs and temporary files live somewhere else.

This guide explains the most common Linux directories in very simple English.

---

# Linux Root Directory (/)

Everything starts from:

```text
/
```

This is called the **root directory**.

All other directories exist inside it.

Example:

```text
/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── srv
├── sys
├── tmp
├── usr
└── var
```

---

# /boot

## Purpose

Contains files needed to start Linux.

## Examples

```text
/boot/vmlinuz
/boot/grub
```

### Simple Example

When you power on a computer:

```text
Computer Starts
      ↓
/boot files load
      ↓
Linux starts
```

### Easy Memory

```text
/boot = Start Linux
```

---

# /usr

## Purpose

Contains most programs and libraries.

Think of it as the main software area of Linux.

### Contains

```text
/usr/bin
/usr/sbin
/usr/lib
```

### Easy Memory

```text
/usr = Programs and software
```

---

# /usr/bin

## Purpose

Stores normal user programs.

### Examples

```text
/usr/bin/ls
/usr/bin/cat
/usr/bin/cp
/usr/bin/python3
```

### Example

Command:

```bash
ls
```

Actual program:

```bash
/usr/bin/ls
```

### Easy Memory

```text
/usr/bin = User programs
```

---

# /usr/sbin

## Purpose

Stores system administration programs.

These are mostly used by administrators.

### Examples

```text
/usr/sbin/fdisk
/usr/sbin/useradd
/usr/sbin/fsck
```

### Easy Memory

```text
/usr/sbin = Admin programs
```

---

# What is a Program?

A program is a tool that performs a job.

Examples:

```text
ls      -> Show files
cp      -> Copy files
cat     -> Read file contents
python3 -> Run Python
```

Think of a program as:

```text
Program = Worker
```

---

# /usr/lib

## Purpose

Stores libraries.

Libraries help programs work.

### Examples

```text
/usr/lib/libc.so.6
/usr/lib/libm.so
```

---

# What is a Library?

A library is reusable code used by many programs.

Instead of every program writing the same code:

```text
Program A
Program B
Program C
```

All programs share a library.

```text
Program A \
Program B  ---> Library
Program C /
```

### Easy Memory

```text
Program = Worker

Library = Toolbox used by the worker
```

---

# /lib

## Purpose

Stores shared libraries.

In modern Linux:

```text
/lib -> /usr/lib
```

This means:

```text
/lib
```

is usually a shortcut (link) to:

```text
/usr/lib
```

### Easy Memory

```text
/lib = Shared helper files for programs
```

---

# /etc

## Purpose

Stores configuration files.

Configuration means:

```text
Settings
```

### Examples

```text
/etc/hostname
/etc/passwd
/etc/ssh/sshd_config
```

### Example

Hostname file:

```text
server01
```

### Easy Memory

```text
/etc = Settings
```

---

# /var

## Purpose

Stores changing data.

### Examples

```text
Logs
Cache
Temporary data
```

### Common Subdirectories

```text
/var/log
/var/cache
/var/tmp
```

### Easy Memory

```text
/var = Frequently changing files
```

---

# /var/log

## Purpose

Stores logs.

Logs record system events.

### Examples

```text
/var/log/syslog
/var/log/auth.log
```

### Stores

```text
Errors
Warnings
Login records
System messages
```

### Easy Memory

```text
/var/log = System diary
```

---

# /home

## Purpose

Stores regular user files.

### Examples

```text
/home/john
/home/mary
/home/ahamed
```

### Example

```text
/home/ahamed/Documents
/home/ahamed/Downloads
```

### Easy Memory

```text
/home = Users' personal folders
```

---

# /root

## Purpose

Home directory of the root user.

### Example

```text
/root/script.sh
```

### Difference

Regular user:

```text
/home/ahamed
```

Administrator:

```text
/root
```

### Easy Memory

```text
/root = Administrator's home
```

---

# /opt

## Purpose

Stores optional third-party software.

### Examples

```text
/opt/google
/opt/microsoft
/opt/myapp
```

### Easy Memory

```text
/opt = Extra software
```

---

# /srv

## Purpose

Stores data used by services.

Examples:

```text
Web servers
FTP servers
```

### Example

```text
/srv/www/index.html
```

### Easy Memory

```text
/srv = Data served to users
```

---

# /tmp

## Purpose

Stores temporary files.

Many systems clear these files after reboot.

### Examples

```text
/tmp/test.txt
/tmp/app.tmp
```

### Easy Memory

```text
/tmp = Temporary files right now
```

---

# /var/tmp

## Purpose

Stores temporary files that may need to stay after reboot.

### Examples

```text
/var/tmp/download.tmp
/var/tmp/backup.tmp
```

### Easy Memory

```text
/var/tmp = Temporary but keep longer
```

---

# Difference Between /tmp and /var/tmp

## /tmp

Use when:

```text
File is needed only temporarily
Can be deleted after reboot
```

Example:

```text
/tmp/image.tmp
```

---

## /var/tmp

Use when:

```text
File is needed after reboot
Should stay longer
```

Example:

```text
/var/tmp/backup.tmp
```

---

## Simple Rule

Ask:

```text
Need file after reboot?
```

If:

```text
No  -> /tmp
Yes -> /var/tmp
```

---

# /run

## Purpose

Stores information about currently running services.

### Examples

```text
/run/sshd.pid
/run/systemd
```

### Easy Memory

```text
/run = What is running now
```

---

# /proc

## Purpose

Virtual filesystem containing process and system information.

Files are created by Linux automatically.

### Examples

```text
/proc/cpuinfo
/proc/meminfo
```

### Get CPU Information

```bash
cat /proc/cpuinfo
```

### Get Memory Information

```bash
cat /proc/meminfo
```

### Easy Memory

```text
/proc = Live system dashboard
```

---

# /sys

## Purpose

Contains hardware and kernel information.

### Examples

```text
/sys/devices
/sys/kernel
```

### Shows

```text
CPU
USB devices
Disks
Network cards
```

### Easy Memory

```text
/sys = Hardware dashboard
```

---

# /dev

## Purpose

Contains device files.

Linux treats hardware as files.

### Examples

```text
/dev/sda
/dev/null
/dev/tty
```

---

## /dev/sda

Represents a hard disk.

```text
/dev/sda = Disk
```

---

## /dev/null

Special device that throws away data.

Example:

```bash
echo hello > /dev/null
```

Result:

```text
Output disappears
```

### Easy Memory

```text
/dev/null = Black hole
```

---

# /mnt

## Purpose

Temporary mount location.

### Example

Mount a disk:

```bash
sudo mount /dev/sdb1 /mnt
```

### Easy Memory

```text
/mnt = Temporary parking space
```

---

# /media

## Purpose

Stores mounted removable devices.

### Examples

```text
USB Drives
DVDs
CDs
```

Example:

```text
/media/ahamed/USB_DRIVE
```

### Easy Memory

```text
/media = USB parking area
```

---

# /data

## Purpose

Usually stores application data.

Not a standard Linux directory.

Often used in:

```text
WSL
Docker
Applications
Servers
```

### Example

```text
/data/backups
/data/projects
```

### Easy Memory

```text
/data = Storage warehouse
```

---

# Symbolic Links

Modern Linux often uses:

```text
/bin  -> /usr/bin
/sbin -> /usr/sbin
/lib  -> /usr/lib
```

Meaning:

```text
/bin
```

actually points to:

```text
/usr/bin
```

and similarly for the others.

### Easy Memory

```text
-> means shortcut
```

Example:

```text
/bin -> /usr/bin
```

means:

```text
/bin is a shortcut to /usr/bin
```

---

# Ultimate Cheat Sheet

```text
/boot   = Start Linux

/usr    = Programs and software
/usr/bin  = User programs
/usr/sbin = Admin programs
/usr/lib  = Libraries

/lib    = Shared libraries

/etc    = Settings

/var    = Changing files
/var/log = Logs
/var/tmp = Temporary files kept longer

/home   = User files
/root   = Admin files

/opt    = Extra software

/srv    = Service data

/tmp    = Temporary files

/run    = Running process information

/proc   = System dashboard
/sys    = Hardware dashboard

/dev    = Devices

/mnt    = Temporary mount point
/media  = USB/CD mount point

/data   = Application data
```

# Final Memory Trick

```text
/boot   = Start
/usr    = Programs
/lib    = Helpers
/etc    = Settings
/var    = Logs
/home   = User files
/root   = Admin files
/tmp    = Temporary
/proc   = Processes
/sys    = Hardware
/dev    = Devices
/mnt    = Temporary mount
/media  = USB mount
/data   = Storage
```
