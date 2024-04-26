# Installation

JoularJX is a Java agent, and therfore provided as a ```.jar``` file.
Just use the compiled jar package for JoularJX.

You can also run the installation script in the ```install/``` folder:
- On Windows, run in a command line: ```windows-install.bat```. This will install JoularJX jar and ProgramMonitor to ```C:\joularjx```.
- On GNU/Linux, run in a terminal: ```sh linux-install.sh```. This will install JoularJX to ```/opt/joularjx```.
- on macOS, we don't provide a quick installation script, but you follow the compilation instructions.

JoularJX requires a minimum version of Java 11+.

On Windows, JoularJX uses a custom power monitor program that uses the [Windows RAPL driver by Hubblo](https://github.com/hubblo-org/windows-rapl-driver), and therefore require installing the driver first, and runs on Intel or AMD CPUs (since Ryzen).

On x86_64 Linux, JoularJX uses Intel RAPL interface through powercap, and therefore requires running on an Intel CPU or an AMD Ryzen CPU.

On supported ARM Linux, JoularJX uses our own research-based regression models to estimate CPU power consumption with support for the following, therefore not requiring any additional dependencies.

On macOS, JoularJX uses `powermetrics`, a tool bundled with macOS which requires running with `sudo` access.
It is recommended to authorize the current users to run `/usr/bin/powermetrics` without requiring a password by making the proper modification to the `sudoers` file.