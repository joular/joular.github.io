# Compilation

Joular Core is written in Ada and is built with GPRBuild, or with Alire. A modern GNAT compiler is the only requirement.

## Default Build

With [Alire](https://alire.ada.dev):

```bash
alr build
```

Or directly with GNAT:

```bash
gprbuild -P joularcore.gpr
```

The build produces a static library by default, `libjoularcore.a` in `lib/static/`, for Ada programs.

## Choosing the OS

The build detects the OS on its own to compile the appropriate version: Linux, Windows, macOS, and BSD systems are each recognised from the target GPRBuild reports.

`-XPJ_OS` overrides it when the version to build is not the one of the machine building it, with `linux`, `windows`, `macos` or `bsd`:

```bash
gprbuild -P joularcore.gpr -XPJ_OS=windows
```

OpenBSD has no target name of its own in GPRBuild, so build there with `-XPJ_OS=bsd`. On BSD, build with GPRBuild: the Alire crate is only available on Linux, Windows and macOS.

On Windows, reading the RAPL registers through a driver needs the CPUID instruction of 64 bits x86 processors, to know whether they are the Intel or the AMD ones. Whether the processor is one is detected from the target too, and `-XPJ_X86=True` or `-XPJ_X86=False` overrides it. With `False`, only the Energy Meter Interface is used.

## Library Types

For other library types, set `-XJOULARCORE_LIBRARY_TYPE` (when it is not set, the generic `LIBRARY_TYPE` is used if set, and `static` otherwise):

```bash
gprbuild -P joularcore.gpr -XJOULARCORE_LIBRARY_TYPE=relocatable
```

| Library type | Default | Description |
|---------|:-------:|-------------|
| `static` | **on** | `libjoularcore.a`, linked into an Ada program. |
| `relocatable` | off | The shared library (`libjoularcore.so` / `.dll` / `.dylib`) that carries the C interface, for programs in other languages. |
| `static-pic` | off | A static library built position independent, to go inside someone else's shared library. |

Each type is built in its own folder: `lib/static/`, `lib/relocatable/` or `lib/static-pic/`.

The `relocatable` library is stand-alone (it starts itself up when loaded) and, on every OS but macOS, encapsulated: it carries the Ada runtime too, so it is one self-contained file. On Linux and BSD, it is versioned as `libjoularcore.so.0`.

On macOS it cannot be encapsulated, so the Ada runtime stays a file of its own: the library records the folder of the runtime of the compiler that built it, and loads it from there with nothing to set (no `DYLD_LIBRARY_PATH`, so it also works under `sudo` and from `/usr/bin/java` or the system's Python).

## Building the Examples

The Ada example uses the static library:

```bash
gprbuild -P example/example.gpr
./example/example_joular_core
```

The C example comes with a Makefile that builds the shared library and the program:

```bash
make -C example/c
```

To build it by hand instead, from the root of the repository, first compile the library:

```bash
gprbuild -P joularcore.gpr -XJOULARCORE_LIBRARY_TYPE=relocatable
```

Then compile the C program:

```bash
gcc example/c/main.c -Iinclude -Llib/relocatable -ljoularcore -Wl,-rpath,"$PWD/lib/relocatable" -o example/c/example_c
```

`-I` is the folder holding `joularcore.h`, `-L` and `-l` the library to link with, and `-rpath` the folder where the program looks for the library when it runs. Without `-rpath`, the program still compiles but stops on start because it cannot find the library, unless you set `LD_LIBRARY_PATH` (Linux) or `DYLD_LIBRARY_PATH` (macOS) yourself. Windows has no `-rpath`: put a copy of the DLL next to the program instead.

The Python example only needs the shared library, which its Makefile builds:

```bash
make -C example/python run
```