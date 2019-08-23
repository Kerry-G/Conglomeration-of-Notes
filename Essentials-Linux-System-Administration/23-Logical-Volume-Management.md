# Logical Volume Management

- [Logical Volume Management](#logical-volume-management)
  - [Description](#description)
  - [LVM and RAID](#lvm-and-raid)
  - [Volumes and Volume Groups](#volumes-and-volume-groups)
    - [Tools for Volume Groups](#tools-for-volume-groups)
    - [Tools for physical partitions](#tools-for-physical-partitions)
  - [Creating Logical Volumes](#creating-logical-volumes)
  - [Displaying Logical Volumes](#displaying-logical-volumes)
  - [Resizing Logical Volumes](#resizing-logical-volumes)
  - [LVM Snapshots](#lvm-snapshots)

## Description

LVM breaks up one virtual partition into multiple chunks, each of which can be on different partitions and/or disks.

Many advantages of using LVM. It's really easy to change the size of a logical partitions/fs.

One of more physical volumes are grouped together into a volume group. Then, the volume group is subdivided into logical volumes, which mimic to the system nominal physical disk partitions and can be formatted to contain mountable fs. 

There are varierty of cli utilities to create/delete/resize physical and logical volumes. For a graphical tool, some linux uses system-config-lvm. 

LVM does impact performance. There is a definite additional cost that comes from the overhead of the LVM layer. However, even on non-raids, if you use striping, you can achieve some parallelization improvements.

## LVM and RAID

Like RAID, the use of logical volumes is mechanism for creating fs which can span more than one physical disks.

Logical volumes are creted by puttin all the devices into a large pool of disk space, and then allocating space frmo the pool to create a logical volume. Logical volumes have features similar to RAID devices. They can actually be built on top of a RAID device. This would give the logical volume the redundancy of a RAID device with the scalability of LVM. LVM has better scalability than RAID. Logical volumes can be easily resized/enlarged/shrunk as needs require.

## Volumes and Volume Groups

Partitions are converted to physical volumes and multiple physical volumes are grouped into volume groups; there can be mroe than one volume group on the system. Space in the volume group is devided into extents; these are 4MB in size by default, bu the size can be change when being allocated. 

### Tools for Volume Groups

* `vgcreate`: creates volume groups
* `vgextend`: adds to volume groups
* `vgreduce`: shrinks volume groups

### Tools for physical partitions

* `pvcreate`: Converts a partition to a physical volume
* `pvdisplay`: shows the physical volumes being used
* `pvmove`: move the data from one pv within the vg to others
* `pvremoev`: remove a partition from a physical volume.

## Creating Logical Volumes

lvcreate allocates lv from withtin vg. The size can be specified either in bytes or number of extents (4MB by default). Names can be anything desired.

lvdisplay reports on avail. logical volumes

FS are placed in logical volumes and are formatted with mkfs as usual.

Starting with possibly creating a new vg, the steps involded ni setting up and using a new lv are:

1. Create partitions on disk drives
2. Create physical volumes from the partitions
3. Create the vg
4. Allocate lv from the vg
5. Format the lv
6. Mount the lv

## Displaying Logical Volumes

* `pvdisplay`: Shows pv. Can specify which one or not
* `vgdisplay`: Show vg. Can specify which one or not
* `lvdisplay`: Show lv. Can specify which one or not.

## Resizing Logical Volumes

One of the advatange of using LVMs is that you can quickly resize a LV. This can be done with `lvresize` as in `sudo lvresize -r -L 20 GB /dev/VG/mylvm`

## LVM Snapshots

LVM snapshots create an exact copy of an existing lv. They are useful for backups, application testing and deploying VMs. The original state of the snapshot is kept as the block map. 

Snapshots only use space for storing deltas:

* When the original lv changes
* If data is added to snapshot, it's stored only there

to create a snapshot of an existing lv:

`sudo lvcraete -l 128 -s -n mysnap /dev/vg/mylvm`

To then make a mount point and mount the snapshot

```bash
mkdir /mysnap
mount -o ro dev/vg/mysnap /mysnap
```

to use the snapshot and to remove the snapshot

```bash 
sudo umount /mysnap
sudo lvremove /dev/vg/mysnap
```

Always be sure to remoev the snapshot when you are through with it. if you do not remvoe the snapshot and it filss up because of changes. it will be automatically disbaled. A snapshot with the size of the original will never overflow.