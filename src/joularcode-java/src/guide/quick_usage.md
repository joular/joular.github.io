# Quick Usage

Joular Code for Java is a Java agent where you can simply hook it to the Java Virtual Machine when starting your Java program's main class:

```
java -javaagent:joularcodejava-<version>.jar YourProgramMainClass
```

If your program is a JAR file, then just run it as usual while adding Joular Code for Java:

```
java -javaagent:joularcodejava-<version>.jar -jar yourProgram.jar
```

On Linux PC/Servers, Joular Code for Java reads the CPU power from RAPL directly, which needs root, or read access to the RAPL files (see [Installation](./installation.md)).

Everywhere else (Windows, macOS, Raspberry Pi), start [PowerJoular](https://github.com/joular/powerjoular) with the `-r` option, so it writes its power data to the shared memory ring buffer Joular Code for Java reads (with `sudo` on macOS, and from a terminal with administrative rights on Windows with the PawnIO driver):

```
powerjoular -r
```

On Windows, start PowerJoular before the application, and do not restart it while the application runs.
On Linux, you can also run `sudo powerjoular -r` and set `power-source-type=ringbuffer`, so the Java application does not need root.

Joular Code for Java will generate two CSV files, and will create these files in a `joular-code-java-results` folder:
- `methods-power-all.csv`: the power and energy of every execution branch, including the JDK ones.
- `methods-power-app.csv`: the power and energy of the execution branches of your application, according to the `methods-filtering-prefix` setting.

To focus the second file on your own code, set the packages of your application in `joularcodejava.properties`, in the folder where you run the Java command:

```properties
methods-filtering-prefix=com.example
```

To use a configuration file from another path:

```
java -Djoularcodejava.properties=/path/to/joularcodejava.properties -javaagent:joularcodejava-<version>.jar -jar yourProgram.jar
```

Joular Code for Java writes its messages on the standard error, and leaves the standard output to the application.
At startup, it says which power source it reads and where the results are written.