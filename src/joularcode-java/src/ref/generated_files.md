# Generated Files

Joular Code for Java will generate two CSV files, five with `lines-monitoring=true`, in a folder per run inside the `joular-code-java-results` folder (which can be changed with the `results-path` property):

| File | Contents |
|---|---|
| `methods-power-all.csv` | Power and energy of all observed execution branches, including the JDK ones |
| `methods-power-app.csv` | Power and energy of the execution branches of your application, according to the `methods-filtering-prefix` setting |
| `lines-power-all.csv` | As `methods-power-all.csv`, with the line of each method (only with `lines-monitoring=true`) |
| `lines-power-app.csv` | As `methods-power-app.csv`, with the line of each method (only with `lines-monitoring=true`) |
| `lines-power-app-self.csv` | Power and energy of each line of your application itself, added up over the branches that reach it (only with `lines-monitoring=true`) |

At the end of every monitoring cycle (about 1 second), one row is added to each file for every execution branch that consumed power during the cycle.
Branches with a power of zero are not written.
With `aggregate=true`, the files get their rows once, when the JVM stops (see [CSV format](#csv-format)).

Each run has its own folder, `<pid>-<yyyyMMdd-HHmmss>`, named after the process id of the JVM and the time the agent started, so runs never mix.
When that folder is already there, `-2`, `-3`... is added to the name: JVMs in containers that share a results folder often have the same process id, and may start in the same second.
At startup, Joular Code for Java says which folder it writes to.

Joular Code for Java never follows a symbolic link when it opens its result files, since it may run as root, and the results folder may be one every user can write to.

## The filtered file

The second file is not just a subset of the first one, but rather a recalculation done by Joular Code for Java to provide accurate data: the methods that start with the filtered prefixes are allocated the power or energy of the methods they call that do not match a prefix (the JDK's, but also those of libraries and frameworks).

For example, if `Package1.MethodA` calls `java.io.PrintStream.println` to print some text to a terminal, then we calculate:

- In the first file, the power or energy of the branch ending in `println` (through `MethodA`), separately from the branch ending in `MethodA`. The latter won't include the power consumed by `println`.
- In the second file, if we filter methods from `Package1`, then the power consumption of `println` will be added to `MethodA` power consumption, and the file will only provide power or energy of `Package1` methods.

We manage to do this by analyzing the stack trace of all running threads at runtime.

In the filtered file, a branch only has the methods that match a prefix, and the branches that end up the same are added up into one row.
The samples with no matching method at all are left out of this file, so its totals are lower than those of the first file.
When `methods-filtering-prefix` is empty, both files hold the same data.

## Lines of code

With `lines-monitoring=true`, the energy is also measured by line of code, and three more files are written:

- `lines-power-all.csv` and `lines-power-app.csv` have the same rows as the methods files, with the same filtering, but each method of a branch comes with its line, as the JVM stack trace writes it: `com.example.Main.main(Main.java:12);com.example.Service.process(Service.java:120)`. The line of the newest method is the line being executed, and the line of every other method is the line calling the next one.
- `lines-power-app-self.csv` has one row per line of your application per cycle, whatever it was called from: the newest method of each branch that matches `methods-filtering-prefix` (the newest of all without a filter), so the line that was running, or that called into the JDK. It is the total of a line itself, where the two other files split it by execution branch.

The methods files stay the same.

- Thread dumps are taken at safepoints, which the JVM places on loop back edges, calls and returns, so the samples gather on those lines: the energy per line is accurate to the loop or the block of statements, not to the single line.
- Classes need line numbers in their debug information, which `javac` and Maven include by default. Without them a method is written `(Service.java)`, and `(Unknown Source)` when the file is unknown too. Native methods are written `(Native Method)`.
- Lines give more distinct branches than methods, so the files grow faster.

## CSV format

All files share the same format:

```
Timestamp,branch,power_watts,energy_joules,interval_seconds,coverage
```

| Column | Type | Description |
|---|---|---|
| `Timestamp` | double (s) | Unix time in seconds, with milliseconds, at the end of the monitoring cycle |
| `branch` | string | The call chain, from the oldest to the newest method, separated by semicolons (e.g., `com.example.Main.run;com.example.Service.process`) |
| `power_watts` | double | Estimated power consumed by this branch during the cycle (W) |
| `energy_joules` | double | Energy = power_watts × interval_seconds (J) |
| `interval_seconds` | double | Duration of the monitoring cycle (s), usually about 1.0 |
| `coverage` | double | The share of the JVM's CPU time during the cycle that belonged to threads Joular Code for Java actually sampled, from 0.0 to 1.0 |

A branch with a comma, a double quote or a line break in it is written between double quotes (standard CSV), so read the files with a CSV parser.

To get the total energy of a branch over the whole execution, add up its `energy_joules` values, or set `aggregate=true`.

With `aggregate=true`, each branch has one row, written when the JVM stops: `power_watts` is its average over the run, `energy_joules` its total, `interval_seconds` how long the run was monitored, leaving out the cycles whose power could not be read, and `coverage` the average over those same cycles.

### Example output

```
Timestamp,branch,power_watts,energy_joules,interval_seconds,coverage
1746000000.000,com.example.Main.main;com.example.Worker.compute,2.341500000,2.341500000,1.000000000,1.0000
1746000001.000,com.example.Main.main;com.example.Worker.compute,2.158300000,2.158300000,1.000000000,0.9974
```

### What `coverage` means

Power is split between threads by the CPU time each one used, and the denominator is every thread that used CPU, not only the ones that were caught in a sample.
Power drawn by a thread that was never sampled is therefore left unattributed, rather than shared out over the threads it did see.

`coverage` says how much of the JVM's CPU time is represented: at `1.0` everything was accounted for, and at `0.6` only 60% of what the JVM consumed is attributed to the observed threads at that timestamp.
Below `1.0`, the power reported per branch is a lower bound. When `coverage` drops below 0.5, Joular Code for Java shows a warning, once per run.

A sampled thread that ends during a cycle is charged up to the sample that last saw it, because the JVM stops reporting a thread's CPU time once it has ended.
A thread that ends without ever being sampled is not counted here. Its CPU time is still part of the JVM's, so its power is spread over the threads still alive at the end of the cycle (like the power of the garbage collector), and `coverage` does not drop because of it.