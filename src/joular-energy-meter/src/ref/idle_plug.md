# Idle Power and Plug Meter

Both are optional, and set in the Setup tab.

## Idle power

The idle power is what the machine draws doing nothing, in watts.
It can be typed, or measured with **Measure…**: a window of its own measures the whole machine for 30 seconds while you leave it alone, without recording and without touching the readings, then offers its average.
On macOS and Windows, the machine does not go to sleep while it measures.

A typed value is split between the processor and the graphics card as the last measured one was (all to the processor without one).
The value is kept for the next launches, and an empty field means no idle power.

With an idle power, the chart of the machine shows a dashed *idle* line, and the **Above idle** switch shows the readings without it: what the program, or anything you run, adds to an idle machine.

- For the machine, the processor and the graphics card: the power minus their idle power.
- For the program: its share of the processor's power above the processor's idle power.
- Energies: the energy minus the idle power times the measured time (for the program, times its share of the processor over that time), so the ups and downs around the idle power even out.

A figure below zero shows as 0.

The idle power is only as good as the 30 seconds it was measured in: measure it again after a change (power settings, a dock, other programs started at login).

## Plug meter

*Whole machine* is the processor and the graphics card only.
A power meter between the plug and the socket measures the whole computer, screen and power supply included.

**Choose…** picks the CSV file the meter's program writes as it measures, such as [PowerSpyCli](https://github.com/joular/powerspycli) for PowerSpy 2 meters.
From the next Start, the readings get an *At the plug* row and the chart an *At the plug* choice.
**Remove** stops using it.

The rows of the file are ```time;watts```, ```time,watts``` or separated by a tab, and a header is skipped.
When the meter's program starts the file again, it is read again from the top.
Only the rows written after Start count: each measurement takes the average of the rows written during it, or the last one for up to 5 seconds.
After that, a warning shows, and the plug has no reading until a new row comes.

The plug's readings are not written to the recording CSV files.
