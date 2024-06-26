# Supported Platforms

JoularJX support the following platforms and operating systems:

- PC/Servers using a RAPL supported Intel processor (since Sandy Bridge) or a RAPL supported AMD processor (since Ryzen), on GNU/Linux, on Windows and on macOS.
- Virtual machines (any supported guest on any guest)

The supported list of Raspberry Pi and Asus Tinker Board models are listed below.
We support all revisions of each model lineup.
However, the model is generated and trained on a specific revision (listed between brackets), and the accuracy is best on this particular revision.

- Raspberry Pi devices (multiple models) on GNU/Linux:
  - Model Zero W (rev 1.1), for 32 bits OS
  - Model 1 B (rev 2), for 32 bits OS
  - Model 1 B+ (rev 1.2), for 32 bits OS
  - Model 2 B (rev 1.1), for 32 bits OS
  - Model 3 B (rev 1.2), for 32 bits OS
  - Model 3 B+ (rev 1.3), for 32 bits OS
  - Model 4 B (rev 1.1, and rev 1.2), for both 32 bits and 64 bits OS
  - Model 400 (rev 1.0), for 64 bits OS
  - Model 5 B (rev 1.0), for 64 bits OS
- Asus Tinkber Board (S)

| Platform | Supported OS | Based on | Supported Architecture |
|:--------------:|:---------------------:|:-----------------------------:|:-----------------------------:|
|     Linux PC/Server    |        GNU/Linux        |             RAPL (using powercap)            |             x86, x86_64            |
|     Windows PC/Server    |        Windows        |             Hubblo's Windows RAPL driver            |             x86, x86_64            |
|     macOS PC    |        macOS        |             Powermetrics            |             x86_64, ARM            |
|        Raspberry Pi       |        GNU/Linux       |             Our regression models             |             ARM            |
|        Asus Tinker Board       |        GNU/Linux       |             Our regression models             |             ARM            |
|     Virtual Machine    |        Supported guests (Windows, Linux, macOS), any host       |             Host's architecture (RAPL, regression models, others)            |             x86, x86_64, ARM            |