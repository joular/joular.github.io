# Systemd Service

A systemd service is provided for Linux and can be installed (by copying ```powerjoular.service``` in ```systemd``` folder to ```/usr/lib/systemd/system/``` or ```/etc/systemd/system/```).
The service runs ```/usr/bin/powerjoular```, so copy the binary there too, or change ```ExecStart``` in the service.
The service will run the program with the ```-o``` option (which only saves the latest power data) and saves data to ```/run/powerjoular/powerjoular-service.csv```.
The ```/run/powerjoular``` folder is made by systemd when the service starts and removed when it stops, and anyone can read the file in it.

The service runs as root, as reading the energy counters needs it, but with most of the other privileges taken away (it only reads the energy counters and the CPU load, and writes one file in its own folder).
If it stops on its own (not through ```systemctl```), it is restarted after 30 seconds.

The service can be started, and enabled to run automatically on boot, with:

```
sudo systemctl start powerjoular.service
sudo systemctl enable powerjoular.service
```

The systemd service is automatically installed (but not enabled) when installing PowerJoular using the installer script or the Linux packages.

Version 1 installed the service in ```/etc/systemd/system/```, which takes precedence over the one now installed in ```/usr/lib/systemd/system/```.
If you installed version 1 with the installer script, delete ```/etc/systemd/system/powerjoular.service``` and run ```sudo systemctl daemon-reload```.