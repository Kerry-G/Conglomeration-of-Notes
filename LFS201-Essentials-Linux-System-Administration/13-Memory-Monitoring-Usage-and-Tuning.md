# Memory Monitoring Usage and Tuning

Systems are always eating more memory and at the same time RAM became cheaper and more performant. Yet, it's often the bottleneck in mdoern systems. There are many tools for monitoring/debugging/tuning a system's behavior with regard to its memory.

- [Memory Monitoring Usage and Tuning](#memory-monitoring-usage-and-tuning)
    - [Memory tuning considerations](#memory-tuning-considerations)
    - [Memory Monitoring Tools](#memory-monitoring-tools)
    - [/proc/sys/vm](#procsysvm)
    - [vmstat](#vmstat)
        - [Options](#options)
        - [Fields](#fields)
            - [Generals Fields](#generals-fields)
            - [Disk Fields](#disk-fields)
    - [/proc/meminfo](#procmeminfo)
    - [OOM Killer](#oom-killer)

## Memory tuning considerations

It is a complex process. First of all., memory and I/O is highly related. Often memory is just a cache for files on disk. Thus, changing settings can have a large effect on I/O performance, and changing I/O settings could have an effect on memory.

When tweaking params in /proc/sys/vm, the best practice is to adujst one thing at a time and look for effects. The primary tasks are:

* Controlling flushing params: how may pages are allowed to be dirty and how often they are flushed out to disk
* Controlling swap behaviour: how much pages that reflect file contens are allowed to remain in memory, as opposed to those that need to be swapped out as they ahve no backing store
* Controlling how much memory overcommision is allowed, since many programs never need the full amount of memory they request, particuarly because of copy on write (COW) techniques.

## Memory Monitoring Tools

See chapter 11. for available monitoring tools. 

## /proc/sys/vm 

This dir contains a lot of tunable knobs to control the virtual memory. Almost everything is writable by root there.

The full documentation about this directory is in the kernel source, usually under Documentation/sysctl/vm.txt.

| Entry                      | Purpose                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------- |
| admin_reserve_kbytes       | Reserved free memory for privileged users                                                   |
| block_dump                 | Enables block I/O debugging                                                                 |
| compact_memory             | Turns on or off memory compaction (defrag)                                                  |
| dirty_background_bytes     | Dirty memory threshold that triggers writing uncommited pages to disk                       |
| dirty_background_ratio     | % of total pages at which kernel will start writing dirty data out to disk                  |
| dirty_bytes                | Amount of dirty memory a process needs to initiate writing on its own                       |
| dirty_expire_centisecs     | When dirty data is old enough to be written out                                             |
| dirty_ratio                | % of pages at which a process writing will start writing out dirty data on its own          |
| dirty_writeback_centisecs  | Interval in which periodic writeback daemons wake up to flush. If 0, no automatic writeback |
| drop_caches                | Echo 1 to free page cache, 2 to free dentry and inode caches, 3 to free all. (clean)        |
| extfrag_threshold          | Controls when the kernel should compact memory                                              |
| hugepages_treat_as_movable | Toggle how huge pages are treated                                                           |
| hugetlb_shm_group          | Sets a GID that can be used for System V huge pages                                         |
| laptop_mode                | Can control a number of features to save power on labtops                                   |
| legacy_va_layout           | Use old layout (2.4 kernel) for how memory mappings are displayed                           |
| lowmen_reserve_ratio       | Controls how much low memory is reserved for pages that can only be there                   |

## vmstat

It's a multipurpose tool that shows infos about memory, paging I/O, procesor activity and processes. The general form is

`vmstat [options] [delay] [count]`

* delay: Given in seconds of n number of counts
* count: if not given, will repeat forever

### Options

* -S: memory is given in MB instead of KB.
* -a: active/inactive memory. 
* -s: table of memory statistics.
* -d: table of disk statistics.
* -p: quick stats on only one partition 
  
### Fields

#### Generals Fields

| field     | subfield | Purpose                             |
| --------- | -------- | ----------------------------------- |
| Processes | r        | # of p. waiting to be scheduling in |
| Processes | b        | # of p. in uninterruptible sleep    |
| memory    | swpd     | Virtual memory used                 |
| memory    | free     | Free (idle) memory                  |
| memory    | bugg     | Bugger memory                       |
| memory    | cache    | Cached memory                       |
| swap      | si       | Memory swapped in                   |
| swap      | so       | Memory swapping out                 |
| I/O       | bi       | Blocks written to devices           |
| I/O       | bo       | blocks read from devices            |
| system    | in       | Interrupts/seconds                  |
| system    | cs       | Context switches/seconds            |
| CPU       | us       | CPU time running user code          |
| CPU       | sy       | CPU time running kernel code        |
| CPU       | id       | CPU time idle                       |
| CPU       | wa       | Time waiting for I/O                |

#### Disk Fields

| field  | subfield | Purpose                             |
| ------ | -------- | ----------------------------------- |
| reads  | total    | Total reads completed successfully  |
| reads  | merged   | Grouped reads                       |
| reads  | ms       | Ms spent reading                    |
| writes | total    | Total writes completed successfully |
| writes | merged   | Grouped writes                      |
| writes | ms       | Ms spent writing                    |
| I/O    | cur      | I/O in progress                     |
| I/O    | sec      | s spent for I/O                     |

## /proc/meminfo

| Entry           | Purpose                                                       |
| --------------- | ------------------------------------------------------------- |
| MemTotal        | Total usable RAM (Physical - kernel reserved mem)             |
| MemFree         | Free memory in both low/high zones                            |
| Buffers         | Memory used for temp. block I/O storage                       |
| Cached          | Page cache memory, mostly for file I/O                        |
| SwapCached      | Memory that was swapped back in but is still in the swap file |
| Active          | Recently used memory, not to be claimed first                 |
| Inactive        | Memory not recently used, more eligible for reclamation       |
| Active(anon)    | Active memory for anon. pages                                 |
| Inactive(anon)  | Inactive memory for anon. pages                               |
| Active(file)    | Active memory for file-backed pages                           |
| Inactive(file)  | Inactive memory for file-backed pages                         |
| Unevictable     | Pages which can't be sawpped out of memory or released        |
| Mlocked         | Pages which are locked in memory                              |
| SwapTotal       | Total swap space available                                    |
| SwapFree        | Swap space not being used                                     |
| Dirty           | Memory which needs to be written back to disk                 |
| Writeback       | Memory actively being written back to disk                    |
| AnonPages       | Non-file back pages in cache                                  |
| Mapped          | Memory mapped pages, such as libs                             |
| Shmem           | Pages used for shared memory                                  |
| Slab            | Memory used in slabs                                          |
| SReclaimable    | Cached memory in slabs that can be reclaimed                  |
| SUnreclaim      | Memory in slabs that can't be reclaimed                       |
| KernelStack     | Memory used in kernel stack                                   |
| PageTables      | Memory being used by page table structures                    |
| Bounce          | Memory used for block device bounce buffers                   |
| WritebackTmp    | Memory used by FUSE fs for writeback buffers                  |
| CommitLimit     | Total memory available to be used, including overcommision    |
| Committed_AS    | Total memory presently allocated, whether or not it is used   |
| VmallocTotal    | Total memory available in kernel for vmalloc allocations      |
| VmallocUsed     | Memory used by vmalloc allocations                            |
| VmallocChunk    | Largest possible contiguous vmalloc area                      |
| HugePages_Total | Total size of the huge page pool                              |
| HugePages_Free  | Huge pages that are not yet allocated                         |
| HugePages_Rsvd  | Huge pages that have been reserved, but not yet used          |
| HugePages_size  | Size of a huge page                                           |

## OOM Killer

The simple way would be to allocate memory as long as the system has some, and fail when the memory is all exhausted

The second simplest way, would be to do the same, but with swapping.

Linux however permets the system to overcommit memory, so thati t can grant memory requests that exeed the size of RAM + swap.

On kernel processes, memory is not swappable and are always allocated at request time.

One can modify, and even turn off this overcommision by setting the value of `/prof/sys/vm/overcommit_memory`:

* 0 (default): Permit overcommission, but refuse obvious overcommits, and give roots users more mem alloc than normal users
* 1 : All memory request are allowed to overcommit
* 2: Turn off overcomission. Memory request will fail when the total memory commit reaches the size of swap + (factor) RAM

`(factor)` is modifyable by `/proc/sys/vm/overcommit_ratio`

When all the memory is taken, Linux invokes the OOM Killer (Out of Memory). This isn't a precise science. 