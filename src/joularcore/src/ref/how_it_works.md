# How Joular Core Works

This page describes the internal architecture of Joular Core: how it detects the hardware, how it reads each source on each OS, and how to add new hardware or a new OS.

## High-Level Architecture

The library has two sources, the CPU and the GPU, each measured by its own monitor:

1. `Open` asks each requested monitor to detect its hardware. The monitor tries, in order, every way it knows to read that hardware on this OS, and keeps the first one that answers. It also takes a first reading of cumulative counters (e.g. RAPL), so the next reading can report the energy consumed since then.
2. `Read` asks each monitor that could be opened for one measurement: the energy consumed since the previous reading, or the power drawn, depending on what the hardware reports.
3. `Close` closes what each monitor opened (e.g. a driver on Windows, the `powermetrics` process on macOS).

```
            Open / Read / Close
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   CPU_Monitor             GPU_Monitor
   (one body per OS)       (one body per OS)
        │                       │
        ├── RAPL (Linux)        ├── NVML (Nvidia)
        ├── Board models (SBC)  ├── hwmon sysfs (AMD, Linux)
        ├── RAPL (Windows)      ├── ADLX (AMD, Windows)
        ├── RAPL (FreeBSD)      └── powermetrics
        └── powermetrics
```

Each source is opened, read and closed on its own: a source that fails while being opened is closed again and reported as not available, and a source that fails while being read reports zero. Neither stops the other source.

## Linux

**CPU.** The CPU monitor tries RAPL first, then the power models of single-board computers.

RAPL is read from the powercap sysfs interface at `/sys/class/powercap/intel-rapl:N`. Joular Core looks for the first domain whose name begins with `package` (it is usually `intel-rapl:0`, but on some machines another domain such as `psys` comes first), and reads its `energy_uj` counter, in microjoules. The `max_energy_range_uj` file gives where the counter wraps. The counter reads zero without root on most systems, and the CPU is then reported as not available.

On Raspberry Pi and Asus Tinker Board, the board is detected from `/proc/device-tree/model`. The power is calculated from CPU utilization using polynomial regression models:

```
power = c₀ + c₁·u + c₂·u² + … + c₉·u⁹
```

