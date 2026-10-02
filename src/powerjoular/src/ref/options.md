# Command Line Options

To use PowerJoular, just run the command ```powerjoular```.
On Linux PC/servers, PowerJoular uses Intel's RAPL through the Linux powercap sysfs, and therefore requires root/sudo access on the latest Linux kernels (5.10 and newer): ```sudo powerjoular```.
On macOS, ```powermetrics``` also requires root/sudo access: ```sudo powerjoular```.

By default, the software will show the total power consumption, and the power of the CPU and of the GPU (when there is one), on the terminal, every second, until Ctrl+C.

The following options are available:
- ```-h```: show the help message
- ```-v```: show version number
- ```-p pid```: specify a particular PID to monitor
- ```-a appName```: specify a particular application name to monitor (will monitor all PIDs of the application)
- ```-f filename```: save monitoring data to the given filename path, in CSV format
- ```-o filename```: save only last monitoring data to the given filename path (file overwritten with only latest power measures)
- ```-t```: print the power data to the terminal
- ```-d```: print the available hardware components on start up (the versions of PowerJoular and its libraries, whether the CPU and the GPU can be measured, and the path of the ring buffer with ```-r```)
- ```-r```: write the power data to a shared memory ring buffer, for another program to read with low latency
- ```-m filename```: specify a filename for the power consumption of the virtual machine (written by the host)
- ```-s format```: specify the format of the VM power, either ```powerjoular``` format (generated with the ```-o``` option for a monitored process: 3 columns csv file with the 3rd containing the power consumption of the VM), or ```watts``` format (1 column containing just the power consumption of the VM)

You can mix options, i.e., ```powerjoular -tp 144``` will monitor PID 144 and will print to the terminal.

A few rules apply when mixing them:
- ```-p``` and ```-a``` cannot be used together.
- With ```-f```, ```-o``` or ```-r```, the terminal is only used if ```-t``` is also given.
- ```-m``` and ```-s``` go together, and the file given to ```-m``` must already hold a power value in that format when PowerJoular starts.

When monitoring a process (```-p```) or an application (```-a```), ```-f``` and ```-o``` write **two** CSV files: the given filename for the whole system, and the same name with the PID or the application name added to it, for the process or the application.
For example, ```powerjoular -p 144 -f power.csv``` writes ```power.csv``` and ```power.csv-144.csv```.

See [Exporting Power Data](./exports.html) for the content of the CSV files and of the ring buffer.

On Windows, the ```JOULARCORE_WINDOWS_RAPL``` environment variable picks how the RAPL counter is read (```emi```, ```pawnio``` or ```hubblo```), instead of trying the three in turn (see [Supported Platforms](../guide/supported_platforms.html)).