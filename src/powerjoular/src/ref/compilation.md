# Compilation

PowerJoular is written with Ada, and requires a modern Ada compiler, such as GNAT, together with the [Joular Core](https://github.com/joular/joularcore) and [CPU Load](https://github.com/joular/cpuload) libraries.

PowerJoular depends on the following libraries and tools for certain of its functions, but can function without them:
- Linux powercap with RAPL support: for monitoring power consumption of Intel and AMD processors
- The Energy Meter Interface, PawnIO or Hubblo's RAPL driver: for monitoring power consumption of Intel and AMD processors on Windows
- The ```cpuctl``` driver (```kldload cpuctl```): for monitoring power consumption of Intel and AMD processors on FreeBSD
- ```powermetrics``` (installed with macOS): for monitoring power consumption of Macs
- NVML (installed with the Nvidia driver): for monitoring power consumption of Nvidia graphic cards
- The amdgpu driver (its hwmon sysfs files): for monitoring power consumption of AMD graphic cards on Linux
- ADLX (installed with the AMD driver): for monitoring power consumption of AMD graphic cards on Windows

None of them is needed to compile PowerJoular: they are looked for when PowerJoular starts.

On a modern Linux distribution, just install the GNAT compiler (and GPRBuild), usually available from the distribution's repositories:

```
Fedora:
sudo dnf install fedora-gnat-project-common gprbuild gcc-gnat

Debian, Ubuntu or Raspberry Pi OS:
sudo apt install gnat gprbuild
```

For other distributions, use their package manager to download the compiler, or check [this article for easy instruction for various distributions](https://www.noureddine.org/articles/ada-on-windows-and-linux-an-installation-guide), including RHEL and its clones which does not ship with Ada support in GCC.

On FreeBSD, use GNAT 15 or newer, which Alire also takes from ```PATH```: ```pkg install gprbuild gnat15``` brings GPRBuild and GNAT 15, whose folder ```/usr/local/gnat15/bin``` has to be added to ```PATH```. GNAT 12, which ```pkg install gprbuild``` uses, crashes while compiling Joular Core.

On Windows and macOS, the easiest way is to install [Alire](https://alire.ada.dev/), which also downloads the GNAT compiler and GPRBuild.

### Compilation with the GNAT compiler and GPRBuild

Check out the [Joular Core](https://github.com/joular/joularcore) and [CPU Load](https://github.com/joular/cpuload) repositories next to the PowerJoular folder, then point GPRBuild at them:

```
gprbuild -P powerjoular.gpr -aP../joularcore -aP../cpuload -p
```

The PowerJoular binary will be created in the ```bin/``` folder.

Linux, macOS, Windows and FreeBSD are each detected on their own, from the target GPRBuild identifies.
To build for another OS than the one you are on, use a cross compiler for that OS (given to GPRBuild with ```--target```), and set ```PJ_OS``` to ```linux```, ```macos```, ```windows``` or ```freebsd```:

```
gprbuild -P powerjoular.gpr -aP../joularcore -aP../cpuload -XPJ_OS=windows -p
```

### Compilation with Alire

If you have [Alire](https://alire.ada.dev/) installed, you can use it to build PowerJoular with:

```
alr build
```

Alire fetches the two libraries on its own, so this is the shortest way.

The PowerJoular binary will be created in the ```bin/``` folder.

### Static and dynamic linking

By default, the project will statically link the Ada runtime and libgcc inside the binary, and therefore the PowerJoular binary can be copied to any compatible system and used as-is.

To build with dynamic linking of the Ada runtime, replace the static switch with the shared one in the ```powerjoular.gpr``` file (removing it is not enough, as GNAT links its runtime statically by default):

```
package Binder is
    for Switches ("Ada") use ("-shared");
end Binder;
```

To leave nothing at all outside the binary, including the C library, build with ```POWERJOULAR_LINKING``` set to ```full```:

```
gprbuild -P powerjoular.gpr -aP../joularcore -aP../cpuload -XPOWERJOULAR_LINKING=full -p
```

On Linux and FreeBSD, a fully static binary cannot reliably load a library while it runs, so the Nvidia graphic card readings, which load NVML while running, can't be counted on with this option.
The processor readings are not affected, and PowerJoular carries on without the GPU rather than failing.

On macOS, Apple ships no static C library, so this option does nothing: the binary is built the same way it is by default.