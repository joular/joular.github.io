# Supported Platforms

CPU Load runs on Linux, Windows and macOS, on PCs, servers, Macs and single-board computers such as the Raspberry Pi. It reads what the operating system counts, not the hardware, so it works the same whatever the processor.

| What is measured | OS | Method |
|---|---|---|
| The whole system | Linux | The `cpu` line of `/proc/stat` |
| The whole system | macOS | `host_statistics`, macOS' CPU counters |
| The whole system | Windows | `GetSystemTimes` |
| One process, by its number | Linux | `utime` + `stime` of `/proc/<pid>/stat` |
| One process, by its number | macOS | `proc_pidinfo`, the user and system time of the process |
| One process, by its number | Windows | `OpenProcess` + `GetProcessTimes` |
| An application, every process of it | Linux | `/proc` scanned, each process named by `/proc/<pid>/exe` |
| An application, every process of it | macOS | `proc_listpids`, each process named by `proc_pidpath` |
| An application, every process of it | Windows | `EnumProcesses` + `QueryFullProcessImageNameW` |

BSD support is planned and will come in a future version.

## Matching an Application

An application is given by the name of its program, without its folder. The name must match exactly, but upper or lower case does not matter.
Every OS matches the program with the process that actually runs, so `firefox` finds every process of Firefox, its content processes included.

**Linux**  
A process is named by the program `/proc/<pid>/exe` points to. A process whose `/proc/<pid>/exe` cannot be read (such as a kernel thread, which runs no program directly, or another user's process) falls back on `/proc/<pid>/comm`: the name the process was given, cut to 15 characters.

**macOS**  
A process is named by the program inside the bundle, so `firefox` finds the application inside `Firefox.app`, and also every program inside `Firefox.app`: for example, its content processes run `plugin-container`, from a helper bundle inside it.

**Windows**  
A trailing `.exe` is ignored, so `firefox` also finds `firefox.exe`.

## Macs

macOS is supported on Apple Silicon and Intel Macs, with the library built for the chip it runs on.
An x86_64 build running under Rosetta on an Apple Silicon Mac reads process times about 40 times too low, so process and application loads come out close to 0%.