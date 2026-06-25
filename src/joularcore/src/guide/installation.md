# Installation

Joular Core is distributed as pre-built binaries and installers for the most common platforms. For anything else, you can compile it from source in a few minutes.

## Pre-built Packages and Installers

### Windows

- MSI installer for 64-bit Windows (x86_64)

### macOS

- PKG installer for Apple Silicon (aarch64)

### Linux

- DEB packages for Debian-based systems (Debian, Ubuntu, Raspberry Pi OS, etc.) — available for x86_64 and arm64 (aarch64)
- RPM packages for Red Hat / Fedora-based systems (RHEL, Fedora, etc.) — x86_64
- AUR package for Arch-based systems — x86_64

For other architectures or distributions (32-bit Windows, Intel macOS, other Linux setups), compile from source. See [Compilation](../ref/compilation.md).

## Binaries

Compiled binaries for all supported targets are attached to each release on the GitHub repository. Download the binary for your platform, place it somewhere on your `PATH`, and you're ready to go. You can also run it from any directory using its full path.

There are two binaries per release:

- `joularcore` / `joularcore.exe` — the command-line interface
- `joularcoregui` / `joularcoregui.exe` — the graphical user interface (opens a window directly, no terminal needed)

## Platform-specific Requirements

### Linux (PC / servers)

CPU power is read via the RAPL sysfs interface. On Linux kernel 5.10 and newer, RAPL files are only readable by root. You have two options:

- Run Joular Core with `sudo`
- Grant read access to the RAPL files for your user (see [this GitHub issue](https://github.com/joular/joularcore/issues/12) for instructions)

GPU monitoring (Nvidia/AMD) requires `nvidia-smi` or `amd-smi` / `rocm-smi` to be installed separately if needed.

### Windows

CPU power monitoring requires a RAPL kernel driver. The supported driver is [Hubblo's windows-rapl-driver](https://github.com/hubblo-org/windows-rapl-driver). The simplest way to install a signed version is via the [Scaphandre installer](https://github.com/hubblo-org/scaphandre/releases/download/v1.0.0/scaphandre_v1.0.0_installer.exe), which installs the driver as a side effect.

Once the driver is installed, Joular Core itself runs without administrator rights.

GPU monitoring (Nvidia/AMD) requires `nvidia-smi` or `amd-smi` to be installed separately if needed.

### macOS

No additional software is required. Power data is read via `powermetrics`, which ships with macOS. Because `powermetrics` requires elevated access to read hardware counters, run the CLI with `sudo` or use a GUI/default build that can ask for elevation.

### Raspberry Pi and SBC

No dependencies and no `sudo` required. Use a binary built with the `sbc` feature for your architecture (arm, armv7, or aarch64). Unsupported boards run but report 0 W for CPU power.
