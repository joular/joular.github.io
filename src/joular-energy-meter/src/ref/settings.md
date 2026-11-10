# Options and Settings

## Command line options

Everything is chosen in the window, and the command line has only these options:

| Option | What it does |
|---|---|
| ```--start``` | Start measuring at once, with the choices kept from the last time |
| ```--idle``` | Open the window that measures the idle power |
| ```-v```, ```--version``` | Print the versions of Joular Energy Meter, Joular Core and CPU Load, and stop |
| ```-h```, ```--help``` | Print the help, and stop |

## Settings file

The choices are kept for the next launch: what to measure (except a process ID), the interval, the length, the recording and its file, the electricity, the price, the idle power, the plug meter, the look and the zoom.
They are saved at each Start and on quit, as ```key=value``` lines, in:

| OS | File |
|---|---|
| Linux, FreeBSD | ```~/.config/joularenergymeter/settings``` (or in ```$XDG_CONFIG_HOME``` when set) |
| macOS | ```~/Library/Application Support/Joular Energy Meter/settings``` |
| Windows | ```%APPDATA%\Joular Energy Meter\settings``` |

Under ```sudo```, they go to your own folder, not root's.
Delete the file to go back to the defaults.

## Look and zoom

The sun or moon key at the top right picks a light or a dark look, or one like the system's (the default): the appearance of macOS and Windows, and GNOME's style on Linux.
On Linux and FreeBSD under ```sudo```, "like the system" is light.

**Ctrl +** and **Ctrl -** (**Cmd** on macOS) zoom the window from 80 to 150 %, and **Ctrl 0** goes back to 100 %.
The window grows with the zoom, so a zoom larger than the screen can hold is refused.

## Keyboard and accessibility

Every control works from the keyboard: Tab moves between them, Space or Enter uses one, and the arrows choose in a list.
In the file browser, the arrows move, Enter opens, Backspace goes up, letters jump to a name, and Esc cancels.

The help shown on hover needs a mouse.
Screen readers cannot read the window, as LVGL draws it itself, but **Copy summary** gives the figures as text.

## Sleep

While measuring, macOS and Windows do not go to sleep on idle (the screen still can, and closing the lid still does).
On Linux, use ```systemd-inhibit --what=sleep``` for long runs.
