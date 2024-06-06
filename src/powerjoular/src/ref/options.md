# Command Line Options

To use PowerJoular, just run the command ```powerjoular```.
On PC/servers, PowerJoular uses Intel's RAPL through the Linux powercap sysfs, and therefore requires root/sudo access on the latest Linux kernels (5.10 and newer): ```sudo powerjoular```.

By default, the software will show the power consumption of the CPU and its utilization.
The difference (increase or decrease) of power consumption from the last metric will also be shown.

The following options are available:
- ```-h```: show the help message
- ```-v```: show version number
- ```-p pid```: specifiy a particular PID to monitor
- ```-a appName```: specifiy a particular application name to monitor (will monitor all PIDs of the application)
- ```-f filename```: save monitoring data to the given filename path
- ```-o filename```: save only last monitoring data to the given filename path (file overwritten with only latest power measures)
- ```-t```: print energy data to the terminal
- ```-d```: print debug info to the terminal
- ```-l```: use linear regression models (less accurate than the default polynomial models) for Raspberry Pi energy models
 - ```-m```: specify a filename for the power consumption of the virtual machine
- ```-s```: specify the format of the VM power, either ```powerjoular``` format (generated with the ```-o``` option: 3 columns csv file with the 3rd containing the power consumption the VM), or ```watts``` format (1 column containing just the power consumption of the VM)

You can mix options, i.e., ```powerjoular -tp 144``` will monitor PID 144 and will print to the terminal.