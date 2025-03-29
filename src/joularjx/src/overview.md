# Overview

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue)](https://www.gnu.org/licenses/gpl-3.0)
[![Java](https://img.shields.io/badge/Made%20with-Java-orange)](https://openjdk.java.net)

![JoularJX Logo](joularjx.png)

JoularJX is part of the <a href="https://www.noureddine.org/research/joular/"><img src="https://raw.githubusercontent.com/joular/.github/main/profile/joular.png" alt="Joular Project" width="64" /></a> project.

JoularJX is a Java-based agent for power monitoring at the source code level with support for modern Java versions and multi-OS to monitor power consumption of hardware and software.

JoularJX is a Java agent where you can simply hook it to the Java Virtual Machine when starting your Java program. To get power readings, JoularJX uses a custom PowerMonitor program (based on Hubblo's Windows RAPL driver) on Windows, Intel RAPL (through powercap) on GNU/Linux, Powermetrics on macOS, and our accurate power models on Raspberry Pi and similar devices.

JoularJX is the successor of [Jalen](https://www.noureddine.org/research/jalen), and the core approach of statistical sampling is based and inspired by the work we did in monitoring energy hotspots in software ([ASE 2012 conference paper](https://hal.inria.fr/hal-00715331/document), and [ASE Journal paper in 2015](https://hal.inria.fr/hal-01069142/document)).

The official website of JoularJX is: [https://www.noureddine.org/research/joular/joularjx](https://www.noureddine.org/research/joular/joularjx).

## Features

- Monitor power consumption of each method at runtime
- Monitor the evolution of power consumption of each method
- Monitor power consumption of methods' call tree and execution branches
- Monitor energy of Java applications running in virtual machines
- Uses a Java agent, no source code instrumentation needed
- Uses Intel RAPL (powercap interface) for getting accurate power reading on GNU/Linux, our research-based regression models on Raspberry Pi devices, and a custom program monitor (based on Intel Power Gadget) for accurate power readings on Windows
- Provides real-time power consumption of every method in the monitored program
- Provides total energy for every method on program exit

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

JoularJX is licensed under the GNU GPL 3 license only (GPL-3.0-only).

Copyright (c) 2021-2025, Adel Noureddine, Université de Pau et des Pays de l'Adour.
All rights reserved. This program and the accompanying materials are made available under the terms of the GNU General Public License v3.0 only (GPL-3.0-only) which accompanies this distribution, and is available at: https://www.gnu.org/licenses/gpl-3.0.en.html

Author : [Adel Noureddine](https://www.noureddine.org/)