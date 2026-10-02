# Generated Files

Joular Code for Java will generate two CSV files, and will create these files in the `joular-code-java-results` folder (which can be changed with the `results-path` property):

| File | Contents |
|---|---|
| `methods-power-all.csv` | Power and energy of all observed execution branches, including the JDK ones |
| `methods-power-app.csv` | Power and energy of the execution branches of your application, according to the `methods-filtering-prefix` setting |

At the end of every monitoring cycle (about 1 second), one row is added to each file for every execution branch that consumed power during the cycle.
Branches with a power of zero are not written.

The files are added to, and not overwritten: running the application again with the same results folder adds its rows after the previous ones.
Use another `results-path`, or move the files away, to keep each run on its own.

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

## CSV format

Both files share the same format:

```
timestamp,branch,power_watts,energy_joules,interval_seconds,coverage
```

| Column | Type | Description |
|---|---|---|
| `timestamp` | long (ms) | Unix timestamp in milliseconds at the end of the monitoring cycle |
| `branch` | string | The call chain, from the oldest to the newest method, separated by semicolons (e.g., `com.example.Main.run;com.example.Service.process`) |
| `power_watts` | double | Estimated power consumed by this branch during the cycle (W) |
| `energy_joules` | double | Energy = power_watts × interval_seconds (J) |
| `interval_seconds` | double | Duration of the monitoring cycle (s), usually about 1.0 |
| `coverage` | double | The share of the JVM's CPU time during the cycle that belonged to threads Joular Code for Java actually sampled, from 0.0 to 1.0 |

A branch with a comma, a double quote or a line break in it is written between double quotes (standard CSV), so read the files with a CSV parser.

To get the total energy of a branch over the whole execution, add up its `energy_joules` values.

### Example output

```
timestamp,branch,power_watts,energy_joules,interval_seconds,coverage
1746000000000,com.example.Main.main;com.example.Worker.compute,2.341500000,2.341500000,1.000000000,1.0000
1746000001000,com.example.Main.main;com.example.Worker.compute,2.158300000,2.158300000,1.000000000,0.9974
```

### What `coverage` means

Power is split between threads by the CPU time each one used, and the denominator is every thread that used CPU, not only the ones that were caught in a sample.
Power drawn by a thread that was never sampled is therefore left unattributed, rather than shared out over the threads it did see.

`coverage` says how much of the JVM's CPU time is represented: at `1.0` everything was accounted for, and at `0.6` only 60% of what the JVM consumed is attributed to the observed threads at that timestamp.
Below `1.0`, the power reported per branch is a lower bound. When `coverage` drops below 0.5, Joular Code for Java shows a warning (once, until it rises again).

Threads that end during a cycle, even ones started before it, are not counted here, because the JVM stops reporting a thread's CPU time once it has ended.
Their CPU time is still part of the JVM's, so their power is spread over the threads still alive at the end of the cycle (like the power of the garbage collector), and `coverage` does not drop because of them.