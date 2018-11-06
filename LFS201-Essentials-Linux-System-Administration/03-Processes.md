# Processes

- [Processes](#processes)
    - [Processes, Programs and Threads](#processes-programs-and-threads)
    - [The init process](#the-init-process)
    - [Processes](#processes-1)
    - [Process Attributes](#process-attributes)
    - [Controlling Processes with ulimit](#controlling-processes-with-ulimit)
    - [Process Permissions and setuid](#process-permissions-and-setuid)
    - [Process States](#process-states)
        - [Running](#running)
        - [Sleeping](#sleeping)
        - [Stopped](#stopped)
        - [Zombie](#zombie)
    - [Execution Modes](#execution-modes)
        - [User Mode](#user-mode)
        - [Kernel Mode](#kernel-mode)
    - [Daemons](#daemons)
    - [Creating Processes in a Command Shell](#creating-processes-in-a-command-shell)
    - [Kernel-Created processes](#kernel-created-processes)
    - [Process creating and forking](#process-creating-and-forking)
    - [Using nice to Set Priorieties](#using-nice-to-set-priorieties)
    - [Static and Shared libs](#static-and-shared-libs)
        - [Static](#static)
        - [Shared](#shared)
    - [Finding Shared Libs](#finding-shared-libs)

## Processes, Programs and Threads

A **process** is an executing program. A program can run one or more process at the same time. If everything is shared, we talk about a multi-threaded process. Some OS have heavy weight/light weight process. On linux, the situation is a bit different: Each thread of execution is considered individually. The difference between an heavy and light process only matters about the sharing of ressource and somewhat faster context switching. Linux respect POSIX and other standards for multi-threaded processes. Each thread returns the same process ID (group ID) and a different thread ID (process ID).

## The init process

The first **user** process on the system is **init** which has a process ID = 1. It's started as soon as possible when the kernel is initialized. It runs until the system is shut down, it serves as the ancestral parent of all other **user** process.

## Processes

A process is an instance of a program in execution. It can be on different states (running/sleeping). Every process have a **pid** (Process ID), a **ppid** (Parent process ID), **pgid** (Process group ID). It also have a program code, data, variables, file descriptors and an environment. If a parent process die before the child, its ppid is set to 1 (adopted by init; replaced by kthreadd on systemd system).

Zombie process: a child process which terminates before its parent. They release almost all their resources and remains alive just to convey their exit status. The init process checks on its adopted children to let those who have terminated die gracefully. Known as the zombie killer or the child reaper.

Processes are controlled by **scheduling**, which is completely preemptive. Only the kernel can kill a process. The largest PID has been limited to 16-bit number, but it's possible to change it with `/proc/sys/kernel/pid_max`, since it may be indaequate.

## Process Attributes

All processes have certain attributes:

* The program being executed
* Context
* Permissions
* Associated ressources

## Controlling Processes with ulimit

**ulimit** is a built-in bash command that displays or resets a number of resource limits

A sysadmin can change some of their value in either direction:

* To **restrict** capabilities (so a process can't take all the cpu time, or system ressources)

* To **expand** capabilities (so a server can open more files)

There is two kinds of limits: Hard (limit set by root user) & Soft (the current value that the user can change to the hard limit).

One can change the limit with ulimit, however it will only change for the current shell. To change for everyone, one needs to modify `/etc/security/limits.conf`

## Process Permissions and setuid

Every process has permissions based on which specific user invoked it. May also have permissions based on who owns its program files.

Programs marked with an s execute bit have a different effective user is than their real user id. These are referred to as **setuid** programs. They run with the user id of the user who **owns** the program. **setuid** programs owned by root can be a security problems. The passwd is a example of a steuid program.

## Process States

Process can be in one of several states, the main ones being

### Running

The process is on the CPU, or sitting in the **run queue**

### Sleeping

The process is waiting on a request (usually I/O)

### Stopped

The process has been suspended. Usually when a process is run under a debugger or Ctrl-Z

### Zombie

The process enters this state when it terminates, and no process (usually the parent) has inquired about its state. Known as **defunct** process

## Execution Modes

A process can be run in either user mode or system mode (known as **kernel mode** by kernel dev). What instruction can be execeuted depends on the mode and is enfored at a hardware-level. The mode is not a state of the sytem; it's a state of the processor. In Intel-world, user mode is also termed Ring 3 and system mode is termed Ring 0.

### User Mode

Except when using system call, processes execute in user mode. It's isolated in its own user space, this promote security and stability, known as **process resource isolation**

### Kernel Mode

In kernel mode, the CPU has full access to all hardware. If an app wants those ressource it needs to issue a sytem call, which cause a context switch.

## Daemons

A **daemon** process is a background process whose sole purpose is to provide a specific service to the user. Daemons can be quite efficient, often ends with d, many are started on boot, no controlling terminal/ no standard I/O.

## Creating Processes in a Command Shell

What happen when a user execuse a command?

1- A new process is created (forked from the user's login shell)

2- A wait system call puts the parent shell process to sleep

3- The cmomand is loaded onto the child process's space via the **exec** system call. i.e, the code for the command replaces the bash program in the child process's memory space

4- The command complete executing, and the child process dies via the exit system call.

5- The parent shell is re-awakened by the death of the child process and proceeds to issue a new shell prompt. The parent shell then waits for the net command requrest from the user, at which time the cycle will be repeated. If there's a `-&-` at the end of the command line (background processing), the parent shell skips the wait request and is free to issue a new shell prompt. Some program don't involve loading of program files such as echo or kill.

## Kernel-Created processes

Not all Processes are created/forked by user parents. The kernel make 2 kinds of processes:

* Internal kernel processes: take care of maintenance work (flushing buffers, load on different CPUs balancing, device drivers queue)
* External user processes: processes that run in user space like normal application but which the kernel started.

`ps -elf` list all processes on the system while showing the parent process IDs. (PPID=2 -> kthreadd and name will be in [])

## Process creating and forking

Linux is always creating new processes. Often called forking. On single-cpu computer, children comes first, but now it's simultaneous.

A fork is usually followed by an exec, where the parent process terminates and the child process inherits the process ID of the parent. Called **fork and exec**.

## Using nice to Set Priorieties

Process priority can be controlled through the `nice` and `renice` commands. A nice process yields more often. It can range between -20(highest priority) to +19(lowest).

example of running nice: `nice -n 5 command [ARGS]

## Static and Shared libs

### Static

Code inserted at combile time, and don't change after.

### Shared

Code is loaded into the program at run time, and if the lib change, the running program runs with the lib modifications.

Shared libs are also called **DLL** (Dynamic Link Library)

They need to be carefully versionned, otherwise **DLL Hell**.

Shared libs under linux have the extension **.so**. The template is filename.so.N where N is the major version of the file.

## Finding Shared Libs

Idd can be used to know which shared libs a exe. requires. It shows the soname of the libs and what file it actually points to.

Idconfig is generally rnu at boot time (can be run anytime) and use the file /etc/ld.so.conf which list the dir that will be searched for shared libs.
