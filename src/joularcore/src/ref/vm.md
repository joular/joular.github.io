# Virtual Machines

Joular Core also works inside virtual machines.
All its functionalities (such as monitoring a PID or an application) work the same inside a virtual machine as with bare metal installation.

In virtual machines, Joular Core in the guest OS needs to get the power consumption of the virtual machine instance itself.
This can only be done by installing on the host OS, a power monitoring tool (such as Joular Core itself or other ones), and monitoring the power consumption of the specific guest virtual machine process.

The power data of the VM process need to be written to a shared file between the host and the guest.
Inside the guest, Joular Core will read this file continuously and use the reported power value as the power of the entire virtual machine.

Joular Core is agnostic to what power tools in installed in the host and can work with any available tool that is capable of monitoring the VM process.

For virtual machines, two environment variables must be set depending on whether you want to monitor the CPU, the GPU, or both:
- `VM_CPU_POWER_FILE`: Path to the CPU power data file
- `VM_CPU_POWER_FORMAT`: Format of the CPU power data (`joularcore`  or `powerjoular` or `watts`, default is `watts` if not set)
- `VM_GPU_POWER_FILE`: Path to the GPU power data file
- `VM_GPU_POWER_FORMAT`: Format of the GPU power data (`joularcore` or `powerjoular` or `watts`, default is `watts` if not set)

## Use case example with Joular Core on host and guest

A use case example is using Joular Core on both the host and the guest.

### In the host OS

- Install Joular Core
- Run Joular Core while specifying the PID of the virtual machine of the guest OS (or the application name), and writing the power data in a CSV file in overwrite mode.
- For instance, you can run Joular Core with the following command: ```joularcore -p $VM_PID -o /home/vm/vm.csv```
- Share the ```/home/vm/vm.csv``` between the host OS and the guest OS

### In the guest OS

- Install Joular Core
- Share the ```/home/vm/vm.csv``` between the host OS and the guest OS, potentially having a different path of the file inside the guest. For instance, ```/opt/vm/vm.csv```
- Run Joular Core after setting the needed environemental variables.
- For example, in our use case example, set `VM_CPU_POWER_FILE` to ```/opt/vm/vm.csv```, and `VM_CPU_POWER_FORMAT` to `joularcore`
- You can add other options for your needs, such as `-p` or `-a` to monitor a specific process or application inside your virtual machine.