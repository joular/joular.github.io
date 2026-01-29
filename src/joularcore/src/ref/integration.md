# Integration with Systems and Tools

Joular Core exports its power data with multiple outputs and approaches: to the terminal, to CSV files, to an HTTP and WebSockets API, and to a ring buffer shared memory (IPC).
Check [this page](./exports.md) for details on these export approaches in Joular Core.

Therefore, Joular Core can be used in IDEs, integration systems CI/CD, alongside other profilers or debugging tools, connect with dashboard for real-time power monitoring, send data to remote databases, or anywhere needed!

All what is needed is to start Joular Core and configure the export you want to use, and that's it.

Joular Core can also be run as a CLI service, on all platforms.