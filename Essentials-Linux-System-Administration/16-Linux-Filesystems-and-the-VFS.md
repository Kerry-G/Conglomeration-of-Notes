# Linux Filesystems and the VFS

- [Linux Filesystems and the VFS](#Linux-Filesystems-and-the-VFS)
  - [Filesystem basics](#Filesystem-basics)
  - [Inodes](#Inodes)
  - [Hard and Soft links](#Hard-and-Soft-links)
  - [Filesystem Tree Organization](#Filesystem-Tree-Organization)
  - [Virtual File System](#Virtual-File-System)
  - [Available Filesystems](#Available-Filesystems)
  - [Jounalling Filesystems](#Jounalling-Filesystems)
  - [Special filesystems](#Special-filesystems)

## Filesystem basics

Programs r/w on files instead of the physical location on the hardware. Files are an abstraction camoufling the physical I/O layer. Otherwise, it would be very dangerous to write on the memory itself.

## Inodes

An inode is a data structure on disk that define the file attributes. Every file has its inode. The attributes stored in a inode are:

* Permissions
* User and group ownership
* Size
* Timestamp (Last access, modification and change)

Their name is not stored in a file's inode, but in a directory file.

## Hard and Soft links

A director files is a kind of file that associates file names with their inode. A hard link is a direct point to an inode where as a soft links point to a filename which has an assocaited inode.

When a process refers to a pathname, the kernel searches directories to find the corresponding inode number. Then, the inode is laoded into memory and is used by subsequent requests.

## Filesystem Tree Organization

All Linux systems uses an inverted tree hierarchy branching off the / directory. Multiple element can be mounted such as mount points (drives) and virtual pseudo fs, but to the applications and OS, it all appears in one unified tree structure. 

## Virtual File System

Linux implements a Virtual File System (VFS). When a process needs a file, it interact with it, which then translates all the I/O system calls to specific code relevant to that particular filesystem.

Some variants of filesystems doesn't have distinct r/w/e permissions. The VFs then make assumptions about the distinct permissions. 

## Available Filesystems

* ext4: Linux native fs
* XFS: high-performance fs created by SGI
* JFS: high-performance fs created by IBM
* Windows-natives: FAT12, FAT16, FAT32, VFAT, NTFS
* Pseudo-filesystems resides in memory only, including proc, sysfs, devfs, debugfs
* Network fs: NFS, coda, afs
* etc...

You can see a list of the fs type currently registed and understood by the running Linux kernel by doing `cat /proc/filesystems`

## Jounalling Filesystems

Journalling fs recover from system crashes or ungraceful shutdowns with little or no curruption, and very rapidly. In a journalling fs, operations are grouped into transactions. A transaction must be completed without error, atomically; otherwise the fs is not changed. ext3, ext4, reiserfs, jfs , xfs and btrfs are all journalling fs. 

## Special filesystems

| FS          | Mount Point       | Purpose                                               |
| ----------- | ----------------- | ----------------------------------------------------- |
| rootfs      | None              | During kernel load, provides an empty root dir        |
| hugetlbfs   | Anywhere          | Provides extended page access                         |
| bdev        | None              | Used for block devices                                |
| proc        | /proc             | Pseudo fs access to many kernel struct and subsystems |
| sockfs      | None              | Used by BSD Sockets                                   |
| tmpfs       | Anywhere          | RAM disk with swapping, re-sizing                     |
| shm         | None              | Used by System V IPC Shared Memory                    |
| pipefs      | None              | Used for pipes                                        |
| binfmt_misc | Anywhere          | Used by various executable formats                    |
| devpts      | /dev/pts          | Used by Unix98 pseudo-terminals                       |
| usbfs       | /prov/bus/usb     | Used by USB sub-system for dynamical devices          |
| sysfs       | /sys              | Used as a device tree                                 |
| debugfs     | /sys/kernel/debug | Used for simple debugging file access                 |

