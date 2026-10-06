# Quick Usage

CPU Load has one simple interface: take a sample, wait, take another, then compare the two.

- `Take` reads the CPU counters of the whole machine, and of one process or of one application when one is given.
- `System_Usage` gives the load of the whole machine between two samples.
- `Process_Usage` gives the load of the process or of the application between two samples.

Every load is a share of the whole machine, from `0.0` to `1.0`: a process using all of one core of an eight core machine reads `0.125`, not `1.0`. A negative load means the process or the application could not be read at all. See [Reading the Numbers](../ref/numbers.md) for the details.

## Using from Ada

```ada
with Ada.Text_IO; use Ada.Text_IO;
with CPU_Load; use CPU_Load;

procedure Measure is
    --  The first sample, which the first reading below is measured against
    Before : Sample := Take ("firefox");
    After : Sample;
begin
    for I in 1 .. 5 loop
        delay 1.0;
        After := Take ("firefox");

        --  The same pair of samples gives both figures
        Put_Line ("machine:" & Long_Float'Image (100.0 * System_Usage (Before, After)) & " %");
        Put_Line ("firefox:" & Long_Float'Image (100.0 * Process_Usage (Before, After)) & " %");

        --  This reading becomes the one the next is measured against
        Before := After;
    end loop;
end Measure;
```

To follow several things at once, read the machine once and measure each of them against that one reading. Each is then measured over exactly the same period of time, and the machine's counters are read once instead of once per thing:

```ada
Machine := Take;
Ours := Take (Our_PID, Machine);
Theirs := Take ("firefox", Machine);
```

With Alire, add the library to your project with `alr with cpuload`.

A full example program is in [example/src/example_cpu_load.adb](https://github.com/joular/cpuload/blob/HEAD/example/src/example_cpu_load.adb). It follows the machine, itself, and an application named on the command line, once per second until stopped with Ctrl+C. To build and run it:

```bash
gprbuild -P example/example.gpr -p
./example/example_cpu_load firefox
```

A number after the name stops it after that many readings (`""` follows no application):

```bash
./example/example_cpu_load "" 3
```

## Using from C

The C declarations are in [include/cpuload.h](https://github.com/joular/cpuload/blob/HEAD/include/cpuload.h). Build the relocatable (shared) library (see [Compilation](../ref/compilation.md)), then:

```c
#include <stdio.h>
#include <unistd.h>
#include "cpuload.h"

cpuload_sample before, after;

cpuload_take_app("firefox", &before);
sleep(1);
cpuload_take_app("firefox", &after);

printf("machine: %.2f%%\n", 100.0 * cpuload_system_usage(&before, &after));
printf("firefox: %.2f%%\n", 100.0 * cpuload_process_usage(&before, &after));
```

`cpuload_take_pid_with` and `cpuload_take_app_with` are the same as `cpuload_take_pid` and `cpuload_take_app`, but measure against a machine sample already taken instead of taking another one:

```c
cpuload_sample machine, mine, theirs;

cpuload_take_system(&machine);
cpuload_take_pid_with(getpid(), &machine, &mine);
cpuload_take_app_with("firefox", &machine, &theirs);
```

A full example program is in [example/c/main.c](https://github.com/joular/cpuload/blob/HEAD/example/c/main.c). Like the Ada one, it follows the machine, itself, and an application named on the command line, once per second until stopped with Ctrl+C. It comes with a Makefile that builds the shared library and the program:

```bash
make -C example/c run APP=firefox
```

Run from the root of the repository with no application, for two readings:

```bash
./example/c/example_c "" 2
```

It prints:

```
CPU Load 0.0.4
Following the machine and this program. Name an application to follow it as well: ./example/c/example_c firefox
machine 8.29% | this program 0.00%
machine 7.89% | this program 0.00%
Stopping
```

## Using from Python

From Python, the same interface through ctypes:

```python
import ctypes, time

class Sample(ctypes.Structure):
    _fields_ = [("busy", ctypes.c_int64), ("total", ctypes.c_int64), ("used", ctypes.c_int64)]

lib = ctypes.CDLL("lib/relocatable/libcpuload.so")  # libcpuload.dylib on macOS, libcpuload.dll on Windows
lib.cpuload_system_usage.restype = ctypes.c_double
lib.cpuload_process_usage.restype = ctypes.c_double
lib.cpuload_version.restype = ctypes.c_char_p

before, after = Sample(), Sample()

lib.cpuload_take_app(b"firefox", ctypes.byref(before))
time.sleep(1)
lib.cpuload_take_app(b"firefox", ctypes.byref(after))

print("machine:", 100.0 * lib.cpuload_system_usage(ctypes.byref(before), ctypes.byref(after)), "%")
print("firefox:", 100.0 * lib.cpuload_process_usage(ctypes.byref(before), ctypes.byref(after)), "%")
```

`cpuload_take_pid_with` and `cpuload_take_app_with` are there as well, taking the machine sample as their middle argument.

A full example program is in [example/python/main.py](https://github.com/joular/cpuload/blob/HEAD/example/python/main.py), with a Makefile that builds the shared library:

```bash
make -C example/python run APP=firefox
```

Other languages (C++, Java, Rust, etc.) use the same C interface. See [Integration with Systems and Tools](../ref/integration.md).