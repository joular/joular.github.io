# Virtual Machines

PowerJoular also works inside virtual machines, with a Linux, Windows or macOS guest.
All its functionalities (such as monitoring a PID or an application) work the same inside a virtual machine as with bare metal installation.

In virtual machines, PowerJoular in the guest OS needs to get the power consumption of the virtual machine instance itself.
This can only be done by installing on the host OS, a power monitoring tool (such as PowerJoular itself or other ones), and monitoring the power consumption of the specific guest virtual machine process.

The power data of the VM process need to be written to a shared file between the host and the guest.
Inside the guest, PowerJoular will read this file continuously and use the reported power value as the CPU power of the entire virtual machine.
A GPU passed through to the virtual machine is still measured directly, as on bare metal.

PowerJoular is agnostic to what power tools are installed in the host and can work with any available tool that is capable of monitoring the VM process.

The file can have one of two formats, given with the ```-s``` option:
- ```powerjoular```: the 3 columns CSV file that ```-o``` writes for a monitored process (timestamp, CPU usage and power), where the power is the third column.
- ```watts```: a file holding the power in watts and nothing else.

PowerJoular reads the first line of the file.
When PowerJoular starts, the file must already be there and hold a power value, otherwise PowerJoular stops with an error, so start the host side first.
Afterwards, a negative power in the file, such as the ```-1.0000``` PowerJoular writes for a process it cannot read, is ignored: the last power read is kept, as it is when the file cannot be read at all.

## Use case example with PowerJoular on host and guest

A use case example is using PowerJoular on both the host and the guest.

### In the host OS

- Install PowerJoular
- Run PowerJoular while specifying the PID of the virtual machine of the guest OS, and writing the power data in a CSV file in overwrite mode.
- For instance, you can run PowerJoular with the following command: ```powerjoular -p $VM_PID -o /home/vm/vm.csv```
- This writes two files: ```/home/vm/vm.csv``` with the power of the whole host, and ```/home/vm/vm.csv-$VM_PID.csv``` with the power of the virtual machine process.
- Share the ```/home/vm/vm.csv-$VM_PID.csv``` between the host OS and the guest OS (read-only for the guest, as PowerJoular on the host usually runs as root)

### In the guest OS

- Install PowerJoular
- Share the ```/home/vm/vm.csv-$VM_PID.csv``` between the host OS and the guest OS, potentially having a different path of the file inside the guest. For instance, ```/opt/vm/vm.csv```
- Run PowerJoular with the -m and -s options to specify the file and the power data format.
- For example, in our use case example: ```powerjoular -m /opt/vm/vm.csv -s powerjoular```
- You can add other options for your needs, such as -p or -a to monitor a specific process or application inside your virtual machine.

## Use case example with another tool on the host

Have the tool on the host write the power of the virtual machine in watts, and nothing else, to the shared file.
Then read that file in the guest with the ```watts``` format: ```powerjoular -m /opt/vm/vm-power.txt -s watts```