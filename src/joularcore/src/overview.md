# Overview

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue)](https://www.gnu.org/licenses/gpl-3.0) ![Made with Rust](https://img.shields.io/badge/Made%20with-Rust-2b2b2b?logo=rust&logoColor=white)

![Joular Core Logo](joularcore.png)

Joular Core is a cross-platform power and energy monitoring tool that runs on Linux, Windows, macOS, Raspberry Pi, and inside virtual machines. It measures CPU and GPU power consumption in real time, and can break that down to individual processes or named applications.

Joular Core is part of the <a href="https://www.noureddine.org/research/joular/">Joular project</a>.

The official website is: [https://www.noureddine.org/research/joular/joularcore](https://www.noureddine.org/research/joular/joularcore).

It is written in Rust and compiles to a native binary with minimal overhead. It ships with both a command-line interface and a graphical user interface, and can export data to CSV files, a shared-memory ring buffer, and an HTTP/WebSocket API.

> Joular Core is under active development and currently in beta quality. Expect rough edges and features still being worked on and polished.

## Key Features

- Real-time CPU and GPU power monitoring on PCs, servers, and single-board computers
- Per-process power monitoring: track the energy consumption of a specific PID
- Per-application power monitoring: track a named application across all its processes
- Monitor power from inside virtual machines using data from the hypervisor or an external meter
- Export power data to CSV files (append or overwrite mode)
- Write power data to a shared-memory ring buffer for low-latency IPC with other programs
- Expose power data over HTTP (`GET /data`) and WebSocket (`/ws`) endpoints
- Filter output to only show CPU power, GPU power, or both
- CPU idle baseline calibration: automatically or manually subtract idle CPU power before attributing it to a process or application
- Command-line interface (CLI) with colored, human-readable output or a numeric-only mode
- Graphical user interface (GUI) built with egui, with live power graphs and a 60-second rolling history
- Linux systemd service for running as a background daemon

## License

Joular Core is licensed under the GNU GPL 3 license only (GPL-3.0-only).

Copyright © 2025–2026, Adel Noureddine.  
All rights reserved. This program and the accompanying materials are made available under the terms of the [GNU General Public License v3.0 only (GPL-3.0-only)](https://www.gnu.org/licenses/gpl-3.0.en.html).

Author: [Adel Noureddine](https://www.noureddine.org/)
