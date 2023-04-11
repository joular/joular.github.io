# Compilation

PowerJoular is written with Ada, and requires a modern Ada compiler, such as GNAT.

PowerJoular depends on the following commands and libraries for certain of its functions, but can function without them:
- nvidia-smi: for monitoring power consumption of Nvidia graphic cards
- Linux powercap with Intel RAPL support: for monitoring power consumption of Intel processors and SoC

On a modern GNU/Linux distribution, just install the GNAT compiler (and GPRBuild), usually available from the distribution's repositories:

```
Fedora:
sudo dnf install fedora-gnat-project-common gprbuild gcc-gnat

Debian, Ubuntu or Raspberry Pi OS:
sudo apt install gnat gprbuild
```

For other distributions, use their package manager to download the compiler, or check [this article for easy instruction for various distributions](https://www.noureddine.org/articles/ada-on-windows-and-linux-an-installation-guide), including RHEL and its clones which does not ship with Ada support in GCC.

### Compilation with the GNAT compiler and GPRBuild

To compile the project, just type ```gprbuild``` if using the latest GPRBuild versions.

Or, on older versions, create the ```/obj``` folder first, then type ```gprbuild powerjoular.gpr```.

The PowerJoular binary will be created in the ```obj/``` folder.

By default, the project will statically link the required libraries, and therefore the PowerJoular binary can be copied to any compatible system and used as-is.

To build with dynamic linking, remove or comment the static switch in the ```powerjoular.gpr``` file, in particular these lines:

```
package Binder is
    for Switches ("Ada") use ("-static");
end Binder;
```

### Compilation with the GNAT compiler only

You can also compile PowerJoular with the GNAT compiler only (without the need for GPRBuild).

Just compile using gnatmake. For example, to compile from ```obj/``` folder (so .o and .ali files are generated there), type the following:

```
mkdir -p obj
cd obj
gnatmake ../src/powerjoular.adb
```

### Compilation with Alire

If you have [Alire](https://alire.ada.dev/) installed, you can use it to build PowerJoular with:

```
alr build