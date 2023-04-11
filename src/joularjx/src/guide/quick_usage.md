# Quick Usage

JoularJX is a Java agent where you can simply hook it to the Java Virtual Machine when starting your Java program's main class:

```
java -javaagent:joularjx-$version.jar YourProgramMainClass
```

If your program is a JAR file, then just run it as usual while adding JoularJX:

```
java -javaagent:joularjx-$version.jar -jar yourProgram.jar
```

JoularJX will generate multiple CSV files according to the configuration settings (in ```config.properties```), and will create these files in a ```joularjx-results```folder.