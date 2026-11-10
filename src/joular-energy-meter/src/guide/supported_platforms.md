# Supported Platforms

Joular Energy Meter runs on Linux, Windows, macOS and FreeBSD, and reads:

| OS | CPU | GPU |
|---|---|---|
| Linux | Intel, AMD, Raspberry Pi | Nvidia, AMD |
| Windows | Intel, AMD | Nvidia, AMD |
| macOS | Apple Silicon, Intel | Apple Silicon |
| FreeBSD | Intel, AMD | Nvidia |

The hardware is read by our [Joular Core](https://github.com/joular/joularcore) library, the same way as in [PowerJoular](https://joular.github.io/powerjoular/guide/supported_platforms.html):
- Intel and AMD processors through RAPL: the powercap files on Linux, the Energy Meter Interface (Windows 11), PawnIO or Hubblo's driver on Windows, and the ```cpuctl``` driver on FreeBSD.
- Macs through ```powermetrics```, installed with macOS. Apple Silicon gives the CPU and the GPU of the chip, Intel Macs the CPU only.
- Raspberry Pi through our regression power models.
- Nvidia cards through NVML (installed with the driver), and AMD cards through the amdgpu hwmon files on Linux and ADLX on Windows.

The CPU usage of the machine, of a process and of an application is read by our [CPU Load](https://github.com/joular/cpuload) library.

*Whole machine* means the processor and the graphics card only.
The screen, memory, disks and the losses of the power supply are not measured, so the computer draws more at the plug.
A power meter at the plug can be added for that (see [Idle Power and Plug Meter](../ref/idle_plug.html)).

## Required privileges

The CPU load is always read, but the CPU power needs some rights on most systems.
When they are missing, a notice says what to do: on Linux, macOS and Windows, with a button that asks the system for the administrator (through the system's own dialog, never a password field of ours), and on Linux, macOS and FreeBSD, with a button to copy the command instead.

- **Linux**: the RAPL files are readable by root only (CVE-2020-8694). **Allow reading power** asks for the administrator through polkit (```pkexec```), and makes the processor's counters readable by every user of the machine until the next boot. The application itself keeps running as you. The same by hand, after each boot: ```sudo chmod -R a+r /sys/class/powercap/intel-rapl```. Or start the application with ```sudo```, under X11 (not Wayland).
- **Windows**: nothing to do on Windows 11. On older versions, install the [PawnIO](https://pawnio.eu) driver, then **Restart as administrator** (UAC asks), or install [Hubblo's RAPL driver](https://github.com/hubblo-org/windows-rapl-driver), which needs no administrator.
- **macOS**: ```powermetrics``` needs root. **Restart with admin rights** asks for your password in macOS's dialog, then starts the application again as root, with your files and settings still yours. Or from Terminal: ```sudo "/Applications/Joular Energy Meter.app/Contents/MacOS/joularenergymeter"```.
- **FreeBSD**: load the cpuctl driver (```sudo kldload cpuctl```, or ```cpuctl_load="YES"``` in ```/boot/loader.conf```), and start the application with ```sudo``` under X11. The ```kmem``` group also works, but it can read all memory through ```/dev/mem```.
- Nvidia and AMD graphics cards, and Raspberry Pi, need nothing.

On Windows and macOS, following a process of another user also needs the administrator.
