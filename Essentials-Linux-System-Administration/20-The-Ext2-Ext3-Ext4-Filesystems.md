# The Ext2/Ext3/Ext4 Filesystems

- [The Ext2/Ext3/Ext4 Filesystems](#the-ext2ext3ext4-filesystems)
  - [History and Basics](#history-and-basics)
  - [Ext4 fs Features](#ext4-fs-features)
  - [Ext4 layout](#ext4-layout)
  - [Superblock Information](#superblock-information)
  - [Block groups](#block-groups)
  - [Data blocks and Inodes](#data-blocks-and-inodes)
  - [Utilities](#utilities)
    - [dumpe2fs](#dumpe2fs)
    - [tune2fs](#tune2fs)

## History and Basics

* Ext2: Linux original nativ varierty and is avail. on all Linux systems, but rarely used
* Ext3: First journalling extension of ext2. It had the same ondisk layout as ext2, expect for the presence of a journal file
* Ext4: Appeared in kernel 2.6.19, and it experimental designation was remvoed in version 2.6.28. It included significant enhancement such as the use of extents for large files, instaed of lists of file blocks. An extent is simply a group of cotiguous block. 

Ext4 is the default fs on msot enterprise Linux distributions, altough RHEL 7 adopted XFS as default.

## Ext4 fs Features

* Ext fs was built with VFS in mind. 
* The linux kernel memory management system requires only an integral number of blocks must fit into a page of memroy. thus, you can't have 8kb blocks wher pages are 4kb.
* The number of indoes on the fs can also be tuned
* A feature called inode reservation consists ofreserving several inodes when a directory is created, expecting them to be used in the future.
* Ext4 is backward compatible
* Maximum fs size to 1EB and the maximum filesize to 16TB. 
* No limit on # of subdirectories, which was limited to 32K in ext3.
* Splits large files into the largest possible extents instead of using indirect block mapping.
* Uses multiblock allocation which can allocate space all at once, instead of one block at a time. 
* Can pre-allocate disk space for a file.
* Uses allocate-on-flush, a performance technique which delays block allocation until data is written to disk.
* uses fast fsck which can make the speed of a fs check go up b an order of magnitude or more.
* Uses checksums for the journal which improves reliability. 
* used improved timestamps measured in nanoseconds.

## Ext4 layout

All fields are written to disk in little-endian order, except the journalDisk blocks are partitioned into block groups, each of which contains inode and data blocks stored adjacently. Data blocks are pre-allocated to files before they are actually used. Thus, when a file's size is increased, adjacent space has already been reserved and file fragmentation is reduced. The fs superblock contains bit-fields which are used to ascertain whether or not the fs requires checking when it is first mounted during sysinit. The status can be:

* clean: fs cleanly unmounted previously
* dirty: usually means mounted
* unknown: not cleanly unmounted, such as when there is a syscrash

In the latter two cases, fsck will be run to check the fs and fix any problems before it is mounted. 

The frst block on the fs is the so-called boot block, and is reserved for the partition boot sector.

## Superblock Information

The superblock contains global information about the fs including:

* Mount count and max. mount cuont. One can also force a fs after 180 by default.
* Block size
* Blocks per group
* Free block count
* Free inode count
* OS ID

## Block groups

After the boot block, there's a series of block groups, each of which is the same size. It looks like

| Super Block | Group Descriptors | Data Block Bitmap | Inode Bitmap | Inode Table (n) | Data Block (n) |

The first two are the same in every block group as a redundacy feature. Makes it difficult to fry an ext2/3/4 fs.

The number of block gruops is constrained by the fact that the block bitmap has to fit in a single block. Example: if we have a block of 4kb in size, then the block group can contain no mroe than 32k blocks (128MB). if we take the largest possible block group size, a 10GB partition would have to have atleast 80 block groups.

## Data blocks and Inodes

The data block and inode bitmaps are blocks whose bits contain 0 for each fre block or inode and 1 for each used one. There is only one of those bitmap per block group

The inode table contains as many consecutive blocs as are necessary to cover the number of inodes in the block group. Each inode requires 128 bytes; thsu a 4KB block can contain 32 inodes.

## Utilities

### dumpe2fs

`dumpe2fs` can be use to scan the fs informations (limits, flags, capabilities).

### tune2fs

`tune2fs` can be used to chagne fs parameters

* `sudo tune2fs -c 25 /dev/sda1`: change the max number of mounts between fs checks
* `sudo tune2fs -i 10 /dev/sda1`: change the time inteval between checks
* `sudo tune2fs -l /dev/sda1`: List the contents of the superblock, including the current values of parameters which can be changed