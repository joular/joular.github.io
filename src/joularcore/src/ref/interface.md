# Library Interface

Joular Core has the same interface in Ada and in C: open, read, close, and the version of the library.

## Ada

The whole interface is the `Joular_Core` package.

### Types

| Type | Description |
|------|-------------|
| `Source` | The hardware sources to measure: `CPU` or `GPU`. |
| `Source_List` | An array of `Boolean` indexed by `Source`: which sources to measure (the ones set to `True`). |
| `All_Sources` | A `Source_List` with every source set to `True`. |
| `Measurement_Unit` | `Energy` (joules consumed since the previous reading) or `Power` (watts being drawn, when read or averaged since the previous reading). |
| `Measurement` | One measurement of one source: `Available` (`Boolean`), `Value` (`Long_Float`, joules or watts) and `Unit` (`Measurement_Unit`). |
| `Reading` | An array of `Measurement` indexed by `Source`: one measurement per source. |

### Subprograms

| Subprogram | Description |
|------------|-------------|
| `procedure Open (Sources : in Source_List := All_Sources)` | Detect the hardware sources asked for, and open the files, drivers or processes needed to read them. A source that is not there, or cannot be read, is reported as not available by `Read`. Calling `Open` again closes what was open first. |
| `function Read return Reading` | Take one reading of every source `Open` could open. A source that fails to answer reports a value of zero, with `Available` still `True`. |
| `procedure Close` | Close what `Open` opened. |
| `function Version return String` | The version of the library. |

For example, to measure the CPU only:

```ada
Open ((CPU => True, GPU => False));
```

## C

The C declarations are in [include/joularcore.h](https://github.com/joular/joularcore/blob/main/include/joularcore.h). Use them with the relocatable (shared) library, which starts itself up when loaded: no other initialization call is needed.

### Types

```c
typedef struct joularcore_measurement {
    double value;      /* energy or power value, see unit */
    int    available;  /* 1 when the source was requested and opened, 0 otherwise */
    int    unit;       /* 0 when value is energy in joules, 1 when it is power in watts */
} joularcore_measurement;

typedef struct joularcore_reading {
    joularcore_measurement cpu;
    joularcore_measurement gpu;
} joularcore_reading;
```

### Functions

| Function | Description |
|----------|-------------|
| `void joularcore_open(int cpu, int gpu)` | Same as `Open`: a source is measured when its flag is not zero. |
| `void joularcore_read(joularcore_reading *out)` | Same as `Read`: writes one reading of every source into `*out`. |
| `void joularcore_close(void)` | Same as `Close`. |
| `const char *joularcore_version(void)` | Same as `Version`. The string is owned by the library, so do not free it. |

## Environment Variable

| Variable | Description |
|----------|-------------|
| `JOULARCORE_WINDOWS_RAPL` | On Windows, picks how the RAPL counter is read: `emi`, `pawnio` or `hubblo`. Any other value, or not setting it at all, tries the Energy Meter Interface first, then PawnIO, then Hubblo's driver, and keeps the first that answers. |

When the one picked does not answer, the others are not tried and the CPU is reported as not available. Upper or lower case does not matter.

All three end up on the same package counter of the processor, so they report the same energy. Picking one is useful to test each of them on a machine carrying several, or to work around a broken one.

```
set JOULARCORE_WINDOWS_RAPL=emi
set JOULARCORE_WINDOWS_RAPL=pawnio
set JOULARCORE_WINDOWS_RAPL=hubblo
```

## Constraints

- The library is **not thread safe**: call `Open`, `Read` and `Close` from a single thread (or task), as one monitoring loop is the intended use for the current version.
- Energy counters (RAPL) wrap after a few minutes under load, so read at least once per minute to not miss a wrap. On Windows with the Energy Meter Interface, the wrap is handled by Windows.
- The first reading after `Open` counts the energy from `Open`.