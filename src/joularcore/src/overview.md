# Overview

[![License: LGPL v3](https://img.shields.io/badge/License-LGPLv3-blue)](https://www.gnu.org/licenses/lgpl-3.0) [![Ada](https://img.shields.io/badge/Made%20with-Ada-blue)](https://www.adaic.org)

![Joular Core Logo](joularcore.png)

Joular Core is an Ada library that measures the energy or power consumption of hardware components.
It detects automatically what the machine offers (which CPU, which GPU, and how to read them), and gives one simple interface: open, read, close.

Joular Core is part of the <a href="https://www.noureddine.org/research/joular/">Joular project</a>.

The official website is: [https://www.noureddine.org/research/joular/joularcore](https://www.noureddine.org/research/joular/joularcore).

It compiles to native code with minimal overhead, and also provides a C interface, so it can be used from any language with a C FFI (C, C++, Java, Python, Rust, etc.).

Joular Core is the library [PowerJoular](https://github.com/joular/powerjoular) uses to measure the hardware.

## Key Features

- Measure the CPU and the GPU on Linux, Windows and macOS, and Nvidia GPUs on BSD
- Measure Intel and AMD processors through RAPL, on Linux and Windows
- Measure Apple Silicon and Intel Macs through powermetrics
- Measure Raspberry Pi and Asus Tinker Board through our research-based regression power models
- Measure Nvidia, AMD and Apple Silicon GPUs
- Detect automatically the hardware and how to read it, with nothing to configure
- Report a source that is not there or cannot be read as not available, while the other sources keep working
- Static library for Ada programs, and a shared library with a C interface for other languages (one self-contained file on Linux and Windows)

## License

Joular Core is licensed under the GNU LGPL 3 license only (LGPL-3.0-only).

Copyright © 2026, Adel Noureddine.  
All rights reserved. This program and the accompanying materials are made available under the terms of the [GNU Lesser General Public License v3.0 only (LGPL-3.0-only)](https://www.gnu.org/licenses/lgpl-3.0.en.html).

Author: [Adel Noureddine](https://www.noureddine.org/)