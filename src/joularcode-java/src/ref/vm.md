# Virtual Machines

Joular Code for Java also works inside virtual machines.
All its functionalities work the same inside a virtual machine as with bare metal installation.

In virtual machines, Joular Code for Java in the guest OS needs to get the power consumption of the virtual machine instance itself.
This can only be done by installing on the host OS, a power monitoring tool (such as PowerJoular or other ones), and monitoring the power consumption of the specific guest virtual machine process.

The power data of the VM process need to be written to a shared file between the host and the guest (virtiofs, 9p, a shared folder).
Inside the guest, Joular Code for Java will read this file every monitoring cycle (every second) and use the reported power value as the CPU power of the entire virtual machine.
Nothing else has to run in the virtual machine, and the host may start writing the file before or after the Java application.

Joular Code for Java is agnostic to what power tools are installed in the host and can work with any available tool that is capable of monitoring the VM process, as long as it writes the power alone, in watts (`vm-power-format=watts`), or the row PowerJoular writes.

Only the first line of the file is read, in one of the two formats given with `vm-power-format`:

- `powerjoular` (the default): the row PowerJoular rewrites every second with `-o` for a monitored process, `timestamp,cpu_usage,cpu_power`. Use `-o` on the host, not `-f`: `-f` starts the file with a header, which is refused. The file of the whole host, with five columns, is refused too, since it would charge all of the host's power to one virtual machine.
- `watts`: the power alone, in watts, written by any tool. The value is used as it is, however long ago it was written.

With the `powerjoular` format, the host writes the file every second, on its own clock.
When a cycle finds no new row, or catches the file while it is rewritten, the last row is used again, for up to two cycles.
After that, the host is taken as stopped, and no energy is attributed until it writes again.
A negative value, such as the `-1.0000` PowerJoular writes for a process it cannot read, counts as no reading.

Some shared folders cache files in the guest (9p with `cache=loose`, virtiofs with `cache=always`), which can hide the host's updates: mount the share without caching.

## Use case example with Joular Code for Java on guest and PowerJoular on host

A use case example is using Joular Code for Java on the guest OS and PowerJoular on the host OS.

### In the host OS

- Install [PowerJoular](https://github.com/joular/powerjoular)
- Run PowerJoular while specifying the PID of the virtual machine of the guest OS, and writing the power data in a CSV file in overwrite mode.
- For instance, you can run PowerJoular with the following command: `powerjoular -p $VM_PID -o /home/vm/vm.csv`
- This writes two files: `/home/vm/vm.csv` with the power of the whole host, and `/home/vm/vm.csv-$VM_PID.csv` with the power of the virtual machine process.
- Share the `/home/vm/vm.csv-$VM_PID.csv` between the host OS and the guest OS (read-only for the guest, as PowerJoular on the host usually runs as root)

### In the guest OS

- Get Joular Code for Java (download or compile it)
- Share the `/home/vm/vm.csv-$VM_PID.csv` between the host OS and the guest OS, potentially having a different path of the file inside the guest. For instance, `/opt/vm/vm.csv`
- Modify `joularcodejava.properties` and set `power-source-type` to `vm`, `vm-power-file` to the shared file `/opt/vm/vm.csv`, and `vm-power-format` to the proper format (in this case to `powerjoular`).
- Start your Java application with the Joular Code for Java agent as usual.

```properties
power-source-type=vm
vm-power-file=/opt/vm/vm.csv
vm-power-format=powerjoular
```

## Use case example with another tool on the host

Have the tool on the host write the power of the virtual machine in watts, and nothing else, to the shared file.
Then read that file in the guest with the `watts` format:

```properties
power-source-type=vm
vm-power-file=/opt/vm/vm-power.txt
vm-power-format=watts
```