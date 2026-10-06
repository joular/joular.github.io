# Installation

PowerJoular is written in Ada and can be easily compiled, and its unique binary added to your system PATH.
The binary can be copied to any machine of the same architecture and run as it is.

Ready-made binaries and packages are released in our [GitHub repository](https://github.com/joular/powerjoular) release page and in the build workflow:
- Linux: binaries, Debian installation packages (.deb, for Debian, Ubuntu, Raspberry Pi OS, etc.) and RPM packages (.rpm, for Fedora, RHEL, etc.), for x86_64 and aarch64.
- Windows: ```powerjoular.exe```, for x86_64.
- macOS: one binary per chip, ```powerjoular-macos-arm64``` for Apple Silicon and ```powerjoular-macos-x86_64``` for Intel Macs. Take the one of your Mac, copy it where you want it, and make it executable with ```chmod +x```.
- FreeBSD: ```powerjoular-freebsd-amd64```, for x86_64, built on FreeBSD 14 so it also runs on FreeBSD 15. Copy it where you want it, and make it executable with ```chmod +x```.

## Which Linux build to use

The PowerJoular binary runs on the glibc version it was built on or a newer one (but not an older one), so more than one build is published, and every file says which glibc version was used:

| File | Architectures | Runs on |
|---|---|---|
| ```powerjoular-glibc-2.35```, ```powerjoular-glibc-2.35_*.deb```, ```powerjoular-glibc-2.35-*.rpm``` | x86_64 and aarch64 | Ubuntu 22.04 and newer, Debian 12 and newer, Raspberry Pi OS bookworm and newer, Fedora 36 and newer |
| ```powerjoular-glibc-2.34```, ```powerjoular-glibc-2.34_*.deb```, ```powerjoular-glibc-2.34-*.rpm``` | x86_64 only | The same, and also RHEL 9, AlmaLinux 9, Rocky 9 and CentOS Stream 9 |

Take the ```2.34``` build if you are unsure, or if the other one gives you a ```version 'GLIBC_2.35' not found``` error.
Both are built from the same code, the difference being the C library (glibc) they were linked against.

On a 32 bits ARM board or OS (such as the Raspberry Pi Zero W, 1 and 2, the Asus Tinker Board, or a 32 bits Raspberry Pi OS), no ready-made build is published, so compile PowerJoular (see [Compilation](../ref/compilation.html)).

The Linux packages install the binary in ```/usr/bin``` along with the systemd service (see [Systemd Service](../ref/systemd.html)).

## Installation scripts

Easy-to-use installation scripts are available in the ```installer/bash-installer``` folder.
Just open the folder and run the appropriate file to build and install, or uninstall, the program and systemd service.

- ```build-install.sh```: will build (using Alire if it is installed, otherwise ```gprbuild```) and install the program binary to ```/usr/bin``` and the systemd service. It requires having installed Alire, or GNAT and gprbuild with the Joular Core and CPU Load repositories checked out next to the PowerJoular folder (see [Compilation](../ref/compilation.html)).
- ```uninstall.sh```: stops the systemd service, and deletes the program binary and the systemd service.

These scripts and the packages are for Linux. A ```PKGBUILD``` for Arch Linux is also available in the ```installer/aur``` folder.