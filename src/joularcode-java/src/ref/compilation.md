# Compilation

To build Joular Code for Java, you need Java 21 or later and Maven (3.6.3 or later), then just clone the repository and build:

```
git clone https://github.com/joular/joularcode-java.git
cd joularcode-java
mvn clean install
```

This produces the JAR in the `target` folder:

```
target/joularcodejava-<version>.jar
```

The build also runs the unit tests. JUnit is the only dependency, and is used for the tests only.
The agent itself has no runtime dependencies, so the JAR holds nothing but its own classes.