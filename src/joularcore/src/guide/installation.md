# Installation

Joular Core is added to the program that uses it rather than installed on its own. It can be taken from Alire, or compiled from source.

## With Alire

In an Ada project managed by [Alire](https://alire.ada.dev), add the library with:

```bash
alr with joularcore
```

Alire fetches the library and builds it along with your program. Then add `with Joular_Core;` to your code (see [Quick Usage](./quick_usage.md)).

Alire offers Joular Core on Linux, Windows, macOS and FreeBSD. On FreeBSD, it builds with the GNAT in `PATH`, which has to be GNAT 15 or newer (see [Compilation](../ref/compilation.md)).

## From Source

Clone the [GitHub repository](https://github.com/joular/joularcore) and build it with GNAT and GPRBuild, or with Alire. The build gives:

- a static library (`libjoularcore.a`), to link into an Ada program, which is the default
- a shared library (`libjoularcore.so` on Linux and FreeBSD, `libjoularcore.dll` on Windows, `libjoularcore.dylib` on macOS), which carries the C interface for programs in C, C++, Java, Python, Rust, etc.

On Linux, Windows and FreeBSD, the shared library carries the Ada runtime too, so it is the only file to ship with your program (on Linux and FreeBSD, `libjoularcore.so.0`, the name programs look for). On macOS, the Ada runtime stays a file of its own, which the library loads from the folder of the compiler that built it.

See [Compilation](../ref/compilation.md) for the build commands and options.

## Platform-specific Requirements

Joular Core is a library, so the privileges below are needed for the *program using it*.

### Linux (PC / servers)

CPU energy is read via the RAPL powercap sysfs interface. On Linux kernel 5.10 and newer, the RAPL `energy_uj` files are only readable by root. You have two options:

- Run your program with `sudo`
- Grant read access to the RAPL files for your user (see [this GitHub issue](https://github.com/joular/powerjoular/issues/1) for instructions)

GPU monitoring needs the Nvidia driver for Nvidia cards (NVML is installed with it), and the amdgpu kernel driver for AMD cards. Neither needs special privileges.

### Windows

CPU energy is read in one of three ways, depending on what the machine offers:

- The [Energy Meter Interface](https://learn.microsoft.com/en-us/windows-hardware/drivers/powermeter/energy-meter-interface) is the one used by default and checked first. It needs **no installation and no elevated access**, and works only on Windows 11.
- [PawnIO](https://pawnio.eu) is the main RAPL driver used (after EMI): it is maintained and properly signed, and its installer is all that is needed, as Joular Core carries the modules it loads. It needs elevated access, so run the program using the library from a terminal with administrative rights.
- [Hubblo's RAPL driver](https://github.com/hubblo-org/windows-rapl-driver) still works and is used when PawnIO does not answer (not installed, or the program is not run with administrative rights). It does not require elevated access, but its development has paused. The easiest way to install a signed version is through the [Scaphandre installer](https://github.com/hubblo-org/scaphandre/releases/download/v1.0.0/scaphandre_v1.0.0_installer.exe).

GPU monitoring needs the Nvidia driver for Nvidia cards (NVML is installed with it), or the AMD driver for AMD cards (ADLX is installed with it).

### macOS

No additional software is required. Power data is read via `powermetrics`, which ships with macOS. Because `powermetrics` only runs as the superuser, run your program with `sudo`. Without it, both the CPU and the GPU are reported as not available.

### FreeBSD

CPU energy is read from the RAPL registers through the `cpuctl(4)` driver, which is a module not in the GENERIC kernel: load it with `kldload cpuctl` (or `cpuctl_load="YES"` in `/boot/loader.conf`), and run your program as root, or as a member of the `kmem` group (which `/dev/cpuctl0` is part of).

GPU monitoring needs the Nvidia driver for Nvidia cards (NVML is installed with it), and no special privileges.

### Raspberry Pi and SBC

No dependencies and no `sudo` required. Boards with no power model report the CPU as not available.