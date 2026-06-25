# How Joular Core Works

This page describes the internal architecture of Joular Core: how it reads hardware power data, how it attributes that power to processes and applications, how it produces output, and how the different subsystems fit together.

## High-Level Architecture

At startup, Joular Core:

1. Parses command-line arguments
2. Detects the platform and initialises the appropriate power readers and CPU utilisation trackers
3. Optionally calibrates an idle CPU baseline
4. Enters a monitoring loop that fires once per second
5. On each iteration: reads power, reads CPU utilisation, attributes process/app power, and sends the result to all configured output channels
6. On Ctrl+C: flushes outputs and exits cleanly

```
┌──────────────────────────────────────────────────────────┐
│                   Argument Parsing                       │
└──────────────────────────┬───────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────┐
│                  Platform Setup                          │
│  Detect OS → create CPU/GPU energy readers               │
│             → create CPU utilisation trackers            │
│             → set up ring buffer (if -r)                 │
│             → set up API server (if --api-port)          │
└──────┬──────────────────────┬──────────────────┬─────────┘
       │                      │                  │
       ▼                      ▼                  ▼
  Monitor Loop          Ring Buffer          API Server
  (1 Hz)                Writer               (async)
       │
       ├── CPU energy reader
       ├── GPU energy reader
       ├── CPU utilisation reader
       ├── Process tracker (optional)
       └── App tracker (optional)
            │
            ▼
       MonitorSample
            │
            ├── Terminal output
            ├── CSV file output
            ├── Ring buffer write
            └── API broadcast
```

## Platform Abstraction

The codebase is built around three traits defined in `src/energy.rs`:

- **`CPUEnergy`** — returns the current CPU power in watts via `get_power()`
- **`GPUEnergy`** — returns the current GPU power in watts via `get_power()`
- **`PlatformEnergy`** — a factory that creates the above readers and the CPU utilisation trackers, and provides the process/app power attribution formula

Each supported OS has a concrete implementation of these traits. The monitoring loop talks only to these trait objects, so it is identical on every platform.

### Linux

CPU power is read from the RAPL (Running Average Power Limit) package counter exposed through the powercap sysfs interface at `/sys/class/powercap/intel-rapl/`. The interface exposes cumulative energy counters in microjoules; Joular Core reads two consecutive values and converts the energy delta over elapsed time to watts. If the package counter is absent or unreadable, Joular Core warns once and reports 0 W for CPU power until the interface becomes usable.

CPU utilisation is computed from `/proc/stat`. Joular Core reads the `user`, `nice`, `system`, `idle`, `iowait`, `irq`, `softirq`, and `steal` tick counters on each sample, computes the delta from the previous sample, and calculates utilisation as:

```
utilisation = 1 − (Δidle / Δtotal)
```

Per-process utilisation is computed from `/proc/<pid>/stat`, which exposes cumulative user and kernel CPU time (in clock ticks) for each process. Joular Core computes the delta from the previous sample normalised by the total CPU time delta from `/proc/stat`.

GPU power on Linux is read by:
- **Nvidia**: calling `nvidia-smi --query-gpu=power.draw --format=csv,noheader,nounits` and parsing the output. Power values from all GPUs are summed.
- **AMD**: calling `amd-smi` or `rocm-smi` with JSON output and extracting the power fields.

### Windows

CPU power is read from Hubblo's/Scaphandre's RAPL Windows kernel driver via `DeviceIoControl`. The driver exposes Intel or AMD RAPL MSR energy counters through a Windows device interface; Joular Core reads those counters and converts energy deltas over elapsed time to watts.

CPU utilisation is read from the Win32 API via `GetSystemTimes()`, which returns `idle`, `kernel`, and `user` times as `FILETIME` structures. Utilisation is computed from the deltas between two successive calls.

Per-process CPU time is read via `GetProcessTimes()`. Application monitoring enumerates processes using the Windows Toolhelp32 API (`CreateToolhelp32Snapshot`, `Process32First`, `Process32Next`).

GPU power on Windows follows the same approach as Linux: `nvidia-smi` for Nvidia, `amd-smi` for AMD.

### macOS

CPU power, and GPU power on Apple Silicon, are read from Apple's `powermetrics` tool, which ships with macOS. Joular Core spawns `powermetrics` as a subprocess, forces the C locale, and parses the text sample blocks. On Apple Silicon, `powermetrics` reports CPU and GPU power separately; on Intel Macs it reports CPU package power only.

Because `powermetrics` requires elevated privileges to access hardware counters, Joular Core must be run with elevated access on macOS.

CPU utilisation on macOS is read via the Mach kernel API (`host_statistics64` with `HOST_CPU_LOAD_INFO`), which returns per-CPU user, system, and idle tick counters.

