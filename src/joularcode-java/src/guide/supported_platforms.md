# Supported Platforms

Joular Code for Java supports the following platforms and operating systems:

- PC/Servers using a RAPL supported Intel processor (since Sandy Bridge) or a RAPL supported AMD processor (since Ryzen), on Linux and on Windows.
- Macs, with Apple Silicon or Intel processors.
- Raspberry Pi devices and Asus Tinker Board, on Linux.
- Virtual machines (any supported guest on any host).

On Linux PC/Servers, Joular Code for Java can read RAPL directly, with nothing else to install.
On Windows, macOS and Raspberry Pi, it gets the CPU power from [PowerJoular](https://github.com/joular/powerjoular) (version 2.0.0 or later), which runs alongside the monitored application and writes its power data to a shared memory ring buffer.
In a virtual machine, it reads the power that the host writes to a file shared with the guest.
See [Power Sources](../ref/power_sources.md) for the details.

Only the power of the CPU is attributed to methods. The power of the GPU is not taken into account.

On Raspberry Pi and Asus Tinker Board, the power comes from the research-based regression models of PowerJoular.
The supported Raspberry Pi and Asus Tinker Board models are listed below.
We support all revisions of each model lineup.
However, the model is generated and trained on a specific revision (listed between brackets), and the accuracy is best on this particular revision.

- Raspberry Pi devices (multiple models) on Linux:
  - Model Zero W (rev 1.1), for 32 bits OS
  - Model 1 B (rev 2), for 32 bits OS
  - Model 1 B+ (rev 1.2), for 32 bits OS
  - Model 2 B (rev 1.1), for 32 bits OS
  - Model 3 B (rev 1.2), for 32 bits OS
  - Model 3 B+ (rev 1.3), for 32 bits OS
  - Model 4 B (rev 1.1, and rev 1.2), for both 32 bits and 64 bits OS
  - Model 400 (rev 1.0), for 64 bits OS
  - Model 5 B (rev 1.0), for 64 bits OS
- Asus Tinker Board (S)

The models listed for 32 bits OS are also used on a 64 bits OS.

| Platform | Supported OS | Based on | Supported Architecture |
|:--------------:|:---------------------:|:-----------------------------:|:-----------------------------:|
|     Linux PC/Server    |        Linux        |             RAPL (using powercap), or PowerJoular            |             x86, x86_64            |
|     Windows PC/Server    |        Windows        |             PowerJoular (RAPL through EMI, PawnIO or Hubblo's driver)            |             x86_64            |
|     Mac    |        macOS        |             PowerJoular (powermetrics)            |             Apple Silicon (ARM), Intel (x86_64)            |
|        Raspberry Pi       |        Linux       |             PowerJoular (our regression models)             |             ARM            |
|        Asus Tinker Board       |        Linux       |             PowerJoular (our regression models)             |             ARM            |
|     Virtual Machine    |        Supported guests (Windows, Linux, macOS), any host       |             Host's architecture (RAPL, regression models, others)            |             x86, x86_64, ARM            |

## Java Virtual Machine

Joular Code for Java requires Java 21 or later.

It also requires `com.sun.management.OperatingSystemMXBean`, to measure the CPU time of the process and the CPU load of the system.
This is available in all standard HotSpot JVMs (OpenJDK, Oracle JDK). Minimal or embedded JVMs that do not provide this class are not supported.

The JVM must also report the CPU time of its threads (`ThreadMXBean.isThreadCpuTimeSupported()`).
If any of these is missing, Joular Code for Java does not start, and the application runs unmonitored.