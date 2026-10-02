# Quick Usage

To use PowerJoular, just run the command ```powerjoular``` (or ```powerjoular.exe``` on Windows).
On Linux PC/servers, PowerJoular uses Intel's RAPL through the Linux powercap sysfs, and therefore requires root/sudo access on the latest Linux kernels (5.10 and newer): ```sudo powerjoular```.
On macOS, ```powermetrics``` also requires root/sudo access: ```sudo powerjoular```.
On Windows and Raspberry Pi, no special access is needed, except with the PawnIO driver on Windows, which needs a terminal with administrative rights (see [Supported Platforms](./supported_platforms.html)).

By default, the software will show the total power consumption, and the power of the CPU and of the GPU (when there is one), on the terminal, every second, until Ctrl+C:

```
Total Power: 18.45 Watts (CPU: 15.20 W, GPU: 3.25 W)
```

To monitor a specific application, just run PowerJoular with the ```-a appName``` parameter, or with ```-p pid``` for a specific process.
The terminal then shows the CPU usage and the CPU power of the application or the process, with the ones of the whole system between brackets:

```
Application monitoring:	CPU: 3.10 % (24.60 %)	1.92 Watts (15.20 Watts)
```

To save the power data to a CSV file, add ```-f filename```, or ```-o filename``` to keep only the latest values in the file.
When a file (or the ring buffer, with ```-r```) is given, PowerJoular no longer prints on the terminal: add ```-t``` to print there too.

PowerJoular is a powerful tool with many functionalities that can be accessed using specific parameters. Check the [Command Line Options page](../ref/options.html) for the detailed list. 