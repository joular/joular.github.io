# Exporting power and energy measurements

Joular Core can export the power and energy measurements through multiple interfaces and approaches.
All the following export approaches works on all supported platforms and operating systems.

## Write on terminal and stdout redirection

By default, Joular Core will write data to the terminal.
However, this data could be friendly for stdout redirection by using the `-i` argument: `joularcore -i` or `joularcore.exe -i`

This will output just a float on the terminal that can be redirected anywhere.
By default, the float will be the total power consumption.
If combined with other arguments, such as the `-c` argument to select a specific component, it will provide only that specific component.

## CSV files

Joular Core can export data to a CSV file with the `-f` argument, while supplying a filename or path.

```joularcore -f myfile.csv``` will write a CSV with a header and a new line for every second of power monitoring.
The format of the CSV file is the following:
```Timestamp,Total Power (W), CPU Power (W),GPU Power (W),CPU Usage (%),Process Power (W)```

## Inter-process communication with a shared memory ring buffer

Joular Core can also write data to a shared memory ring buffer, on all OSes, when executed with the `-r` argument: `joularcore -r` or `joularcore.exe -r`
This allows much faster communication and sharing power data between Joular Core and other programs.

Default ring buffer paths are:
- `/dev/shm/joularcorering` on Linux
- `/tmp/joularcorering` on macOS
- `Local\\JoularCoreRing` on Windows

The ring buffer have a size of 5 data stuctures with the following values (by order): `CPU power`, `GPU power`, `Total power`, `CPU usage`, and `PID or APP power`.
Unavailable values, such as when monitoring entire CPU and not a specific application, will be at 0.

## HTTP and WebSockets API

Joular Core can expose power data through an HTTP server or WebSockets.
To use this API, you need to enable the `api` feature when building Joular Core.

To run Joular Core and export data through the API, use the `--api-port` option to specify the port for the HTTP and WebSockets server endpoint. For example:
```bash
joularcore --api-port 8080
```

The API will expose the following endpoints:
- `/data`: HTTP endpoint to get the latest power data
- `/ws`: WebSockets endpoint for real-time power data

The API exposes data in JSON format:
- `cpu_power`: CPU power in Watts
- `gpu_power`: GPU power in Watts
- `total_power`: Total power in Watts
- `cpu_usage`: CPU usage in percentage
- `pid_or_app_power`: PID or application power in Watts (0 if option not selected)