# Process Monitoring

Keeping track of process is an essential sysadmin task. ps has been a main tool for UNIX-based OS for decades. However, it can get confusing since it got a large assortment of options that can be applied with it.

- [Process Monitoring](#process-monitoring)
    - [Monitoring Tools](#monitoring-tools)
    - [Viewing Process States with PS](#viewing-process-states-with-ps)
    - [BSD](#bsd)
    - [UNIX](#unix)
    - [pstree](#pstree)
    - [top](#top)

## Monitoring Tools

See chapter 11. for available monitoring tools. 

## Viewing Process States with PS

ps is a workhouse for viewing statistics about processes, all of which are from /proc.

ps is a old command, and evolved in a weird way where UNIX, BSD and GNU options aren't write the same way. Can be confusing.

## BSD

`ps aux` 

Most are elf-explanatory, for the other:

* VSZ: Process' vm size in KB
* RSS: Resident set size; the non-swapped physical memory a task is using in KB
* STAT: State of the process. S for sleeping and R for running. Can also contain:
    * <: high priority (not nice)
    * N: low priority (nice)
    * L: pages locked in memory
    * s: session leader
    * l: multi-threaded
    * +: Foreground process group

When adding a f option, it will show processes connect by ancestry

`ps auxf`

## UNIX

`ps -elf`

This one contain also the Parent Process ID (PPID) and the niceness (NI).

* -A/-e: Select all process
* -N: Negate selection
* -C: Select by command name
* -G: Select by real group ID
* -U: Select by real user ID
* -o: Print out a customized list of ps field with:
    * pid: Process id
    * uid: User id
    * cmd: Command w/ all args.
    * cputime: Cumulative CPU time
    * pmem: Ratio of the process' resident set size/physical memory

## pstree

pstree gives a visual desc. of the process ancestry/multi-threaded app.

```bash
machine:~/$ pstree -aAp 3730

dockerd,3730 -H unix://
  |-{dockerd},3804
  |-{dockerd},3805
  |-{dockerd},3806
  |-{dockerd},3807
  |-{dockerd},3809
  |-{dockerd},3830
  |-{dockerd},3832
  |-{dockerd},3866
  |-{dockerd},3942
  |-{dockerd},3943
  |-{dockerd},3944
  |-{dockerd},3991
  |-{dockerd},4082
  |-{dockerd},4097
  |-{dockerd},4098
  `-{dockerd},4099
```

## top

When one want to know whats the system is spending its time on: top is the first tool to use. By default, it refresh every 3s.

When active, you can press 1 to switch CPU, and i for seeing only active processes.

* h/?: bried list of interactice commands
* q: quits
* k: kill
* r: renice