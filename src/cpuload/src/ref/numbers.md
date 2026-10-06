# Reading the Numbers

A sample holds the CPU counters of one moment, and the load is what changed between two samples.

## Units

Every counter in a `Sample` is in **microseconds**, on every OS.

Both loads, `System_Usage` and `Process_Usage`, run from `0.0` to `1.0` and are a share of the **whole machine**, not of one core: a process using all of one core of an eight core machine reads `0.125`, not `1.0`.
A core here is a CPU as the OS counts them, so a hardware thread on a processor with Hyper-Threading or SMT. To get a load as a share of one of them, multiply it by their number.

## Negative and Zero

**A negative load means the process could not be read at all**: not running (a process that has ended but is not yet cleaned up by the system counts as not running), or not allowed to get the information needed. That is not the same as `0.0`, which means no CPU time used at all.

A process is read in both samples, and the load is negative when it could not be read in one of them.

## Applications

For an application, a process that could not be read is left out of the sum, so the figure is short by what it used. The answer is negative only when some of the application's processes are running and none of them would say anything at all, or when the processes could not be listed. An application that is not running reads `0.0`.

On Windows, the processes of other users cannot even be named, so they are never part of an application: an application made only of them reads `0.0`.

An application is every process running its program when the sample is taken. A process of the application that ends between two samples takes all of its time out of the second one, so that stretch reads low, or `0.0`.

## Samples That Cannot Be Compared

- A sample with a total of 0 could not be taken at all (the counters of the machine could not be read). The loads computed with it are `0.0`, or negative when the process could not be read.
- Two samples given the wrong way round give `0.0`.

A total of 0 on the very first sample is worth checking: it is what a library built for another OS gives (see [Compilation](./compilation.md)), and without the check a program would print 0% forever and look idle. The example programs do this check.

## How Often to Sample

Sample about a second apart: Linux counts a process in 10 ms units, Windows in about 15 ms, and FreeBSD the machine in ticks of about 8 ms, too coarse for shorter waits. macOS counts a process in nanoseconds, and reads it well below a second, though the machine itself is still counted in 10 ms ticks.

On macOS, the counters of the machine are 32 bits, counting 100 times per second for each busy core: on a 10 core machine kept busy, they can wrap after about 50 days. Only the load measured across the wrap shows 0%.