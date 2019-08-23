# Filesystem Features Swap Quotas Usage

- [Filesystem Features Swap Quotas Usage](#filesystem-features-swap-quotas-usage)
  - [Using swap](#using-swap)
  - [Filesystem Quotas](#filesystem-quotas)
  - [Setting up Quotas](#setting-up-quotas)
  - [Utilities](#utilities)
  - [df: Filesystem usage](#df-filesystem-usage)
  - [du: Disk Usage](#du-disk-usage)

## Using swap

Linux uses a VM (Virtual Memory) system, in which the OS can function as it had more memory than it really does. This kind of memory overcommission functions in two ways:

* Many programs do not actually use all the memory they are given permission to use. Sometimes, this si because child processes inherit a copy of the parent's memory regiosn utilizing a COW (Copy On Write) technique. 

* When memory pressure is important, less active memory regions may be swapped out to disk. 

Such swapping is usually done to one or more dedicated partitions or files; Linux permits multiple swap areas, so the needs can be adjusted dynamically. Each area has a priority, and lower priority areas are not used until higher priority areas are filled. 

Usually the recommended swap size is the total RAM on the system. 

* `cat /proc/sawps`: cat /proc/swaps

* `free -m`: Basic memory stats

The only commands involving swap are:

* `mkswap`: format a swap partition or file
* `swapon`: Activate a swap partition or file
* `swapoff` deactivate a swap partition or file

The memory used by the linux kernel is never swapped out.

## Filesystem Quotas

Linux can enforce quotas on fs. Disk quotas allows admin. to control the max. space a user are allowed.

Utilies to help manages quotas:

* quotacheck: generates and updates quota account files
* quotaon: enable quota
* quotaoff: disables quota
* edquota: used for editing user or group quotas
* quota: reports on uasge and limits

Quota op. require the existence of the files aquota.user and aquota.group in the root directory of the fs using quotas. 

## Setting up Quotas

Basic steps of setting up a quota:

1. Mount the fs with user/group quota options
2. Run quotacheck on the fs to setup quota
3. Enable quotas on the fs
4. Set qutoas with the edquota program

## Utilities

## df: Filesystem usage

The df (Disk Free) utility examines fs capacity and usage. `-h` will turn in into human-readable and -T will show the fs tpye.

## du: Disk Usage

To display disk usage for the current directory, run `du`, to list all the files, and not just directories uses `-a`. `-h` will turn in into human-readable.