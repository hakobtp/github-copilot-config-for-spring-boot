---
applyTo: "**/*.java"
description: Java logging rules. Use SLF4J via @Slf4j (Lombok) or a private static final Logger. Wrap every parameter placeholder in square brackets for log parsing. Always use parameterized logging. Never log secrets.
---

# Java Logging Instructions

These rules apply whenever a class needs to emit log output, or whenever a logger
is added, modified, or removed.

## How to obtain the logger

1. **Detect the logging stack** by inspecting the project:
   - `pom.xml` / `build.gradle` for one of:
     `logback-classic`, `log4j-core` (Log4j 2), `slf4j-api`,
     `spring-boot-starter-logging` (which brings in Logback by default).
   - Configuration files: `logback.xml`, `logback-spring.xml`, `log4j2.xml`,
     `log4j2.properties`, `simplelogger.properties`.
2. **Pick the logger declaration form**:
   - **Lombok present** → use `@Slf4j` on the class. The field is named `log`.
   - **No Lombok** → declare a standard SLF4J logger:
     `private static final Logger log = LoggerFactory.getLogger(MyClass.class);`
   - **No SLF4J / no logging framework at all** → fall back to
     `java.util.logging.Logger`, but **prefer to add SLF4J** to the project first.
3. **Field name must always be `log`** (not `LOG`, not `logger`) so messages look
   the same regardless of how the field was generated.

## Message format — the `[ ]` rule

Every parameter placeholder must be **wrapped in square brackets** in the log
message. This is the project convention for structured log parsing.

```java
log.info("User created [{}] with id [{}]", username, userId);
```

**Why:** downstream log parsers extract values by matching `[...]`. Removing
the brackets, replacing them with `()` or `<>`, or moving the value outside the
brackets all break parsing.

## Rules

1. **Always use parameterized logging.** Never use string concatenation or
   `String.format` to build log messages. SLF4J does the substitution lazily
   and only when the level is enabled.
2. **Wrap every placeholder in `[ ]`.** One placeholder = one `[{}]` group.
3. **Pass the throwable as the last argument** when logging an exception — do
   **not** put it inside the placeholder list:
   `log.error("Failed to save user [{}]", userId, ex);`
4. **Pick the right level**:
   - `error` — operation failed, manual intervention may be needed.
   - `warn`  — unexpected condition the application can recover from.
   - `info`  — significant lifecycle / business events.
   - `debug` — diagnostic data useful while developing or troubleshooting.
   - `trace` — very verbose, low-level execution flow.
5. **Guard expensive arguments with `isDebugEnabled()` / `isTraceEnabled()`**
   only when computing the value is genuinely costly (e.g. serializing a large
   object). Otherwise the parameterized form is enough.
6. **Never log secrets.** This includes passwords, tokens, API keys, JWTs,
   session IDs, full credit-card numbers, social-security / personal-ID numbers,
   email bodies, request/response bodies that may contain PII. Mask or omit.
7. **Do not log inside tight loops** at `info` level or above.
8. **Do not use `System.out.println` or `System.err.println` for logging.**

## Examples

### With Lombok

```java
@Slf4j
@Service
public class UserService {

    public void createUser(String username) {
        log.info("Creating user [{}]", username);
    }
}
```

### Without Lombok

```java
public class UserService {

    private static final Logger log = LoggerFactory.getLogger(UserService.class);

    public void createUser(String username) {
        log.info("Creating user [{}]", username);
    }
}
```

### Multiple parameters

```java
log.info("Order placed by user [{}] for product [{}] amount [{}]",
        userId, productId, amount);
```

### Logging exceptions

```java
try {
    repository.save(user);
} catch (DataAccessException ex) {
    log.error("Failed to save user [{}]", user.getId(), ex);
}
```

### Expensive-to-build debug message

```java
if (log.isDebugEnabled()) {
    log.debug("Resolved user graph [{}]", expensiveSerialize(userGraph));
}
```

### Masking sensitive data

```java
log.info("Login attempt for user [{}]", username);
// password is never logged

log.debug("Token issued ending in [{}]", token.substring(token.length() - 4));
```

## Anti-patterns to reject

```java
// BAD: string concatenation
log.info("Creating user " + username);
```

```java
// BAD: String.format
log.info(String.format("Creating user [%s]", username));
```

```java
// BAD: missing [ ] around the placeholder
log.info("Creating user {}", username);
```

```java
// BAD: variable outside the brackets
log.info("Creating user []" + username);
```

```java
// BAD: throwable inside the placeholder list — its message is rendered into the text instead of being attached as a stack trace
log.error("Failed to save user [{}] error [{}]", userId, ex);
```

```java
// BAD: logging a secret
log.info("Auth header [{}]", request.getHeader("Authorization"));
```

```java
// BAD: System.out used as a logger
public void save(Order order) {
    System.out.println("Saving order " + order.getId());
}
```

```java
// BAD: field named LOGGER instead of log — inconsistent with @Slf4j
private static final Logger LOGGER = LoggerFactory.getLogger(UserService.class);
```

## Imports cheat sheet

```java
import lombok.extern.slf4j.Slf4j;                       // Lombok form
import org.slf4j.Logger;                                // standard form
import org.slf4j.LoggerFactory;                         // standard form
```

Avoid:

```java
import java.util.logging.Logger;          // JUL — only as a last resort
import org.apache.commons.logging.Log;    // legacy commons-logging
```