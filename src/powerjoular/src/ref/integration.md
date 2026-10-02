# Integration with Systems and Tools

PowerJoular exports its power data to CSV files and to a shared memory ring buffer, or prints the values directly on the terminal.
As such, PowerJoular can be used in IDEs, integration systems CI/CD, or alongside other profilers or debugging tools.

All what is needed is to start PowerJoular and read the generated CSV files.
PowerJoular can write these CSV files in ```overwrite``` mode, and thus reducing the overhead and size of these files.

For a program running on the same machine, the ring buffer (```-r```) is the fastest way: the program maps the shared memory area once, and reads the latest measurement whenever it needs it, with no file to open or parse.
See [Exporting Power Data](./exports.html) for the format of the CSV files and the layout of the ring buffer.

With the systemd service provided, PowerJoular can run automatically on startup and write power consumption to ```/run/powerjoular``` folder (which can be changed in the service: the service can only write in its own folder, so a new folder also needs a ```ReadWritePaths=``` line there).
Therefore, offering an automatic way for multiple tools and dashboards to get accurate and real-time power metrics in multiple platforms.