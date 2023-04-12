# How PowerJoular Works

PowerJoular monitors the CPU and GPU power consumption in computers and servers. It is written in Ada in order to provide a low-impact tool as Ada is constantly ranked among the most energy efficient programming languages, while also improving code maintainability and safety.

PowerJoular is aimed to software developers, system administrators and to automated tools, with a goal to help these users understand the power consumption of their devices and software, and to build more in-depth tools using our proposed platform.

PowerJoular's power monitoring is based on the Intel RAPL through the [Linux Power Capping Framework](https://www.kernel.org/doc/html/latest/power/powercap/powercap.html), and, optionally [NVIDIA's System Management Interface](https://developer.nvidia.com/nvidia-system-management-interface).
PowerJoular automatically detects the computer configuration and supported modules, and provides power data accordingly.

On Raspberry Pi and Asus Tinker Board (and similar devices), PowerJoular uses its own research-based empirical regression models to estimate the power consumption of the ARM processor.

## CPU Power Monitoring

For the CPU, PowerJoular uses the Intel RAPL power data through the Linux powercap interface by reading the appropriate system files (or our regression models on Raspberry Pi and similar devices).

It first detects which power domains are supported by the CPU:
- Pkg: which is supported since Intel Sandy Bridge CPUs, and provides energy consumption for the CPU cores, integrated graphics, memory controller and last level caches.
PowerJoular also checks if DRAM power domain is supported (RAM attached to the memory controller) and adds its power readings to the total.
- Psys: which is supported since Intel Skylake CPUs, and provides energy consumption for the entire SOC (including Pkg along with other components, such as eDRAM, PCH, System Agent). If Psys is supported, it will be exclusively used by PowerJoular for CPU power consumption instead of Pkg and DRAM, as it provides a more comprehensive power reading of the CPU SOC.

On Raspberry Pi and similar devices, PowerJoular uses regression models we developed (which provides high accuracy, and publish in [this repository](https://github.com/joular/powermodels)).
The model uses CPU utilization (calculated from CPU cycles statistics from ```/proc/stat``` and ```/proc/pid/stat``` files).

## GPU Power Monitoring

For the GPU, PowerJoular checks if NVIDIA SMI is installed, then uses it to verify if GPU power monitoring is supported to the specific graphic card, and to read GPU power consumption every second.

Finally, PowerJoular aggregates power readings from all supported components to provide an overall power consumption.
For instance, if both Intel RAPL and NVIDIA SMI are supported, the tool will provide an aggregated power value for both CPU and GPU.

## Monitoring a PID

For monitoring a specific process through its ID (PID), PowerJoular uses the Linux statistics in ```/proc/stat``` and ```/proc/pid/stat```, and calculates the proportion of CPU cycles used by the process, and thus calculates the power consumed by the process accordingly.

## Monitoring an application by its name

PowerJoular monitors an application by searching for all its PIDs, on every monitoring cycle (by default, every second) because an application can create or destroy process during its runtime.
Then, PowerJoular applies the same approach to monitor a PID, but by monitoring all the applications PIDs and calculating the sum of the power of these PIDs.

## Systemd service

PowerJoular also provides a systemd service which writes power consumption to /tmp folder, allowing it to bypass the added restrictions on reading powercap energy meters to [non-privileged users in Intel CPUs](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=949dd0104c496fa7c14991a23c03c62e44637e71).
The restriction was added due to the recently discovered [PLATYPUS vulnerability](https://platypusattack.com/).
However, the systemd service still requires privileged access (root/sudo) to be enabled, and the data provided is the runtime power consumption (every second) from aggregate sources (CPU and GPU when available).