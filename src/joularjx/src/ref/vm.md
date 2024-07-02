# Virtual Machines

JoularJX also works inside virtual machines.
All its functionalities work the same inside a virtual machine as with bare metal installation.

In virtual machines, JoularJX in the guest OS needs to get the power consumption of the virtual machine instance itself.
This can only be done by installing on the host OS, a power monitoring tool (such as PowerJoular or other ones), and monitoring the power consumption of the specific guest virtual machine process.

The power data of the VM process need to be written to a shared file between the host and the guest.
Inside the guest, JoularJX will read this file continuously and use the reported power value as the power of the entire virtual machine.

JoularJX is agnostic to what power tools in installed in the host and can work with any available tool that is capable of monitoring the VM process.

## Use case example with JoularJX on guest and PowerJoular on host

A use case example is using JoularJX on the guest OS and PowerJoular on the host OS.

### In the host OS

- Install [PowerJoular](https://github.com/joular/powerjoular)
- Run PowerJoular while specifying the PID of the virtual machine of the guest OS, and writing the power data in a CSV file in overwrite mode.
- For instance, you can run PowerJoular with the following command: ```powerjoular -p $VM_PID -o /home/vm/vm.csv```
- Share the ```/home/vm/vm.csv``` between the host OS and the guest OS

## In the guest OS

- Get and compile JoularJX
- Share the ```/home/vm/vm.csv``` between the host OS and the guest OS, potentially having a different path of the file inside the guest. For instance, ```/opt/vm/vm.csv```
- Modify ```config.properties``` and set ``vm-power-path``` to the shared file ```/opt/vm/vm.csv```, and ```vm-power-format``` to the proper format (in this case to ```powerjoular```).
- Start your Java application with JoularJX agent as usual.