---
mode: agent
description: Create or replace a Spring Boot banner.txt file with fun ASCII art and helpful environment placeholders
---

# Wave Your Banner

Create or replace the Spring Boot banner file at:

`src/main/resources/banner.txt`

## What to build

Create a custom Spring Boot `banner.txt` file for this application.

## Banner requirements

1. Build fun ASCII art based on the project name.
2. Do not ask the user for any input.
3. Automatically choose a playful ASCII style that fits the project name and a Spring Boot development theme.
4. Keep the ASCII art readable in a normal terminal.
5. After the ASCII art, show a neatly formatted diagnostics section that displays useful Spring Boot banner placeholders and environment values.

## Placeholder content to include

Include a nicely formatted section that uses as many useful banner placeholders as are appropriate, including these where available:

- `${spring.application.name}`
- `${application.title}`
- `${application.version}`
- `${spring-boot.version}`
- `${spring.profiles.active}`
- `${server.port}`
- `${PID}`
- `${java.version}`
- `${java.vendor}`
- `${os.name}`
- `${os.version}`
- `${user.name}`

You may include additional safe and useful placeholders if they fit well.

## Formatting rules

- Make the banner visually fun, but keep it practical for developers
- Align labels so the diagnostics section is easy to scan
- Do not add code comments
- Do not modify Java source files unless absolutely necessary
- Only create or update `src/main/resources/banner.txt`
- If the project name is not obvious, infer it from the Maven artifactId, Gradle root project name, or the folder name
- Prefer a polished developer-friendly result over a giant banner

## Output behavior

- Create the file directly
- Then summarize what was added
- Mention which project name was detected
- Mention which placeholders were included