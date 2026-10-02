# Quick Usage

Joular Core has one simple interface: open the sources, read them as often as you need, then close them.

- `Open` detects the hardware asked for (the CPU, the GPU, or both) and opens what is needed to read it.
- `Read` takes one reading of every source that could be opened.
- `Close` closes what was opened.

Each reading gives, for the CPU and for the GPU, whether the source is available, its value, and its unit: energy in joules consumed since the previous reading, or power in watts. See [Reading the Measurements](../ref/measurements.md) for what each one means.

## Using from Ada

```ada
with Ada.Text_IO; use Ada.Text_IO;
with Joular_Core; use Joular_Core;

procedure Measure is
    Measurements : Reading;
begin
    Open; -- Detect and open every supported hardware source

    for I in 1 .. 5 loop
        delay 1.0;
        Measurements := Read;

        if Measurements (CPU).Available then
            Put_Line ("CPU:" & Long_Float'Image (Measurements (CPU).Value)
                      & (if Measurements (CPU).Unit = Energy then " J" else " W"));
        end if;
    end loop;

    Close;
end Measure;
```

With Alire, add the library to your project with `alr with joularcore`.

A full example program is in [example/src/example_joular_core.adb](https://github.com/joular/joularcore/blob/main/example/src/example_joular_core.adb). It reads once per second until stopped with Ctrl+C, which closes the sources cleanly. To build and run it:

```bash
gprbuild -P example/example.gpr
./example/example_joular_core
```

It takes two optional arguments, in any order: on Windows, `emi`, `pawnio` or `hubblo` to say how the RAPL counter is read (rather than trying them in turn), and a number of readings to do before stopping. A summary is printed at the end.

```bash
./example/example_joular_core emi 10
```

## Using from C

The C declarations are in [include/joularcore.h](https://github.com/joular/joularcore/blob/main/include/joularcore.h). Build the relocatable (shared) library (see [Compilation](../ref/compilation.md)), then:

```c
#include <stdio.h>
#include "joularcore.h"

joularcore_reading reading;

joularcore_open(1, 1);   /* measure the CPU and the GPU */
joularcore_read(&reading);
if (reading.cpu.available)
    printf("CPU: %f %s\n", reading.cpu.value, reading.cpu.unit == 0 ? "J" : "W");
joularcore_close();
```

A full example program is in [example/c/main.c](https://github.com/joular/joularcore/blob/main/example/c/main.c). Like the Ada one, it reads once per second until stopped with Ctrl+C. It comes with a Makefile that builds the shared library and the program:

```bash
make -C example/c
```

## Using from Python

From Python, the same interface through ctypes:

```python
import ctypes

class Measurement(ctypes.Structure):
    _fields_ = [("value", ctypes.c_double), ("available", ctypes.c_int), ("unit", ctypes.c_int)]

class Reading(ctypes.Structure):
    _fields_ = [("cpu", Measurement), ("gpu", Measurement)]

lib = ctypes.CDLL("lib/relocatable/libjoularcore.so")
lib.joularcore_read.argtypes = [ctypes.POINTER(Reading)]
lib.joularcore_version.restype = ctypes.c_char_p

lib.joularcore_open(1, 1)   # measure the CPU and the GPU
r = Reading()
lib.joularcore_read(ctypes.byref(r))
if r.cpu.available:
    print("CPU:", r.cpu.value, "J" if r.cpu.unit == 0 else "W")
lib.joularcore_close()
```

The library is `libjoularcore.dylib` on macOS and `libjoularcore.dll` on Windows.

A full example program is in [example/python/main.py](https://github.com/joular/joularcore/blob/main/example/python/main.py), with a Makefile that builds the shared library.

Other languages (C++, Java, Rust, etc.) use the same C interface. See [Integration with Systems and Tools](../ref/integration.md).