where `u` is the CPU utilization from 0.0 to 1.0, measured from `/proc/stat` over the interval between two readings, and `c₀…c₉` are the model coefficients measured for each board (models fitted at a lower degree have their remaining coefficients at zero). Some boards have separate models for 32-bit and 64-bit systems. The models come from our [Joular Power Models Database](https://github.com/joular/powermodels).

**GPU.** The GPU monitor tries Nvidia cards through NVML first, then AMD cards through the hwmon sysfs of the amdgpu kernel driver. For AMD, Joular Core looks in `/sys/class/hwmon` for the first amdgpu sensor with a readable power file, and prefers `power1_average` (an average over a short period) to `power1_input` (an instant reading).

## Windows

**CPU.** The same RAPL package counter of the first socket can be reached in three ways, tried in this order, and the first one that answers is kept:

1. The Energy Meter Interface (EMI), built into Windows 11. The processor driver publishes the RAPL domains as channels of a meter, and Joular Core opens the meter publishing the package channel. A meter publishing something else (a board rail, a battery) is refused. Windows hands over a 64-bit counter already unwrapped, so there is no wrap to correct.
2. The MSR registers through the PawnIO driver. The driver reads no register on its own: Joular Core loads into it a signed module saying which registers it lets through (one for Intel, one for AMD from family 17h onwards; older AMD processors are refused, and are left to Hubblo's driver).
3. The MSR registers through Hubblo's RAPL driver, which only lets the RAPL registers through, so opening the device is enough.

When reading the registers directly, Joular Core asks the processor for its vendor with the CPUID instruction, to know which registers hold the energy unit and the package counter on Intel and AMD. Silvermont and Airmont Atoms, whose energy unit would be read wrongly, are refused.

**GPU.** The GPU monitor tries Nvidia cards through NVML first, then AMD cards through ADLX. ADLX gives the whole board power where the card offers it, and the graphics processor alone where it does not. Which of the two is used is chosen when the card is opened, and does not change afterwards.

## macOS

The CPU, and the GPU on Apple Silicon, are read from Apple's `powermetrics` tool (`/usr/bin/powermetrics`, given by its full path so that `PATH` cannot point at another program). One `powermetrics` process serves both sources: it is started by the first `Open`, and stopped by the last `Close`.

Each reading asks `powermetrics` for a sample (with the `SIGINFO` signal), so a reading is the average power since the previous one. `powermetrics` still takes a sample of its own every ten minutes (this is what makes it stop once the program using the library is gone), so readings should be closer than that. The CPU and the GPU are read one after the other, and readings within 0.1 seconds share one sample, so both values come from the same sample.

On Apple Silicon, `powermetrics` reports the CPU and the GPU each on a line. On Intel Macs, it reports the whole chip on one line, and no GPU.

`powermetrics` only runs as root, so Joular Core checks that before starting it. If it stops answering, its sources read zero, and it is started again on a reading about ten seconds later.

## FreeBSD

**CPU.** The RAPL package counter of the first processor is read from its registers, through the `cpuctl(4)` driver (`/dev/cpuctl0`). As on Windows, Joular Core asks the processor for its vendor with the CPUID instruction, to know which registers hold the energy unit and the package counter on Intel and AMD, and refuses Silvermont and Airmont Atoms.

**GPU.** The GPU monitor reads Nvidia cards through NVML.

## Nvidia GPUs on Linux, Windows and FreeBSD: NVML

NVML is the library installed with the Nvidia driver (`libnvidia-ml.so.1` on Linux and FreeBSD, `nvml.dll` on Windows, in the system folder, or in the Nvidia folder of Program Files for older drivers). Joular Core loads it when the GPU is opened, rather than linking to it, so the library works the same on a machine with no Nvidia driver. It reads the power of the first card listed. The same applies to ADLX on Windows (`amdadlx64.dll`, or `amdadlx32.dll` for a 32 bits build).

## Energy Counters

The RAPL counters of Linux, of Windows when read through a driver, and of FreeBSD, only count up, and wrap back to zero once full. Joular Core keeps the previous value of each counter, and turns two readings into the energy consumed between them:

- When the new value is lower, the counter wrapped, so the range it wraps at is added.
- A drop that does not look like a wrap (one that would mean more than half the range was used between two readings) is taken as a counter reset, for example after the machine was suspended, and gives zero.
- A counter that could not be read gives zero, and keeps the previous value, so the next reading that works covers the gap.

Only one wrap can be corrected between two readings, so read more often than the counter takes to fill half its range. Under load, this is a few minutes, hence the advice to read at least once per minute.

## Code Layout

| Path | Purpose |
|------|---------|
| `src/joular_core.ads` | The Ada interface: `Open`, `Read`, `Close`, `Version` |
| `src/joular_core-c_api.ads` | The C interface, matching `include/joularcore.h` |
| `src/joular_core-cpu_monitor.ads`, `src/joular_core-gpu_monitor.ads` | The two monitors, with one body per OS |
| `src/joular_core-energy_counters.adb` | The energy between two readings of a counter that wraps (RAPL on Linux, Windows and FreeBSD) |
| `src/joular_core-gpu_nvidia_nvml.adb` | Nvidia GPUs through NVML, on Linux, Windows and FreeBSD |
| `src/joular_core-processor.adb` | The CPUID vendor check, to read the RAPL registers on Windows and FreeBSD |
| `src/linux/` | RAPL through powercap, the power models of single-board computers, AMD GPUs through hwmon |
| `src/windows/` | RAPL through EMI, PawnIO and Hubblo's driver, AMD GPUs through ADLX, loading a shared library and finding NVML, and the Win32 bindings they share |
| `src/macos/` | `powermetrics`, and the specs file recording where the shared library finds the Ada runtime |
| `src/freebsd/` | RAPL through `cpuctl` |
| `src/posix/` | What Linux, macOS and FreeBSD do the same way (loading a shared library) |
| `include/joularcore.h` | The C declarations |
| `example/` | Example programs in Ada, C and Python |
| `tools/` | The PawnIO modules, and the script turning them into an Ada package |

## Adding New Hardware or a New OS

Each hardware component is one package with three functions: `Open` (detect and open, returning `False` when the hardware is not there or cannot be read), `Get_Power` or `Get_Energy` (one reading, in watts or joules), and `Close`.

The code shared by every OS is in `src`, and the code of each OS is in its own folder, picked by `joularcore.gpr` from `PJ_OS`. Each OS folder has its own body of the two monitors, `CPU_Monitor` and `GPU_Monitor`, which lists the packages that can read that hardware on that OS, tries them in order and keeps the first one that answers.

- To support new hardware, write such a package in the folder of the OS it runs on (or in `src` if it is portable, like NVML), and add it to the monitor of that OS.
- To support a new OS, add its folder with its two monitors, and add it to `PJ_OS` in the project file.

Windows RAPL splits this further, as the same counter is reached in several ways. `RAPL_Windows` tries each way in order and keeps the first that answers: `RAPL_EMI_Windows`, which reads RAPL from the Energy Meter Interface, then `RAPL_MSR_Windows`, which reads the MSR registers with a driver. That second one keeps the vendor detection and the counter, and hands the reading of a single register to one of two interchangeable packages, `MSR_PawnIO` and `MSR_Hubblo`. Supporting another driver means writing another such package with `Open`, `Read` and `Close`, adding it to `Driver_Kind` in `RAPL_MSR_Windows`, and to the list tried in `RAPL_Windows.Open`.

## Third Party Components

Joular Core carries two [PawnIO modules](https://github.com/namazso/PawnIO.Modules), `IntelMSR.bin` and `AMDFamily17.bin`, taken byte for byte from release 0.2.11. The PawnIO driver checks their signature before running them, so they are shipped as they are and cannot be rebuilt here.

They are licensed under the GNU Lesser General Public License version 2.1 or later, copyright namazso and contributors. A copy of that license is in `tools/pawnio/COPYING`, next to the modules themselves.

They are turned into `src/windows/joular_core-pawnio_modules.ads` by `tools/gen_pawnio_modules.py`, which is also how that file is regenerated when a newer release is taken:

```bash
python3 tools/gen_pawnio_modules.py
```

The SHA-256 of each module is pinned in that script, which refuses to write anything when a module on disk is not the one it expects. Running it with `--check` writes nothing and only reports whether the modules, their digests and the committed package still agree.