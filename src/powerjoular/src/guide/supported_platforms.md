# Supported Platforms

PowerJoular monitors the following platforms:
- PC/Servers using a RAPL supported Intel processor (since Sandy Bridge) or a RAPL supported AMD processor (since Ryzen, including EPYC), and optionally an Nvidia or an AMD graphic card.
- Macs, with Apple Silicon (M series) or Intel processors.
- Raspberry Pi devices (multiple models) and Asus Tinker Board.
- Inside virtual machines in all supported host platforms.

PowerJoular works on Linux, Windows and macOS.

PowerJoular does the energy and CPU usage measuring through two Ada libraries we developed:
- [Joular Core](https://github.com/joular/joularcore): for CPU and GPU energy and power consumption.
- [CPU Load](https://github.com/joular/cpuload): for CPU usage of the whole system, of a specific PID, and of a specific application (all its PIDs).

On Linux PC/Servers, PowerJoular uses powercap Linux interface to read RAPL (Running Average Power Limit) energy consumption, for Intel and AMD processors.

PowerJoular reads the RAPL package domain (cores, integrated graphics, memory controller and last level caches).
On a server with several sockets, only the first one is measured.

On Windows PC/Servers, PowerJoular reads the same RAPL package counter in one of three ways, and keeps the first one that answers:
- The [Energy Meter Interface](https://learn.microsoft.com/en-us/windows-hardware/drivers/powermeter/energy-meter-interface) (EMI), built into Windows 11. Nothing to install.
- The RAPL registers through the [PawnIO](https://pawnio.eu) driver.
- The RAPL registers through [Hubblo's RAPL driver](https://github.com/hubblo-org/windows-rapl-driver).

Setting the ```JOULARCORE_WINDOWS_RAPL``` environment variable to ```emi```, ```pawnio``` or ```hubblo``` picks one instead of trying them in turn. Nothing has to be configured otherwise.

On macOS, PowerJoular uses ```powermetrics```, which is installed with macOS and reports the power drawn over each cycle.
Apple Silicon Macs give their CPU and the GPU built into the same chip.
Intel Macs give the CPU only (the whole chip: cores, integrated graphics and system agent).

For the GPU, PowerJoular supports:
- Nvidia graphic cards on Linux and Windows, through NVML, which is installed with the Nvidia driver.
- AMD graphic cards on Linux, through the hwmon sysfs of the amdgpu driver.
- AMD graphic cards on Windows, through ADLX, which is installed with the AMD driver.
- The GPU of Apple Silicon Macs, through ```powermetrics```.

On virtual machines, PowerJoular requires two steps:
- Installing PowerJoular itself or another power monitoring tool in the host machine.
Then monitoring the virtual machine power consumption every second and writing it to a file (to be shared with the guest VM).
- Installing PowerJoular in the guest VM, then running PowerJoular while specifying the path of the power file shared with the host and its format.

On Raspberry Pi and Asus Tinker Board, PowerJoular uses its own research-based empirical regression models to estimate the power consumption of the ARM processor.

The supported list of Raspberry Pi and Asus Tinker Board models are listed below.
We support all revisions of each model lineup.
However, the model is generated and trained on a specific revision (listed between brackets), and the accuracy is best on this particular revision.

We currently support the following Raspberry Pi models and Asus Tinker Board models:
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
|        Raspberry Pi       |        Linux       |             Our regression models             |             ARM            |
|        Asus Tinker Board       |        Linux       |             Our regression models             |             ARM            |
|     Linux PC/Server    |        Linux        |             RAPL (using powercap), Nvidia NVML, AMD hwmon sysfs            |             x86, x86_64            |
|     Windows PC/Server    |        Windows        |             RAPL (using EMI, PawnIO or Hubblo's driver), Nvidia NVML, AMD ADLX            |             x86_64            |
|     Mac    |        macOS        |             powermetrics            |             Apple Silicon (ARM), Intel (x86_64)            |
|     Virtual Machine    |        Linux, Windows or macOS guest, any host       |             Host's architecture (RAPL, regression models, others)            |             x86, x86_64, ARM            |

## Required privileges

Without the privileges below, and with no graphic card to read, PowerJoular finds no power source and stops. With a graphic card, it carries on and the CPU power reads 0 (```-d``` shows what can be measured).

- **Linux, PC or server**: reading RAPL files needs elevated privileges on the recent kernels (5.10 and newer), so run ```sudo powerjoular```, or give read rights to the files. See [this issue](https://github.com/joular/powerjoular/issues/1).
- **Windows**: with the Energy Meter Interface (EMI), no special privileges or driver are needed. Otherwise, a RAPL driver is needed: [PawnIO](https://pawnio.eu), which needs PowerJoular to run from a terminal with administrative rights, or [Hubblo's RAPL driver](https://github.com/hubblo-org/windows-rapl-driver), which does not. The easiest way to get a signed version of Hubblo's driver is through the [Scaphandre installer](https://github.com/hubblo-org/scaphandre/releases). Reading the CPU time of a process belonging to another user also needs a terminal with administrative rights, so ```-p``` and ```-a``` on someone else's process need it too.
- **macOS**: ```powermetrics``` only runs as the superuser, so run ```sudo powerjoular```. Without it, PowerJoular finds no power source and stops. Reading the CPU time of a process belonging to another user also needs root, so ```-p``` and ```-a``` on someone else's process need ```sudo``` too.
- **Raspberry Pi, and GPU readings on Linux and Windows**: no special privileges needed.