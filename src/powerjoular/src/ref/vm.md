# Virtual Machines

PowerJoular also works inside virtual machines.
All its functionalities (such as monitoring a PID or an application) work the same inside a virtual machine as with bare metal installation.

In virtual machines, PowerJoular in the guest OS needs to get the power consumption of the virtual machine instance itself.
This can only be done by installing on the host OS, a power monitoring tool (such as PowerJoular itself or other ones), and monitoring the power consumption of the specific guest virtual machine process.

The power data of the VM process need to be written to a shared file between the host and the guest.
Inside the guest, PowerJoular will read this file continuously and use the reported power value as the power of the entire virtual machine.

PowerJoular is agnostic to what power tools in installed in the host and can work with any available tool that is capable of monitoring the VM process.

## Use case example with PowerJoular on host and guest

A use case example is using PowerJoular on both the host and the guest.

### In the host OS

- Install PowerJoular
- Run PowerJoular while specifying the PID of the virtual machine of the guest OS, and writing the power data in a CSV file in overwrite mode.
- For instance, you can run PowerJoular with the following command: ```powerjoular -p $VM_PID -o /home/vm/vm.csv```
- Share the ```/home/vm/vm.csv``` between the host OS and the guest OS

## In the guest OS

- Install PowerJoular
- Share the ```/home/vm/vm.csv``` between the host OS and the guest OS, potentially having a different path of the file inside the guest. For instance, ```/opt/vm/vm.csv```
- Run PowerJoular with the -m and -s options to specify the file and the power data format.
- For example, in our use case example: ```powerjoular -m /opt/vm/vm.csv -s powerjoular```
- You can add other options for your needs, such as -p or -a to monitor a specific process or application inside your virtual machine.