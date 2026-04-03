# Systemd Service (Linux)

Joular Core ships with a ready-to-use systemd unit file located at `systemd/joularcore.service`. It lets you run Joular Core as a background daemon that starts on boot and restarts automatically if it crashes.

## Default Service Configuration

The included unit file looks like this:

```ini
[Unit]
Description=Joular Core service

[Service]
Type=simple
Restart=always
User=root
ExecStart=/usr/bin/joularcore -o -f /tmp/joularcore-service.csv

[Install]
WantedBy=multi-user.target
```

By default it runs Joular Core as root (required for RAPL access on Linux), writes the latest power reading to `/tmp/joularcore-service.csv` in overwrite mode (`-o`), and restarts automatically on failure.

## Installation

Copy the unit file to the systemd directory and reload the daemon:

```bash
sudo cp systemd/joularcore.service /etc/systemd/system/
sudo systemctl daemon-reload
```

Enable automatic start on boot and then start the service immediately:

```bash
sudo systemctl enable joularcore
sudo systemctl start joularcore
```

Check that it is running:

```bash
sudo systemctl status joularcore
```

## Customisation

You can edit the `ExecStart` line to pass any Joular Core arguments. For example, to also expose an HTTP API and monitor a specific application:

```ini
ExecStart=/usr/bin/joularcore -o -f /tmp/joularcore-service.csv --api-port 8080
```

Or to write to a ring buffer without any file output:

```ini
ExecStart=/usr/bin/joularcore -s -r
```

After editing the file, reload systemd for the change to take effect:

```bash
sudo systemctl daemon-reload
sudo systemctl restart joularcore
```

## Reading the Output File

When using the default `-o -f /tmp/joularcore-service.csv` configuration, the file always contains exactly one data row — the most recent measurement. Any script or tool can poll this file at its own pace:

```bash
cat /tmp/joularcore-service.csv
```

Example output:
```
Timestamp,Total Power (W),CPU Power (W),GPU Power (W),CPU Usage (%)
1712345678,18.45,15.20,3.25,24.60
```

## Stopping and Disabling

```bash
sudo systemctl stop joularcore
sudo systemctl disable joularcore
```
