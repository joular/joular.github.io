# Quick Usage — Command-Line Interface

## Starting Joular Core

Run `joularcore` (Linux / macOS) or `joularcore.exe` (Windows). The program starts monitoring immediately and updates the terminal display once per second.

**Linux (PC/server)** — requires elevated access to read RAPL power files:
```bash
sudo joularcore
```

**macOS** — requires elevated access for `powermetrics`; run with `sudo` for CLI use:
```bash
sudo joularcore
```

**Windows**:
```
joularcore.exe
```

**Raspberry Pi and SBC** — no `sudo` needed:
```bash
joularcore
```

Press **Ctrl+C** to stop.

## Default Output

With no arguments, Joular Core displays total power, CPU power, GPU power, and CPU usage — updated every second in place on the same terminal line:

```
⚡ Total 18.45 W | CPU 15.20 W | GPU 3.25 W | CPU Usage 24.60%
```

## Common Use Cases

### Monitor a specific process by PID

```bash
joularcore -p 1234
```

Output includes the power attributed to that process:
```
⚡ Total 18.45 W | CPU 15.20 W | GPU 3.25 W | CPU Usage 24.60% | PID 1.84 W
```

### Monitor an application by name

```bash
joularcore -a firefox
```

Joular Core finds all processes whose name matches `firefox` and sums their attributed power:
```
⚡ Total 18.45 W | CPU 15.20 W | GPU 3.25 W | CPU Usage 24.60% | App 3.12 W (4 PIDs)
```

### Write output to a CSV file

```bash
joularcore -f power.csv
```

A new row is appended every second. See [Exporting Power Data](../ref/exports.md) for the exact CSV format.

### Run silently and only write to CSV

```bash
joularcore -s -f power.csv
```

The `-s` flag suppresses terminal output while keeping all other outputs (CSV, ring buffer, API) active.

### Show only CPU or only GPU power

```bash
joularcore -c cpu
joularcore -c gpu
```

### Numeric-only output (for scripting and piping)

```bash
joularcore -i
```

Prints a single float (total power in watts) per line with no labels or formatting. Combined with `-c`, it prints only the selected component:

```bash
joularcore -c cpu -i
```

### Expose power data over HTTP and WebSocket

```bash
joularcore --api-port 8080
```

Starts an HTTP server. `GET http://localhost:8080/data` returns the latest reading as JSON, and `ws://localhost:8080/ws` streams a new JSON reading every second.

The server binds to `127.0.0.1`. Browser access is limited to localhost origins by default; use `--api-allowed-origin <ORIGIN>` to add another CORS origin.

### Expose power data over a shared memory buffer ring

```bash
joularcore -r
```

### Subtract idle CPU baseline for process attribution

```bash
# Auto-calibrate: measure idle power over 5 seconds, then start monitoring
joularcore -p 1234 --calibrate-cpu-idle-baseline

# Or supply a known baseline manually
joularcore -p 1234 --cpu-idle-baseline 4.5
```

## Full Options Reference

See the [Command Line Options](../ref/options.md) page for a complete list of all flags and their descriptions.
