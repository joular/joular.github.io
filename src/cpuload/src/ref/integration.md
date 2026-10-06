# Integration with Systems and Tools

CPU Load is designed to be embedded in other programs and tools. It only measures the CPU load, and leaves to the program using it what to do with the numbers: show them, write them to files, share power out to processes, etc.

For example, [PowerJoular](https://github.com/joular/powerjoular) uses CPU Load together with our [Joular Core](https://github.com/joular/joularcore) library: Joular Core gives the power of the CPU, and CPU Load the load of the machine and of the monitored process or application. The power of the process is then the CPU power times its load, divided by the load of the machine.

See [Library Interface](./interface.md) for the details of the interface. This page focuses on using it from different languages.

## Ada Programs

Ada programs use the `CPU_Load` package directly, and link the static library. With Alire, add the library to your project with:

```bash
alr with cpuload
```

Without Alire, add the folder of the library to the project search path of GPRBuild, and add `with "cpuload.gpr";` to your project file:

```bash
gprbuild -P your_project.gpr -aP../cpuload
```

## C, C++ and Other Languages

Any language with a C FFI can use the shared library (`libcpuload.so` on Linux, `libcpuload.dll` on Windows, `libcpuload.dylib` on macOS) through the C interface in [include/cpuload.h](https://github.com/joular/cpuload/blob/HEAD/include/cpuload.h): C and C++ directly, Python through ctypes, Java through FFM or JNA, Rust through `libloading` or FFI declarations, etc.

A sample is three 64-bit integers, `busy`, `total` and `used`, in this order, with the same layout on every target.

On Linux and Windows, the shared library is one self-contained file, carrying the Ada runtime too, so it is the only file to ship with your program. On Windows, the program looks for the DLL next to it, so put a copy there.

On macOS, the library loads the Ada runtime from the folder of the compiler that built it (see [Compilation](./compilation.md)), so it runs on a Mac where that same compiler is installed in the same folder.

## Python

The [Quick Usage](../guide/quick_usage.md) page shows how to call the library from Python with ctypes, and [example/python/main.py](https://github.com/joular/cpuload/blob/HEAD/example/python/main.py) is a full example.

ctypes takes every result as an `int` unless told otherwise, so declare `ctypes.c_double` as the result of `cpuload_system_usage` and `cpuload_process_usage`, or the loads come back wrong. The full example declares the types of every function it calls.

The library leaves the signal handlers of the program loading it in place, so Ctrl+C raises `KeyboardInterrupt` in Python as usual. The example still puts Python's Ctrl+C handler back after loading the library, as a safeguard.

## Java

From Java, the library can be called through the Foreign Function and Memory API (FFM) or JNA. The shared library does not install any handler for the signals the JVM relies on (such as `SIGSEGV` and `SIGBUS`), so it does not get in the way of the JVM.

## Threads

The shared library must be called from one thread at a time. A sample is a plain record, so it can be taken in one thread and compared in another.

Linked statically into an Ada program, the library can be called from any number of tasks, as it keeps no state.