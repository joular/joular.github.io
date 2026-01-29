# Command Line Options

To use Joular Core, just run the command ```joularcore``` or ```joularcore.exe```.

The following options are available:
- `-p`, `--pid <PID>`              : Monitor a specific process by its PID
- `-a`, `--app <APP>`              : Monitor a specific application by its name (all PIDs)
- `-f`, `--file <FILE>`            : Write output to CSV file
- `-c`, `--component <COMPONENT>`  : Monitor only specific component (values: `cpu`, `gpu`)
- `-i`, `--numeric`                : Output only numeric value (no formatting)
- `-o`, `--overwrite`              : Overwrite file instead of append (only with -f)
- `-g`, `--gui`                    : Start GUI interface
- `-r`, `--ringbuffer`             : Send power data to ring buffer
- `-h`, `--help`                   : Print help
- `-V`, `--version`                : Print version

To use Joular Core inside virtual machines, two environment variables must be set depending on whether you want to monitor the CPU, the GPU, or both:
- `VM_CPU_POWER_FILE`: Path to the CPU power data file
- `VM_CPU_POWER_FORMAT`: Format of the CPU power data (`joularcore`  or `powerjoular` or `watts`, default is `watts` if not set)
- `VM_GPU_POWER_FILE`: Path to the GPU power data file
- `VM_GPU_POWER_FORMAT`: Format of the GPU power data (`joularcore` or `powerjoular` or `watts`, default is `watts` if not set)

On SBC (Raspberry Pi, Asus TinkerBoard), Joular Core uses our own regression models to calculate CPU power consumption.
These models are hard-coded in the code, but you can use your own models by supplying a JSON file with the model detail with the `SBC_POWER_MODEL_JSON` environmental variable.
The format needs to follow the one used in our [Power Models Database](https://github.com/joular/powermodels).