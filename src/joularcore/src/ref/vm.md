# Virtual Machines

Joular Core works inside virtual machines with the same feature set as on bare metal: system-wide monitoring, per-process monitoring, per-application monitoring, and all export channels.

The challenge specific to virtual machines is that the guest OS has no direct access to RAPL or other hardware power interfaces — those are managed by the hypervisor. Joular Core solves this by reading power data from a file written by a tool running on the host.

## How It Works

1. On the **host OS**, run a power monitoring tool (Joular Core itself, or any other tool) and have it continuously write the power consumption of the VM process to a file that is shared between the host and the guest.
2. On the **guest OS**, set environment variables telling Joular Core where that file is and what format it uses.
3. Joular Core in the guest reads the file every second and treats the reported value as its total CPU (or GPU) power.
4. From there, process and application attribution works normally — proportional to CPU utilisation within the guest.

Joular Core is agnostic to what tool runs on the host. Any tool that can write power data in one of the supported formats will work.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `VM_CPU_POWER_FILE` | Path (inside the guest) to the file containing CPU power data |
| `VM_CPU_POWER_FORMAT` | Format of that file: `joularcore`, `powerjoular`, or `watts` (default: `watts`) |
| `VM_GPU_POWER_FILE` | Path (inside the guest) to the file containing GPU power data |
| `VM_GPU_POWER_FORMAT` | Format of that file: `joularcore`, `powerjoular`, or `watts` (default: `watts`) |

You only need to set the variables for the components you want to monitor. If `VM_CPU_POWER_FILE` is not set, the VM feature is inactive and Joular Core falls back to whatever hardware interface is available in the guest (which may return 0 if there is none).

## Supported File Formats

### `watts`
A plain text file containing a single numeric value in watts, with no header. Updated by the host on every measurement cycle.

```
18.45
```

### `powerjoular`
A CSV file with at least three columns. The third column (index 2) contains the power value in watts.

```
...,18.45,...
```

### `joularcore`
A CSV file in Joular Core's standard output format, with a header row. The file should be written in overwrite mode (`-o`) so it always contains exactly one data row.

```
Timestamp,Total Power (W),CPU Power (W),GPU Power (W),CPU Usage (%)
1712345678,18.45,15.20,3.25,24.60
```

When using this format, Joular Core reads the `CPU Power (W)` column for CPU power and `GPU Power (W)` for GPU power.

## Step-by-Step Example: Joular Core on Both Host and Guest

This example uses Joular Core itself on the host to monitor the VM process, and Joular Core inside the guest to attribute that power to individual processes.

### Host Setup

Find the PID of the virtual machine process (for QEMU/KVM this is `qemu-system-x86_64`, for VirtualBox it is `VirtualBoxVM`, etc.).

Run Joular Core on the host, monitoring that process and writing to a shared file in overwrite mode:

```bash
joularcore -p <VM_PID> -o -f /shared/vm_power.csv -s
```

The file `/shared/vm_power.csv` must be accessible from inside the guest — mount the directory using VirtFS, a shared folder, or any other mechanism supported by your hypervisor.

### Guest Setup

Inside the guest, set the environment variables so Joular Core knows where to find the power data. Assuming the shared file is mounted at `/mnt/host/vm_power.csv` inside the guest:

```bash
export VM_CPU_POWER_FILE=/mnt/host/vm_power.csv
export VM_CPU_POWER_FORMAT=joularcore
```

Then run Joular Core as usual. You can monitor any process or application inside the guest:

```bash
joularcore -a myapp
```

Joular Core reads the total VM power from the shared file and distributes it proportionally to processes based on their CPU utilisation within the guest.

## Notes

- The shared file must be on a file system visible to both host and guest. tmpfs (`/dev/shm`) on Linux is a good choice for low-latency sharing.
- Write the file in overwrite mode (`-o`) on the host so Joular Core in the guest always sees the latest value rather than stale data from earlier rows.
- If the file is temporarily unavailable or empty, Joular Core in the guest treats power as 0 for that sample and continues.
- The `VM_CPU_POWER_FILE` and `VM_GPU_POWER_FILE` paths are interpreted inside the guest. They do not need to match the host path.
