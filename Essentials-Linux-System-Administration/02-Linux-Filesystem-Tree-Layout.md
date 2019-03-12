# Linux Filesystem Tree Layout

- [Linux Filesystem Tree Layout](#linux-filesystem-tree-layout)
    - [Overview](#overview)
    - [/bin/](#bin)
    - [/boot/](#boot)
    - [/dev/](#dev)
    - [/etc/](#etc)
    - [/home/](#home)
    - [/lib/ and /lib64/](#lib-and-lib64)
    - [/media/](#media)
    - [/mnt/](#mnt)
    - [/opt/](#opt)
    - [/proc/](#proc)
    - [/sys/](#sys)
    - [/root/](#root)
    - [/sbin/](#sbin)
    - [/srv/](#srv)
    - [/tmp](#tmp)
    - [/usr/](#usr)
    - [/var/](#var)
    - [/run/](#run)

Most UNIX-like system have similar layout for their FS. There's actually a standard called Filesystem Hierarchy Standard. It is old 
and a lot of distros respect the FHS but don't follow it exactly.

## Overview

| Directory | FHS? | Purpose                                                                       |
| --------- | ---- | ----------------------------------------------------------------------------- |
| /         | Yes  | Root of the entire FS                                                         |
| /bin      | Yes  | Essential exe that must be available in single user mode                      |
| /boot     | Yes  | Files needed to boot the system (kernel,initrd,etc...)                        |
| /dev      | Yes  | Device Nodes                                                                  |
| /etc      | Yes  | System wise config files                                                      |
| /home     | Yes  | User home directories                                                         |
| /lib      | Yes  | Lib required by exe in /bin and /sbin                                         |
| /lib64    | No   | 64 libs required in /bin and /sbin, for sys that run both 32/64               |
| /media    | Yes  | Mount points for removable media (USB,CD)                                     |
| /mnt      | Yes  | Temp. mounted filesystems                                                     |
| /opt      | Yes  | Opt. app. software packages                                                   |
| /proc     | Yes  | Virtual pseudo-fs giving info about the sys and process                       |
| /sys      | No   | Same as /proc + Similar to a device tree and part of the Unified Device Model |
| /root     | Yes  | Home directory for root user                                                  |
| /sbin     | Yes  | Essential system bin.                                                         |sys
| /srv      | Yes  | Site-specific data served up by the system. Rarely used                       |
| /tmp      | Yes  | Temp. files, lost after a reboot on many distros                              |
| /usr      | Yes  | Multi-user app, utilities and data; theoretically read-only                   |
| /var      | Yes  | Variable data that changes during sys. op                                     |

/ is often in a special/dedicated partition with othe components with /home, /var, /opt to be mounted later.

## /bin/

* Contains exe programs and scripts needed by both system admin/unprivileged user.
* May not include subdirectories
* cat, chmod, cp, date, ls, mkdir, etc... should be included in /bin/
* Command not important enough to be in /bin/ are located in /usr/bin/

## /boot/

Files needed to boot the for thesystem. The two files which are absolutely essential are:

* vmlinuz (the compressed kernel)
* initramfs (the initial RAM fs, which is mounted before the real root filesystem) - Sometimes called initrd

Sometime multiple kernel versions available in /boot/ and for each one, there's 4 files available.

The other two files (not necessary for booting/running the sys) beside the two important one are:

* config (config file used when compiling the kernel. )
* System.map (kernel symbol table, very useful for debugging)

## /dev/

This dir contains special device files (also known as **device nodes**). Essential for the system to function properly.

Such dev files represent character and block I/O devices. Network dev do not have device nodes.

## /etc/

This dir contains machine-local config files and some startup scripts; No exe binary programs.

This dir may includes files such as: csh.login, exports, fstab, ftpusers, gateways, gettydefs, group, host.conf, hosts.allow, hosts.deny, hosts.lpd, inetd.conf, inittab, issue, motd, mtab, mtools, passwd, profile, protocols, resolv.conf, rpc, securetty, services, shells, syslog.conf.

Distros often add config file and dir to /etc

## /home/

User dir are conventionally placed under /home such as /home/kerry/ or /home/gen/. All personal stuff are placed in this dir. It could also be for group or associations of users such as /home/students or /home/friends

A user can always substitute the env variable $HOME, those are equivalent

```bash
ls -l $HOME/public_html
ls -l ~/public_html
```

The only one exception is for the root user which is always found under /root.

## /lib/ and /lib64/

Should only contain the libs needed to execute the bin in /bin and /sbin. Kernel modules are under /lib/modules/{kernel-version}

PAM (Pluggable Auth Modules) files are stored in the /lib/security/

## /media/

This dir is used to mount filesystems on removable media (CD,USB). Modern Linux distro mount such media dynamically upon insertion.

If the media has more than one partition, more than one entry will appear on /media.

## /mnt/

This dir is for mounting a filesystem when needed. A common use is for network filesystems (NFS, Samba, CIFS, AFS).

## /opt/

This dir is for software packages that wants most of their files in one isolated places. Makes (un)installing easy, but not the only way.

## /proc/

This dir is the mount point for a pseudo-filesystem where all the informatino resides only in memory, not on disk. It is empty on a non-running system. The kernel exposes some data structures through /proc entries. Each running process has its own subdirectory containing information about it.

Often refered as virtual files, they appear to have 0 bytes in size, however, when viewed, they can contain a large amount of information. Information get loaded only when viewed.

## /sys/

This dir is the mount point for the sysfs pseudo-filesystem, it is very similar to /proc/, but has adhered to stricter standards. Most of the entries only contains one lines.

## /root/

This is pronounced "slash-root", since / is the root directory. Similar to a /home/user directory, but require superuser privilege.

## /sbin/

This dir contains binaries essential for booting, restoring, recovering, and/or repairing in addition to the bin in the /bin dir. It should contain fdisk, fsck, getty, halt, ifconfig, init, mkfs, mkswap, reboot, route, swapon, swapoff, update.

## /srv/

This dir contains site-specific data which is served by this system. Some people really use it, some don't. Single location for data files for a particular services. No good methodology, some structure the data under /src by protocol, eg. ftp, rsync, www,cvs. Often confusion between /var/ and /srv/

## /tmp

This dir is used to store temp. files, and can be accessed by anyone, however file cannot be depended on staying forever there. Some distro runs cron jobs that remove the file, some remove on every reboot and then some use a virtual filesystem making it reseting on each reboots.

In the last case, one should avoid creating huge files in /tmp/ because it's living in the ram.

## /usr/

This dir can be thought of as a secondary hierarchy. It is used for files not needed for system booting. In fact, can be in a different partition than the root directory, and can be shared over a network. Typically read-only. It contains binaries not needed in single user mode. man apges are stored under /usr/share/man.

## /var/

This directory contains volatile files that can change frequently like log files, admin. data files, sppol dir and files for printing,  and cache content. It's a good idea to mount /var as a separate fs.

## /run/

The purpose of /run/ is to store transient files: those that contain runtime information. Mostly run on memory.
