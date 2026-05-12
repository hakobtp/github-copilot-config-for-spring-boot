---
mode: Agent
description: Create a .gitignore file with appropriate content for a Spring Boot project, including common IDE and build tool ignores.
---

# Add .gitignore File

Create a `.gitignore` file in the root directory of the workspace (@workspace).

If the file already exists, overwrite it with the content below.

The file must contain exactly the content provided between the **gitignore code block**.

Do not modify, remove, or add any extra lines. Copy the content exactly as it appears.

```gitignore
HELP.md
target/
.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/
!**/src/main/**/build/
!**/src/test/**/build/

### VS Code ###
.vscode/
```