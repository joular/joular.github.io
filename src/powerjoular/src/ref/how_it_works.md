# How PowerJoular Works

PowerJoular monitors the CPU and GPU power consumption in computers and servers. It is written in Ada in order to provide a low-impact tool as Ada is constantly ranked among the most energy efficient programming languages, while also improving code maintainability and safety.

PowerJoular is aimed to software developers, system administrators and to automated tools, with a goal to help these users understand the power consumption of their devices and software, and to build more in-depth tools using our proposed platform.

Since version 2, PowerJoular measures the hardware through our [Joular Core](https://github.com/joular/joularcore) library, and the CPU usage through our [CPU Load](https://github.com/joular/cpuload) library.
Both are written in Ada too, and PowerJoular is compiled with them into one binary.

PowerJoular's power monitoring is based on the Intel RAPL through the [Linux Power Capping Framework](https://www.kernel.org/doc/html/latest/power/powercap/powercap.html) on Linux, and through the Energy Meter Interface, PawnIO or Hubblo's driver on Windows.
On macOS, it is based on Apple's ```powermetrics```.
For the GPU, it optionally uses [NVIDIA's Management Library (NVML)](https://developer.nvidia.com/management-library-nvml), and AMD's hwmon sysfs (Linux) or ADLX (Windows).
PowerJoular automatically detects the computer configuration and supported modules, and provides power data accordingly.

On Raspberry Pi and Asus Tinker Board (and similar devices), PowerJoular uses its own research-based empirical regression models to estimate the power consumption of the ARM processor.

## CPU Power Monitoring

For the CPU, PowerJoular uses the Intel RAPL power data through the Linux powercap interface by reading the appropriate system files (or our regression models on Raspberry Pi and similar devices).

It reads the Pkg domain, which is supported since Intel Sandy Bridge CPUs (and on AMD Ryzen and EPYC), and provides energy consumption for the CPU cores, integrated graphics, memory controller and last level caches.
On a server with several sockets, only the first one is read.

On Windows, the same Pkg counter is read through the Energy Meter Interface built into Windows 11, or through the PawnIO driver or Hubblo's driver, whichever answers first.

RAPL gives an energy counter that only counts up, and Joular Core turns it into the energy consumed since the last reading.
PowerJoular divides it by how long the cycle actually took, rather than assuming it was exactly one second, so the watts reported are the watts drawn, even on a busy machine where a cycle takes longer than one second.
RAPL counters wrap when they fill, and Joular Core corrects that, as long as the counter is read before it fills half of its range, which is the case with a reading every second.

On macOS, ```powermetrics``` is asked for a sample on every cycle, so each value is the average power since the previous cycle.
On Apple Silicon, it gives the CPU and the GPU of the chip from the same sample. On Intel Macs, it gives the whole chip (cores, integrated graphics and system agent), and no GPU.

On Raspberry Pi and similar devices, PowerJoular uses regression models we developed (which provides high accuracy, and publish in [this repository](https://github.com/joular/powermodels)).
The model uses CPU utilization (calculated from CPU cycles statistics from the ```/proc/stat``` file), and the board is detected from ```/proc/device-tree/model```.

## GPU Power Monitoring

For the GPU, PowerJoular checks if the Nvidia driver is installed, then uses its NVML library to verify if GPU power monitoring is supported to the specific graphic card, and to read GPU power consumption every second.
When there is no Nvidia card, it looks for an AMD card: through the hwmon sysfs of the amdgpu driver on Linux, and through ADLX, which comes with the AMD driver, on Windows.
On Apple Silicon Macs, the GPU power comes from ```powermetrics```, with the CPU power.
Only the first graphic card found is read.

Finally, PowerJoular aggregates power readings from all supported components to provide an overall power consumption.
For instance, if both Intel RAPL and NVML are supported, the tool will provide an aggregated power value for both CPU and GPU.

## Monitoring a PID

For monitoring a specific process through its ID (PID), PowerJoular reads the CPU time used by the process and by the whole system (```/proc/stat``` and ```/proc/pid/stat``` on Linux, ```proc_pidinfo``` and ```host_statistics``` on macOS, ```GetProcessTimes``` and ```GetSystemTimes``` on Windows), and calculates the proportion of CPU cycles used by the process, and thus calculates the power consumed by the process accordingly:

```
process power = CPU power × process CPU usage / system CPU usage
```

The process power is never more than the CPU power, and the GPU power is not shared out to processes.
When the process cannot be read (it has stopped, or the system does not allow it), its usage and power are written as ```-1``` in the CSV file and the ring buffer (and shown as ```n/a``` on the terminal), and not as ```0```.

## Monitoring an application by its name

PowerJoular monitors an application by searching for all its PIDs, on every monitoring cycle (every second) because an application can create or destroy process during its runtime.
Then, PowerJoular applies the same approach to monitor a PID, but by summing the CPU time of all the application's PIDs and calculating the power from that sum.
An application with no running process is reported as ```0```, and not as ```-1```.

A process is part of the application when the program it runs has that exact name (without its folder, in any case), so ```firefox``` finds every process of Firefox, its content processes included.
On macOS, the name is the one of the program inside the bundle, and every program inside ```Firefox.app``` is found too, as Firefox runs its content processes (```plugin-container```) from a helper bundle inside it. On Windows a trailing ```.exe``` is ignored.

## Systemd service

PowerJoular also provides a systemd service which writes power consumption to the ```/run/powerjoular``` folder, allowing it to bypass the added restrictions on reading powercap energy meters to [non-privileged users in Intel CPUs](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=949dd0104c496fa7c14991a23c03c62e44637e71).
The restriction was added due to the recently discovered [PLATYPUS vulnerability](https://platypusattack.com/).
However, the systemd service still requires privileged access (root/sudo) to be enabled, and the data provided is the runtime power consumption (every second) from aggregate sources (CPU and GPU when available).