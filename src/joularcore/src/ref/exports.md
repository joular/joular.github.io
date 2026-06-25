# Exporting Power Data

Joular Core can send power measurements to multiple destinations. File output, the shared-memory ring buffer, and the HTTP/WebSocket API can be combined. In the current CLI, `-f` switches the main writer from the live terminal display to the selected file.

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

Use `-s` / `--silent` to suppress terminal output while keeping file, ring buffer, and API output active.

### Numeric-only mode (`-i`)

Adding `-i` prints a bare float with no labels or formatting, one value per line:

```bash
joularcore -i        # prints total power
joularcore -c cpu -i # prints CPU power only
```

This mode is useful for stdout redirection or piping into other tools. When combined with `-f`, the file receives the same numeric-only values instead of CSV rows.

## CSV Files (`-f`)

Use `-f <FILE>` to write measurements to a file. Without `-i`, this is CSV output. In append mode, a header line is written once at startup, then one data row is appended every second.

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

By default, rows are appended to the file. With `-o`, the file is truncated before each write so it contains only the latest data row. No CSV header is rewritten in overwrite mode. This is useful when another program is polling the file for the latest value.

## Shared-Memory Ring Buffer (`-r`)

Use `-r` or `--ringbuffer` to write power data to a shared-memory region. Any process on the same machine can read from this region without file I/O or network overhead.

### Ring Buffer Paths

| OS      | Path |
|---------|------|
| Linux   | `/dev/shm/joularcorering` |
| macOS   | `/tmp/joularcorering` |
| Windows | `Local\\JoularCoreRing` |

### Ring Buffer Layout

The shared memory region starts with an 8-byte native-endian `u64` head counter, followed by 5 slots. Each slot contains one C-compatible `RingBufferStruct`:

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | `u64` | Unix timestamp in seconds |
| `cpu_power` | `f64` | CPU power in watts |
| `gpu_power` | `f64` | GPU power in watts |
| `total_power` | `f64` | Total (CPU + GPU) power in watts |
| `cpu_usage` | `f64` | System CPU usage as a percentage (0–100) |
| `pid_app_power` | `f64` | Power attributed to the monitored PID or application in watts; `0.0` if no process/app is selected |

The structure uses Rust's `repr(C)` layout, so consumers should read it using the platform's native C layout and native endianness.

### Multiple Ring Buffer Entries

The writer stores the next sample in `head % 5`, then increments the head counter. Readers should track the head index to detect new data and use the entry `timestamp` to detect stale samples.

## HTTP and WebSocket API (`--api-port`)

When built with the `api` feature (on by default), Joular Core can expose power data over a local HTTP and WebSocket server. Start it with:

```bash
joularcore --api-port 8080
```

The server binds to `127.0.0.1:<PORT>`. By default, browser CORS access is allowed only from `http://127.0.0.1:<PORT>` and `http://localhost:<PORT>`.

To allow another browser origin, pass `--api-allowed-origin <ORIGIN>` along with `--api-port`. The flag is repeatable, and `--api-allowed-origin "*"` allows any origin.

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

File output, the ring buffer, and the API can be active at the same time. For example:

```bash
# File output + ring buffer + API, no terminal output
joularcore -s -f power.csv -r --api-port 8080
```

```bash
# Process monitoring, CSV, and API
joularcore -p 1234 -f power.csv --api-port 8080
```
