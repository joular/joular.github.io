# Reading the Measurements

Each reading gives one measurement for the CPU and one for the GPU. A measurement says whether the source is available, its value, and its unit.

## Energy and Power

- Some hardware reports **energy**: the joules consumed since the previous reading (RAPL on Linux and Windows). The first reading counts from `Open`.
- Others report **power**: the watts being drawn when read (GPUs), or averaged since the previous reading (Raspberry Pi and Asus Tinker Board models).
- On macOS, the value is the **average power since the previous reading**: Joular Core asks `powermetrics` for a sample at each reading. Opening the sources waits for the first sample of `powermetrics` (up to about three seconds).

To get watts from an energy reading, divide it by the time elapsed since the previous reading. To get joules from a power reading, multiply it by that time. [PowerJoular](https://github.com/joular/powerjoular) does the first on every cycle, dividing by the time the cycle actually took rather than assuming it was exactly one second.

## Available, Not Available, and Zero

- A source that is not present, not supported, or not accessible is reported as **not available**. This is not an error: the other sources keep working.
- A source that was available but stops answering reports a value of **zero**, and is still available. For RAPL, the energy is not lost: the next reading that works also counts what was used during the one that failed.
- On macOS, if `powermetrics` stops answering, its sources read zero, and it is started again around ten seconds later.

## What Each Source Actually Measures

| Source | What the number covers |
|---|---|
| RAPL on Linux | **One** package of powercap, the first one whose name begins with `package` |
| RAPL on Windows | The package domain of the first socket |
| Raspberry Pi and Asus Tinker Board | A model-based estimate: a regression on CPU load, evaluated over the interval between two readings. It is not a reading of the board's actual draw |
| Nvidia (NVML) | What the card reports for the whole GPU board. Depending on the architecture and driver, this is an average over about a second rather than an instant value |
| AMD on Linux (hwmon) | What the amdgpu driver reports (`power1_average`, or `power1_input`), which the kernel documents as the power used by the SoC, the GPU chip. On an APU that chip includes the CPU cores, so work done on the CPU raises what this library calls the GPU, and adding CPU and GPU together counts some of it twice |
| AMD on Windows (ADLX) | Whole GPU board power where the card offers it, and the graphics processor alone where it does not. Which of the two is chosen when the card is opened and does not change while the program runs, so a series of measurements always means one thing |
| macOS | Apple Silicon: the CPU and the GPU parts of the same chip, from the same sample, as `powermetrics` estimates them. Intel Macs: the whole chip (cores, integrated GPU and system agent), like the RAPL package on Linux and Windows |

## Counter Wraps

RAPL counters wrap when they fill, and the library corrects that. The correction only works if less than half of what the counter holds was used between two readings, so read at least once per minute. How long the counter takes to fill depends on the energy unit of the processor.

A counter that goes back without looking like a wrap is taken as a reset (for example after the machine was suspended), and that reading gives zero.

The Energy Meter Interface (EMI) on Windows corrects for the wrap by itself, so Joular Core doesn't need to do the correction.