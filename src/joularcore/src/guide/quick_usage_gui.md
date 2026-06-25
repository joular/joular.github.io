# Quick Usage — Graphical User Interface

## Starting the GUI

There are two ways to open the GUI:

- **Dedicated binary**: launch `joularcoregui` (Linux/macOS) or `joularcoregui.exe` (Windows) directly — double-click in a file manager or run from a terminal.
- **CLI flag**: add `-g` or `--gui` to any `joularcore` command, for example:
  ```bash
  joularcore --gui
  joularcore -g --api-port 8080
  ```

On RAPL-based Linux systems, `sudo` is still required:
```bash
sudo joularcoregui
sudo joularcore --gui
```

On macOS, it'll ask for elevated access to read `powermetrics`.

## Interface Overview

The GUI has two screens: **Options** and **Monitor**. Switch between them using the buttons at the top of the window.

### Options Screen

Before starting a monitoring session, the Options screen lets you configure:

- **Monitor mode**: choose between monitoring the whole system, a specific PID, or a named application
- **Application / PID**: type a process name or PID number directly; a dropdown lists running processes or applications
- **CSV export**: toggle CSV file output on or off, and choose the file path using the native file picker
- **Advanced options**: choose CPU/GPU filtering, set or keep the CPU idle baseline, enable the ring buffer, and configure the local HTTP/WebSocket API

When launched as `joularcore --gui`, the flag conflicts with terminal/file-oriented options such as `--pid`, `--app`, `--file`, `--overwrite`, `--silent`, and `--numeric`. Startup options such as `--ringbuffer`, `--api-port`, `--api-allowed-origin`, `--app-refresh-interval`, and the CPU idle baseline flags can still preconfigure the GUI session.

Once configured, click **Start Monitoring** to begin.

### Monitor Screen

The Monitor screen shows live power readings updated every second:

- **Total power** (watts)
- **CPU power** (watts)
- **GPU power** (watts)
- **CPU usage** (%)
- **Process / application power** (watts, if a PID or app was selected)

A rolling graph plots each of these values over the last 60 seconds. Click **Stop** to return to the Options screen.
