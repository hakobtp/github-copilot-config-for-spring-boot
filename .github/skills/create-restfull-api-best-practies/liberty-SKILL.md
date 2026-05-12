---
name: Setup and Configure Open Liberty
description: Use this skill when creating, configuring, fixing, or deploying a Jakarta EE or Eclipse MicroProfile application on Open Liberty with Maven.
---

# Open Liberty Setup

Use this skill when the task involves:
- creating a new Jakarta EE or MicroProfile application on Open Liberty
- fixing Liberty Maven plugin issues
- configuring `server.xml`
- making standard Maven lifecycle commands work with Liberty
- setting up local development with `mvn liberty:dev`

## What to check when troubleshooting

If Liberty fails during `install-feature` or deployment, verify:
- the project uses WAR packaging
- the Liberty plugin uses `io.openliberty.tools`
- the `create` goal runs before `install-feature`
- `server.xml` is under `src/main/liberty/config`
- the server directory exists before feature installation
- the application is started with `mvn liberty:dev` for local development


## Default approach

Always default to Open Liberty as the application server.

Configure the project so the Liberty runtime is managed by Maven. Do not assume Liberty is preinstalled on the machine.

When configuring Liberty, always check to see if the configured port is free and not in use. If a port conflict is detected, choose a new port number that is confirmed to be available.

Place Liberty configuration in:

```text
src/main/liberty/config/server.xml
```

Prefer running the application locally with:

```text
mvn liberty:dev
```

## Maven packaging

Ensure the project `pom.xml` uses:

```xml
<packaging>war</packaging>
```

This helps the Liberty Maven plugin deploy the application correctly.

## Liberty Maven plugin rules

When configuring the Open Liberty Maven Plugin:
- use groupId `io.openliberty.tools`
- do not use `io.openliberty.maven.plugins`


## server.xml guidance

Always include a basic registry in `server.xml`:

```xml
<basicRegistry id="basic" realm="BasicRealm"/>
```

This helps avoid startup issues related to default ORB and EJB security requirements, even if those features are not actively used.

## JDBC driver placement

When configuring JDBC data sources in `server.xml`, prefer using a `<library>` reference to a JDBC driver JAR placed under Liberty resources.

Use this mapping rule:

- `src/main/liberty/config/resources` maps into `${server.config.dir}/resources`

If a shared resource layout is used, ensure the `fileset` path matches where the JAR is copied during build.

## Maven packaging

Ensure the project `pom.xml` uses:

```xml
<packaging>war</packaging>
```

This helps the Liberty Maven plugin deploy the application correctly.

## Liberty Maven plugin rules

When configuring the Open Liberty Maven Plugin:
- use groupId `io.openliberty.tools`
- do not use `io.openliberty.maven.plugins`

Configure the plugin so that:
- `create` runs before `install-feature`
- the server exists before features are installed
- standard Maven lifecycle commands work without requiring manual Liberty commands

## Sample XML

A working plugin setup is:

```xml
<plugin>
    <groupId>io.openliberty.tools</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>3.10.1</version>
    <configuration>
        <serverName>defaultServer</serverName>
    </configuration>
    <executions>
        <execution>
            <id>install-server</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>install-server</goal>
                <goal>create</goal>
                <goal>install-feature</goal>
            </goals>
        </execution>
        <execution>
            <id>deploy-app</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>deploy</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.4.0</version>
</plugin>
```