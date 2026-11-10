# Installation

Ready-made packages are released in our [GitHub repository](https://github.com/joular/joular-energy-meter) release page and in the build workflow.
They need no compiler, Alire or MSYS2.

Every package has the program, the carbon intensities of the countries, the README and the licences.

## Linux

For x86_64 and aarch64:

```
Debian, Ubuntu:
sudo apt install ./joularenergymeter_<version>-1_amd64.deb

Fedora:
sudo dnf install ./joularenergymeter-<version>-1.x86_64.rpm

Arch Linux:
sudo pacman -U joularenergymeter-<version>-1-x86_64.pkg.tar.zst
```

These packages also install SDL2.
The AppImage has SDL2 inside and runs on any distribution: make it executable with ```chmod +x```, then start it.
The tar.gz needs the SDL2 package (```libsdl2-2.0-0``` on Debian and Ubuntu, ```SDL2``` on Fedora).

All of them need glibc 2.35 or newer (Debian 12, Ubuntu 22.04, Fedora 36 and later), as they are built on Ubuntu 22.04.
On Raspberry Pi, this means a 64 bits OS.

## macOS

Open the ```.dmg``` and drag the application to Applications.
It is one application for Apple Silicon and Intel Macs, with SDL2 inside, and needs macOS 15 or newer (GNAT's Ada runtime is built for 15).

The application is only signed ad hoc, so macOS blocks it the first time: open *System Settings > Privacy & Security* and click *Open Anyway*.

Started from Finder, it writes its CSV files to the home folder.
To read the power, it needs root (see [Required privileges](./supported_platforms.html#required-privileges)).

## Windows

Run the ```.msi``` (it installs to Program Files, adds a Start menu shortcut, and is removed from *Settings > Apps*), or unzip the ```.zip``` and run ```joularenergymeter.exe```.
Both ship SDL's own ```SDL2.dll```.

The installer is not signed, so SmartScreen asks once (*More info*, then *Run anyway*).

## FreeBSD

```
sudo pkg install sdl2
sudo pkg add joularenergymeter-<version>-freebsd14-amd64.pkg
```

```pkg add``` looks for dependencies only beside the file, hence SDL2 first.
A tar.gz is also available.
The package is made for the FreeBSD version it was built on.

## From the source code

See [Compilation and Packages](../ref/compilation.html).
