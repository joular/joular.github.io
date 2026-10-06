# How CPU Load Works

This page describes the internal architecture of CPU Load: what a sample holds, how a load is calculated, how each OS is read, and how to add a new OS.

## Samples and Loads

A sample holds three counters, read at one moment, in microseconds:

- `Busy`: the machine time not spent idle, summed over every core.
- `Total`: the machine time there was to spend: the elapsed time times the number of cores.
- `Used`: the CPU time of the process or of the application sampled, and 0 for the system alone.

These counters count up, so a load is what changed between two samples:

```
system load  = (Busy after − Busy before) / (Total after − Total before)
process load = (Used after − Used before) / (Total after − Total before)
```

As `Total` covers every core, a load is a share of the whole machine.

When a process or an application is sampled, the counters of the machine are read first, and the process right after. The small lag between the two is the same in both samples, so it cancels out.

An application is every process running its program at the moment of the sample: CPU Load lists the processes, keeps the ones running the program named, and adds up their CPU time. A process that could not be read is left out of the sum.

## Linux

**The machine.** The first line of `/proc/stat` gives the time of all the cores together, in clock ticks (usually 100 per second, as `sysconf` says). `Busy` is user, nice, system, irq, softirq and steal. The idle time is idle and iowait. The guest times are already inside user and nice, so they are not added again.

**A process.** `utime` and `stime` of `/proc/<pid>/stat` give the time the process spent running its own code and in the kernel. The name of the process, in brackets, may hold spaces or brackets, so the fields are counted from the last `)`. A process in state `Z` (zombie), `X` or `x` (dead) has ended, and is not read.

**An application.** `/proc` is scanned, and each process is named by the program `/proc/<pid>/exe` points to, without the ` (deleted)` the kernel adds when the program's file was replaced while it runs. When it cannot be read, the name in `/proc/<pid>/comm` is used instead.

The files are read through file descriptors rather than `Ada.Text_IO`, whose `Open` fails when several tasks call it at once.

## macOS

**The machine.** `host_statistics` gives the user, system and nice ticks of all the cores (100 per second), which make `Busy`. `Total` comes from the monotonic clock (`mach_absolute_time`) times the number of cores, rather than from the idle ticks, which macOS updates only every 90 ms or so.

**A process.** `proc_pidinfo` gives the user and system time of the process, in the units of `mach_absolute_time`, turned into microseconds with `mach_timebase_info` (125/3 on Apple Silicon, 1/1 on Intel). It fails for the processes of other users unless the program runs as root.

**An application.** `proc_listpids` lists the processes, and `proc_pidpath` gives the full path of each one's program. A process is part of the application when its program has the name given, or when it runs from inside the bundle of that name (`<name>.app`), as Firefox runs its content processes from a helper bundle inside `Firefox.app`.

Under Rosetta, a process is still counted in the units of Apple Silicon (24 MHz), which an x86_64 build reads with the units of an Intel Mac: process times come out about 40 times too low.

## Windows

**The machine.** `GetSystemTimes` gives the idle, kernel and user times, in 100 ns units. The kernel time includes the idle time, so `Total` is the kernel and user times, and `Busy` is `Total` without the idle time.

**A process.** `OpenProcess` (asking for limited information only), then `GetProcessTimes`, give the kernel and user times of the process. The processes of other users cannot be opened, so they are not read. A process that has ended can still be queried while a handle to it is held. Its exit time, which stays zero until it ends, tells it apart, and such a process is not read.

**An application.** `EnumProcesses` lists the processes (with room for twice as many each time the list fills up, up to 65536), and `QueryFullProcessImageNameW` gives the path of each one's program, through the same `OpenProcess`: the processes of other users cannot be named, so they are left out. A trailing `.exe` is ignored on both sides of the comparison.

## Code Layout

| Path | Purpose |
|------|---------|
| `src/cpu_load.ads`, `src/cpu_load.adb` | The interface (`Sample`, `Take`, `System_Usage`, `Process_Usage`, `Version`), and what an application is, the same on every OS |
| `src/cpu_load-platform.ads` | The four functions each OS provides |
| `src/cpu_load-c_api.ads`, `src/cpu_load-c_api.adb` | The C interface, matching `include/cpuload.h` |
| `src/linux/`, `src/macos/`, `src/windows/` | One body of `CPU_Load.Platform` per OS (`src/macos` also holds the specs file recording where the shared library finds the Ada runtime) |
| `include/cpuload.h` | The C declarations |
| `example/` | Example programs in Ada, C and Python |

## Adding a New OS

The package spec `src/cpu_load.ads` and its body `src/cpu_load.adb` are shared by every OS. They hold the `Take` and usage functions, and what an application is: every process running its program, their times added up.

Each OS has its own body of `src/cpu_load-platform.ads`, which has four functions about the machine:

- `Measure_System`: the machine's own CPU counters
- `Used_By_PID`: the CPU time one process has used
- `Runs`: whether a process runs the program an application is named by
- `For_Each_Process`: every process running

One body per OS lives in `src/linux`, `src/macos` and `src/windows`, and `cpuload.gpr` picks the folder for the OS being built from `PJ_OS`.

To support a new OS, write a body of `CPU_Load.Platform` for it, then add the OS and its folder to `PJ_OS` in `cpuload.gpr`, and to `alire.toml`.