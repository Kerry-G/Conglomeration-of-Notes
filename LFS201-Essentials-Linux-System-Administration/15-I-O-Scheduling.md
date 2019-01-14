# I/O Scheduling

System oten depends very heavily on optimizing the I/O scheduling strategy. Linux offers a variety of I/O schedulers to choose from, each of which has tunable parameters, and a # of utilities for reporting on and analyzing performance.

- [I/O Scheduling](#io-scheduling)
  - [I/O Scheduler](#io-scheduler)
    - [I/O Scheduler Choices](#io-scheduler-choices)
  - [I/O Scheduling and SSD Devices](#io-scheduling-and-ssd-devices)
  - [Tunables and Switching the I/O Scheduler at Runtime](#tunables-and-switching-the-io-scheduler-at-runtime)
- [CFQ Scheduler](#cfq-scheduler)
    - [CFQ Tunables](#cfq-tunables)
  - [Deadline Scheduler](#deadline-scheduler)
    - [Deadline Tunables](#deadline-tunables)

## I/O Scheduler

The I/O Scheduler provide the interface between the generic block layer and low-level physical device driver. Both VM (Virtual Memory) and VFS (Virtual File System) layers submit I/O requests to the block device. The Scheduler will prioritize and order these requests before they are given to the block devices.

Any I/O scheduling algorithm has to satisfy certain requirements:

* Hardware access should me minimized (Requests should be ordered according to physical location on the disk; lead to an **elevator** scheme)

* Request should be merged to the extent possible to get as big a contiuous region as possible; which also minizes disk access

* Requests should be satisfied with as low a latency as is feasible; determinism (in the sense of deadline) may be important.

* Write ops can usually wait to migrate from caches to disk without stalling processes. Read ops almost always require a process to wait for completion before proceeding further. Favoring reads > writes leads to better parallelism and system responsiveness.

* Processes should share the I/O bandwidth in a fair fashion; even if it means some overall performance slowdown of the I/O layer.

### I/O Scheduler Choices

Since these demands can be conflicting; different I/O schedulers may be appriopriate for different workloads. In order to provide the flexibility, the Linux kernel has an object oriented scheem, in which pointers to the various needed functions are supplied in a data structure, the particular one of which can be selected at boot on the kernel command : `linux ... elevator=[cfq|deadline|noop]`

At least one of the I/O scheduling algorithms must be complied into the kernel. The current choices are: 

* Completely Fair Queuing (CFQ)
* Deadline Scheduling
* noop (a simple scheme)

Modern distros choose eitehr CFQ or Deadline.

## I/O Scheduling and SSD Devices

SSD Devices does not require an elevator scheme and benefits from wear leveling to spread I/O over the devices which has limited write/erase cycles.

One can examine `/sys/block/sda/queue/rotational` to see wether or not the device is an SSD or not. (1 means HDD in the sense that it is rotational type and 0 means SSD)

## Tunables and Switching the I/O Scheduler at Runtime

Each of the I/O schedulers exposes parameters that can be tuned at run time. The parameters are accessed through the pseudo-filesystem mounted at /sys

IN addition, it is possible to use different I/O schedulers for different devices. 

```
$ cat /sys/block/sda/queue/scheduler
noop deadline [cfq]

$ echo noop > /sys/block/sda/queue/scheduler
$ cat /sys/block/sda/queue/scheduler
[noop] deadline cfq
```

# CFQ Scheduler

The CFQ method has the goal of equal spreading of I/O bandwidth among all processes submitting requests. 

Theoretically, each processes has its own I/O queue, which works together with a dispatch queue which receives the actual requests on the way to the device. In reality, the # of queues is fixed (at 64) and a hash process based on the pid is used to select a queue when a request is submitted.

Dequeing of requests is done round robin style on al the queus, each one of which works in FIFO order. Thus, the work is spread out. To avoid excessive seeking operations, and entire round is selected, and then sorted into the dispatch queue before actual I/O requests are issued to the device.

### CFQ Tunables

In the examples below, the parameter HZ is a kernel-configured quantity, that coreresponds to the # of **jiffes** per second, which the kernel uses as a coarse measure of time. Without getting into details, let's assume that HZ/2 = 0.5 seconds and 5HZ = 5 seconds, etc.

| Name              | Meaning                                   | Default |
| ----------------- | ----------------------------------------- | ------- |
| quantum           | Max. queue length in one round of service | 4       |
| queue             | Min request alloc per queue               | 8       |
| fifo_expire_sync  | FIFO timeout for sync request             | HZ/2    |
| fifo_expire_async | FIFO timeout for async requests           | 5HZ     |
| fifo_batch_expire | Rate at which the FIFO's expire           | HZ/8    |
| back_seek_max     | Max backwards seek, in KB                 | 16K     |
| back_seek_penalty | Penalty for a backwards seek              | 2       |

## Deadline Scheduler

The Deadline I/O scheduler aggressively re-orders requests with the simultaneous goals of improving overall performance and preventing large latencies for individual requests (limiting starvation)

Every request, the kernal associates a **deadline**. Read requests get higher priority than write requests.

Five separate I/O queue are maintained:

* Two sorted list, one for reading and one for writing, and arranged by starting block

* Two **FIFO** lists, one for reading and one for writing. These lists are sorted by submission time.

* A fifth queue contains the requests that are to be shoveled to the device driver iteseld. This is called the dispatch queue.

Exactly how the requests are peeled off the first four queues and placed on the fifth (dispatch queue) is where the art of the algorithm is.

### Deadline Tunables 

| Name           | Meaning                                                                                                                          | Default |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------- |
| read_expire    | How long a read request is guaranteed to occur within                                                                            | HZ/2    |
| write_expire   | How long a write req. is guaranteed to occur within                                                                              | 5HZ     |
| writes_starved | How many req. we should give the preference to reads over write                                                                  | 2       |
| fifo_batch     | How many requests should be moved from the sorted scheduler list to the dispath queue, when deadline has exipred                 | 16      |
| front_merged   | Back merges are more common than front merges as a contiguous req. usually continues to the next block. 0: disabled front merges | 1       |

