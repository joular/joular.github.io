# Installation

CPU Load is added to the program that uses it rather than installed on its own. It can be taken from Alire, or compiled from source.

## With Alire

In an Ada project managed by [Alire](https://alire.ada.dev), add the library with:

```bash
alr with cpuload
```

Alire fetches the library and builds it along with your program. Then add `with CPU_Load;` to your code (see [Quick Usage](./quick_usage.md)).

Alire offers CPU Load on Linux, Windows and macOS.

## From Source

Clone the [GitHub repository](https://github.com/joular/cpuload) and build it with GNAT and GPRBuild, or with Alire. The build gives:

- a static library (`libcpuload.a`), to link into an Ada program, which is the default
- a shared library (`libcpuload.so` on Linux, `libcpuload.dll` on Windows, `libcpuload.dylib` on macOS), which carries the C interface for programs in C, C++, Java, Python, Rust, etc.

On Linux and Windows, the shared library carries the Ada runtime too, so it is the only file to ship with your program (on Linux, `libcpuload.so.0`, the name programs look for). On macOS, the Ada runtime stays a file of its own, which the library loads from the folder of the compiler that built it.

See [Compilation](../ref/compilation.md) for the build commands and options.

## Platform-specific Requirements

CPU Load is a library, so the privileges below are needed for the *program using it*.

The counters need no particular rights on Linux, Windows or macOS: any user can read the load of the whole machine, and of their own processes.

- **Linux**: the load of other users' processes can be read too, but not the program they run, so they are matched by the name in `/proc/<pid>/comm` instead (see [Supported Platforms](./supported_platforms.md)).
- **macOS**: the processes of other users cannot be read unless the program runs as root (with `sudo`), so their load is negative.
- **Windows**: the processes of other users cannot be read. One of them followed by its ID reads a negative load, and in an application they are left out, as their program cannot be found either (an application made only of them reads `0.0`).