# Quick Usage

Start Joular Energy Meter, with the rights it needs to read the power (see [Required privileges](./supported_platforms.html#required-privileges)).

The window has two tabs:
- **Setup**: what to measure (the whole machine, a process by its ID, an application by its name, or a command to start), how often (every 1 to 10 seconds), for how long, and whether to record to a CSV file. Optionally, the electricity used for the carbon emissions and the cost, the idle power, and a plug meter.
- **Readings**: the power now, and the energy and carbon emissions since the start, of the followed program and of the whole machine, the share of the processor and the graphics card, and a chart of the last 120 measurements.

Press **Start** (or Enter in a field) to start measuring.
The bar at the bottom says what is measured, for how long, and the last message.

While measuring, the interval, the length and the electricity can still be changed (a new interval restarts the chart).
**Pause** stops the clock and the energy count until you resume, and the length only counts the measured time.
After **Stop**, the power shown is the average of the session.

**Copy summary** puts the session in plain text on the clipboard, for a report or an email: what was measured, when and for how long, the average power, the energy, the carbon emissions, and the price per kWh (with the cost of the followed program, when there is one).

Resting the pointer on the name of a figure shows what it means.

To see what a program adds to an idle machine, measure the idle power first (see [Idle Power and Plug Meter](../ref/idle_plug.html)).
