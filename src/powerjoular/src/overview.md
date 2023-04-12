# Overview

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue)](https://www.gnu.org/licenses/gpl-3.0)
[![Ada](https://img.shields.io/badge/Made%20with-Ada-blue)](https://www.adaic.org)

![PowerJoular Logo](powerjoular.png)

PowerJoular is a command line software to monitor, in real time, the power consumption of software and hardware components.

PowerJoular is part of the <a href="https://www.noureddine.org/research/joular/"><img src="https://raw.githubusercontent.com/joular/.github/main/profile/joular.png" alt="Joular Project" width="64" /></a> project.

PowerJoular can monitor the power consumption, in real time, of the CPU and the GPU in general, and also for a specific application (through its name or PID).
It works on a Linux system and on multiple platforms (x86_64, ARM) and devices (computers, servers, Raspberry Pi, etc.).

The official website of PowerJoular is: [https://www.noureddine.org/research/joular/powerjoular](https://www.noureddine.org/research/joular/powerjoular).

## Features

- Monitor power consumption of CPU and GPU of PC/servers
- Monitor power consumption of individual applications or processes in GNU/Linux
- Expose power consumption to the terminal and CSV files
- Provides a systemd service (daemon) to continuously monitor power of devices
- Low overhead (written in Ada and compiled to native code)

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

Copyright (c) 2020-2023, Adel Noureddine, Université de Pau et des Pays de l'Adour.
All rights reserved. This program and the accompanying materials are made available under the terms of the GNU General Public License v3.0 only (GPL-3.0-only) which accompanies this distribution, and is available at: https://www.gnu.org/licenses/gpl-3.0.en.html

Author : [Adel Noureddine](https://www.noureddine.org/)