# Integration with Systems and Tools

Joular Code for Java is a Java agent, and the compiled JAR file has no external dependencies.
Therefore, it can be integrated and used in any setup running a standard JVM (OpenJDK, Oracle JDK), Java 21 or later (see [Supported Platforms](../guide/supported_platforms.md)).

For instance, you can add Joular Code for Java to the run/execute parameters of your favorite IDE (Eclipse, IntelliJ IDEA, NetBeans, etc.), or to your development workflow or continuous integration and delivery processes (CI/CD).

Joular Code for Java runs with the same configuration options and generated files on all supported platforms, from Windows to Linux, from x86_64 servers and PC to ARM Raspberry Pi devices.

The generated CSV files are added to at the end of every monitoring cycle (about every second), so other tools can read them while the application runs, or after it ends.

## Attaching to a running JVM

The agent can also be loaded into a JVM that is already running, without restarting it, from another Java program using the Attach API (`com.sun.tools.attach.VirtualMachine`, in the `jdk.attach` module):

```java
VirtualMachine vm = VirtualMachine.attach(pid);
vm.loadAgent("/path/to/joularcodejava-<version>.jar");
vm.detach();
```

Monitoring starts when the agent is loaded, so whatever the application did before that is not in the results.
JDK 21 and later allow the attach by default but print a warning: start the target JVM with `-XX:+EnableDynamicAgentLoading` to silence it, or with `-XX:-EnableDynamicAgentLoading` to forbid it.

The configuration is read when the agent attaches, in the target JVM: `-Djoularcodejava.properties=...` belongs on its own command line, and the default `joularcodejava.properties` and a relative `results-path` are taken from its working folder.

Attaching again while Joular Code for Java is monitoring is ignored, with a warning, rather than starting a second monitor.
If the first one has stopped (for instance, because its power source could not be opened), attaching again reads the configuration again and starts monitoring.

## Application servers and long-running applications

Joular Code for Java monitors the JVM until it stops, and closes its files when the JVM shuts down.
It works the same for a program that ends on its own and for an application server or a service that keeps running (Spring Boot, Tomcat, etc.).
Unlike JoularJX, there is no `application-server` property to set.

The cycle in progress when the JVM shuts down is not written, so the last second or so of a run is not in the results, and a program that ends within its first cycle leaves files with only their header.

## Messages

Joular Code for Java writes its messages on the standard error, and leaves the standard output to the application.
The first line gives its version (`Joular Code for Java: version <version>`), and the next ones read as `dd-MM-yyyy HH:mm:ss LEVEL: message`.

If anything goes wrong in Joular Code for Java, it says so there.
When the power cannot be read, or a cycle fails, that cycle gets no rows and the monitoring resumes by itself.
When Joular Code for Java cannot start, or cannot open its power source, the application carries on unmonitored.