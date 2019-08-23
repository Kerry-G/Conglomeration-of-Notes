# Filesystem Features: Attributes, Creating, Checking, Mouting

- [Filesystem Features: Attributes, Creating, Checking, Mouting](#filesystem-features-attributes-creating-checking-mouting)
  - [Extended Attributes and Isattr/chattr](#extended-attributes-and-isattrchattr)
  - [Creating and Formatting Filesystems](#creating-and-formatting-filesystems)
  - [Checking and Repairing Filesystems](#checking-and-repairing-filesystems)
  - [Mounting Filesystems](#mounting-filesystems)
  - [Network Shares (NFS)](#network-shares-nfs)
  - [Mounting at Boot](#mounting-at-boot)
  - [Automatic Filesystem Mounting](#automatic-filesystem-mounting)

## Extended Attributes and Isattr/chattr

**Extended Attributes** associate metadata not interpreted directly by the fs with files. Four namespaces exists: user, trusted, security and system. The system namespace is used for Access Control Lists (ACLs), and the security namespace is used by SELinux.

Flag values are stored in the file inode and may be modified and set only by the root user. They are viewed with lsattr and set with chattr.

| Flags | Meaning         | Description                                                                                                                                                               |
| ----- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| i     | Immutable       | Can't be modified (not even by root), can't be deleted or renamed. No hard link and no data can be written. Only the superuser can set or clear this attr.                |
| a     | Append-only     | Can only be opened in append mode for writing. Only the superuser can set/clear this.                                                                                     |
| d     | No-Dump         | Is ignored when the dump program is run. Useful for swap/cache files.                                                                                                     |
| A     | No atime update | Will not modify its atime record when the file is accessed but not otherwise modified. Can increase performance on some systems, since it's reduce the I/O on the system. |

There are more flags, and all the description is found with `man chattr`

## Creating and Formatting Filesystems 

Every fs type has a utility for formatting a fs on a partition. The generic name for these utilities is mkfs. The general format is `mkfs [-f fstype] [options] [device-file]` where device-file is usualyl a device name like /dev/sda3

Each fs type has its own particular options that can be set when formatting. One should look at the man page for each of the mkfs.* programs to see details.

## Checking and Repairing Filesystems

Every fs type has a utility desgined to check for errors. The generic name for these utilities is fsck. The general format for fsck is `fsck[-f fstype] [options] [device-file]. Usually you don't need to specify the fs type as fsck can figure it out. You can control whether any errors should be fixed one by one manually with the -r option, or automatically, as best possible, by using the -a options. 

Note that journallnig fs are much faster to check than older generation fs for two reasons: 

* You rarely need to scan the entire partition for errors, as everything but the very last transaction has been logged and confirmed. 

* Even if you do check the whole fs, newer fs have been designed with **fast fsck** in mind. 

## Mounting Filesystems

All accessible files in Linux are organized into one large tree with the head of the tree being the root directory (/). However, it's common to have more than one partition joined together in the same tree. 

The mount program allows attaching at any point in the tree structure; umount allows detaching them.

The general form for mount is `mount [options] <source> <directory>`
And for umount is `umount [device-file | mount-point]

The most common error when unmounting a device is that it's currently in use.

## Network Shares (NFS)

It is common to mount remote fs through network shares. The most used one has been NFS. There are other one like Andrew File System (AFS), Server Message Book (SMB) aka Common Internet File System (CIFS).

Thus, a fs could be unavaiable and the system should be prepared for it. These can be specified in the mount command. Mount has a lot of options for NFS, so one should verify the man pages

## Mounting at Boot

During sysinit, the command mount -a is executed. This mounts all fs listed in the /etc/fstab config file. They will contains a white space seperated field with those information

* Device node, label or UUID
* Mount point
* Fs type
* Comma separated list of options
* dump frequency (or 0)
* fsck pass number (or 0)

## Automatic Filesystem Mounting

Linux system have long had the ability to mount a filesystem only when it is needed. Historicall, this was done using autofs. This utility requires installation of the autofs package using the appropriate package manager and config file in /etc. 

However, systemd-based systems comes with automount facilities built into the systemd framework. Configuring this is as simlpe as adding a line /etc/fstab specifying the proper device, mount point and mounting optinos such as:

`LABEL=wtv /SAM ext4 noauto,x-systemd.automount,x-systemd.device-timeout=10,x-system.idle-timeout=30 0 0` and then either rebooting or call daemon-reload on systemctl.

* noauto: Do not mount at boot. here auto =!= automount
* x-systemd.automount: Use the systemd automount facility
* x-systemd.automount.device-timeout=10: If this device isn't avail. timeout after 10s
* x-systemd.autmount.idle-timeout=30: if the device is not used for 30sec, unmount it.

