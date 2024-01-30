# Supported Platforms

PowerJoular monitors the following platforms:
- PC/Servers using a RAPL supported Intel processor (since Sandy Bridge) or a RAPL supported AMD processor (since Ryzen), and optionally an Nvidia graphic card.
- Raspberry Pi devices (multiple models) and Asus Tinker Board.

In all platforms, PowerJoular works currently only on GNU/Linux.

On PC/Servers, PowerJoular uses powercap Linux interface to read Intel RAPL (Running Average Power Limit) energy consumption.

PowerJoular supports RAPL package domain (core, including integrated graphics, and dram), and for more recent processors, we support Psys package (which covers the energy consumption of the entire SoC).

On Raspberry Pi and Asus Tinker Board, PowerJoular uses its own research-based empirical regression models to estimate the power consumption of the ARM processor.

The supported list of Raspberry Pi and Asus Tinker Board models are listed below.
We support all revisions of each model lineup.
However, the model is generated and trained on a specific revision (listed between brackets), and the accuracy is best on this particular revision.

We currently support the following Raspberry Pi models and Asus Tinker Board models:
- Model Zero W (rev 1.1), for 32 bits OS
- Model 1 B (rev 2), for 32 bits OS
- Model 1 B+ (rev 1.2), for 32 bits OS
- Model 2 B (rev 1.1), for 32 bits OS
- Model 3 B (rev 1.2), for 32 bits OS
- Model 3 B+ (rev 1.3), for 32 bits OS
- Model 4 B (rev 1.1, and rev 1.2), for both 32 bits and 64 bits OS
- Model 400 (rev 1.0), for 64 bits OS
- Model 5 B (rev 1.0), for 64 bits OS
- Asus Tinker Board (S)

| Platform | Supported OS | Based on | Supported Architecture |
|:--------------:|:---------------------:|:-----------------------------:|:-----------------------------:|
|        Raspberry Pi       |        GNU/Linux       |             Our regression models             |             ARM            |
|        Asus Tinker Board       |        GNU/Linux       |             Our regression models             |             ARM            |
|     Linux PC/Server    |        GNU/Linux        |             RAPL (using powercap), Nvidia-smi            |             x86, x86_64            |