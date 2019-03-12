# Monitoring and Tuning 

- [Monitoring and Tuning](#monitoring-and-tuning)
  - [Disk Bottlenecks](#disk-bottlenecks)
  - [iostat](#iostat)
    - [Options](#options)
  - [iotop](#iotop)
  - [Using ionice to Set I/O priorities](#using-ionice-to-set-io-priorities)
    - [I/O Scheduling Classes](#io-scheduling-classes)

## Disk Bottlenecks

Disk performance problems can be coupled with other problems, like insufficient memory or inadequate network hardware/tuning.

A system can be considered as I/O-bound when the CPU is idle waiting for an I/O operation to be done. There are many variables to this and I/O is complex, we will consider I/O scheduling later.

## iostat

iostat is a utility for monitoring I/O device activity on the system. It generate reports with a lot of precise informations.

After a brief summary of the CPU usage, I/O stats are given: 

* tps (I/O transaction per seconds)
* blocks read and written per unit time (blocks are generally sectors of 512 bytes)
* total blocks read and written

Information are broken out by disk partition.

### Options

| Options | Purpose              |
| ------- | -------------------- |
| -k      | results in kb        |
| -m      | results in mb        |
| -N      | show by device name  |
| -x      | More detailed report |

When using -x; more fields are in the report such as (time are in msecs):

| Field    | Meaning                                                                             |
| -------- | ----------------------------------------------------------------------------------- |
| Device   | device/partition name                                                               |
| rrqm/s   | # of read request merged /s, queued to device                                       |
| wrqm/s   | # of write request merged /s, queued to device                                      |
| r/s      | # of read request /s, queued to device                                              |
| w/s      | # of write request  /s, queued to device                                            |
| rkB/s    | KB read from the device /s                                                          |
| wkB/s    | KB written to the device /s                                                         |
| avgrq-sz | Average request size in 512 bytes sectors / s                                       |
| avgqu-sz | Average queue length of request issued to the device                                |
| await    | Average time I/O requests between when a request is issued and when it is completed |
| svctm    | Average service time for I/O requests                                               |
| %util    | Percentage of CPU time during the device serviced requests                          |

Notes: : await is queue time + service time

## iotop

Another usefull tool but must be run as root. It display a table of current I/O usage and update periodically; like top.

## Using ionice to Set I/O priorities

The **ionice** utility can set the I/O scheduling class and priority for a given process. It takes the form of `ionice [-c class] [-n priority] [-p pid] [COMMAND [ARGS] ]`. If the pid is given, it will apply the changes on the given process, otherwise it will apply them on the process started with the COMMAND.

### I/O Scheduling Classes

| I/O Scheduling Class | -c value | Meaning                                                        |
| -------------------- | -------- | -------------------------------------------------------------- |
| None or Unknown      | 0        | Default Value                                                  |
| Real Time            | 1        | Get first access to the disk, can starve others                |
| Best Effort          | 2        | All programs serviced in a round-robin fashion                 |
| Idle                 | 3        | No access to disk I/O unless no other program has asked for it |

Both Real Time and Best Effort takes the the -n argument which gives the priority of range 0-7 with 0 being the highest priority. 

Note: ionice works only when using the *CFQ* I/O Scheduler.