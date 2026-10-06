# Supported Platforms

Joular Core runs on Linux, Windows, macOS and FreeBSD, on PCs, servers, Macs and single-board computers. The tables below summarise what is supported on each platform and architecture.

## Operating Systems and Architectures

### CPU

| OS / Architecture     | x86_64 | x86 | Apple Silicon | arm | aarch64 |
|-----------------------|:------:|:---:|:-------------:|:---:|:-------:|
| Linux (PC / servers)  | ✓      | ✓   |               |     |         |
| Windows               | ✓      |     |               |     |         |
| macOS                 | ✓      |     | ✓             |     |         |
| FreeBSD               | ✓      |     |               |     |         |
| SBC (Raspberry Pi, Asus Tinker Board) | |  |            | ✓   | ✓       |

### GPU

| OS / Architecture    | Nvidia | AMD | Apple GPU |
|----------------------|:------:|:---:|:---------:|
| Linux (PC / servers) | ✓      | ✓   |           |
| Windows              | ✓      | ✓   |           |
| macOS                |        |     | ✓         |
| FreeBSD              | ✓      |     |           |
| SBC (Raspberry Pi)   |        |     |           |

## Platform Details

| Platform | OS | Power source | Reports |
|---|---|---|---|
| Linux PC / Server | Linux | RAPL through powercap sysfs, Nvidia through NVML, AMD through amdgpu hwmon sysfs | Energy (CPU), power (GPU) |
| Windows PC / Server | Windows | RAPL through the Energy Meter Interface, PawnIO or Hubblo's driver, Nvidia through NVML, AMD through ADLX | Energy (CPU), power (GPU) |
| macOS (Apple Silicon) | macOS | `powermetrics` (CPU and GPU) | Power |
| macOS (Intel) | macOS | `powermetrics` (CPU only) | Power |
| Raspberry Pi | Linux | Regression power models | Power |
| Asus Tinker Board (S) | Linux | Regression power models | Power |
| FreeBSD PC / Server | FreeBSD | RAPL through the `cpuctl` driver, Nvidia through NVML | Energy (CPU), power (GPU) |

Energy is the joules consumed since the previous reading, and power is the watts being drawn. See [Reading the Measurements](../ref/measurements.md) for the details.

## Supported Single-Board Computers

Joular Core includes power models for the following devices. Every revision of each model is supported, though the power model was trained on one particular revision (two for the 4 B, rev 1.1 and rev 1.2), on which the accuracy is at its best.

**Raspberry Pi**:
- Zero W, 1 B, 1 B+, 2 B, 3 B, 3 B+ (model measured on a 32-bit OS, also used on a 64-bit one)
- 4 B (32-bit and 64-bit OS)
- 400, 5 B (64-bit OS only)

**Asus Tinker Board (S)**

## CPU Power Monitoring Details

**Linux (x86 / x86_64)**  
CPU energy is read from the RAPL package counter exposed through the powercap sysfs interface (`/sys/class/powercap/intel-rapl:N`), for Intel and AMD processors. Joular Core reads one package, the first one whose name begins with `package`, so on a server with several sockets only the first socket is measured. On recent kernels, the counter is only readable by root (see [Installation](./installation.md)). If it cannot be read, the CPU is reported as not available.

**Windows**  
CPU energy is read from the same RAPL package counter, through one of three approaches, tried in this order:
- The [Energy Meter Interface](https://learn.microsoft.com/en-us/windows-hardware/drivers/powermeter/energy-meter-interface) (EMI), built into Windows 11. Nothing to install, and no administrative rights needed.
- The RAPL registers through the [PawnIO](https://pawnio.eu) driver, which needs administrative rights.
- The RAPL registers through [Hubblo's RAPL driver](https://github.com/hubblo-org/windows-rapl-driver), which does not.

The first one that answers is kept. Setting the `JOULARCORE_WINDOWS_RAPL` environment variable to `emi`, `pawnio` or `hubblo` picks one instead (see [Library Interface](../ref/interface.md)).

**macOS**  
CPU power, and GPU power on Apple Silicon, are read from Apple's `powermetrics` tool, which ships with macOS. It covers both Apple Silicon and Intel Macs, and only runs as root. On Intel Macs, the CPU power is the whole chip (cores, integrated graphics and system agent), and there is no GPU power.

**Raspberry Pi and SBC**  
Power is calculated from CPU utilization using polynomial regression models that were measured against each supported board at various load levels. The board is detected from `/proc/device-tree/model`, and the CPU utilization is read from `/proc/stat`. No special permissions are needed. On a board with no power model, the CPU is reported as not available.

**FreeBSD (x86_64)**  
CPU energy is read from the same RAPL package counter, from the registers of the first processor, through the [cpuctl(4)](https://man.freebsd.org/cgi/man.cgi?query=cpuctl&sektion=4) driver (`/dev/cpuctl0`), for Intel and AMD processors. The driver is a module to load first, and its device is only readable by root and the `kmem` group (see [Installation](./installation.md)). If it cannot be read, the CPU is reported as not available.

**Virtual Machines**  
Inside a virtual machine, the hardware counters are usually not reachable, so the CPU is reported as not available. [PowerJoular](https://github.com/joular/powerjoular) can read the power of a virtual machine from a file the host writes.

## GPU Power Monitoring Details

**Nvidia (Linux, Windows and FreeBSD)**  
GPU power is read through NVML, the library installed with the Nvidia driver, and loaded by Joular Core when the GPU is opened. The first card listed is the one read. If the driver is not installed or that card does not report its power, the GPU is reported as not available. On Linux and Windows, Nvidia is tried first, then AMD, and only one GPU is read: on a machine with both, the Nvidia card is the one measured.

**AMD (Linux)**  
GPU power is read from the hwmon sysfs of the amdgpu kernel driver (`/sys/class/hwmon`), with nothing to install. The first amdgpu sensor with a readable power file is the one read.

**AMD (Windows)**  
GPU power is read through ADLX, the library installed with the AMD driver. The first card listed is the one read.

**Apple Silicon**  
GPU power comes from the same `powermetrics` sample as the CPU power.

**SBC**  
GPU monitoring is not supported on single-board computers. The GPU is reported as not available.