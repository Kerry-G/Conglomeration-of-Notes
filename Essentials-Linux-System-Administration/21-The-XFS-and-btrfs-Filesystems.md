# The XFS and btrfs Filesystems

- [The XFS and btrfs Filesystems](#the-xfs-and-btrfs-filesystems)
  - [XFS](#xfs)
    - [Features](#features)
    - [Details](#details)
  - [btrfs](#btrfs)

## XFS

### Features

Designed and created by Silicon Graphics Inc. (SGI) and used in the IRIX OS and was ported to Linux. It was engineered to deal with alrge data sets for SGI systems

It can handle:

* Up to 16EB fs
* Up to 8EB for an individual file

High performance is one of the key design elements of XFS, which implements methods for:

* Employing DMA (Direct Memory Access) I/O
* Guaranteeing an I/O rate
* Adjusting stripe size to match underlying RAID or LVM storage devices

### Details

Maintaining is made easier by the fact that most of the maintenance tasks can be done on-line (When the fs is fully mounted). These includes:

* Defragmenting
* Resizing (Enlarging...)
* Dumping/Restoring

Backup/restore can be done with the native XFS utiltiies `xfsdump` and `xfsrestore` which can be paused and then resumed at a later time. It's also multi-threaded. 

## btrfs

B-tree filesystem (btrfs) is intended to address the lack of pooling/snapshots/checksums/integral multi-device spanning in other Linux fs such as ext4. 

Such features are crucial into larger enterprise storage configuration.

One of the main features is the ability to take frequent snapshots of entire fs, or sub-volumes of entire fs in virtually no time. One can easily revert to the state described by earlier snapshots and even induce the kernel to reboot off an earlier root fs snapshot. 