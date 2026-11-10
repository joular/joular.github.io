# CSV Files

With *Record to a CSV file* on, Joular Energy Meter writes a row at every measurement, in the same format as [PowerJoular](https://joular.github.io/powerjoular/ref/exports.html), so the same scripts read both.

The file is added to, not emptied, and a new or empty file starts with a header:

```
Timestamp,CPU Usage,Total Power,CPU Power,GPU Power
1756681930,0.2460,18.4500,15.2000,3.2500
```

The timestamp is in whole Unix seconds, the CPU usage goes from 0.0 to 1.0, the power is in watts, with four digits after the dot.
Each row is written to the disk at once, so the file can be read while measuring, and its rows survive a crash.

When following a process, an application or a command, a second file is written next to the first, named after the process ID or the program's name (```power-144.csv``` or ```power-firefox.csv``` for ```power.csv```).
For a command, the name is its program's, without its folder.
The characters a file name cannot hold are replaced by ```_```.
It has the CPU usage and the power of the followed program:

```
Timestamp,CPU Usage,CPU Power
1756681930,0.0310,1.9154
```

Both value columns hold ```-1.0000``` when the program could not be read at all: it has ended, or the system does not let us read it.
That is not the same as ```0.0000```, a program that was read and used no CPU time.
An application with no running process is written as ```0.0000```, as in PowerJoular.

The carbon emissions, the cost, the idle power and the plug meter are not written to the CSV files, which keep PowerJoular's columns.

## Writing files safely

On Linux, macOS and FreeBSD, the CSV files are only written to plain files: a symbolic link, a hard link, or anything else that is not a plain file is refused, and the reason is shown.
A file created while running with ```sudo``` belongs to you, not to root.

A file that cannot be written is reported once, and the measuring goes on.
