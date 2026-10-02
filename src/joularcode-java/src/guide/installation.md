# Installation

Joular Code for Java is a Java agent, and therefore provided as a `.jar` file.
Just use the compiled JAR package of Joular Code for Java, from the releases of our [GitHub repository](https://github.com/joular/joularcode-java/releases), or compile it yourself (see [Compilation](../ref/compilation.md)).

The JAR has no runtime dependencies: it holds nothing but its own classes, and needs no additional classpath setup.
You can install it wherever you want, and give its full path to `-javaagent`.

Joular Code for Java requires, at minimum, Java 21, to run the application being monitored.

Joular Code for Java gets the CPU power from one of three sources, depending on the platform or operating system:
- On Linux PC/Servers, it reads RAPL directly through powercap, on Intel or AMD CPUs (since Ryzen). The RAPL files are only readable by root on most distributions (Linux kernel 5.10 and newer): run the application as root, give its user read access to these files (for example with a udev rule), or run PowerJoular with `-r` as root and set `power-source-type=ringbuffer`.
- On Windows, macOS and Raspberry Pi devices (and on Linux, with `power-source-type=ringbuffer`), it reads the shared memory ring buffer of [PowerJoular](https://github.com/joular/powerjoular). Install PowerJoular (version 2.0.0 or later), and run it with the `-r` option alongside your application. On Windows, start PowerJoular before the application, and do not restart it while the application runs.
- In virtual machines, set `power-source-type=vm` and `vm-power-file`: it then reads the power consumption of the virtual machine (measured in the host) from a file shared between the host and the guest (see [Virtual Machines](../ref/vm.md)).

PowerJoular needs `sudo` on macOS, and on Linux PC/Servers unless its user can read the RAPL files, and a terminal with administrative rights on Windows with the PawnIO driver.
The Java application needs none of these.

A commented example of the configuration file, `joularcodejava.properties.example`, is available in the repository and with the releases.
Copy it as `joularcodejava.properties` to the folder where you run the Java command, or give its path with `-Djoularcodejava.properties` (see [Configuration Properties](../ref/configuration.md)).
Without a configuration file, the default values are used.