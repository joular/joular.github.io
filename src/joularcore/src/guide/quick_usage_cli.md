# Quick Usage of the command-line version

To use Joular Core, just run the command ```joularcore``` (on Linux or macOS), or ```joularcore.exe``` (on Windows).

On Intel or AMD PC/servers on Linux, Joular Core uses Intel's RAPL through the Linux powercap sysfs, and therefore requires root/sudo access on the latest Linux kernels (5.10 and newer): ```sudo joularcore```.
An alternative is to change the permissions on the RAPL powercap files to provide read access to all. Check [this issue in our GitHub](https://github.com/joular/joularcore/issues/12) on the matter. This is a limitation for all power monitoring tools on Linux using RAPL, and not specific to Joular Core.

On macOS, you also need to run Joular Core with elevated access (thus `sudo`): ```sudo joularcore```.

On Windows, just run ```joularcore.exe```.

On Raspberry Pi devices with Linux, just run ```joularcore```.

By default, the software will show the power consumption of the CPU, GPU, total power, and CPU utilization.

To monitor a specific application, just run PowerJoular with the ```-a appName``` parameter, and a specific process with the ```-p PID```.
PowerJoular is a powerful tool with many functionalities that can be accessed using specific parameters. Check the [Command Line Options page](../ref/options.html) for the detailed list. 