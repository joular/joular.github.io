# Supported Platforms

Joular Core runs on all major desktop and server operating systems and a selection of single-board computers. The table below summarises what is supported on each platform and architecture.

## Operating Systems and Architectures

### CPU

| OS / Architecture     | x86_64 | i686 | Apple Silicon | arm | armv7 | aarch64 |
|-----------------------|:------:|:----:|:-------------:|:---:|:-----:|:-------:|
| Linux (PC / servers)  | ✓      | ✓    |               |     |       |         |
| Windows               | ✓      | ✓    |               |     |       |         |
| macOS                 | ✓      |      | ✓             |     |       |         |
| SBC (Raspberry Pi, Asus) |     |      |               | ✓   | ✓     | ✓       |
| Virtual Machines      | ✓      | ✓    | ✓             | ✓   | ✓     | ✓       |

### GPU

| OS / Architecture    | Nvidia | AMD | Apple GPU |
|----------------------|:------:|:---:|:---------:|
| Linux (PC / servers) | ✓      | ✓   |           |
| Windows              | ✓      | ✓   |           |
| macOS                |        |     | ✓         |
| SBC (Raspberry Pi)   |        |     |           |
| Virtual Machines     | ✓      | ✓   | ✓         |

## Platform Details

| Platform | OS | Power source | Architectures |
|---|---|---|---|
| Linux PC / Server | Linux | Intel RAPL (sysfs), Nvidia via `nvidia-smi`, AMD via `amd-smi` / `rocm-smi` | x86, x86_64 |
| Windows PC / Server | Windows | Hubblo's RAPL driver, Nvidia via `nvidia-smi`, AMD via `amd-smi` | x86, x86_64 |
| macOS (Intel) | macOS | `powermetrics` | x86_64 |
| macOS (Apple Silicon) | macOS | `powermetrics` (CPU + GPU) | aarch64 |
| Raspberry Pi | Linux | Regression power models | arm, armv7, aarch64 |
| Asus Tinker Board S | Linux | Regression power models | arm |
| Virtual Machine (guest) | Windows, Linux, macOS | Shared file from host power tool | x86, x86_64, arm, aarch64 |

## Supported Single-Board Computers

The SBC build includes built-in power models for the following devices:

**Raspberry Pi** (all revisions of each model):
- Zero W (32-bit OS)
- 1 B, 1 B+ (32-bit OS)
- 2 B (32-bit OS)
- 3 B, 3 B+ (32-bit OS)
- 4 B (32-bit and 64-bit OS)
- 400 (64-bit OS)
- 5 B (64-bit OS)

**Asus Tinker Board S**

## CPU Power Monitoring Details

**Linux and Windows (x86 / x86_64)**  
CPU power is read from Intel's Running Average Power Limit (RAPL) interface. RAPL is supported on Intel processors since Sandy Bridge (2011) and on AMD processors since Ryzen. On Linux, RAPL is exposed via the powercap sysfs interface (`/sys/class/powercap/intel-rapl/`). On Windows, a kernel driver is required (see [Installation](./installation.md)).

**macOS**  
CPU (and GPU on Apple Silicon) power is read from Apple's `powermetrics` command. This tool ships with macOS and covers both Intel and Apple Silicon hardware. It requires elevated access to read power data.

**Raspberry Pi and SBC**  
Power is calculated from CPU utilization using polynomial regression models that were measured against each supported board at various load levels. No hardware interface or special permissions are needed.

**Virtual Machines**  
Power is read from a shared file written by a monitoring tool running on the host OS. See [Virtual Machines](../ref/vm.md) for the setup details.

## GPU Power Monitoring Details

**Nvidia (Linux and Windows)**  
GPU power is read by calling `nvidia-smi --query-gpu=power.draw`. Power values from all detected GPUs are summed. If `nvidia-smi` is not installed or no Nvidia GPUs are found, the GPU power reading is 0 and monitoring continues normally.

**AMD (Linux and Windows)**  
GPU power is read by calling `amd-smi` or `rocm-smi` (whichever is available). As with Nvidia, if neither tool is available the GPU reading is 0.

**Apple Silicon**  
GPU power is included in the output from `powermetrics` alongside CPU power.

**SBC**  
GPU monitoring is not supported on single-board computers. GPU power is always reported as 0.
