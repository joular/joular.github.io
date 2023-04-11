# Installation

PowerJoular is written in Ada and can be easily compiled, and its unique binary added to your system PATH.

Easy-to-use installation scripts are available in the ```installer``` folder.
Just open the installer folder and run the appropriate file to build and/or install or uninstall the program and systemd service.

- ```build-install.sh```: will build (using ```gprbuild```) and install the program binary to ```/usr/bin``` and systemd service. It requires having installed GNAT and gprbuild (see [Compilation](../ref/compilation.html)).
- ```uninstall.sh```: deletes the program binary and systemd service.