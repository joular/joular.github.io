# Compilation and Packages

Joular Energy Meter is written in C, and is linked with the [Joular Core](https://github.com/joular/joularcore) and [CPU Load](https://github.com/joular/cpuload) libraries, written in Ada.
It needs GNAT and GPRBuild, SDL2 (2.0.18 or newer), and LVGL 9.6.

LVGL goes in the ```lvgl``` folder, at the commit of its ```release/v9.6``` branch that the CI builds against (see ```.github/workflows/build.yml```):

```
git clone https://github.com/joular/joular-energy-meter
cd joular-energy-meter
git init lvgl && git -C lvgl fetch --depth 1 https://github.com/lvgl/lvgl.git 8df3de9c4c5e45ad7b0fef5adc2fa6eaa5d21579 && git -C lvgl checkout FETCH_HEAD
```

The program is created as ```bin/joularenergymeter```.

## SDL2

```
macOS:
brew install sdl2

Debian, Ubuntu:
sudo apt install libsdl2-dev

Fedora:
sudo dnf install SDL2-devel

Windows, in Alire's msys2 shell:
pacman -S mingw-w64-x86_64-SDL2

FreeBSD:
sudo pkg install sdl2
```

## Compilation with Alire

[Alire](https://alire.ada.dev/) downloads GNAT, GPRBuild and the two libraries, and checks that SDL2 is installed (and offers to install it):

```
alr build
```

When SDL2 is somewhere else:

```
alr build -- -XSDL2_CFLAGS="-I<folder>" -XSDL2_LIBS="-L<folder> -lSDL2"
```

On macOS, use ```alr build -- -XDev=True``` for a program run from the source folder.
Homebrew's SDL2 loads SDL3 by name, and this switch tells dyld where to find it.
The packaged application is built without it, so it looks for nothing outside the application when run with ```sudo```.

## Compilation with GNAT and GPRBuild

Check out the two libraries next to the Joular Energy Meter folder, then point GPRBuild at them:

```
cd ..
git clone --depth 1 --branch 0.1.0 https://github.com/joular/cpuload.git
git clone --depth 1 --branch 0.1.0 https://github.com/joular/joularcore.git
cd joular-energy-meter
gprbuild -P joularenergymeter.gpr -aP../joularcore -aP../cpuload -p
```

The OS is detected from the compiler, and can be forced with ```-XPJ_OS=linux```, ```windows```, ```macos``` or ```freebsd```.

GPRBuild does not look where Homebrew puts SDL2, so on macOS add:

```
-XSDL2_CFLAGS="-I$(brew --prefix)/include" -XSDL2_LIBS="-L$(brew --prefix)/lib -lSDL2"
```

On FreeBSD, this is the only way to build, as Alire has no FreeBSD SDL2 package.
Use GNAT 15 or newer (```sudo pkg install gprbuild gnat15```, then put ```/usr/local/gnat15/bin``` first in ```PATH```), as GNAT 12 cannot compile Joular Core's FreeBSD code.

## Packages and installers

The ```packaging/``` folder has one script per system.
Each script builds the program, checks it, and makes the packages in ```dist/```.
The CI runs the same scripts and keeps the packages as artifacts.
The top of each script lists what it needs.

| System | Run | Makes |
| --- | --- | --- |
| Linux | ```packaging/linux.sh``` | ```.tar.gz```, ```.deb```, ```.rpm```, Arch Linux ```.pkg.tar.zst```, ```.AppImage``` |
| macOS | ```packaging/macos.sh``` | ```.zip``` with *Joular Energy Meter.app*, ```.dmg``` |
| Windows | ```powershell -ExecutionPolicy Bypass -File packaging\windows.ps1``` | ```.zip```, ```.msi``` |
| FreeBSD | ```packaging/freebsd.sh``` | ```.tar.gz```, ```.pkg``` |

The Linux packages need the glibc of the build machine or newer, so build them on an old distribution (the CI uses Ubuntu 22.04).
On macOS, the script builds the application's binary in ```bin/app```, so ```bin/joularenergymeter``` stays the one that runs from the source folder, and two binaries (arm64 and x86_64) make one universal application.

A binary built by hand needs SDL2 from where it was built (Homebrew, MSYS2), so ship the packages instead.
