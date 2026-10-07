# Compilation

CPU Load is written in Ada and is built with GPRBuild, or with Alire. A modern GNAT compiler is the only requirement.

On FreeBSD, GNAT 15 or newer is needed: `pkg install gprbuild gnat15` brings GPRBuild and GNAT 15, whose folder `/usr/local/gnat15/bin` has to be added to `PATH`. GNAT 12, which `pkg install gprbuild` uses, ignores the pragma that keeps the shared library away from the signal handlers of the program loading it. Alire takes the GNAT in `PATH` there too, and refuses an older one.

## Default Build

With [Alire](https://alire.ada.dev):

```bash
alr build
```

Or directly with GNAT:

```bash
gprbuild -P cpuload.gpr
```

The build produces a static library by default, `libcpuload.a` in `lib/static/`, for Ada programs.

## Choosing the OS

The build detects the OS on its own: Linux, macOS, Windows and FreeBSD are each recognised from the target GPRBuild reports, so nothing has to be passed.

`-XPJ_OS` still says which OS to build for (`linux`, `macos`, `windows` or `freebsd`) when it is not the one of the machine building it:

```bash
gprbuild -P cpuload.gpr -XPJ_OS=windows
```

A library built for another OS reads no counters at all: the machine reads 0%, and processes and applications read negative.

## Library Types

For other library types, set `-XCPULOAD_LIBRARY_TYPE` (when it is not set, the generic `LIBRARY_TYPE` is used if set, and `static` otherwise):

```bash
gprbuild -P cpuload.gpr -XCPULOAD_LIBRARY_TYPE=relocatable
```

| Library type | Default | Description |
|---------|:-------:|-------------|
| `static` | **on** | `libcpuload.a`, linked into an Ada program. |
| `relocatable` | off | The shared library (`libcpuload.so` / `.dll` / `.dylib`) that carries the C interface, for programs in other languages. |
| `static-pic` | off | A static library built position independent, to go inside someone else's shared library. |

Each type is built in its own folder: `lib/static/`, `lib/relocatable/` or `lib/static-pic/`.

The `relocatable` library is stand-alone (it starts itself up when loaded) and, on Linux, Windows and FreeBSD, encapsulated: it carries the Ada runtime too, so it is one self-contained file. On Linux and FreeBSD, it is versioned as `libcpuload.so.0`.

On macOS it cannot be encapsulated, so the Ada runtime stays a file of its own: the library records the folder of the runtime of the compiler that built it, and loads it from there with nothing to set (no `DYLD_LIBRARY_PATH`, so it also works under `sudo` and from the system's Python).

## Building the Examples

The Ada example uses the static library:

```bash
gprbuild -P example/example.gpr -p
./example/example_cpu_load firefox
```

The C example comes with a Makefile that builds the shared library and the program:

```bash
make -C example/c run APP=firefox
```

On FreeBSD, the Makefiles need GNU make: run `gmake` instead of `make`.

To build it by hand instead, from the root of the repository, first compile the library:

```bash
gprbuild -P cpuload.gpr -XCPULOAD_LIBRARY_TYPE=relocatable
```

Then compile the C program:

```bash
gcc example/c/main.c -Iinclude -Llib/relocatable -lcpuload -Wl,-rpath,"$PWD/lib/relocatable" -o example/c/example_c
```

`-I` is the folder holding `cpuload.h`, `-L` and `-l` the library to link with, and `-rpath` the folder where the program looks for the library when it runs. Without `-rpath`, the program still compiles but stops on start because it cannot find the library, unless you set `LD_LIBRARY_PATH` (Linux and FreeBSD) or `DYLD_LIBRARY_PATH` (macOS) yourself. Windows has no `-rpath`: put a copy of the DLL next to the program instead (which is what the Makefile does).

The Python example only needs the shared library, which its Makefile builds:

```bash
make -C example/python run APP=firefox
```