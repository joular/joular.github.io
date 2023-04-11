# Integration with Systems and Tools

PowerJoular exports its power data to CSV files, or print the values directly on the terminal.
As such, PowerJoular can be used in IDEs, integration systems CI/CD, or alongside other profilers or debugging tools.

All what is needed is to start PowerJoular and read the generated CSV files.
PowerJoular can write these CSV files in ```overwrite``` mode, and thus reducing the overhead and size of these files.

With the systemd service provided, PowerJoular can run automatically on startup and write power consumption to ```/tmp``` folder (which can be changed in the service).
Therefore, offering an automatic way for multiple tools and dashboards to get accurate and real-time power metrics in multiple platforms.