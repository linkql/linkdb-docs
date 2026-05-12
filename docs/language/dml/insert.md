# INSERT
Inserts one or more rows or documents into a table or collection.

```grammar title="Grammar"
insert_stmt     ::= INSERT INTO identifier ( '(' identifier ( ',' identifier )* ')' )?
                      ( with_free )?
                    insert_source
                    ( ON CONFLICT conflict_target conflict_action )?

insert_source   ::= select_stmt
                  | VALUES '(' insert_value ( ',' insert_value )* ')'
                        ( ',' '(' insert_value ( ',' insert_value )* ')' )*

insert_value    ::= expr | DEFAULT

with_free       ::= WITH FREE free_source

free_source     ::= '(' identifier ( ',' identifier )* ')'
                  | UNPACK '(' expr ')'
                  | UNPACK '(' ')'

conflict_target ::= '(' identifier ( ',' identifier )* ')'

conflict_action ::= DO NOTHING
                  | DO UPDATE SET conflict_set ( ',' conflict_set )*

conflict_set    ::= identifier '=' expr
                  | identifier '=' EXCLUDED '.' identifier
```

## Description
`INSERT INTO` adds one or more rows or documents to a table or collection. Values can
be provided directly with `VALUES`, or sourced from a `SELECT` statement.

For collections, free fields can be included in the insert using the `WITH FREE` clause.
Free fields are additional fields beyond the defined fields of the collection. If a field
is included in the column list that the engine does not recognize as a defined field, an
error is returned — use `WITH FREE` to explicitly declare free fields.

`ON CONFLICT` handles cases where the insert would violate a unique or primary key
constraint. The conflicting row can be silently skipped or updated with new values.

## Parameters

`identifier`
:   The name of the table or collection to insert into.

`'(' identifier (',' identifier)* ')'`
:   An optional column list specifying which defined fields to insert values for. If
    omitted, values must be provided for all defined fields in their defined order and
    free fields cannot be included.

`WITH FREE free_source`
:   Declares free fields to include in the insert. Only valid on collections. Three
    forms are supported:

    - **Named list** `( field, ... )` — use when the free field names are known upfront
    - **`UNPACK( expr )`** — pass a JSON literal or column reference to unpack as free
      fields. The engine resolves a type for each key in the JSON object. See
      [Functions](../../functions.md) for full `UNPACK()` behavior.
    - **`UNPACK()`** — no argument form, INSERT only. Signals the engine to take the
      last value in the `VALUES` list as the JSON object to unpack as free fields.

    Using `WITH FREE` on a table returns an error.

`VALUES`
:   Provides one or more rows of values to insert. Multiple rows are comma separated.
    Each value corresponds to a column in the column list, or all defined fields in order
    if no column list is provided. When using `UNPACK()` with no arguments, the last
    value in the list is treated as the JSON object to unpack as free fields.

`DEFAULT`
:   Uses the defined default value for that field. Returns an error if no default is
    defined.

`select_stmt`
:   Sources insert values from a `SELECT` statement. The result set must match the
    column list in order and type.

`ON CONFLICT conflict_target`
:   Specifies the unique or primary key column or columns that define a conflict.

`DO NOTHING`
:   Silently skips the insert if a conflict is detected.

`DO UPDATE SET`
:   Updates the conflicting row with the specified values if a conflict is detected.

`EXCLUDED`
:   References the values from the attempted insert. Available in `DO UPDATE SET`
    for both defined and free fields.

## Notes
- If no column list is provided, values must be given for all defined fields in their
  defined order. Free fields cannot be inserted without a column list and `WITH FREE`.
- `WITH FREE` is only valid on collections. Using it on a table returns an error.
- Unrecognized fields in the column list return an error. Use `WITH FREE` to declare
  free fields explicitly.
- `ON CONFLICT` only triggers on unique or primary key constraint violations.
- `EXCLUDED` references the values from the attempted insert and is available for
  both defined and free fields.
- Multiple rows can be inserted in a single statement by providing multiple value
  tuples.
- `UNPACK()` with no arguments is INSERT only and takes the last value in the `VALUES`
  list as the JSON object to unpack.

## Examples

```sql title="Basic insert into a table"
INSERT INTO users (username, first_name, last_name)
VALUES ('john', 'John', 'Smith');
```

```sql title="Insert multiple rows"
INSERT INTO users (username, first_name, last_name)
VALUES
    ('john', 'John', 'Smith'),
    ('jane', 'Jane', 'Smith');
```

```sql title="Insert with DEFAULT"
INSERT INTO users (username, first_name, last_name, phone)
VALUES ('john', 'John', 'Smith', DEFAULT);
```

```sql title="Insert into a collection with defined fields only"
INSERT INTO posts (user_id, content)
VALUES (1, 'Hello world');
```

```sql title="Insert with named free fields"
INSERT INTO posts (user_id, content) WITH FREE (mood, weather)
VALUES (1, 'Hello world', 'happy', 'sunny');
```

```sql title="Insert with UNPACK and JSON literal"
INSERT INTO posts (user_id, content) WITH FREE UNPACK('{"mood": "happy", "weather": "sunny"}')
VALUES (1, 'Hello world');
```

```sql title="Insert with UNPACK and JSON in VALUES"
INSERT INTO posts (user_id, content) WITH FREE UNPACK()
VALUES (1, 'Hello world', '{"mood": "happy", "weather": "sunny"}');
```

```sql title="Insert from a SELECT"
INSERT INTO archived_users (username, first_name, last_name)
SELECT users.username, users.first_name, users.last_name
FROM users
WHERE users.last_login < '2020-01-01';
```

```sql title="Insert from SELECT with UNPACK"
INSERT INTO posts (user_id, content) WITH FREE UNPACK(source.metadata)
SELECT source.user_id, source.content
FROM staging_posts AS source;
```

```sql title="ON CONFLICT DO NOTHING"
INSERT INTO users (user_id, username)
VALUES (1, 'john')
ON CONFLICT (user_id) DO NOTHING;
```

```sql title="ON CONFLICT DO UPDATE"
INSERT INTO users (user_id, username)
VALUES (1, 'john')
ON CONFLICT (user_id) DO UPDATE SET username = EXCLUDED.username;
```

```sql title="ON CONFLICT with free fields"
INSERT INTO posts (user_id, content) WITH FREE (mood)
VALUES (1, 'Hello world', 'happy')
ON CONFLICT (user_id) DO UPDATE SET
    content = EXCLUDED.content,
    mood = EXCLUDED.mood;
```

## See Also
[UPDATE](update.md), [DELETE](delete.md), [SELECT](select.md), [Functions](../../functions.md)

---
[← Back to DML](../index.md)
