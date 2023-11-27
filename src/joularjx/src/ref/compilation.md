# Compilation

To build JoularJX, you need Java 11+ and Maven, then just build:

```
mvn clean install
```

Alternatively, you can use the Maven wrappen shipped with the project with the command:

```
Linux and macOS: ./mvnw clean install
Windows: mvnw.cmd clean install
```

To compile the Windows power monitor tool, required by JoularJX on Windows, open the project in Visual Studio and compile there.
Or open, Developer Command Prompt for VS (or Developer PowerShell for VS), and compile with this command:
```
msbuild.exe PowerMonitor.sln /property:Configuration=Release
```