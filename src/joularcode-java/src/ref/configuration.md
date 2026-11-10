# Configuration Properties

Joular Code for Java can be configured by modifying the `joularcodejava.properties` file, read from:

1. The path given as the agent's argument: `-javaagent:joularcodejava-<version>.jar=<path>`
2. Otherwise, the path given with the `-Djoularcodejava.properties=<path>` JVM property
3. Otherwise, `joularcodejava.properties` in the folder where you run the Java command

A missing file, or a property left empty, means the default value is used.
A path given as the agent's argument or with `-Djoularcodejava.properties` where there is no file also means the defaults are used, with a warning: the working folder is not searched then.

The configuration is read once, when the agent starts.
When the agent is attached to a running JVM, the path can be given as the agent's argument there too (see [Integration with Systems and Tools](./integration.md)). Otherwise, the other two are those of that JVM: its own `-Djoularcodejava.properties`, and its working folder.

In this file, `\` starts an escape sequence, so on Windows write paths with `/` or `\\`, for example `results-path=C:/Users/me/results`.
Written `C:\Users\me\results`, the backslashes are dropped, and a `\u` (as in `C:\users`) makes the whole file unreadable, so the defaults are used, with a warning.

[joularcodejava.properties.example](https://github.com/joular/joularcode-java/blob/HEAD/joularcodejava.properties.example) is a commented example you can start from.

The following properties are available:

- `power-source-type`: where the CPU power comes from: `auto`, `rapl`, `ringbuffer` or `vm` (default: `auto`). See [Power Sources](./power_sources.md).
- `powerjoular-ringbuffer-path`: the path to the PowerJoular ring buffer (default: `/dev/shm/powerjoular` on Linux, `/tmp/powerjoular` on macOS and FreeBSD, `%PROGRAMDATA%\powerjoular` on Windows).
- `vm-power-file`: the path for the power consumption of the virtual machine. Inside a virtual machine, indicate the file containing the power consumption of the VM (which is usually a file in the host that is shared with the guest). It must be set when `power-source-type` is `vm` (default: empty). See [Virtual Machines](./vm.md).
- `vm-power-format`: power format of the shared VM power file: `watts` (a file containing one value, the power consumption of the VM), or `powerjoular` (the row [PowerJoular](https://github.com/joular/powerjoular) writes with `-o` in the host, containing 3 columns: timestamp, CPU utilization of the VM and CPU power of the VM) (default: `powerjoular`).
- `stack-monitoring-sample-rate`: the sample rate (in milliseconds) for the agent to monitor methods usage by calling the JVM stack, from 1 to 1000 (default: `10`). A higher value means less accurate energy data, and a lower value means a higher overhead of the agent. Below 5 milliseconds, a warning is shown, as sampling can noticeably slow the application down. A value that is not a number from 1 to 1000 is replaced by the default, with a warning.
- `max-stack-depth`: how many methods of each thread's stack are kept, from the newest (default: `64`). A deeper stack loses its oldest methods, so its branch starts lower, and a smaller value makes sampling cheaper. A value that is not a positive number is replaced by the default, with a warning.
- `results-path`: the folder where the CSV files are written, in a sub-folder per run (default: `joular-code-java-results`). A relative path starts from the folder where the Java command runs. See [Generated Files](./generated_files.md).
- `methods-filtering-prefix`: list of package or class name prefixes, separated by commas, which will be used to filter the methods of your application, for example `com.example,org.myapp` (default: empty, no filter). See [Generated Files](./generated_files.md) for explanations.
- `lines-monitoring`: `true` also writes the power of each line of code, to `lines-power-all.csv`, `lines-power-app.csv` and `lines-power-app-self.csv` (default: `false`). See [Generated Files](./generated_files.md).
- `aggregate`: `true` writes one row per branch when the JVM stops, with its energy and average power over the run (the cycles whose power could not be read are left out), instead of a row per monitoring cycle (default: `false`).

The values of `power-source-type`, `vm-power-format`, `lines-monitoring` and `aggregate` can be written in upper or lower case.
A value other than `true` or `false` for `lines-monitoring` or `aggregate` is replaced by `false`, with a warning.
An unknown value for `power-source-type` or `vm-power-format` is not replaced by the default: Joular Code for Java says why on the standard error, and the monitoring is off (the application still runs as usual).
The same happens with `vm` without `vm-power-file`, or `rapl` on another OS than Linux.

## Method filtering

The `methods-filtering-prefix` property controls which methods appear in the `methods-power-app.csv` file:

- If left empty, all methods (except those of the agent's own thread) appear in both result files, which then hold the same data.
- When set, each branch is written to `methods-power-app.csv` with only its methods that match a prefix, and the branches that end up the same are added up. So the energy of the methods that do not match (e.g., the JDK ones) and that are called by a matched method goes to the matched method. A branch with no matching method is left out of this file.
- `methods-power-all.csv` always contains every observed branch, regardless of this setting.

A method matches when its full name (`package.Class.method`) starts with one of the prefixes.
The prefix is compared as plain text, not as a package: `com.example` also matches `com.examples`. Use `com.example.` to match the `com.example` package and its sub-packages, but not `com.examples`.

## When the application runs as root

Give the configuration file as the agent's argument or with `-Djoularcodejava.properties`, rather than leaving it in a folder others can write to, since the file sets where the results are written.