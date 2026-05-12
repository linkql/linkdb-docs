# JSON Querying
Querying and manipulating JSON and JSONB values in LinkQL.

```grammar title="Grammar"
json_access     ::= expr '[' string_literal ']' ( '[' string_literal ']' )*

json_function   ::= HAS '(' expr ',' string_literal ')'
                  | KEYS '(' expr ')'
                  | VALUES '(' expr ')'
                  | MERGE '(' expr ',' expr ( ',' ( LEFT | RIGHT ) )? ')'
                  | CONTAINS '(' expr ',' expr ')'
                  | UNPACK '(' expr ')'
                  | UNPACK '(' ')'
```

## Description
LinkQL provides bracket notation and a set of functions for querying and manipulating
JSON and JSONB values. Bracket notation works on both `JSON` and `JSONB` columns and
fields — the syntax is identical, the difference is purely storage and performance.
See [Data Types](data_types.md) for the distinction between `JSON` and `JSONB`.

Bracket notation and JSON functions work on both table columns and collection fields:

```sql
-- Table column
users.profile_data['preferences']['theme']

-- Collection field
posts::metadata['ip']
```

## Bracket Notation
Bracket notation navigates into a JSON value by key. Keys are string literals.
Notation can be chained to navigate nested objects.

```sql
-- Single key
users.profile_data['theme']

-- Nested keys
users.profile_data['preferences']['theme']

-- On a collection field
posts::metadata['preferences']['theme']
```

### NULL and MISSING Behavior
- If a key exists but its value is `null`, bracket notation returns `NULL`.
- If a key does not exist at any point in the path, bracket notation returns `MISSING`.

```sql
-- Key exists, value is null → NULL
users.profile_data['theme']  -- { "theme": null } → NULL

-- Key does not exist → MISSING
users.profile_data['theme']  -- { "preferences": {} } → MISSING
```

## JSON Functions

### HAS
Returns `true` if the specified key path exists in the JSON value, `false` otherwise.
Dot notation in the path checks the full depth — all keys in the path must exist for
`HAS` to return `true`.

```sql
-- Top level key
HAS(metadata, 'ip')

-- Nested path — true only if preferences exists AND theme exists within it
HAS(profile_data, 'preferences.theme')

-- On a collection field
HAS(posts::metadata, 'ip')
```

### KEYS
Returns all top level keys of a JSON object as an `ARRAY(TEXT)`.

```sql
KEYS(users.profile_data)

-- On a collection field
KEYS(posts::metadata)
```

### VALUES
Returns all top level values of a JSON object as an `ARRAY(JSON)`.

```sql
VALUES(users.profile_data)

-- On a collection field
VALUES(posts::metadata)
```

### MERGE
Merges two JSON objects together. If both objects share a key, the direction determines
which value is kept. Defaults to `RIGHT` — the second object's value wins on conflict.

```sql
-- Right wins by default
MERGE(json1, json2)

-- Explicit left — json1 wins on conflict
MERGE(json1, json2, LEFT)

-- Explicit right — json2 wins on conflict
MERGE(json1, json2, RIGHT)
```

### CONTAINS
Returns `true` if a JSON array contains the specified value, `false` otherwise.

```sql
-- Check if an array contains a value
CONTAINS(users.profile_data['tags'], 'admin')

-- On a collection field
CONTAINS(posts::tags, 'featured')
```

### UNPACK
Unpacks a JSON object into free fields. Each key in the JSON object becomes a free
field. The engine resolves a type for each value:

| JSON type | Resolved type |
|-----------|--------------|
| String | `TEXT` |
| Integer | `INT` |
| Decimal | `FLOAT` |
| Boolean | `BOOLEAN` |
| Array | `ARRAY` of resolved type |
| Object | `JSONB` |
| null | `NULL` |

If the engine cannot resolve a type, an error is returned.

```sql
-- JSON literal
UNPACK('{"mood": "happy", "weather": "sunny"}')

-- Column reference
UNPACK(source.metadata)
```

`UNPACK()` with no arguments is INSERT only — it takes the last value in the `VALUES`
list as the JSON object to unpack. See [INSERT](dml/insert.md) for details.

## Notes
- Bracket notation syntax is identical for `JSON` and `JSONB`. The difference is
  storage and performance only.
- A key that does not exist returns `MISSING`, not `NULL`. A key that exists with a
  null value returns `NULL`.
- `HAS` with a dot path checks the full depth — all keys in the path must exist.
- `MERGE` defaults to `RIGHT` — the second object's value wins on conflict.
- `KEYS` and `VALUES` only return top level keys and values.
- `UNPACK()` with no arguments is INSERT only.
- JSON functions work on both table columns and collection fields.

## Examples

```sql title="Bracket notation in SELECT"
SELECT
    users.username,
    users.profile_data['preferences']['theme'] AS preferred_theme
FROM users;
```

```sql title="Bracket notation in WHERE"
SELECT users.username
FROM users
WHERE users.profile_data['preferences']['theme'] = 'dark';
```

```sql title="HAS in a CASE expression"
SELECT
    users.username,
    CASE
        WHEN HAS(users.metadata, 'ip') THEN users.metadata['ip']
        ELSE 'n/a'
    END AS ip_address
FROM users;
```

```sql title="HAS with nested path"
SELECT users.username
FROM users
WHERE HAS(users.profile_data, 'preferences.theme');
```

```sql title="KEYS and VALUES"
SELECT
    KEYS(users.profile_data) AS profile_keys,
    VALUES(users.profile_data) AS profile_values
FROM users;
```

```sql title="MERGE two JSON objects"
SELECT MERGE(users.profile_data, users.metadata) AS combined
FROM users;
```

```sql title="MERGE with explicit direction"
SELECT MERGE(users.profile_data, users.metadata, LEFT) AS combined
FROM users;
```

```sql title="CONTAINS on a JSON array"
SELECT users.username
FROM users
WHERE CONTAINS(users.profile_data['tags'], 'admin');
```

```sql title="UNPACK in INSERT"
INSERT INTO posts (user_id, content) WITH FREE UNPACK('{"mood": "happy", "weather": "sunny"}')
VALUES (1, 'Hello world');
```

## See Also
[Data Types](data_types.md), [INSERT](dml/insert.md), [Functions](functions.md)

---
[← Back to Language](index.md)
