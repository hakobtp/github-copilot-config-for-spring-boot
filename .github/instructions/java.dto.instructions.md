---
applyTo: "**/*.java"
description: Rules for creating Java DTO (Data Transfer Object) classes. Choose between record, plain class, or Lombok depending on Java version, project dependencies, and whether the DTO must be immutable.
---

# Java DTO Instructions

These rules apply whenever a DTO class is created, modified, or generated.

## Naming

- The class name **must end with the `DTO` suffix** (e.g. `UserDTO`, `OrderItemDTO`).
- Place DTOs in a package named `dto` (e.g. `com.acme.user.dto`).
- One DTO per file. Do not nest DTOs inside service or controller classes.

## Decision tree

Pick the right form for the DTO:

```
Need a DTO
│
├── Should it be immutable? ── YES ──► Java >= 16 ?
│                                       ├── YES ──► Use a record
│                                       └── NO  ──► Lombok present?
│                                                    ├── YES ──► @Data on a class with `final` fields
│                                                    └── NO  ──► Plain class with `final` fields + constructor + getters
│
└── Should it be mutable? ── YES ──► Lombok present?
                                       ├── YES ──► @Getter + @Setter (+ @Accessors(chain = true) when fluent setters are useful)
                                       └── NO  ──► Plain class with private fields, public getters, public setters
```

**Default to immutable.** Only create a mutable DTO when the user explicitly asks for one or when a framework requires it (e.g. some serializers).

## Rules

1. **Prefer records** for immutable DTOs whenever Java is 16 or newer.
2. **Use `@Data` from Lombok** only on classes whose fields are all `final` (i.e. immutable).
3. **Never use `@Data` on a mutable DTO** — it generates a `toString` and `equals/hashCode` over mutable state, which is dangerous in collections and logs. Use `@Getter` + `@Setter` instead.
4. **Do not add Spring annotations** (`@Component`, `@Service`, etc.) on DTOs. DTOs are plain data, not beans.
5. **Do not add JPA annotations** (`@Entity`, `@Table`, `@Column`) on DTOs. Keep DTOs separate from persistence entities.
6. **Use validation annotations** (`@NotNull`, `@Size`, `@Email`, ...) when the DTO is bound to an HTTP request body.
7. **Do not expose internal entity types** through a DTO. Map entity → DTO at the boundary (controller or facade).

## Examples

### 1. Immutable DTO — Java 16+ (preferred)

```java
public record UserDTO(String name, int age) {
}
```

With Bean Validation:

```java
public record UserDTO(
        @NotBlank String name,
        @Min(0) int age) {
}
```

### 2. Immutable DTO — Lombok available, Java < 16

```java
@Data
public class UserDTO {
    private final String name;
    private final int age;
}
```

`@Data` on a class with `final` fields generates getters, `equals`, `hashCode`, `toString`, and a required-args constructor. No setters are generated, so the object stays immutable.

### 3. Immutable DTO — no Lombok, Java < 16

```java
public class UserDTO {

    private final String name;
    private final int age;

    public UserDTO(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public boolean equals(Object o) { /* ... */ }

    @Override
    public int hashCode() { /* ... */ }

    @Override
    public String toString() { /* ... */ }
}
```

### 4. Mutable DTO — Lombok available

```java
@Getter
@Setter
@Accessors(chain = true)
public class UserDTO {
    private String name;
    private int age;
}
```

Usage:

```java
UserDTO dto = new UserDTO()
        .setName("Alice")
        .setAge(30);
```

### 5. Mutable DTO — no Lombok

```java
public class UserDTO {

    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

## Anti-patterns to reject

```java
// BAD: missing DTO suffix
public record User(String name, int age) {}
```

```java
// BAD: @Data on a mutable DTO — equals/hashCode over mutable state
@Data
public class UserDTO {
    private String name;
    private int age;
}
```

```java
// BAD: DTO mixed with persistence concerns
@Entity
@Table(name = "users")
public class UserDTO {
    @Id private Long id;
    private String name;
}
```

```java
// BAD: DTO turned into a Spring bean
@Component
public class UserDTO {
    private String name;
}
```

```java
// BAD: public mutable fields
public class UserDTO {
    public String name;
    public int age;
}
```

## How to detect the right form

When generating a DTO, check the project for these signals:

- **Java version**: read `<java.version>` / `<maven.compiler.release>` in `pom.xml`, or `sourceCompatibility` in `build.gradle`. If `>= 16`, use a record.
- **Lombok availability**: look for `org.projectlombok:lombok` in `pom.xml` or `build.gradle`. If present, prefer the Lombok form for non-record cases.
- **Validation availability**: look for `jakarta.validation` or `javax.validation`. If present and the DTO is a request body, add validation annotations.

If you cannot determine the Java version, **default to records** (the project's `java.instructions.md` already mandates Java 25).