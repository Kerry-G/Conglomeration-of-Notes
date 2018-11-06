# Signals

- [Signals](#signals)
    - [What are signals?](#what-are-signals)
    - [Types of signals](#types-of-signals)
    - [kill](#kill)

Signals are notifications for processes to take action to often unplannified events. Often results in termination. It's possible to sent signals from the cli kill, killall and pkill.

## What are signals?

Used to notify processes about async events. There's is 2 path :

* from the kernel to the user process (exception/programming error)
* From a user process to the kernel which will then send it to a user process. Can be the same process sending/receiving it.

Two signals: **SIGKILL** and **SIGSTOP** cannot be handled and will always terminate the program.

## Types of signals

There's a lot different signals; generally they handle two things:

* Exception detected by hardware
* Exception detected by the environment

`kill -l` will show all the different signals

The signals from **SIGRTMIN** are real-time signals. No predefined purpose and differ in the way that they can be queued up. Typing `man 7 signal` can give more information.

## kill

A process can't send a signal to another process; it must ask the kernel via a **system call**. Users can send such a signal by using kill. Kill is not a good name because it doesn't always kill a process, it's real function is to send signals -- even thought some are benign.

**killall** kills all processes with a giving name. It uses a command name rather than a pid.

**pkill** send a signal to a process using selection criteria `pkill [-signal] [option] [pattern]`
