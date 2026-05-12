---
mode: Agent
description: Generate a Maven multi-module project structure based on input parameters.
---

# Behavior Rules

You are a software architect and team lead, and you must set up a Maven multi-module project.
Please follow the instructions below and design the correct project structure and workflow.


# Create Maven Modules

## Inputs

- PACKAGE_NAME: <base_package>
- SPRING_BOOT_VERSION: <spring_boot_version> default: 4.0.6
- JAVA_VERSION: <java_version> default: 25
- SPRING-MODULITH-VERSION: <spring-modulith.version> default: 2.0.6


Dependencies:
- Add spring boot `modulith` for modular architecture support.
- Add spring boot `web` for rest api.

The project must be structured as a multi-module Maven project with the following modules:

* application
* api
* common

> Note: Do not create any classes unless I explicitly request them. Only create package structures.

---

## Parent POM Configuration

* All modules must be declared in the main `pom.xml` (parent POM).
* The parent POM must define **dependency management** (e.g., versions, BOM imports).
* The parent POM must not include actual dependencies used by modules; it should only manage them.
* Shared build configuration (plugins, compiler settings, etc.) must be defined in the parent POM.

---

## Module Responsibilities

### application

* Contains the main class (entry point).
* Responsible for bootstrapping and configuring the entire application.
* Acts as the **composition root**.
* Includes the actual dependencies required to run the application.

### api

* Contains REST controllers and request/response DTOs.
* Responsible for handling HTTP requests and responses.
* Must not contain business logic.

### common

* Contains shared utilities, exceptions, constants, and base classes.
* Can be used by all modules.

---

## Inter-module Dependency Rules

* `application` → can depend on all modules
* `api` → can depend on `common`
* `common` → must not depend on any other module

### Forbidden Dependencies
* `common` must remain independent
* Avoid circular dependencies between modules

---

## Internal Package Structure


---

### api module

```
PACKAGE_NAME.api
├── controller        --> Contains REST controllers that handle HTTP requests and responses.
   ├── .gitkeep         These classes call the appropriate services in the domain modules
                        or the application module to process requests.

├── dto               --> Contains Data Transfer Objects (DTOs) specifically for API communication.
   ├── .gitkeep         These are often slightly different from domain DTOs, tailored for request
                        payloads or response formats. Example: UserResponseDto, UserCreateRequestDto.

├── mapper            --> Contains classes or interfaces that convert between domain objects
   ├── .gitkeep         (or domain DTOs) and API DTOs. For example, mapping User entity to UserResponseDto or vice versa.
```

---

### application module

```
PACKAGE_NAME.application
├── config              --> Contains global or module-level Spring configuration classes.
   ├── .gitkeep           Examples: security configuration, bean definitions, or shared module settings.
                          This is where you configure anything that other modules or the application 
                          itself may need at runtime.
├── bootstrap           --> Contains the main entry point of the application (the class with the main method)
   ├── .gitkeep           and any classes that are responsible for starting up the Spring Boot application.
                          It orchestrates initialization but does not contain domain logic.
```

---

### common module

```
PACKAGE_NAME.common
├── exception             --> Contains shared exception classes used across multiple modules.
   ├── .gitkeep             Examples: CustomNotFoundException, InvalidInputException.
                            Helps standardize error handling across the project.

├── util                  --> Contains utility or helper classes used by multiple modules.
   ├── .gitkeep             Examples: DateUtils, StringUtils, ValidationUtils.
                            Should be framework-agnostic and contain no business logic.

├── config                --> Contains shared configuration classes that can be reused across modules.
   ├── .gitkeep             Examples: common bean definitions, property loaders, shared converters.
```

---

## Key Architectural Principles

* Follow separation of concerns
* Use `application` as the composition root
* Prefer clear boundaries over convenience
* Design for scalability and maintainability


## Create Main Class

Under `PACKAGE_NAME.application.bootstrap`, create a class that scans the
main package and runs the Spring Boot application. This class must contain the `main` method.

## Dependencies and Module Encapsulation

### API Module
- The API module is fully encapsulated.
- Other modules must not depend on or use code from the API module.
- Within the API module, the controller package can access other internal packages (e.g., dto, mapper).

### Common Module
- The common module must be accessible to all modules.
  
## Module Verification (Testing)
- Add unit tests to verify module boundaries using Spring Modulith.
- Ensure that:
  - Only allowed dependencies exist between modules
  - Internal packages are not accessed from outside
  - No forbidden dependencies are introduced


