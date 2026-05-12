# UPDATE
Modifies existing rows or documents in a table or collection.

```grammar title="Grammar"
update_stmt     ::= UPDATE identifier
                    ( with_free )?
                    SET set_values
                    ( FROM from_clause )?
                    ( WHERE expr )?
                    ( CONFIRM string_literal )?

set_values      ::= set_expr ( ',' set_expr )*

set_expr        ::= identifier '=' expr
                  | identifier '=' EXCLUDED '.' identifier

with_free       ::= WITH FREE free_source

free_source     ::= '(' identifier ( ',' identifier )* ')'
                  | UNPACK '(' expr ')'
                  | UNPACK '(' ')'

from_clause     ::= from_item ( ',' join_clause )*

from_item       ::= identifier ( AS? identifier )?
                  | '(' select_stmt ')' AS identifier
```

## Description
`UPDATE` modifies one or more rows or documents in a table or collection. Rows can be
filtered with a `WHERE` clause. If no `WHERE` clause is provided, all rows or documents
in the table or collection are updated.

For collections, free fields can be updated using the `WITH FREE` clause. If a free
field being updated does not exist on a document, the engine adds it as a new free field
by default. This behavior can be controlled with the `ADD_MISSING_FREE_FIELDS` database
setting.

A `FROM` clause can be used to join against other tables or collections when determining
which rows to update.

## Parameters

`identifier`
:   The name of the table or collection to update.

`WITH FREE free_source`
:   Declares free fields to update. Only valid on collections. Three forms are supported:

    - **Named list** `( field, ... )` — use when the free field names are known upfront
    - **`UNPACK( expr )`** — pass a JSON literal or column reference to unpack as free
      fields. See [Functions](../../functions.md) for full `UNPACK()` behavior.
    - **`UNPACK()`** — not valid in UPDATE. Use `UNPACK( expr )` instead.

    Using `WITH FREE` on a table returns an error.

`SET set_values`
:   One or more field assignments. Each assignment sets a field to a new value. The
    engine resolves whether each field is defined or free automatically via the catalog.

`FROM from_clause`
:   Joins against one or more tables or collections when determining which rows to
    update. Follows the same syntax as the [`FROM`](select/from.md) clause in `SELECT`.

`WHERE expr`
:   Filters which rows or documents are updated. If omitted, all rows or documents in
    the table or collection are updated.

`CONFIRM string_literal`
:   A confirmation string that must match the name of the table or collection being
    updated. Required when `CONFIRM_BULK_OPERATIONS` is enabled on the database and no
    `WHERE` clause is provided.

## Free Field Behavior
When updating a free field on a collection:

| Scenario | Default Behavior | `ADD_MISSING_FREE_FIELDS = false` |
|----------|-----------------|-----------------------------------|
| Field exists on document | Updated normally | Updated normally |
| Field does not exist on document | Added as new free field | Document skipped |
| Field is defined in catalog | Updated normally | Updated normally |

## Database Settings

`CONFIRM_BULK_OPERATIONS`
:   When enabled, requires a `CONFIRM` clause on any `UPDATE` statement with no `WHERE`
    clause. Disabled by default.

`ADD_MISSING_FREE_FIELDS`
:   When enabled, adds free fields to documents that don't have them during an `UPDATE`.
    When disabled, documents missing the free field are skipped. Enabled by default.

## Notes
- If no `WHERE` clause is provided, all rows or documents are updated.
- The engine resolves whether a field is defined or free automatically via the catalog.
- `WITH FREE` is only valid on collections. Using it on a table returns an error.
- `UNPACK()` with no arguments is not valid in UPDATE — use `UNPACK( expr )` instead.
- `FROM` follows the same syntax as the `SELECT` `FROM` clause including shorthand
  and traditional join syntax.
- Free fields that don't exist on a document are added by default. Set
  `ADD_MISSING_FREE_FIELDS = false` to skip those documents instead.

## Examples

```sql title="Basic update"
UPDATE users
SET status = 'inactive'
WHERE users.last_login < '2020-01-01';
```

```sql title="Update multiple fields"
UPDATE users
SET first_name = 'John', last_name = 'Smith'
WHERE user_id = 1;
```

```sql title="Update all rows"
UPDATE users
SET status = 'active';
```

```sql title="Update all rows with confirmation"
UPDATE users
SET status = 'active'
CONFIRM 'users';
```

```sql title="Update collection defined fields"
UPDATE posts
SET content = 'Hello world'
WHERE posts::user_id = 1;
```

```sql title="Update collection with free fields"
UPDATE posts WITH FREE (mood, weather)
SET content = 'Hello world', mood = 'happy', weather = 'sunny'
WHERE posts::user_id = 1;
```

```sql title="Update collection with UNPACK"
UPDATE posts WITH FREE UNPACK('{"mood": "happy", "weather": "sunny"}')
SET content = 'Hello world'
WHERE posts::user_id = 1;
```

```sql title="Update with FROM clause"
UPDATE users
SET status = 'inactive'
FROM sessions
WHERE sessions.user_id = users.user_id
AND sessions.last_active < '2020-01-01';
```

```sql title="Update with JOIN in FROM"
UPDATE posts
SET content = 'archived'
FROM users, LEFT sessions(user_id)
WHERE posts::user_id = users.user_id
AND sessions.last_active < '2020-01-01';
```

## See Also
[INSERT](insert.md), [DELETE](delete.md), [SELECT](select.md), [Functions](../../functions.md)

---
[← Back to DML](../index.md)
