# Partitioning and Formatting Disks

- [Partitioning and Formatting Disks](#partitioning-and-formatting-disks)
  - [Common Disk Types](#common-disk-types)
  - [Disk Geometry](#disk-geometry)
  - [Partition Organization](#partition-organization)
    - [MBR](#mbr)
      - [Partition Table](#partition-table)
    - [GPT](#gpt)
      - [Partition Table](#partition-table-1)
  - [Naming Disk Devices and Nodes](#naming-disk-devices-and-nodes)
  - [Utility](#utility)
    - [blkid](#blkid)
    - [lsblk](#lsblk)
  - [Sizing up partitions](#sizing-up-partitions)
    - [root (/)](#root)
    - [swap](#swap)
  - [Backing Up and Restoring Partition Table](#backing-up-and-restoring-partition-table)
  - [Partition Table Editors](#partition-table-editors)
  - [fdisk](#fdisk)

## Common Disk Types

There are multiple hard disk type, and they categorize by the type of data bus they are attached to, speed, capacity, etc...

* Serial Advanced Technology Attachment (SATA)
* Small Computer Systems Interface (SCSI)
* Serial Attached SCSI (SAS)
* Universal Serial Bus (USB)
* Solid State Drives (SSD)
* (Enhanced) Integrated Drive Electronics (IDE and EIDE) - Obsolete

## Disk Geometry

Disk Geometry is a concept with a long history for rotational media (heads, cylinders, track, sectors). With `sudo fdisk -l /dev/sda`, one can see the disks on a system. 

Rotational disk are usually composed of one or more platters, which is read by one or more heads. The head reads a circular track off a platter as the disk spins.

These circular track are divided into sectors, typically 512 bytes in size. A cylinder is a group which consists of the same track on all platters.

These are less and less relevant with SSDs and stuff. 

## Partition Organization

Disks are divided into partitions. In geometrical terms, these consists of a group of sectors/cylinders. There are two partioning layouts in use:

* MBR (Master Boot Record)
* GPT (GUID Partition Table)

A partition can be useful for: 

* Seperation of user and app data from OS files
* Sharing between OS
* Security enhancement by imposing different quotas and permissions for different system parts
* Size concerns
* Performance enhancement of putting most frequently used data on faster storage media
* Swap space can be isolated from data and also used for hibernation storage

A common partition scheme for linux is a **boot** partition, a partition for the `/` fs, a swap partition and a partition for `/home` directory. 

### MBR

MBR dates back to the early days of MS-DOS. When using MBR, a disk may have four primary partitions which can be subdivided by 15 logicial partitions. 

Max size of a partition is 2 TB.

#### Partition Table

The disk partition table is contained within the disk **MBR**(Master Boot Record). It's 64 bytes follwed by the 446 byte boot record.

Fun fact: At the end of the MBR, there's 2 extra bytes known as the magic number which always have the value `0x55aa`

Each entry in the partition table is 16 bytes long and describe one of the four possible primary partitions (4x16=64)

* Active bit
* Beginning address in cylinder/head/sectors (CHS) formas : ignored by Linux
* Partition type code, indicating: xfs, LVM, ntfs, ext4, swap, etc...
* Ending address in CHS : ignored by Linux
* Start sector, couting linearly from 0
* Number of sectors in partition

Linux uses only the last two fields for addressing, using the linear block adderssing method (LBA)

### GPT

GPT is on all modern system and is based on EUFI (Unified Extensible Firmware Interface). By default it can have up to 128 primary partition. 

Max size of a partitions is 2^33 TB.

#### Partition Table

Modern hardware comes with GPT support; MBR will gradually fade away.

There are 2 copies of the GPT header, at the beginning of the disk and at the end describing metadata: 

* List of usuable blocks on disk
* Number of partition
* Size of partition entries. each partition entry has a minimum size of 128 bytes.

The `blkid` utility can shows information about partitions.

## Naming Disk Devices and Nodes

The linux kernel interacts at a low level with disks through device nodes normally found in the `/dev/` directory. 

Usually only accessed thruogh the infrastructure of the kernel's VFS. Raw access is an efficient way to destroy fs.

Device nodes follow a simple xxy[z] naming convention, where xx is the device type, and y is the letter for the drive number

sda : first hard disk (sd means SCSI or SATA disk)
sdb : second hard disk

Partitions are also easily enumareted as in 

sdb1 : first partition on the second disk
sdc4 : fourth partition on the third disk

For SCSI devices, the numbering is primarily based on the ID number of the SCSI device, rather than its position on the bus itself.

Example, if we have two SCSI controllers with target ID number 1 and 3 on controller 0 and target ID number 2 and 5 on controller 1 (with ID 2 as the last drive)

ID 1 would be /dev/sda
ID 3 would be /dev/sdb
ID 2 (on controller 1) would be /dev/sdc
ID 5 would be /dev/sdd

## Utility

### blkid

blkid is a utility to lcate block devices and report on their attributes. It can determine the type of content (fs, swap), and attributes(token, NAME=value pair) from the metadata

### lsblk

a related utility is lsblk which presents block device information in a tree format

## Sizing up partitions

Most linux systems should use a minimum of two partitions

### root (/)

In pratice, more than one fs on more than one partition which are join together at mount points.

### swap 

Should be around the same size as the physical memory in size, something twice that is recommended.

A system can have multiple swap partition

On a single disk system, try to center the swap partition; on multiple disk system, spread swap over disks.

Adding more and more swap will not necessarily help since at some point it becomes useless.

## Backing Up and Restoring Partition Table

Partitioning and re-partiniong disks are dangerous operations. If something happens, we need to know how to restore them.

Backing up can be easily done with `dd` as in `sudo dd if=/dev/sda of=mbrbackup bs=512 count=1` which will back up the MBR on the first disk including the 64bit partition table

The MBR can then be restored, if necessary, by doing: `sudo dd if=mbrbackup of=/dev/sda bs=512 count=1`

Note that will only copy the primary partition table. Always assume that changing partition table will wipe your data -> make a backup of data beforehand

For GBT, it is best to use the sgdisk tool.

## Partition Table Editors

There are number of utilities which can be used to manage partition tables

| name    | description                                                                                   |
| ------- | --------------------------------------------------------------------------------------------- |
| fdisk   | menu-driven partition table editor. It is the most standard one and one of the most flexible. |
| sfdisk  | non-interactive linux-based partition editor program. Useful for scripting                    |
| parted  | GNU partition manipulation program. It can create, remove, resize, and move partitions.       |
| gparted | GUI interface for parted                                                                      |
| gdisk   | Used for GPT system, but also can work with MBR                                               |
| sgdisk  | script/ cli for gisk                                                                          |

## fdisk

fdisk will **always** be included in any linux installation so it's a good idea to learn how to use it. Must be root.

| CMD | Description                                       |
| --- | ------------------------------------------------- |
| m   | display the menu                                  |
| p   | list the partition                                |
| n   | create a new partion                              |
| d   | delete a partion                                  |
| t   | change a partiton type                            |
| w   | write the new partiton table information and exit |
| q   | quit without making changes                       |

Fortunaly, no actual changes is saved before entering w. And the system will not use the partiton table until a reboot.