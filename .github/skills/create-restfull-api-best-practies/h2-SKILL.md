---
name: H2 Database Setup, Configuration and Troubleshooting
description: Use this skill when configuring H2 for local Jakarta EE development, especially with Open Liberty, JPA, EclipseLink, and persistence.xml or server.xml setup.
---

# H2 Local Development

Use this skill when the task involves:
- configuring H2 for local development
- wiring H2 into Open Liberty
- configuring JPA or EclipseLink for H2
- avoiding schema-generation or identity issues across H2 versions
- setting up datasource definitions in `server.xml`

## Default approach

Prefer H2 as the default local development database for Jakarta EE applications.

## JPA and EclipseLink guidance

For local development with H2 and Open Liberty, prefer this EclipseLink setting in `persistence.xml`:

```xml
<property name="eclipselink.ddl-generation" value="drop-and-create-tables"/>
```

Prefer this over relying on standard Jakarta schema generation properties when targeting local H2 development on Open Liberty.

## Primary key strategy

Prefer:

```java
GenerationType.AUTO
```

instead of:

```java
GenerationType.IDENTITY
```

This improves compatibility across different H2 versions.

## Table naming guidance

Always prefix database table names to avoid conflicts with SQL reserved keywords.

Examples:
- `RPS_USER`
- `APP_ORDER`

Avoid generic names such as:
- `USER`
- `ORDER`
- `MATCH`
- `GROUP`

## H2 JDBC driver handling for Open Liberty

When using H2 with Open Liberty, copy the H2 JAR into a Liberty-accessible resources location during the Maven build.

## What to check when troubleshooting

If H2 is not loading correctly, verify:
- the H2 JAR is copied during the Maven build
- the `fileset` path matches the actual location of `h2.jar`
- the datasource `libraryRef` matches the declared library id
- JPA uses `GenerationType.AUTO`
- table names do not conflict with SQL reserved words

A working Maven dependency plugin example is:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.6.0</version>
    <executions>
        <execution>
            <id>copy-h2</id>
            <phase>package</phase>
            <goals>
                <goal>copy</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>com.h2database</groupId>
                        <artifactId>h2</artifactId>
                        <version>${h2.version}</version>
                        <type>jar</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.directory}/liberty/wlp/usr/shared/resources</outputDirectory>
                        <destFileName>h2.jar</destFileName>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## server.xml datasource template

A working H2 datasource template for `server.xml` is:

```xml
<dataSource id="h2DataSource" jndiName="jdbc/h2DataSource">
    <jdbcDriver libraryRef="H2Lib" javax.sql.DataSource="org.h2.jdbcx.JdbcDataSource"/>
    <properties URL="jdbc:h2:${server.config.dir}/data/rpsdb;AUTO_SERVER=TRUE" user="sa" password=""/>
</dataSource>

<library id="H2Lib">
    <fileset dir="${server.config.dir}/../../shared/resources" includes="h2.jar"/>
</library>
```

