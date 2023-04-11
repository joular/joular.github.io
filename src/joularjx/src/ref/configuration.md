# Configuration Properties

JoularJX can be configured by modifying the ```config.properties``` files:
- ```filter-method-names```: list of strings which will be used to filter the monitored methods (see Generated files below for explanations).
- ```save-runtime-data```: write runtime methods power consumption in a CSV file.
- ```overwrite-runtime-data```: overwrite runtime power data files, or if set to false, it will write new files for each monitoring cycle.
- ```logger-level```: set the level of information (by logger) given by JoularJX in the terminal (allowed values: OFF, INFO, WARNING, SEVERE).
- ```powermonitor-path```: Full path to the power monitor program (only for Windows).
- ```track-consumption-evolution```: generate CSV files for each method containing details of the method's consumption over the time. Each consumption value is mapped to an Unix timestamp.
- ```hide-agent-consumption```: if set to true, the energy consumption of the agent threads will not be reported.
- ```enable-call-trees-consumption```: compute methods call trees energy consumption. A CSV file will be generated at the end of the agent's execution, associating to each call tree it's total energy consumption.
- ```save-call-trees-runtime-data```: write runtime call trees power consumption in a CSV file. For each monitoring cycle (1 second), a new CSV file will be generated, containing the runtime power consumption of the call trees. The generated files will include timestamps in their names.
- ```overwrite-call-trees-runtime-data```: overwrite runtime call trees power data file, or if set to false, it will write new file for each monitoring cycle.

You can install the jar package (and the PowerMonitor.exe on Windows) wherever you want, and call it in the ```javaagent``` with the full path.
However, ```config.properties``` must be copied to the same folder as where you run the Java command.