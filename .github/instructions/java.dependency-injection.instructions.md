---
applyTo: "**/*.java"
description: Java dependency injection rules. Use constructor injection with final fields. No field injection, no setter injection. Applies to Spring, Jakarta EE, and any DI framework.
---

# Java Dependency Injection Instructions

These rules apply to **every Java class** that receives collaborators from a DI container (Spring, Jakarta CDI, Micronaut, Quarkus, etc.).

## Rules

1. **Use constructor injection only.**
   - Do **not** use field injection (`@Autowired` on a field, `@Inject` on a field).
   - Do **not** use setter injection.
2. **All injected dependencies must be `private final` fields.**
3. **Do not annotate the constructor with `@Autowired`** when the class has a single constructor — Spring detects it automatically.
4. **If the project uses Lombok**, prefer `@RequiredArgsConstructor` to generate the constructor for `final` fields.
5. **When extending a base class**, pass the parent's dependencies through `super(...)` from the child constructor.
6. **Do not mix injection styles** in the same class.

## Why constructor injection

- Makes dependencies explicit and visible at the class API level.
- Allows fields to be `final` — the object is fully initialized and immutable after construction.
- Easy to unit test without a DI container (just call `new UserService(mock)`).
- Fails fast at startup if a required bean is missing, instead of throwing `NullPointerException` at runtime.
- Prevents circular dependencies from being silently accepted.

## Examples

### Standard case

```java
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### With Lombok

```java
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final EmailSender emailSender;
}
```

### When extending a base class

```java
public class UserService extends BaseService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository, OtherDependency otherDependency) {
        super(otherDependency);
        this.userRepository = userRepository;
    }
}
```

## Anti-patterns to reject

```java
// BAD: field injection
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

```java
// BAD: setter injection
@Service
public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

```java
// BAD: non-final field even with constructor injection
public class UserService {
    private UserRepository userRepository; // must be `final`

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

## Optional dependencies

If a dependency is truly optional, inject it as `Optional<T>` (Spring) or use a sensible default in a configuration `@Bean`. Do **not** introduce setter injection just to make a dependency optional.

```java
public class UserService {

    private final UserRepository userRepository;
    private final Optional<AuditLog> auditLog;

    public UserService(UserRepository userRepository, Optional<AuditLog> auditLog) {
        this.userRepository = userRepository;
        this.auditLog = auditLog;
    }
}
```