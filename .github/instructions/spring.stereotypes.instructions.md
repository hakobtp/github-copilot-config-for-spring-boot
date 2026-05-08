---
applyTo: "**/*.java"
description: Rules for declaring Spring beans. Choose the correct stereotype (@Service, @Repository, @Controller, @RestController, @Component), use constructor injection with final fields (@RequiredArgsConstructor when Lombok is available), apply @Transactional on service methods that touch repositories, and apply @Validated to beans that validate method arguments.
---

# Spring Stereotype Instructions

These rules apply whenever a Spring bean is created or modified.

## Choosing the right stereotype

Pick exactly **one** stereotype per class. All come from the
`org.springframework.stereotype` package (except web ones, see below).

| Stereotype        | When to use                                                                                            |
|-------------------|--------------------------------------------------------------------------------------------------------|
| `@Service`        | Class contains business logic / orchestration / domain rules.                                          |
| `@Repository`     | Class accesses persistence (JPA, JDBC, Mongo, etc.).                                                   |
| `@Controller`     | HTTP controller that returns view names (server-side rendering).                                       |
| `@RestController` | HTTP controller that returns JSON / response bodies. From `org.springframework.web.bind.annotation`.   |
| `@Component`      | Generic bean that does not fit any other category (utilities, config helpers, infrastructure adapters). |
| `@Configuration`  | Class that declares `@Bean` methods. Not a stereotype, but listed for clarity.                         |

**Do not stack stereotypes** (e.g. `@Service @Component` on the same class).
The more specific one already implies `@Component`.

## Dependency injection

This file follows the project rule from `java.dependency-injection.instructions.md`:

1. **Constructor injection only.** No field injection (`@Autowired` on a field),
   no setter injection.
2. **All injected dependencies are `private final` fields.**
3. **Single-constructor classes need no `@Autowired`** — Spring picks it up
   automatically.
4. **Lombok present** → use `@RequiredArgsConstructor` to generate the
   constructor for `final` fields.
5. **No Lombok** → write the constructor manually.

## Logging

Follows `java.logging.instructions.md`:

- **Lombok present** → `@Slf4j` on the class. Field name is `log`.
- **No Lombok** → `private static final Logger log = LoggerFactory.getLogger(MyClass.class);`
- Wrap every placeholder in `[ ]`.

## Transactions

- **`@Service` classes** that read or write through a repository **must annotate
  the method (preferred) or the class** with
  `@org.springframework.transaction.annotation.Transactional`.
- **Read-only methods** must use `@Transactional(readOnly = true)`.
- **Do not annotate `@Repository` classes with `@Transactional`** — repositories
  are participants, not transaction owners.
- **Do not annotate `@RestController` / `@Controller` with `@Transactional`** —
  transactions belong to the service layer, not the web layer.

## Validation

- **Annotate the bean with `@org.springframework.validation.annotation.Validated`**
  when at least one method takes arguments that should be validated with Jakarta
  Bean Validation constraints (e.g. `@NotNull`, `@Email`, `@Min`).
- For controllers, validation of `@RequestBody` DTOs uses `@Valid` on the
  parameter — `@Validated` on the class is needed when validating
  `@PathVariable` / `@RequestParam` directly.

## Examples

### Service with Lombok

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    @Transactional
    public UserDTO create(CreateUserRequest request) {
        log.info("Creating user [{}]", request.username());
        UserEntity saved = userRepository.save(toEntity(request));
        return toDto(saved);
    }

    @Transactional(readOnly = true)
    public UserDTO findById(Long id) {
        log.debug("Loading user [{}]", id);
        return userRepository.findById(id)
                .map(this::toDto)
                .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

### Service without Lombok

```java
@Service
public class UserService {

    private static final Logger log = LoggerFactory.getLogger(UserService.class);

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional
    public UserDTO create(CreateUserRequest request) {
        log.info("Creating user [{}]", request.username());
        UserEntity saved = userRepository.save(toEntity(request));
        return toDto(saved);
    }
}
```

### Repository

```java
@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {
    Optional<UserEntity> findByUsername(String username);
}
```

`@Repository` on a Spring Data interface is optional (Spring Data registers it
automatically), but adding it makes the role explicit.

### REST controller with parameter validation

```java
@RestController
@RequestMapping("/users")
@Validated
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping
    public UserDTO create(@Valid @RequestBody CreateUserRequest request) {
        return userService.create(request);
    }

    @GetMapping("/{id}")
    public UserDTO findById(@PathVariable @Min(1) Long id) {
        return userService.findById(id);
    }
}
```

### Generic component

```java
@Component
@RequiredArgsConstructor
public class CurrencyConverter {

    private final ExchangeRateClient exchangeRateClient;

    public BigDecimal convert(BigDecimal amount, String from, String to) {
        return amount.multiply(exchangeRateClient.rate(from, to));
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
// BAD: stacked stereotypes
@Service
@Component
public class UserService { }
```

```java
// BAD: @Component used where @Service is the right choice
@Component
public class UserService { }
```

```java
// BAD: @Transactional on a controller
@RestController
@Transactional
public class UserController { }
```

```java
// BAD: @Transactional on a repository
@Repository
@Transactional
public interface UserRepository extends JpaRepository<UserEntity, Long> { }
```

```java
// BAD: read method without readOnly = true
@Service
public class UserService {
    @Transactional
    public UserDTO findById(Long id) {
        return null;
    }
}
```

```java
// BAD: missing @Validated on a class with constraint annotations on parameters
@RestController
public class UserController {
    @GetMapping("/{id}")
    public UserDTO get(@PathVariable @Min(1) Long id) {
        return null;
    }
}
```

```java
// BAD: @Service on a DTO or entity
@Service
public class UserDTO {
    private String name;
}
```

## How to detect the right form

When generating a bean:

- **Lombok available?** Look for `org.projectlombok:lombok` in `pom.xml` /
  `build.gradle`. If yes, use `@Slf4j` + `@RequiredArgsConstructor`.
- **Persistence module imported?** If the project uses Spring Data, prefer
  repository interfaces over repository classes.
- **Web module imported?** Use `@RestController` for JSON APIs (the default for
  Spring Boot REST projects), `@Controller` only for view-rendering apps.

If unclear, default to the Lombok form (matches the rest of the project's
instructions).