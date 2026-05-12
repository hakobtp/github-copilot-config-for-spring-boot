# Git Commit Message Instructions

These rules tell GitHub Copilot how to generate commit messages when the user clicks
**"Generate commit message with Copilot"** in the IntelliJ IDEA commit dialog.

## Context

The user always types the **first line** of the commit message manually in the
IntelliJ IDEA commit message box. That first line follows this format:

```
<TICKET-ID>: [<TAG>] <Short title>
```

Example provided by the user:

```
MPTX-1234: [BE] Add Save action and change request creation TG generation
```

- `TICKET-ID` — the issue tracker ID (e.g. `MPTX-1234`, `JIRA-42`).
- `TAG` — short scope tag in square brackets (e.g. `[BE]` for backend, `[FE]` for frontend, `[INFRA]`, `[DOCS]`).
- `Short title` — imperative, human-written summary of the work.

## Rules for Copilot

1. **Never modify, rewrite, or "fix" the first line.**
   - Do not correct typos in the title.
   - Do not change the ticket ID, the tag, or the wording.
   - Treat the user-typed first line as the authoritative subject.

2. **Add a generated body below the first line.**
   - Leave one blank line between the title and the body.
   - The body must describe what changed, derived **only from the staged files**
     (files actually included in the commit).

3. **Format of the generated body.**
   - Use a bulleted list with `-`.
   - Each bullet describes one logical change, not one file.
   - Group related file changes into a single bullet when they belong to the same change.
   - Keep each bullet under ~100 characters when possible.
   - Use the imperative mood: "Add ...", "Remove ...", "Update ...", "Refactor ...", "Fix ...".
   - Mention the relevant class / module / endpoint name when it adds clarity.

4. **Do not invent context.**
   - Do not describe changes that are not visible in the staged diff.
   - Do not reference files that are not part of the commit.
   - Do not speculate about *why* the change was made unless the diff makes it obvious.

5. **Do not repeat the first line in the body.**

6. **Do not add trailing metadata** (no `Signed-off-by`, no `Co-Authored-By`, no emojis,
   no AI-generated tag) unless the user explicitly asks for it.

## Output template

```
<user-typed first line — keep exactly as written>

- <change 1 derived from staged files>
- <change 2 derived from staged files>
- <change 3 derived from staged files>
```

## Worked example

User typed in IntelliJ:

```
MPTX-1234: [BE] Add Save action and change request creation TG generation
```

Staged files:

- `src/main/java/com/acme/order/SaveOrderAction.java` (new)
- `src/main/java/com/acme/order/OrderController.java` (modified)
- `src/main/resources/templates/tg/change-request.tg` (new)
- `src/test/java/com/acme/order/SaveOrderActionTest.java` (new)

Expected generated commit message:

```
MPTX-1234: [BE] Add Save action and change request creation TG generation

- Add SaveOrderAction to handle order save flow
- Wire SaveOrderAction into OrderController
- Add change-request.tg template for TG generation
- Add unit tests for SaveOrderAction
```

## What NOT to produce

- Do **not** rewrite the first line:
  ```
  fix(MPTX-1234): add save action   ← WRONG, user's title was changed
  ```
- Do **not** list raw file paths without describing the change:
  ```
  - SaveOrderAction.java             ← WRONG, no description
  - OrderController.java
  ```
- Do **not** add unrequested footers:
  ```
  Signed-off-by: ...                 ← WRONG, not requested
  Co-Authored-By: Copilot            ← WRONG, not requested
  ```