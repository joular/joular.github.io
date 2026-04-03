# Command-Line Options

Run `joularcore --help` for the full list of options. This page documents each option in detail.

## Synopsis

```
joularcore [OPTIONS]
```

## Options

### Monitoring Target

| Option | Description |
|--------|-------------|
| `-p`, `--pid <PID>` | Monitor a specific process by its PID. The output will include the power attributed to that process. |
| `-a`, `--app <APP>` | Monitor an application by name. Joular Core finds all running processes whose name matches `<APP>` and sums their attributed power. |

`--pid` and `--app` are mutually exclusive — only one may be used at a time.

### Output and Export

| Option | Description |
|--------|-------------|
| `-f`, `--file <FILE>` | Write power measurements to a CSV file. A header line is written once, then one row per second is appended. |
| `-o`, `--overwrite` | When used with `-f`, overwrite the file on each write instead of appending. Only the latest measurement is kept in the file. Useful when another program is polling the file. |
| `-i`, `--numeric` | Print only a bare numeric value (watts, two decimal places) with no labels or ANSI formatting. Useful for piping or scripting. The value printed is total power by default, or the selected component if `-c` is set. |
| `-s`, `--silent` | Suppress all terminal output. CSV export, ring buffer, and API remain active. Useful when running in the background. |
| `-r`, `--ringbuffer` | Write power data to a shared-memory ring buffer every second. See [Exporting Power Data](./exports.md) for paths and data layout. |
| `--api-port <PORT>` | Start an HTTP and WebSocket API server on the given port. Requires the `api` feature. See [Exporting Power Data](./exports.md) for endpoint details. |

### Component Filter

| Option | Description |
|--------|-------------|
| `-c`, `--component <cpu\|gpu>` | Restrict output to a single hardware component. Use `cpu` for CPU-only power, `gpu` for GPU-only power. When set, only that component's power is shown on the terminal and written to CSV or numeric output. |

### CPU Idle Baseline

These options subtract idle CPU power before attributing energy to a process or application. This gives a more accurate picture of how much power the workload itself is consuming, rather than including the base system cost.

| Option | Description |
|--------|-------------|
| `--cpu-idle-baseline <WATTS>` | Subtract a fixed value (in watts) from CPU power before calculating process or application attribution. |
| `--calibrate-cpu-idle-baseline` | Automatically measure idle CPU power. Joular Core collects 5 samples at 1-second intervals before starting the main monitoring loop, averages them, and uses the result as the baseline. |

`--cpu-idle-baseline` and `--calibrate-cpu-idle-baseline` are mutually exclusive.

### Application Monitoring Refresh

| Option | Default | Description |
|--------|:-------:|-------------|
| `--app-refresh-interval <SECONDS>` | 3 | How often (in seconds) to rescan running processes when monitoring an application by name with `-a`. Setting this to `0` disables caching and rescans on every second. Lower values catch short-lived processes faster at the cost of more frequent process enumeration. |

### Interface

| Option | Description |
|--------|-------------|
| `-g`, `--gui` | Launch the graphical user interface instead of (or alongside) the terminal output. |
| `-h`, `--help` | Print help and exit. |
| `-V`, `--version` | Print the version number and exit. |

## Environment Variables

These environment variables control behaviour that cannot be set via CLI flags.

### Virtual Machines

| Variable | Description |
|----------|-------------|
| `VM_CPU_POWER_FILE` | Path to a file containing CPU power data written by the host |
| `VM_CPU_POWER_FORMAT` | Format of that file: `joularcore`, `powerjoular`, or `watts` (default: `watts`) |
| `VM_GPU_POWER_FILE` | Path to a file containing GPU power data written by the host |
| `VM_GPU_POWER_FORMAT` | Format of that file: `joularcore`, `powerjoular`, or `watts` (default: `watts`) |

See [Virtual Machines](./vm.md) for details.

### Single-Board Computers

| Variable | Description |
|----------|-------------|
| `SBC_POWER_MODEL_JSON` | Path to a JSON file containing a custom SBC power model. If unset, built-in models for supported boards are used. The file format must match the [Joular Power Models Database](https://github.com/joular/powermodels) schema. |

## Constraints and Combinations

- `-p` and `-a` are mutually exclusive.
- `-o` only has effect when `-f` is also set.
- `--cpu-idle-baseline` and `--calibrate-cpu-idle-baseline` are mutually exclusive.
- `--api-port` requires that Joular Core was compiled with the `api` feature (enabled by default).
- `-g` / `--gui` requires the `gui` feature (enabled by default); alternatively, use the `joularcoregui` binary.

## Examples

```bash
# Monitor whole system
joularcore

# Monitor a specific process
joularcore -p 1234

# Monitor application, write to CSV
joularcore -a firefox -f power.csv

# Run silently, write to CSV, overwrite mode
joularcore -s -f /tmp/power.csv -o

# Expose API, suppress terminal output
joularcore -s --api-port 8080

# Write ring buffer + CSV simultaneously
joularcore -r -f power.csv

# CPU-only numeric output (for scripting)
joularcore -c cpu -i

# Monitor process with auto-calibrated idle baseline
joularcore -p 1234 --calibrate-cpu-idle-baseline

# Monitor app, refresh PID list every second
joularcore -a myapp --app-refresh-interval 0

# Launch GUI with API enabled
joularcore -g --api-port 9000
```
