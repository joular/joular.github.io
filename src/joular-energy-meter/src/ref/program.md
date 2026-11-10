# Measuring a Program

Joular Energy Meter follows one program at a time, besides the whole machine:
- **Process**: one process, by its ID.
- **Application**: every process whose program has that name (```firefox```, not *Firefox.app*). On macOS, all the programs inside the application bundle count too. On Windows, a trailing ```.exe``` is ignored.
- **Command**: a command started by Joular Energy Meter at Start, and measured from its first instant.

## The power of a program

The power of the followed program is its share of the processor's power only, as in PowerJoular:

```
program power = CPU power × program CPU usage / machine CPU usage
```

The graphics card is not shared out to programs.

*Share of CPU* is the fraction of the machine's processing capacity the program used (all of one core out of eight is 12.5 %), not of the processor's power.
The share of the machine shown next to the program's name is its power divided by the power of the CPU and the GPU.

So on a quiet machine, a program gets most of the processor's idle power too.
*Above idle* takes it out (see [Idle Power and Plug Meter](./idle_plug.html)).

## A program that cannot be read

A process that has ended, or that the system does not let us read, shows a dash instead of 0 %, and is written as ```-1``` in the CSV files, as PowerJoular does.
A process that ended stays ended: a new process given the same ID later is not followed.

## Commands

On Linux, macOS and FreeBSD, the command runs through ```/bin/sh```, so ```python3 bench.py > out.txt``` works, and the program itself is followed, not the shell.
On Windows, it is a program and its arguments, without the shell's ```>``` or ```&&```.

The command starts in the current folder.
Started with ```sudo```, Joular Energy Meter runs the command as you, not as root.
Started as administrator on Windows, it cannot run commands.

Only the command's own process counts as the program: the processes it starts (the compilers of ```make```, for example) are in the machine's reading, not in the program's.

When the command ends, its exit status is shown, and the measuring stops if the length is *Until it ends*.
**Stop** leaves the command running.
