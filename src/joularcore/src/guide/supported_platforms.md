# Supported Platforms

Joular Core supports a wide variety of platforms, operating systems and CPU architectures.
It is as universal as it can get, and most environements are supported.

- 💻 **Supported Systems**: 🐧 Linux, 🪟 Windows, 🍎 MacOS, 🍓 Raspberry Pi, 💽 Virtual Machines.
- ⚙️ **Supported Architectures**: x86_64 (amd64), x86/i686, aarch64, arm, armv7, GPUs (Nvidia, Apple, AMD).

In particular:
- PC/Servers using a RAPL supported Intel processor (since Sandy Bridge) or a RAPL supported AMD processor (since Ryzen), on Windows and Linux.
- macOS computers on Intel mac or Apple ARM chip.
- Virtual machines (any supported guest on any host).
- For GPU: Nvidia, AMD and Apple
- Raspberry Pi devices (multiple models) on Linux (all revisions for each model are supported):
  - Model Zero W, for 32 bits OS
  - Model 1 B, for 32 bits OS
  - Model 1 B+, for 32 bits OS
  - Model 2 B, for 32 bits OS
  - Model 3 B, for 32 bits OS
  - Model 3 B+, for 32 bits OS
  - Model 4 B, for both 32 bits and 64 bits OS
  - Model 400, for 64 bits OS
  - Model 5 B, for 64 bits OS
- Asus Tinker Board (S)


| Platform | Supported OS | Based on | Supported Architecture |
|:--------------:|:---------------------:|:-----------------------------:|:-----------------------------:|
|     Linux PC/Server    |        Linux        |             RAPL, Nvidia, AMD            |             x86, x86_64            |
|     Windows PC/Server    |        Windows        |             RAPL, Nvidia, AMD            |             x86, x86_64            |
|     macOS PC/Server    |        macOS        |             Powermetrics (Apple Silicon CPU, GPU, Intel CPU)           |             x86, x86_64, ARM            |
|        Raspberry Pi       |        Linux       |             Our regression models (CPU)             |             ARM            |
|        Asus Tinker Board       |        Linux       |             Our regression models (CPU)             |             ARM            |
|     Virtual Machine    |        Supported guests (Windows, Linux, macOS), any host       |             Host's architecture (CPU, GPU)            |             x86, x86_64, ARM            |

**CPU:**

| OS                              | x86_64 | i686 | Apple Silicon | arm  | armv7 | aarch64 |
|---------------------------------|--------|------|---------------|------|-------|---------|
| Windows                         | 🌟     | 🌟   |               |      |       |         |
| Linux                           | 🌟     | 🌟   |               |      |       |         |
| macOS                           | 🌟     |      | 🌟            |      |       |         |
| SBC (Raspberry Pi, Asus)        |        |      |               | 🌟   | 🌟    | 🌟      |
| Virtual Machines                | 🌟     | 🌟   | 🌟            | 🌟   | 🌟    | 🌟      |

**GPU:**

| OS                           | Nvidia GPU | AMD GPU | Apple GPU |
|------------------------------|------------|---------|-----------|
| Windows                      | 🌟         | 🌟      |           |
| Linux                        | 🌟         | 🌟      |           |
| macOS                        |            |         | 🌟        |
| SBC (Raspberry Pi, Asus)     |            |         |           |
| Virtual Machines             | 🌟         | 🌟      | 🌟        |

**Supported SBC platforms**: Raspberry Pi (models: Zero W, 1 B, 1 B+, 2 B, 3 B, 3 B+, 4 B, 400, 5 B), Asus Tinker Board S.