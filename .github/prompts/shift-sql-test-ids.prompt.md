---
mode: agent
description: Renumber the `id` values in a SQL INSERT file so they continue after the biggest existing id for each table found under src/test/resources.
---

# Shift SQL Test Data IDs

You are a code transformation engine.

## Inputs

- `${input:filePath}` — absolute or workspace-relative path to a SQL file with `INSERT` statements that the user wants renumbered.

## Goal

For every table referenced in `${input:filePath}`, find the biggest existing `id`
in all other SQL files under `src/test/resources` (recursive), and renumber the
IDs in `${input:filePath}` so they continue **strictly after** that biggest id,
preserving the original order.

## Behavior

1. **Read the input file** at `${input:filePath}`.
2. **Detect every `INSERT INTO <table> ...` statement** in that file.
   - Match the table name (case-insensitive).
   - Detect the position of the `id` column from the column list of the INSERT.
   - The `id` column may not be the first column — locate it by name.
3. **For each distinct table** found in the input file:
   - Search all `*.sql` files under `src/test/resources/**/*.sql` **except** the
     input file itself.
   - Parse every `INSERT INTO <same_table> ...` statement and collect the value
     of the `id` column.
   - Compute `MAX_ID(table)` = the biggest integer id seen for that table.
   - If no existing rows are found for a table, treat `MAX_ID(table)` as `0`.
4. **Renumber the input file**:
   - Walk the `INSERT` statements in the input file in their original order.
   - For each table, replace the original `id` value with `MAX_ID(table) + N`,
     where `N` starts at `1` and increments by `1` for each subsequent INSERT
     into that table.
   - **Do not change** any other column value, formatting, comment, or line.
5. **If the input file references the new id elsewhere** (for example a foreign
   key in another INSERT in the same file), update those references to keep the
   relations valid. Match by the original id value used in the input file.
6. **Save the file in place** (overwrite `${input:filePath}`).
7. **Report a short summary**: per table, the old id range, the detected
   `MAX_ID`, and the new id range.

## Rules

- Only touch the file at `${input:filePath}`. Do not modify any other SQL file.
- Do not reorder rows.
- Do not add or remove statements.
- Do not change column lists, table names, quoting, or whitespace beyond the id values.
- Treat ids as integers. If an id is non-numeric (e.g. UUID), skip that table
  and report it in the summary.
- Be case-insensitive when matching table and column names.
- Ignore SQL comments (`-- ...` and `/* ... */`) when scanning for ids.

## Example

Input file `src/test/resources/db/migration/test-data-new.sql`:

```sql
INSERT INTO users (id, name) VALUES (1, 'Alice');
INSERT INTO users (id, name) VALUES (2, 'Bob');
INSERT INTO orders (id, user_id, total) VALUES (1, 1, 50);
```

Other files under `src/test/resources` already contain:

- `users` → biggest id is `100`
- `orders` → biggest id is `42`

Expected output (file rewritten in place):

```sql
INSERT INTO users (id, name) VALUES (101, 'Alice');
INSERT INTO users (id, name) VALUES (102, 'Bob');
INSERT INTO orders (id, user_id, total) VALUES (43, 101, 50);
```

Note how `orders.user_id = 1` was updated to `101` because it referenced the old
id of `Alice` inside the same file.

Expected summary printed in chat:

```
users:  old ids 1..2  -> new ids 101..102  (MAX_ID was 100)
orders: old ids 1..1  -> new ids 43..43    (MAX_ID was 42)
```