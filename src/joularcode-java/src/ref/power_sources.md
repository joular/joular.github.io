# Power Sources

Joular Code for Java gets the CPU power of the machine from one of three sources (RAPL, the PowerJoular ring buffer, or the host of a virtual machine), chosen with the `power-source-type` property:

- `auto` (the default): RAPL on Linux, and the PowerJoular ring buffer everywhere else, including Linux machines without RAPL such as the Raspberry Pi.
- `rapl`: Linux RAPL, read directly.
- `ringbuffer`: the shared memory ring buffer PowerJoular writes with `-r`.
- `vm`: inside a virtual machine, the file the host writes the power of the virtual machine to.

With `auto`, RAPL is picked on a Linux machine that has it, even when the application is not allowed to read it.
In that case, the monitoring is off (the application still runs as usual), so either give the application read access to RAPL, or run PowerJoular with `-r` and set `power-source-type=ringbuffer`.

When the source cannot be opened, Joular Code for Java says why on the standard error, and the application carries on without monitoring.
When the power cannot be read, the cycle gets no energy, and the monitoring carries on once it can be read again.

## Linux RAPL (`rapl`)

Joular Code for Java reads the energy counters of the CPU packages directly from `/sys/class/powercap/intel-rapl:N`, on Intel and AMD processors alike.
On a server with several sockets, the packages of every socket are added up.
The sub-domains (cores, uncore, DRAM) and the `psys` domain, which covers more than the CPU, are left out.

Nothing else is needed, but the `energy_uj` files are only readable by root on most distributions. You can:
- Run the application as root
- Give its user read access to those files (for example with a udev rule)
- Run PowerJoular with `-r` as root, and set `power-source-type=ringbuffer`, so the Java application does not need root

PowerJoular reads the first package only, so on a machine with several sockets, `rapl` and `ringbuffer` do not give the same values.

```properties
power-source-type=rapl
```

## PowerJoular ring buffer (`ringbuffer`)

Joular Code for Java reads the shared memory area that [PowerJoular](https://github.com/joular/powerjoular) (version 2.0.0 or later) writes with `-r`.
PowerJoular measures the hardware through [Joular Core](https://github.com/joular/joularcore): RAPL on Linux, Windows and FreeBSD, `powermetrics` on Macs, and our regression power models on Raspberry Pi.
It runs as a process of its own, so the privileges needed to measure the hardware stay out of the Java application:
- On Linux PC/Servers and on macOS, run PowerJoular with `sudo`.
- On FreeBSD, load the `cpuctl` module (`kldload cpuctl`), then run PowerJoular with `sudo`.
- On Windows, PowerJoular needs no special rights with the Energy Meter Interface or Hubblo's RAPL driver, but needs a terminal with administrative rights with the PawnIO driver.
- On Raspberry Pi, no special rights are needed.

Each monitoring cycle ends when PowerJoular publishes a new measurement, so the stack samples and the power describe the same second.

PowerJoular may be started before or after the application, and restarted while it runs, except on Windows: a mapped file cannot be deleted there, so PowerJoular has to be started before the application, and not restarted while it runs.

The default paths of the ring buffer are:
- Linux: `/dev/shm/powerjoular`
- macOS: `/tmp/powerjoular`
- FreeBSD: `/tmp/powerjoular`
- Windows: `%PROGRAMDATA%\powerjoular`

```properties
power-source-type=ringbuffer
powerjoular-ringbuffer-path=/dev/shm/powerjoular
```

When PowerJoular publishes no measurement for two seconds, or when its latest measurement is more than five seconds old, Joular Code for Java takes PowerJoular as stopped, and no energy is attributed until it publishes again.

When PowerJoular cannot measure the CPU (for example, run without `sudo` on a machine with a graphic card), it writes a CPU power of 0: Joular Code for Java then attributes nothing and writes no rows, without a warning. Run `powerjoular -d -r` to check that the CPU can be measured.

Joular Code for Java never follows a symbolic link when it opens the ring buffer, and only opens a regular file there, since it may run as root and the default path is in a folder every user can write to.

## Virtual machine (`vm`)

The CPU of a virtual machine cannot be measured from inside it, but its host can measure the process the virtual machine runs as.
Run PowerJoular on the host for that process, share the file it writes with the virtual machine, and point Joular Code for Java at it.
Nothing else has to run in the virtual machine.

See [Virtual Machines](./vm.md) for the details.

```properties
power-source-type=vm
vm-power-file=/mnt/shared/vm-power.csv-<pid>.csv
vm-power-format=powerjoular
```