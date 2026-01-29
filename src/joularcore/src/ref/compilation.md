# Compilation

Joular Core is written in Rust and uses Cargo for compilation.

## Default build:

```bash
cargo build --release
```

This will build Joular Core with the virtual machine and GUI features, but not with SBC support.

## SBC (Raspberry Pi) build:

```bash
cargo build --release --features sbc
```

This will build Joular Core with the virtual machine and GUI features, and with SBC support.

## Feature selection:

You can build Joular Core with or without virtual machine support or the GUI.
For instane, this is useful when you don't need the GUI (*i.e.*, in server environments) and want to build a smaller binary.

```bash
cargo build --release --no-default-features
```

⚙️ **Available Features:**

- `vm` *(default: ON)*: Enables support for monitoring inside virtual machines.
- `api` *(default: OFF)*: Enables support for the API with an HTTP server or WebSockets. Exporting power data to CSV files or to a ring buffer is enabled by default regardless of this feature.
- `gui` *(default: ON)*: Compiles the application with the graphical user interface.
- `sbc` *(default: OFF)*: Enables support for monitoring on single-board computers (*e.g.*, Raspberry Pi) running Linux.  
  - This disables RAPL-based monitoring and uses our SBC-specific power models.

You can mix and match features, for example:
```bash
# No GUI, but with VM
cargo build --release --no-default-features --features vm

# No GUI, no VM, but with SBC
cargo build --release --no-default-features --features sbc

# No VM, but with GUI and SBC
cargo build --release --no-default-features --features gui,sbc
```

## Cross-Compilation

To cross-compile, use `cargo-make` with one of the targets defined in `Makefile.toml`.

For example, to build for all supported Raspberry Pi architectures (`aarch64`, `arm`, `armv7`):

```bash
cargo make build-sbc
```

With the proper Rust targets installed, you can hence cross-compile on all architectures and operating systems.

## Dependencies
- **Linux (PC, servers)**: Requires the RAPL interface for CPU power readings. RAPL files are usually readable only by the administrator, so you must either grant read access to these files or run Joular Core with sudo. [More information in this issue](https://github.com/joular/joularcore/issues/12).
- **Windows**: a RAPL driver. We currently support [Hubblo's RAPL driver](https://github.com/hubblo-org/windows-rapl-driver). The easiest way to install a signed version of the driver is to install the [Scaphandre](https://github.com/hubblo-org/scaphandre) tool from [this installer](https://github.com/hubblo-org/scaphandre/releases/download/v1.0.0/scaphandre_v1.0.0_installer.exe).
- **macOS**: no dependencies required 😇 Uses powermetrics (already installed by default).
- Raspberry Pi: No dependencies required 😇

## Virtual Machines

For virtual machines, two environment variables must be set depending on whether you want to monitor the CPU, the GPU, or both:
- `VM_CPU_POWER_FILE`: Path to the CPU power data file
- `VM_CPU_POWER_FORMAT`: Format of the CPU power data (`joularcore`  or `powerjoular` or `watts`, default is `watts` if not set)
- `VM_GPU_POWER_FILE`: Path to the GPU power data file
- `VM_GPU_POWER_FORMAT`: Format of the GPU power data (`joularcore` or `powerjoular` or `watts`, default is `watts` if not set)

## Single-Board Computers (SBC)

On SBC (Raspberry Pi, Asus TinkerBoard), Joular Core uses our own regression models to calculate CPU power consumption.
These models are hard-coded in the code, but you can use your own models by supplying a JSON file with the model detail with the `SBC_POWER_MODEL_JSON` environmental variable.
The format needs to follow the one used in our [Power Models Database](https://github.com/joular/powermodels).