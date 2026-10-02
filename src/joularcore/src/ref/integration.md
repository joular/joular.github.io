# Integration with Systems and Tools

Joular Core is designed to be embedded in other programs and tools. It only measures the hardware, and leaves to the program using it what to do with the measurements: show them, write them to files, attribute them to processes, etc.

For example, [PowerJoular](https://github.com/joular/powerjoular) uses Joular Core to measure the CPU and the GPU, together with our [CPU Load](https://github.com/joular/cpuload) library to share the CPU power out to processes and applications, and exports the result to the terminal, CSV files and a shared memory ring buffer.

See [Library Interface](./interface.md) for the details of the interface. This page focuses on using it from different languages.

## Ada Programs

Ada programs use the `Joular_Core` package directly, and link the static library. With Alire, add the library to your project with:

```bash
alr with joularcore
```

Without Alire, add the folder of the library to the project search path of GPRBuild, and add `with "joularcore.gpr";` to your project file:

```bash
gprbuild -P your_project.gpr -aP../joularcore
```

## C, C++ and Other Languages

Any language with a C FFI can use the shared library (`libjoularcore.so` on Linux, `libjoularcore.dll` on Windows, `libjoularcore.dylib` on macOS) through the C interface in [include/joularcore.h](https://github.com/joular/joularcore/blob/main/include/joularcore.h): C and C++ directly, Python through ctypes, Java through FFM or JNA, Rust through `libloading` or FFI declarations, etc.

The two structures of the C interface have the same layout on every target, with no padding: a `double` followed by two `int` for one measurement, and the CPU measurement followed by the GPU one for a reading.

On every OS but macOS, the shared library is one self-contained file, carrying the Ada runtime too, so it is the only file to ship with your program. On Windows, the program looks for the DLL next to it, so put a copy there.

On macOS, the library loads the Ada runtime from the folder of the compiler that built it (see [Compilation](./compilation.md)), so it only runs on a Mac where that same compiler is installed in the same folder.

## Python

The [Quick Usage](../guide/quick_usage.md) page shows how to call the library from Python with ctypes, and [example/python/main.py](https://github.com/joular/joularcore/blob/main/example/python/main.py) is a full example.

The library leaves the signal handlers of the program loading it in place, so Ctrl+C raises `KeyboardInterrupt` in Python as usual.

## Java

From Java, the library can be called through the Foreign Function and Memory API (FFM) or JNA. The shared library does not install any handler for the signals the JVM relies on (such as `SIGSEGV` and `SIGBUS`), so it does not get in the way of the JVM.

## Threads

The library is not thread safe: call `Open`, `Read` and `Close` from a single thread, as one monitoring loop is the intended use for the current version. A program that needs the measurements in several threads can read them in one thread and share the values with the others.