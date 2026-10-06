# Overview

[![License: LGPL v3](https://img.shields.io/badge/License-LGPLv3-blue)](https://www.gnu.org/licenses/lgpl-3.0) [![Ada](https://img.shields.io/badge/Made%20with-Ada-blue)](https://www.adaic.org)

CPU Load is an Ada library that reports the CPU load of the system, of a specific process by its ID, or of a specific application by its name (meaning every one of its processes running when a sample is taken).
It gives one simple interface: take a sample, wait, take another, and compare the two.

CPU Load is part of the <a href="https://www.noureddine.org/research/joular/">Joular project</a>.

It compiles to native code with minimal overhead, and also provides a C interface, so it can be used from any language with a C FFI (C, C++, Java, Python, Rust, etc.).

CPU Load is the library [PowerJoular](https://github.com/joular/powerjoular) uses to measure the CPU load, alongside [Joular Core](https://github.com/joular/joularcore) for the hardware.

## Key Features

- Measure the CPU load of the whole system, of one process by its ID, or of an application by its name (every process of it)
- Measure on Linux, Windows and macOS (Apple Silicon and Intel Macs)
- Match an application with the program its processes actually run, so `firefox` finds every process of Firefox
- Give every load as a share of the whole machine, from 0.0 to 1.0
- Tell apart a process that could not be read (a negative load) from one that used no CPU time (0.0)
- Keep no state: a sample is a plain record, so it can be taken in one thread and compared in another
- Read the counters with no particular rights (except for the processes of other users on macOS and Windows)
- Static library for Ada programs, and a shared library with a C interface for other languages (one self-contained file on Linux and Windows)

## License

CPU Load is licensed under the GNU LGPL 3 license only (LGPL-3.0-only).

Copyright © 2026, Adel Noureddine.  
All rights reserved. This program and the accompanying materials are made available under the terms of the [GNU Lesser General Public License v3.0 only (LGPL-3.0-only)](https://www.gnu.org/licenses/lgpl-3.0.en.html).

Author: [Adel Noureddine](https://www.noureddine.org/)