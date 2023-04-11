# Quick Usage

To use PowerJoular, just run the command ```powerjoular```.
On PC/servers, PowerJoular uses Intel's RAPL through the Linux powercap sysfs, and therefore requires root/sudo access on the latest Linux kernels (5.10 and newer): ```sudo powerjoular```.

By default, the software will show the power consumption of the CPU and its utilization.
The difference (increase or decrease) of power consumption from last metric will also be shown.

To monitor a specific application, just run PowerJoular with the ```-a appName``` parameter.
PowerJoular is a powerful tool with many functionalities that can be accessed using specific parameters. Check the [Command Line Options page](../ref/options.html) for the detailed list. 