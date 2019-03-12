# System Monitoring
- [System Monitoring](#system-monitoring)
    - [Available Monitoring Tools](#available-monitoring-tools)
    - [The /proc and /sys Pseudo-fs](#the-proc-and-sys-pseudo-fs)
    - [/proc](#proc)
        - [/proc/sys](#procsys)
    - [/sys](#sys)
    - [sar](#sar)

## Available Monitoring Tools

Most of monitoring tools uses the mounted pseudo-filesystems, especially /prog and /sys. 

| Utility           | Purpose                                               | Package         |
| ----------------- | ----------------------------------------------------- | --------------- |
| **PROCESS STATS** |
| top               | Process activity, update dynamic                      | procps          |
| uptime            | How long the system has been running & avg. load      | procps          |
| ps                | Detailed infos. about process                         | procps          |
| pstree            | A tree of processes and their connections             | psmisc / pstree |
| mpstat            | Multiple processor usage                              | sysstat         |
| iostat            | CPU utilization & I/O stats                           | sysstat         |
| sar               | Display and collect information about system activity | sysstat         |
| numastat          | Infos. about NUMA (Non-Uniform Memory Architecture    | numactl         |
| strace            | Infos. about all system calls a process makes         | strace          |
| **MEMORY STATS**  |
| free              | Brief summary of memory usage                         | procps          |
| vmstat            | Detailed vm statistics & block I/O, update dynamic    | procps          |
| pmap              | Process Memory Map                                    | procps          |
| **NETWORK STATS** |
| netstat           | Detailde network stats.                               | netstat         |
| iptraf            | Gather infos. on network interfaces                   | iptraf          |
| tcpdump           | Detailed analysis of network packets/traffic          | tcpdump         |
| wireshark         | Detailed network traffic analysis                     | wireshark       |

## The /proc and /sys Pseudo-fs

Pseudo-filesystem means that they only exists in memory. They are non-existent on a drive that isn't running. 

Informations in this file is only updated when it's looked at, no constant polling.

## /proc

This pseudo-filesystem came from UNIX, originally to display informations on all process (Originally meant to have a folder in /proc for every process). Over time, it got a lot more information and it still used often today. 

### /proc/sys 

Contains information and some settings that are modifyable.

| Subdirectories | Purpose                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------- |
| abi/           | App. binary infos. Barely used.                                                          |
| debug/         | Debug params.                                                                            |
| dev/           | Device params. including cdrom/, scsi/, raid/ and parport/                               |
| fs/            | Filesystems params. including quota, file handles used, maximums, inode, dir infos, etc. |
| kernel/        | Kernel params. Very important.                                                           |
| net/           | Network params. Includes ipv4/, netfilter/, etc.                                         |
| vm/            | Virtual Memoery params. Very important.                                                  |

## /sys

Integral parts of what is termed the UDM (Unified Device Model). It's conceptually a device tree, and on walk through it and see the buses,devices, etc.

Most entries contains only one line of text.

Support for sysfs virtual filesystem is built into all modern kernels, and it should be mounted under /sys.

## sar

Systems Activity Reporter is a all-purpose tool for gathering system activity and performance data & creating reports for humans.

It's backend is sadc (system activity data collector) which accumulate the stats and write them to /var/log/sa 

```bash
sar -A                                          # Almost all info
sar -b                              # I/O and transfer rate stats
sar -B                       # Paging stats including page faults
sar -x                                    # Block device activity
sar -n                                       # Network statistics
sar -P                                            # Per CPU stats
sar -q                                            # Queue lengths
sar -r                        # Swap and memory utilization stats
sar -R                                             # Memory stats
sar -u                                          # CPU utilization
sar -v                # Stats about indoes & files & file handles
sar -w                                  # Context switching stats
sar -W           # Swapping statistics, pages in and out /seconds
sar -f   # Extract information from specified file, created by -o
sar -o        # Save reading in the file specified, created by -f
```


