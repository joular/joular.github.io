# How Joular Energy Meter Works

Joular Energy Meter is written in C, with [LVGL](https://lvgl.io) drawing the window on top of SDL2, so it has the same interface on every system.
The hardware is measured by our [Joular Core](https://github.com/joular/joularcore) library, and the CPU usage by our [CPU Load](https://github.com/joular/cpuload) library.
Both are written in Ada, and are linked into the program as static libraries with their Ada runtime, so it is one binary.

## The measuring thread

The libraries are only called from a measuring thread, so a slow reading (```powermetrics``` on macOS, for example) never blocks the window.
The thread queues its measurements for the window, up to four minutes of them at 1 second while the window is blocked (a window dragged on Windows, a menu held open on macOS).
Beyond that, the oldest are dropped, and counted as missed.

An idle window costs nothing: the program sleeps until the next event or the next measurement.

## A measurement

Each cycle, Joular Energy Meter:
1. Reads the CPU time of the whole machine, of the followed program, and of Joular Energy Meter itself, from the same sample.
2. Reads the processor and the graphics card from Joular Core.
3. Divides the energy given by the hardware (RAPL, for example) by how long the cycle actually took, so the watts are right even when a cycle takes longer than the interval, and when the interval is changed while measuring. Hardware that gives watts (```powermetrics```, NVML) is used as it is.
4. Shares the processor's power out to the followed program (see [Measuring a Program](./program.html)).
5. Adds the power times the length of the cycle to the energy since the start.

A cycle during which the machine slept for more than 2 seconds is dropped, and counted as missed.

From the energy since the start:
- The average power is the energy divided by the measured time.
- The spread is the standard deviation of the power around this average, each measurement weighted by the time it covers.
- *A year at this average* is the average power for 365.25 days, day and night.

The share of the CPU used by Joular Energy Meter itself is shown under the chart (on macOS, without the ```powermetrics``` it starts).

## Asking for the administrator

When the power cannot be read for lack of rights, Joular Energy Meter asks through the system's own dialog, never through a password field of its own:
- **Linux**: ```pkexec chmod a+r``` on the energy counter of each top-level RAPL zone (```intel-rapl:N/energy_uj```), which stays readable by every user until the next boot. The application itself keeps running as you.
- **macOS**: the administrator dialog (```osascript``` with administrator privileges) starts the application again as root, as ```powermetrics``` needs.
- **Windows**: UAC starts the application again as administrator, for the PawnIO driver.

Restarting saves the settings first, so the new run starts where the old one was.

## Running as root

Started with ```sudo```, Joular Energy Meter keeps your files yours:
- CSV files it creates belong to you (see [CSV Files](./exports.html)).
- Settings go to your own folder, not root's (see [Options and Settings](./settings.html)).
- **Copy summary** goes to your own clipboard on macOS, not root's.
- Commands it starts run as you.

On Linux, a program run as root cannot open a window on Wayland, so use X11, or **Allow reading power** instead of ```sudo```.
