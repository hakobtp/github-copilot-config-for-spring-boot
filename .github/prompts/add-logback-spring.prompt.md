---
mode: agent
description: Create or replace src/main/resources/logback-spring.xml with the project's standard Logback setup (colored console for local, async wrapper, JSON for default, profile-aware root logger).
---

# Add `logback-spring.xml`

Create or replace the Spring Boot Logback configuration file at:

`src/main/resources/logback-spring.xml`

## What to build

A `logback-spring.xml` that:

1. Defines three appenders — a JSON appender (Logstash encoder), a colored
   pattern-layout console appender, and an async wrapper around the console
   appender.
2. Routes logs through the right appender per Spring profile:
   - `test`  → plain `console` (synchronous, easier to read in CI output).
   - `local` → `async_console` (colored, async, low-latency for dev).
   - `default` → JSON or pattern, chosen at runtime via the `log.format`
     property (defaults to `json`).
3. Reads the property `log.format` through `<springProperty>` so it can be
   overridden in `application.yml` / `application.properties` or via an
   environment variable (`LOG_FORMAT`).
4. Surfaces useful MDC fields in the colored pattern:
   `correlationId`, `userId`, `userType`, `roles`.

## Required dependency

The JSON appender uses `net.logstash.logback.encoder.LogstashEncoder`. Before
writing the file, check the build manifest:

- **Maven** (`pom.xml`): look for `net.logstash.logback:logstash-logback-encoder`.
- **Gradle** (`build.gradle` / `build.gradle.kts`): same coordinates.

If the dependency is missing, add it (latest stable version compatible with the
project's Spring Boot version) and tell the user it was added.

## File content

Write **exactly** this content. Do not reorder, rename, or omit elements.

```xml
<configuration>
    <springProperty name="logs.format" source="log.format"/>

    <appender name="json" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        </encoder>
    </appender>

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %highlight(%-5level) [%magenta(%thread)] %cyan(%logger{36}) [correlationId=%X{correlationId} user=%X{userId} type=%X{userType} roles=%X{roles}] - %msg%n</pattern>
        </layout>
    </appender>

    <!-- Async wrapper for the colored console appender to improve performance -->
    <appender name="async_console" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>512</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <appender-ref ref="console"/>
    </appender>

    <!-- Switch format based on the active Spring profile -->

    <springProfile name="test">
        <root level="INFO">
            <appender-ref ref="console"/>
        </root>
    </springProfile>

    <springProfile name="local">
        <root level="INFO">
            <appender-ref ref="async_console"/>
        </root>
    </springProfile>

    <springProfile name="default">
        <root level="INFO">
            <appender-ref ref="${logs.format:-json}"/>
        </root>
    </springProfile>
</configuration>
```

## Property to expose

So the `default` profile can switch between JSON and colored output without a
rebuild, make sure the `log.format` property is documented (do **not** invent
a value — only add it if it isn't already there):

```properties
# application.properties
log.format=json
```

```yaml
# application.yml
log:
  format: json
```

Valid values are `json` (default) and `console`.

## Rules

- Only create or update `src/main/resources/logback-spring.xml` and, if needed,
  the build file and the `log.format` property documentation.
- Do **not** touch Java source files.
- Do **not** add other appenders, profiles, or log levels beyond what is
  specified above.
- Do **not** rename the appenders — downstream tooling references them by name.
- If a `logback.xml` (without the `-spring` suffix) exists alongside the new
  file, point this out: it would take precedence over the Spring-aware file and
  should usually be deleted.
- This file configures the **logging pipeline**. It does not change how code
  emits logs — Java logging rules still live in
  `.github/instructions/java.logging.instructions.md` (SLF4J `log`, `[ ]`
  placeholders, no secrets).

## Output behavior

- Create or overwrite the file directly.
- If the Logstash encoder dependency was missing, report which file you updated
  and the version you chose.
- Summarize what was done: file path, profiles configured, default format,
  and any dependency / property changes.