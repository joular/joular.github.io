# Exporting Power Data

Joular Core can send power measurements to multiple destinations. All export mechanisms work on all supported platforms and operating systems.

## Terminal Output (default)

By default, Joular Core writes a live display to the terminal. The line is updated in place every second using ANSI escape codes:

```
⚡ Total 18.45 W | CPU 15.20 W | GPU 3.25 W | CPU Usage 24.60%
```

When monitoring a process or application:
```
⚡ Total 18.45 W | CPU 15.20 W | GPU 3.25 W | CPU Usage 24.60% | PID 1.84 W
⚡ Total 18.45 W | CPU 15.20 W | GPU 3.25 W | CPU Usage 24.60% | App 3.12 W (4 PIDs)
```

When a component filter (`-c cpu` or `-c gpu`) is set, only that component is shown:
```
CPU 15.20 W
GPU 3.25 W
```

Use `-s` / `--silent` to suppress terminal output while keeping other export channels active.

### Numeric-only mode (`-i`)

Adding `-i` prints a bare float with no labels or formatting, one value per line:

```bash
joularcore -i        # prints total power
joularcore -c cpu -i # prints CPU power only
```

This mode is useful for stdout redirection or piping into other tools.

## CSV Files (`-f`)

Use `-f <FILE>` to write measurements to a CSV file. A header line is written once at the start, then one data row is appended every second.

### CSV Formats

The columns depend on which options are active:

**Default (no process/app/component filter)**
```
Timestamp,Total Power (W),CPU Power (W),GPU Power (W),CPU Usage (%)
1712345678,18.45,15.20,3.25,24.60
```

**Process monitoring (`-p`)**
```
Timestamp,Total Power (W),CPU Power (W),GPU Power (W),CPU Usage (%),Process Power (W)
1712345678,18.45,15.20,3.25,24.60,1.84
```

**Application monitoring (`-a`)**
```
Timestamp,Total Power (W),CPU Power (W),GPU Power (W),CPU Usage (%),App Power (W),App PIDs
1712345678,18.45,15.20,3.25,24.60,3.12,4
```

**CPU-only component filter (`-c cpu`)**
```
Timestamp,CPU Power (W)
1712345678,15.20
```

**GPU-only component filter (`-c gpu`)**
```
Timestamp,GPU Power (W)
1712345678,3.25
```

The `Timestamp` column contains a Unix epoch timestamp in seconds.

### Overwrite Mode (`-o`)

By default, rows are appended to the file. With `-o`, the file is truncated before each write so it always contains exactly one data row (plus the header). This is useful when another program is polling the file for the latest value.

## Shared-Memory Ring Buffer (`-r`)

Use `-r` or `--ringbuffer` to write power data to a shared-memory region. Any process on the same machine can read from this region without file I/O or network overhead.

### Ring Buffer Paths

| OS      | Path |
|---------|------|
| Linux   | `/dev/shm/joularcorering` |
| macOS   | `/tmp/joularcorering` |
| Windows | `Local\\JoularCoreRing` |

### Ring Buffer Layout

The ring buffer holds 5 consecutive `f64` (double-precision float) values:

| Index | Field | Description |
|:-----:|-------|-------------|
| 0 | `cpu_power` | CPU power in watts |
| 1 | `gpu_power` | GPU power in watts |
| 2 | `total_power` | Total (CPU + GPU) power in watts |
| 3 | `cpu_usage` | System CPU usage as a percentage (0–100) |
| 4 | `pid_or_app_power` | Power attributed to the monitored PID or application in watts; `0.0` if no process/app is selected |

The structure is C-compatible (repr C, packed), so it can be read directly from any language that supports memory-mapped files or shared memory.

### Multiple Ring Buffer Entries

The buffer holds 5 slots. The writer advances a head index on each update. Readers should track the head index to detect new data.

## HTTP and WebSocket API (`--api-port`)

When built with the `api` feature (on by default), Joular Core can expose power data over a local HTTP and WebSocket server. Start it with:

```bash
joularcore --api-port 8080
```

The server binds to `0.0.0.0:<PORT>` and has CORS enabled, so it can be reached from browser-based dashboards.

### Endpoints

| Endpoint | Protocol | Description |
|----------|----------|-------------|
| `/data`  | HTTP GET | Returns the latest power reading as JSON |
| `/ws`    | WebSocket | Pushes a new JSON reading every second |

### JSON Format

Both endpoints use the same JSON schema:

```json
{
  "timestamp": 1712345678,
  "cpu_power": 15.20,
  "gpu_power": 3.25,
  "total_power": 18.45,
  "cpu_usage": 24.60,
  "pid_or_app_power": 1.84
}
```

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | integer | Unix timestamp in seconds |
| `cpu_power` | float | CPU power in watts |
| `gpu_power` | float | GPU power in watts |
| `total_power` | float | Total power (CPU + GPU) in watts |
| `cpu_usage` | float | System CPU usage as a percentage (0–100) |
| `pid_or_app_power` | float | Power attributed to the monitored PID or application in watts; `0.0` if none selected |

### Example: Fetching via curl

```bash
curl http://localhost:8080/data
```

### Example: WebSocket with websocat

```bash
websocat ws://localhost:8080/ws
```

## Combining Export Channels

All export channels can be active at the same time. For example:

```bash
# CSV file + ring buffer + API, no terminal output
joularcore -s -f power.csv -r --api-port 8080
```

```bash
# Process monitoring, CSV, and API
joularcore -p 1234 -f power.csv --api-port 8080
```
