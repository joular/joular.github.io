# Systemd Service

A systemd service is provided and can be installed (by copying ```powerjoular.service``` in ```systemd``` folder to ```/etc/systemd/system/```).
The service will run the program with the ```-o``` option (which only saves the latest power data) and saves data to ```/tmp/powerjoular-service.csv```.
The service can be enabled to run automatically on boot.

The systemd service is automatically installed when installing PowerJoular using the installer script.