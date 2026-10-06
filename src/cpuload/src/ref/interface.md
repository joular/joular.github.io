# Library Interface

CPU Load has the same interface in Ada and in C: take a sample, compare two samples, and the version of the library.

## Ada

The whole interface is the `CPU_Load` package.

### Types

| Type | Description |
|------|-------------|
| `Process_ID` | The ID of a process (a `Natural`). |
| `Sample` | One reading of the CPU counters, in microseconds: `Busy` (machine time not spent idle, summed over every core), `Total` (machine time there was to spend: the elapsed time times the number of cores, `0` if the sample could not be taken) and `Used` (CPU time of the process or application sampled, `0` for the system alone, negative if it could not be read), all three `Integer_64`. |

### Subprograms

| Subprogram | Description |
|------------|-------------|
| `function Take return Sample` | Sample the system alone. |
| `function Take (PID : in Process_ID) return Sample` | Sample the system and one process. `0` samples the system alone. |
| `function Take (App : in String) return Sample` | Sample the system and every process of an application. `App` is the program's name without its folder: the name must match exactly, but upper or lower case does not matter (see [Supported Platforms](../guide/supported_platforms.md)). `""` samples the system alone. |
| `function Take (PID : in Process_ID; Machine : in Sample) return Sample` | The same as `Take (PID)`, measured against a machine sample already taken. |
| `function Take (App : in String; Machine : in Sample) return Sample` | The same as `Take (App)`, measured against a machine sample already taken. |
| `function System_Usage (Before, After : in Sample) return Long_Float` | CPU load of the whole machine between two samples, from `0.0` to `1.0` (samples of any kind: system, process or application). |
| `function Process_Usage (Before, After : in Sample) return Long_Float` | CPU load of the process or application between two samples, from `0.0` to `1.0`, and negative if it could not be read at all. |
| `function Version return String` | The version of the library. |

The two `Take` functions with a `Machine` sample measure everything over exactly the same period of time, and read the machine's counters once:

```ada
Machine := Take;
Ours := Take (Our_PID, Machine);
Theirs := Take ("firefox", Machine);
```

## C

The C declarations are in [include/cpuload.h](https://github.com/joular/cpuload/blob/HEAD/include/cpuload.h). Use them with the relocatable (shared) library, which starts itself up when loaded: no other initialization call is needed.

### Types

```c
typedef struct cpuload_sample {
    int64_t busy;   /* machine time not spent idle, added up over every core */
    int64_t total;  /* machine time altogether, idle included, added up over every core */
    int64_t used;   /* CPU time of what was sampled, 0 for a sample of the system, -1 if it could not be read */
} cpuload_sample;
```

A `total` of 0 means the sample could not be taken at all.

### Functions

| Function | Description |
|----------|-------------|
| `void cpuload_take_system(cpuload_sample *out)` | Same as `Take`: a sample of the whole system. |
| `void cpuload_take_pid(unsigned int pid, cpuload_sample *out)` | Same as `Take (PID)`. `0` samples the system alone, and a `pid` too large to be any process (e.g. a negative `pid_t` turned unsigned) gives a `used` of `-1`. |
| `void cpuload_take_app(const char *app, cpuload_sample *out)` | Same as `Take (App)`. `NULL` or `""` samples the system alone. |
| `void cpuload_take_pid_with(unsigned int pid, const cpuload_sample *machine, cpuload_sample *out)` | Same as `Take (PID, Machine)`. |
| `void cpuload_take_app_with(const char *app, const cpuload_sample *machine, cpuload_sample *out)` | Same as `Take (App, Machine)`. |
| `double cpuload_system_usage(const cpuload_sample *before, const cpuload_sample *after)` | Same as `System_Usage`. |
| `double cpuload_process_usage(const cpuload_sample *before, const cpuload_sample *after)` | Same as `Process_Usage`. |
| `const char *cpuload_version(void)` | Same as `Version`. The string is owned by the library, so do not free it. |

A `NULL` `machine` counts as a sample with a `total` of 0: `cpuload_system_usage` then gives `0.0`, and so does `cpuload_process_usage` unless the process could not be read.
A `NULL` `before` or `after` gives `0.0`, and a `NULL` `out` does nothing.

## Constraints

- The library keeps no state. Linked statically into an Ada program, it can be called from any number of tasks.
- The shared library must be called from one thread at a time, whatever the language: it runs on an Ada runtime without tasking, which has a single working stack for the whole process. A sample is a plain record either way, so it can be taken in one thread and compared in another.
- Sample about a second apart (see [Reading the Numbers](./numbers.md)).