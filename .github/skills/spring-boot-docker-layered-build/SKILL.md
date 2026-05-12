---
name: Spring Boot Docker Layered Build
description: Use this skill when the user wants to dockerize a Spring Boot application, write a Dockerfile for a Spring Boot jar, optimize Docker image layers, use `-Djarmode=layertools`, build a multi-stage Spring Boot Docker image, fix slow Docker rebuilds for Spring Boot, or pick a base image (eclipse-temurin) for a Spring Boot service.
---

# Spring Boot Docker Layered Build

Use this skill when the task involves:

- Creating a `Dockerfile` for a Spring Boot application.
- Optimizing Docker image build time by leveraging Spring Boot **layered jars**.
- Configuring Maven or Gradle to produce a layered jar.
- Picking a JDK / JRE base image for a Spring Boot container.
- Troubleshooting a Spring Boot container that fails to start (`JarLauncher` errors, missing classes, wrong working directory).

## Why layered jars

A regular Spring Boot fat-jar is a **single file**. When you `COPY` it into a Docker image, every code change invalidates one big layer, so every rebuild re-uploads ~50–200 MB of dependencies that did not change.

Spring Boot supports a `layertools` jar mode that splits the jar into 4 distinct
layers, ordered by how often they change:

| Layer                    | Changes                       | Cached effectively |
|--------------------------|-------------------------------|--------------------|
| `dependencies`           | Almost never                  | Yes                |
| `spring-boot-loader`     | Only on Spring Boot upgrades  | Yes                |
| `snapshot-dependencies`  | When you bump SNAPSHOT deps   | Sometimes          |
| `application`            | Every code change             | No (this is fine)  |

Docker caches each `COPY` as its own layer, so only the small **application**
layer is rebuilt and pushed on most code changes. Build and deploy times drop
significantly.

## Reference Dockerfile

This is the canonical multi-stage Dockerfile for a Spring Boot application using
layered jars. Use it as the default starting point.

```dockerfile
# --- Stage 1: extract layers from the fat-jar ---
FROM eclipse-temurin:17.0.5_8-jre-focal AS builder
WORKDIR extracted
ADD ./target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# --- Stage 2: assemble the final runtime image ---
FROM eclipse-temurin:17.0.5_8-jre-focal
WORKDIR application
COPY --from=builder extracted/dependencies/          ./
COPY --from=builder extracted/spring-boot-loader/    ./
COPY --from=builder extracted/snapshot-dependencies/ ./
COPY --from=builder extracted/application/           ./
EXPOSE 8080
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
```

### Important details

1. **The `COPY` order matters.** The least-changing layer (`dependencies`) must
   come first. Reordering breaks the cache benefit.
2. **`JarLauncher` class path changed in Spring Boot 3.2+.**
   - Spring Boot **3.2+**: `org.springframework.boot.loader.launch.JarLauncher`
   - Spring Boot **< 3.2**: `org.springframework.boot.loader.JarLauncher`
   Pick the one that matches your version, or the container will fail with
   `Error: Could not find or load main class`.
3. **Only one jar must exist in `target/`.** `ADD ./target/*.jar app.jar` will
   fail or be ambiguous if Maven produced both `*.jar` and `*.jar.original`.
   Either run `mvn package spring-boot:repackage` (default) or pin the exact
   filename.
4. **`EXPOSE 8080`** is documentation only. It does not publish the port — that
   is `docker run -p 8080:8080`.

## Base image guidance

- **`eclipse-temurin:<version>-jre-<distro>`** — recommended default. Smaller
  than the JDK image, official Adoptium build, broad community support.
- Use a **pinned patch version** (`17.0.5_8`) instead of `17` or `latest` for
  reproducible builds.
- For minimal images, consider `eclipse-temurin:21-jre-alpine` — but verify
  Spring Boot Native or any native libraries support musl libc.
- For native images compiled with GraalVM, switch to a distroless or minimal
  base — see the GraalVM skill.

## Maven configuration

The `spring-boot-maven-plugin` enables layered jars by default since Spring
Boot 2.4. You usually need nothing extra. To be explicit:

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layers>
            <enabled>true</enabled>
        </layers>
    </configuration>
