# Overview

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue)](https://www.gnu.org/licenses/gpl-3.0)
[![Ada](https://img.shields.io/badge/Made%20with-Ada-blue)](https://www.adaic.org)

![PowerJoular Logo](powerjoular.png)

PowerJoular is a command line software to monitor, in real time, the power consumption of software and hardware components, across multiple platforms and virtual machines.

PowerJoular is part of the <a href="https://www.noureddine.org/research/joular/"><img src="https://raw.githubusercontent.com/joular/.github/main/profile/joular.png" alt="Joular Project" width="64" /></a> project.

PowerJoular can monitor the power consumption, in real time, of the CPU and the GPU in general, and also for a specific application (through its name or PID).
It works on Linux, Windows, macOS and FreeBSD, on multiple architectures (x86_64, ARM) and devices (computers, servers, Macs, Raspberry Pi, etc.).

The official website of PowerJoular is: [https://www.noureddine.org/research/joular/powerjoular](https://www.noureddine.org/research/joular/powerjoular).

## Features

- Monitor power consumption of CPU and GPU of PC/servers
- Monitor power consumption of Macs, Apple Silicon and Intel
- Monitor power consumption of Raspberry Pi and Asus Tinker Board devices
- Monitor power consumption inside virtual machines
- Monitor power consumption of individual applications or processes
- Expose power consumption to the terminal, CSV files, and a shared memory ring buffer
- Provides a systemd service (daemon) to continuously monitor power of devices on Linux
- Low overhead (written in Ada and compiled to native code, one binary with nothing to install alongside it)

## What changed in version 2

Version 2 does the measuring using two Ada libraries we developed, [Joular Core](https://github.com/joular/joularcore) and [CPU Load](https://github.com/joular/cpuload), instead of its own code.
This also brings Windows, macOS and FreeBSD support, and AMD graphic cards (Nvidia cards are now read through NVML instead of ```nvidia-smi```).

- **New**: ```-r``` writes the power data to a shared memory ring buffer.
- **Removed**: ```-k```, which measured a process from its threads. It was experimental, and the process readings no longer need it.
- **Removed**: ```-l```, which picked the linear power models of the single-board computers. The polynomial models are now the only ones used, as they are much more accurate and their overhead is minimal.
- **Removed**: ```-c```, which wrote the timestamps in milliseconds.

Other main differences:

- The CSV files start with a Unix timestamp in seconds instead of a date and time, and hold four digits after the dot instead of fourteen. The columns and the header have been renamed, with Timestamp and CPU Usage.
- The terminal no longer shows the difference of power consumption from the last metric, and Ctrl+C no longer prints the total energy consumed.
- On Linux, the CPU power is the RAPL package domain only. Version 1 used the Psys domain when there was one, or added the DRAM domain to the package, so the values can be lower than with version 1.
- The energy a RAPL processor reports is divided by how long the cycle actually took, rather than assumed to be exactly one second. On a busy machine where a cycle takes longer than one second, the watts reported are now the watts drawn.
- A file that cannot be written to, or a ring buffer that cannot be opened, is reported once and the monitoring continues. A power source that stops answering reads 0 until it answers again.
- The systemd service writes to ```/run/powerjoular/powerjoular-service.csv``` instead of ```/tmp```, and runs with reduced privileges.

## Cite this work

To cite our work in a research paper, please cite our paper in the 18th International Conference on Intelligent Environments (IE2022).

- **PowerJoular and JoularJX: Multi-Platform Software Power Monitoring Tools**. Adel Noureddine. In the 18th International Conference on Intelligent Environments (IE2022). Biarritz, France, 2022.

```
@inproceedings{noureddine-ie-2022,
  title = {PowerJoular and JoularJX: Multi-Platform Software Power Monitoring Tools},
  author = {Noureddine, Adel},
  booktitle = {18th International Conference on Intelligent Environments (IE2022)},
  address = {Biarritz, France},
  year = {2022},
  month = {Jun},
  keywords = {Power Monitoring; Measurement; Power Consumption; Energy Analysis}
}
```

## License

PowerJoular is licensed under the GNU GPL 3 license only (GPL-3.0-only).

Copyright (c) 2020-2026, Adel Noureddine.
All rights reserved. This program and the accompanying materials are made available under the terms of the GNU General Public License v3.0 only (GPL-3.0-only) which accompanies this distribution, and is available at: https://www.gnu.org/licenses/gpl-3.0.en.html

Author : [Adel Noureddine](https://www.noureddine.org/)