# Detailed rules

Deeper, scoped rules live in `.github/instructions/`. They are loaded automatically when you edit a matching file:

- [`java.dependency-injection.instructions.md`](instructions/java.dependency-injection.instructions.md) — DI rules and examples.
- [`java.dto.instructions.md`](instructions/java.dto.instructions.md) — DTO patterns (record / Lombok / plain).
- [`java.logging.instructions.md`](instructions/java.logging.instructions.md) — SLF4J usage and `[ ]` placeholder format.
- [`spring.stereotypes.instructions.md`](instructions/spring.stereotypes.instructions.md) — stereotype selection, `@Transactional`, `@Validated`.

## Reusable workflows

Repeatable actions live as prompts under `.github/prompts/` and are invoked from chat with `/<prompt-name>`. 
See `.github/README.md` for the full list and how to use them in IntelliJ IDEA.

## Commit messages

When generating commit messages, follow `.github/git-commit-instructions.md`.