# Integration with Systems and Tools

Joular Core is designed to be embedded in larger workflows. It exposes power data through multiple channels — terminal, CSV files, a shared-memory ring buffer, and an HTTP/WebSocket API — so it can feed into whatever downstream system you are working with.

See [Exporting Power Data](./exports.md) for the technical details of each channel. This page focuses on integration patterns.

## CI/CD Pipelines

Run Joular Core alongside your build or test suite to measure the energy cost of a workload.

**Example: measure energy consumed by a test run**

```bash
# Start Joular Core silently, writing to CSV
joularcore -s -f build_power.csv &
JOULAR_PID=$!

# Run your workload
cargo test

# Stop monitoring
kill $JOULAR_PID
```

The CSV file will contain one row per second of the test run. Sum the power column and multiply by the sample interval (1 second) to get total energy in joules.

## Dashboards and Monitoring Stacks

The HTTP/WebSocket API makes it straightforward to feed power data into any dashboard.

**Example: stream to a Prometheus / Grafana stack via a scraper**

```bash
joularcore --api-port 9001 -s &
```

A small scraper can poll `GET http://localhost:9001/data` on each Prometheus scrape interval and expose the fields as gauges.

**Example: real-time browser dashboard**

Connect a browser-based dashboard directly to the WebSocket endpoint:
```javascript
const ws = new WebSocket("ws://localhost:8080/ws");
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  updateChart(data.timestamp, data.total_power, data.cpu_power, data.gpu_power);
};
```

## IDE and Profiler Integration

Joular Core can run as a background service alongside an IDE or profiler. The ring buffer channel provides the lowest latency option for IPC:

1. Start Joular Core with `joularcore -r -s`
2. In your IDE plugin or profiler, memory-map the ring buffer path (`/dev/shm/joularcorering` on Linux, `/tmp/joularcorering` on macOS, `Local\\JoularCoreRing` on Windows)
3. Read the 5 `f64` values: CPU power, GPU power, total power, CPU usage, PID/app power

The ring buffer is updated every second and is safe to read from multiple processes simultaneously.

## Remote Power Databases and Time-Series Storage

Joular Core does not push to remote databases directly, but the CSV and API outputs make it easy to bridge to any time-series store.

**Example: push to InfluxDB with a small shell loop**

```bash
joularcore -s --api-port 8080 &

while true; do
  DATA=$(curl -s http://localhost:8080/data)
  TS=$(echo $DATA | jq '.timestamp')
  TOTAL=$(echo $DATA | jq '.total_power')
  curl -s -XPOST "http://influxdb:8086/write?db=energy" \
    --data-binary "power,host=$(hostname) total=${TOTAL} ${TS}000000000"
  sleep 1
done
```

## Running as a Service

On Linux, use the included systemd unit file to have Joular Core run continuously in the background without manual intervention. See [Systemd Service](./systemd.md).

On Windows and macOS, you can register Joular Core as a service using the OS-native service managers (Windows Service Manager or `launchd` on macOS). The key is to use `-s -o -f <path>` or `--api-port` so data is accessible to other programs without a terminal attached.

## Virtual Machine Monitoring

For monitoring inside VMs, see the dedicated [Virtual Machines](./vm.md) page, which covers how to bridge host-level power readings into the guest using the shared-file mechanism.
