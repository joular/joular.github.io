# Overview

[![License: LGPL v3](https://img.shields.io/badge/License-LGPLv3-blue)](https://www.gnu.org/licenses/lgpl-3.0)
[![Java](https://img.shields.io/badge/Java-21%2B-orange)](https://openjdk.java.net)

Joular Code for Java is part of the <a href="https://www.noureddine.org/research/joular/"><img src="https://raw.githubusercontent.com/joular/.github/main/profile/joular.png" alt="Joular Project" width="64" /></a> project.

Joular Code for Java is a lightweight and efficient Java agent for monitoring the energy consumption of methods and execution branches at the source code level.

This project is part of [Joular Code](https://github.com/joular/joularcode), and is the successor of [JoularJX](https://github.com/joular/joularjx).

It is a Java agent where you can simply hook it to the Java Virtual Machine when starting your Java program, or attach it to a program already running.
To get power readings, it reads Intel and AMD RAPL directly (through powercap) on Linux, and uses the shared memory ring buffer of [PowerJoular](https://github.com/joular/powerjoular) on Windows, macOS and Raspberry Pi devices (and on Linux too, if you prefer). Inside a virtual machine, it reads the power the host writes to a file shared with the guest.

## Features

- Monitor power consumption and energy of each method and execution branch at runtime
- Uses a Java agent, no source code instrumentation or modification needed
- Samples the JVM stack at high frequency (by default, every 10 milliseconds) and attributes energy every second
- No runtime dependencies: the agent runs on the JDK alone
- Gets the CPU power from Linux RAPL directly, from PowerJoular's shared memory ring buffer, or from the host of a virtual machine
- Generates CSV files with the power (watts) and the energy (joules) of each execution branch
- Provides two sets of results: one for all methods (including the JDK ones), and one filtered and calculated for your application's methods
- Works on Windows, macOS, Linux and Raspberry Pi

## License

Joular Code for Java is licensed under the GNU LGPL 3 license only (LGPL-3.0-only).

Copyright © 2026, Adel Noureddine.
All rights reserved. This program and the accompanying materials are made available under the terms of the [GNU Lesser General Public License v3.0 only (LGPL-3.0-only)](https://www.gnu.org/licenses/lgpl-3.0.en.html) which accompanies this distribution.

Author : [Adel Noureddine](https://www.noureddine.org/)