# Quick Usage of the graphical user interface (GUI) version

Joular Core is shipped with an easy-to-use, yet powerful and complete, graphical user interface (GUI), which works seamlessly and in a coherent way across all platforms and operating systems.

To run the GUI, either:
- Start Joular Core with the `-g` or `--gui` argument.
- Use the dedicated GUI binary directly (double click for instance): `joularcoregui` (on Linux or macOS) or `joularcoregui.exe` (on Windows).

On RAPL-based Linux systems and on macOS, elevated access is needed to monitor CPU energy (hence the need to start it with `sudo`).

Most CLI features are available in the GUI, with easy-to-use buttons and configuration settings to monitor the entire computer, a specific process, or a specific multi-process application.

If the GUI is executed from a terminal, it also accepted the same CLI arguments as the CLI version.
For instance, you can start the gui and an HTTP socket to export power data with: `joularcore -g --api-port 8080`