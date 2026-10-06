# Configuration Properties

Joular Code for Java can be configured by modifying the `joularcodejava.properties` file, read from:

1. The path given with the `-Djoularcodejava.properties=<path>` JVM property
2. Otherwise, `joularcodejava.properties` in the folder where you run the Java command

A missing file, or a property left empty, means the default value is used.
A path given with `-Djoularcodejava.properties` where there is no file also means the defaults are used: the working folder is not searched then.

The configuration is read once, when the agent starts.
When the agent is attached to a running JVM, both are those of that JVM: its own `-Djoularcodejava.properties`, and its working folder.

[joularcodejava.properties.example](https://github.com/joular/joularcode-java/blob/HEAD/joularcodejava.properties.example) is a commented example you can start from.

The following properties are available:

- `power-source-type`: where the CPU power comes from: `auto`, `rapl`, `ringbuffer` or `vm` (default: `auto`). See [Power Sources](./power_sources.md).
- `powerjoular-ringbuffer-path`: the path to the PowerJoular ring buffer (default: `/dev/shm/powerjoular` on Linux, `/tmp/powerjoular` on macOS and FreeBSD, `%PROGRAMDATA%\powerjoular` on Windows).
- `vm-power-file`: the path for the power consumption of the virtual machine. Inside a virtual machine, indicate the file containing the power consumption of the VM (which is usually a file in the host that is shared with the guest). It must be set when `power-source-type` is `vm` (default: empty). See [Virtual Machines](./vm.md).
- `vm-power-format`: power format of the shared VM power file: `watts` (a file containing one value, the power consumption of the VM), or `powerjoular` (the row [PowerJoular](https://github.com/joular/powerjoular) writes with `-o` in the host, containing 3 columns: timestamp, CPU utilization of the VM and CPU power of the VM) (default: `powerjoular`).
- `stack-monitoring-sample-rate`: the sample rate (in milliseconds) for the agent to monitor methods usage by calling the JVM stack, from 1 to 1000 (default: `10`). A higher value means less accurate energy data, and a lower value means a higher overhead of the agent. Below 5 milliseconds, a warning is shown, as sampling can noticeably slow the application down. A value that is not a number from 1 to 1000 is replaced by the default, with a warning.
- `results-path`: the folder where the CSV files are written (default: `joular-code-java-results`). A relative path starts from the folder where the Java command runs.
- `methods-filtering-prefix`: list of package or class name prefixes, separated by commas, which will be used to filter the methods of your application, for example `com.example,org.myapp` (default: empty, no filter). See [Generated Files](./generated_files.md) for explanations.

The values of `power-source-type` and `vm-power-format` can be written in upper or lower case.
An unknown value for either of them is not replaced by the default: Joular Code for Java says why on the standard error, and the monitoring is off (the application still runs as usual).
The same happens with `vm` without `vm-power-file`, or `rapl` on another OS than Linux.

## Method filtering

The `methods-filtering-prefix` property controls which methods appear in the `methods-power-app.csv` file:

- If left empty, all methods (except those of the agent's own thread) appear in both result files, which then hold the same data.
- When set, each branch is written to `methods-power-app.csv` with only its methods that match a prefix, and the branches that end up the same are added up. So the energy of the methods that do not match (e.g., the JDK ones) and that are called by a matched method goes to the matched method. A branch with no matching method is left out of this file.
- `methods-power-all.csv` always contains every observed branch, regardless of this setting.

A method matches when its full name (`package.Class.method`) starts with one of the prefixes.
The prefix is compared as plain text, not as a package: `com.example` also matches `com.examples`. Use `com.example.` to match the `com.example` package and its sub-packages, but not `com.examples`.

## When the application runs as root

Give the configuration file with `-Djoularcodejava.properties`, rather than leaving it in a folder others can write to, since the file sets where the results are written.