</plugin>
```

Verify a layered jar was produced:

```sh
java -Djarmode=layertools -jar target/your-app.jar list
```

Expected output:

```
dependencies
spring-boot-loader
snapshot-dependencies
application
```

## Gradle configuration

The Spring Boot Gradle plugin enables layered jars by default since Spring Boot 2.4. To be explicit:

```groovy
bootJar {
    layered {
        enabled = true
    }
}
```

## Build commands

```sh
# 1. Build the jar
mvn clean package

# 2. Build the image (run from the project root, where the Dockerfile lives)
docker build -t my-spring-app:latest .

# 3. Run it
docker run --rm -p 8080:8080 my-spring-app:latest
```

## Customizing layers

To redefine which classes/resources go into which layer, add
`src/main/resources/META-INF/spring/layers.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<layers xmlns="http://www.springframework.org/schema/boot/layers">
    <application>
        <into layer="spring-boot-loader">
            <include>org/springframework/boot/loader/**</include>
        </into>
        <into layer="application"/>
    </application>
    <dependencies>
        <into layer="snapshot-dependencies">
            <include>*:*:*SNAPSHOT</include>
        </into>
        <into layer="dependencies"/>
    </dependencies>
    <layerOrder>
        <layer>dependencies</layer>
        <layer>spring-boot-loader</layer>
        <layer>snapshot-dependencies</layer>
        <layer>application</layer>
    </layerOrder>
</layers>
```

And reference it in the Maven plugin:

```xml
<configuration>
    <layers>
        <enabled>true</enabled>
        <configuration>${project.basedir}/src/main/resources/META-INF/spring/layers.xml</configuration>
    </layers>
</configuration>
```

## Common issues and fixes

| Symptom                                                        | Cause                                                | Fix                                                                                       |
|----------------------------------------------------------------|------------------------------------------------------|-------------------------------------------------------------------------------------------|
| `Error: Could not find or load main class JarLauncher`         | Wrong `JarLauncher` package for the Spring Boot version | Use `org.springframework.boot.loader.launch.JarLauncher` for 3.2+, otherwise the old path |
| `ADD ./target/*.jar app.jar` matches multiple files            | Both `*.jar` and `*.jar.original` present            | Delete `*.jar.original` before `docker build`, or pin the exact filename                  |
| Image is huge / every code change re-pushes 200MB              | Single-stage build or wrong `COPY` order             | Use the multi-stage Dockerfile above; copy `dependencies` first                           |
| `Unable to access jarfile app.jar` in stage 1                  | `mvn package` was not run, or wrong relative path    | Run `mvn package` first; check `WORKDIR` and the `ADD` path                               |
| App starts but cannot find resources                           | Custom `layers.xml` excluded resource files          | Make sure the catch-all `<into layer="application"/>` is present                          |
| Slow startup inside the container only                         | Missing `-XX:+UseContainerSupport` on old JVMs       | Use Java 17+ — container support is on by default                                         |
| `eclipse-temurin:17.0.5_8-jre-focal` not found                 | Tag was deprecated by upstream                       | Pick a current pinned tag from Docker Hub (e.g. `21.0.4_7-jre-jammy`)                     |

## Best practices

- **Pin the base image patch version.** Don't use floating tags in production builds.
- **Run as a non-root user** in production:
  ```dockerfile
  RUN useradd -r -u 1001 spring
  USER 1001
  ```
- **Add a healthcheck** if Spring Boot Actuator is enabled:
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=3s --start-period=30s \
      CMD curl -fsS http://localhost:8080/actuator/health/liveness || exit 1
  ```
- **Don't bake secrets into the image.** Pass them at runtime via env vars or a
  secrets manager.
- **Pass JVM flags via `JAVA_TOOL_OPTIONS`**, not by editing `ENTRYPOINT`:
  ```sh
  docker run -e JAVA_TOOL_OPTIONS="-Xmx512m" my-spring-app:latest
  ```
- **Keep `.dockerignore`** to avoid sending `target/`, `.idea/`, `.git/` into
  the build context. A minimal one:
  ```
  .git
  .idea
  *.iml
  target/*
  !target/*.jar
  ```

## When NOT to use this pattern

- **Native image builds (GraalVM).** Those produce a single self-contained
  binary — use the GraalVM Native Image skill instead.
- **Spring Boot's built-in image builder** (`mvn spring-boot:build-image`)
  produces a Cloud Native Buildpack image and already does layering for you.
  Use it if you want zero Dockerfile maintenance.