# Installation

JoularJX is a Java agent, and therfore provided as a ```.jar``` file.
Just use the compiled jar package for JoularJX.

JoularJX requires, at minimum, Java 11+.

JoularJX gets CPU power reading from various sources, depending on the platform or operating system.
In particuliar, it can get the data from two main approaches:
- Get the power data from [Joular Core](https://github.com/joular/joularcore), either through its standard output or through a shared memory buffer ring (IPC). Joular Core works on all platforms and operating systems.
- Get the data through its internal calculation from RAPL (Linux, Windows), powermetrics (macOS) or our own regression models (Raspberry Pi).
- In addition, in virtual machines, JoularJX reads the power consumption of the virtual machine (measured in the host) from a file shared between the host and the guest.

If using JoularJX internal calculation, you need:
- On Windows, to read the data from the RAPL driver, we use a custom program monitor called [Power Monitor for Windows](https://github.com/joular/WinPowerMonitor). It used to be part of JoularJX, but now it is in its own repository. Download the binary (or compile the source code), and specify its path in ```config.properties```. Runs on Intel or AMD CPUs (since Ryzen). You can substitute that software with [Joular Core](https://github.com/joular/joularcore).
Either way (WinPowerMonitor or Joular Core), you need to install the [Windows RAPL driver by Hubblo](https://github.com/hubblo-org/windows-rapl-driver). The easiest way to install the driver is to install the windows version of [Scaphandre](https://github.com/hubblo-org/scaphandre) tool from [this link directly](https://github.com/hubblo-org/scaphandre/releases/download/v1.0.0/scaphandre_v1.0.0_installer.exe) (release 1.0.0).
- On PC/server GNU/Linux, JoularJX uses Intel RAPL interface through powercap, and therefore requires running on an Intel CPU or an AMD CPU (since Ryzen).
- On macOS, JoularJX uses `powermetrics`, a tool bundled with macOS which requires running with `sudo` access. It is recommended to authorize the current users to run `/usr/bin/powermetrics` without requiring a password by making the proper modification to the `sudoers` file.
- On Raspberry Pi devices on GNU/Linux, JoularJX uses our own research-based regression models to estimate CPU power consumption with support for the following device models (we support all revisions of each model lineup.