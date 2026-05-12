---
name: RESTful API Creation and Design Best Practices
description: Use this skill when the user needs to build a REST API or create a RESTful API.
---

# RESTful API Design Best Practices

Use this skill whenever the user asks to "build a REST API" or "create a RESTful API".

## Rules

### Use nouns, not verbs

- Do NOT create endpoints like:
  - /add
  - /createUser
  - /getItems
- DO create resource based endpoints that use nouns such as:
  - /sum
  - /users
  - /orders

### Return simple types directly

When the result is a simple type like:
- int
- double
- String
- boolean
Return the raw value directly, NOT wrapped in JSON.

Good:
-42
-"Hello, World!"
-true

Avoid:

- { "result": 42 }