Process and application monitoring on macOS uses Mach task info APIs to read per-process CPU times.

### Single-Board Computers (SBC)

SBC platforms (Raspberry Pi, Asus Tinker Board) do not have a hardware power interface accessible to software. Instead, Joular Core calculates CPU power from CPU utilisation using polynomial regression models:

```
power = c₀ + c₁·u + c₂·u² + … + cₙ·uⁿ
```

where `u` is the current CPU utilisation as a fraction from 0.0 to 1.0 and `c₀…cₙ` are model coefficients measured empirically for each specific board model and revision.

The built-in models cover the supported Raspberry Pi and Asus Tinker Board models when the binary is built with the `sbc` feature. A custom model can be supplied via the `SBC_POWER_MODEL_JSON` environment variable; the file format must match the [Joular Power Models Database](https://github.com/joular/powermodels) schema. Unsupported boards return 0 W.

GPU power is always 0 on SBC platforms.

### Virtual Machines

In a VM, there is no direct hardware interface. Joular Core reads power from a shared file written by the host. The file is read on every sampling cycle. If the file is absent or empty, the power value for that sample is 0. The supported file formats are described in [Virtual Machines](./vm.md).

## The Monitoring Loop

The monitoring loop is in `src/monitor.rs` (`JoularCoreMonitor::poll`). It runs once per second and produces a `MonitorSample`:

```rust
pub struct MonitorSample {
    pub timestamp: u64,       // Unix epoch seconds
    pub cpu_power: f64,       // Watts
    pub gpu_power: f64,       // Watts
    pub total_power: f64,     // cpu_power + gpu_power
    pub cpu_usage: f64,       // Percentage (0–100)
    pub process_power: Option<f64>,       // Watts, if -p is set
    pub app_power: Option<(f64, usize)>,  // (Watts, PID count), if -a is set
}
```

Each field:

- **`timestamp`**: wall-clock time at the moment of reading (`SystemTime::now()`)
- **`cpu_power`** / **`gpu_power`**: raw readings from the hardware interface
- **`total_power`**: sum of CPU and GPU power
- **`cpu_usage`**: system-wide CPU utilisation percentage
- **`process_power`** / **`app_power`**: attributed power, computed as described below

Before the main loop starts, `loop_init()` takes one throwaway reading to warm up the energy counters. RAPL counts cumulative energy since boot; the first real delta needs a prior reference value.

## Process and Application Power Attribution

Joular Core attributes CPU power to a process (or application) using a proportional model:

```
process_power = 100 × (process_cpu_utilisation × attributed_cpu_power) / system_cpu_utilisation
```

where:

- **`process_cpu_utilisation`** is the fraction of total CPU time used by the process (or the sum of all PIDs for an application) in the last second
- **`attributed_cpu_power`** is `max(0, cpu_power − idle_baseline)` — the raw CPU power minus the idle baseline (zero if no baseline is configured)
- **`system_cpu_utilisation`** is the overall CPU utilisation percentage

If `system_cpu_utilisation` is below a tiny threshold, the attributed power is zero to avoid dividing by idle noise.

This model assumes that a process's share of CPU power is proportional to its share of CPU time. This is an approximation — it does not account for frequency scaling within a core, NUMA topology, or work done in kernel threads on behalf of the process — but it is a practical and widely used approach for software-level power attribution.

### CPU Idle Baseline

When the `--cpu-idle-baseline` or `--calibrate-cpu-idle-baseline` option is set, the baseline is subtracted from the raw CPU power before attribution:

```
attributed_cpu_power = max(0, cpu_power − baseline)
```

This removes the base power the CPU consumes just running the operating system at rest, so that the attributed power more accurately reflects the energy consumed by the workload itself.

Auto-calibration (`--calibrate-cpu-idle-baseline`) collects 5 power samples at 1-second intervals before starting the main loop and uses their average as the baseline.

### Application Monitoring and PID Refresh

When monitoring a named application (`-a`), Joular Core maintains a list of all PIDs whose process name matches the supplied string. This list is refreshed periodically (every `--app-refresh-interval` seconds, default 3) to pick up new processes spawned after monitoring started and drop ones that have exited. Setting the interval to 0 rescans on every second.

## Output Pipeline

After each `poll()` call, the `MonitorSample` is sent to an `OutputBundle`, which dispatches it to all active output channels:

1. **Terminal** (`OutputWriter` in `Terminal` mode): formats the reading with ANSI colour codes and writes it to stdout, overwriting the previous line using carriage return and ANSI erase-line sequences. In numeric mode (`-i`), a bare float is printed instead.

2. **CSV file** (`OutputWriter` in `CsvFile` mode): appends one row to the CSV file. In overwrite mode (`-o`), the file is truncated to zero before each write so only the latest row is kept, without a header.

3. **Ring buffer** (`RingBufferWriter`): writes an 8-byte `u64` head counter followed by 5 `RingBufferStruct` slots. Each slot contains a timestamp plus CPU power, GPU power, total power, CPU usage, and PID/app power. On Linux and macOS this is a memory-mapped file (`memmap2`); on Windows it uses Win32 file mapping (`CreateFileMapping` / `MapViewOfFile`).

4. **API** (when the `api` feature is enabled): broadcasts an `ApiData` struct to all connected HTTP and WebSocket clients via a Tokio broadcast channel. The HTTP handler (`GET /data`) reads the latest broadcast value; the WebSocket handler (`/ws`) streams each new broadcast as a JSON message.

## API Server

The API server (`src/api.rs`) is built with the Axum web framework running on a Tokio async runtime. Joular Core starts it on a background thread with its own runtime after successfully binding `127.0.0.1:<PORT>`. Communication from the synchronous monitoring loop to the API server is done via a `tokio::sync::broadcast::Sender<ApiData>`.

- **`GET /data`**: returns the last received value as JSON.
- **`/ws`**: the WebSocket handler subscribes to the broadcast channel and pushes each new value as a JSON message to the connected client.
- CORS is enabled via `tower-http`'s `CorsLayer`. By default only `http://127.0.0.1:<PORT>` and `http://localhost:<PORT>` are allowed. Extra origins come from repeatable `--api-allowed-origin`, and `*` allows any origin.

## Graceful Shutdown

Joular Core installs a Ctrl+C handler via the `ctrlc` crate. When the signal is received:

1. A shared `Arc<AtomicBool>` flag is set to `false`, causing the monitoring loop to exit after the current iteration.
2. The ANSI cursor (hidden at startup) is restored.
3. All buffered output is flushed and file handles are dropped cleanly.

## Code Layout

| Path | Purpose |
|------|---------|
| `src/main.rs` | CLI entry point: argument parsing, monitoring loop, output setup |
| `src/maingui.rs` | GUI binary entry point |
| `src/lib.rs` | Library root: re-exports all public modules |
| `src/args.rs` | Clap-derived argument struct |
| `src/common.rs` | `JoularContext`: platform detection and component initialisation |
| `src/monitor.rs` | `JoularCoreMonitor`: the sampling loop and `MonitorSample` |
| `src/energy.rs` | `CPUEnergy`, `GPUEnergy`, `PlatformEnergy` traits |
| `src/cpu.rs` | `CPUUtilization`, `ProcessCPUUtilization`, `AppCPUUtilization` traits and Linux implementations |
| `src/output.rs` | `OutputWriter`, `OutputBundle`, `OutputSink` traits |
| `src/ringbuffer.rs` | `RingBufferWriter`: shared-memory ring buffer for Linux/macOS/Windows |
| `src/api.rs` | Axum-based HTTP and WebSocket API server |
| `src/logging.rs` | `tracing`-based structured logging helpers |
| `src/platform/linux.rs` | Linux-specific power readers and process trackers |
| `src/platform/windows.rs` | Windows-specific power readers and process trackers |
| `src/platform/macos.rs` | macOS-specific `powermetrics` integration and process trackers |
| `src/platform/sbc.rs` | SBC regression model power calculation |
| `src/platform/nvidia.rs` | Nvidia GPU power via `nvidia-smi` |
| `src/platform/amdgpu.rs` | AMD GPU power via `amd-smi` / `rocm-smi` |
| `src/vm.rs` | Virtual machine shared-file power reader |
| `src/gui/` | egui-based GUI: model, views, history, theme |

## Key Dependencies

| Crate | Version | Purpose |
|-------|---------|---------|
| `clap` | 4.x | Command-line argument parsing |
| `egui` / `eframe` | 0.34 | Cross-platform GUI framework |
| `axum` | 0.8 | HTTP and WebSocket API server |
| `tokio` | 1.x | Async runtime for the API server |
| `tower-http` | 0.6 | CORS middleware |
| `serde` / `serde_json` | 1.x | JSON serialisation for the API |
| `sysinfo` | 0.38 | System and process information (used for process enumeration) |
| `memmap2` | 0.9 | Memory-mapped I/O for the Unix ring buffer |
| `windows` | 0.62 | Win32 API bindings |
| `mach2` | 0.6 | Mach kernel API bindings (macOS) |
| `libc` | 0.2 | C library bindings (macOS) |
| `ctrlc` | 3.x | Cross-platform Ctrl+C handler |
| `tracing` | 0.1 | Structured logging |
| `rfd` | 0.17 | Native file dialog (GUI file picker) |
