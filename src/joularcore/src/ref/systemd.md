# Systemd Service

A systemd service is provided and can be installed (by copying `joularcore.service` in `systemd` folder to `/etc/systemd/system/`).
The service will run the program with the `-o` option (which only saves the latest power data) and the `-f` option to save data to `/tmp/joularcore-service.csv`.
The service can be enabled to run automatically on boot.

You can modify the service as needed to start Joular Core with any argument and output the data with any format (CSV, HTTP API, ring buffer shared memory, etc.).

The systemd service is automatically installed when installing Joular Core using the installer script.