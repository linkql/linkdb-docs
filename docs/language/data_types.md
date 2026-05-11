# Data Types
LinkQL provides a set of primitive and structured data types for use in both tables and collections. For collections, defined fields must declare a type. Free fields are typeless and schemaless by nature, only aquiring a type at runtime.

```title="Grammar"
data_type ::= INT
            | BIGINT
            | FLOAT
            | DECIMAL ( '(' integer_literal ',' integer_literal ')' )?
            | BOOLEAN
            | VARCHAR ( '(' integer_literal ')' )?
            | TEXT
            | DATE
            | TIME
            | DATETIME
            | UUID
            | JSON
            | JSONB
            | ARRAY '(' data_type ')'
```

## Primitives

| <div style="width:14em">Type</div>    | Description | Example |
|---------------------------------------|-------------|---------|
| `INT`                                 | Standard 32-bit integer | `42` |
| `BIGINT`                              | 64-bit integer for large values | `9999999999` |
| `FLOAT`                               | Floating point number | `3.14` |
| `DECIMAL(<size>, <d>)`                | Fixed precision decimal. `size` is total digits, `d` is digits after the decimal. Defaults to arbitrary precision if unspecified. | `DECIMAL(10, 2)` |
| `BOOLEAN`                             | True or false value | `true` |
| `VARCHAR(<length>)`                   | Fixed-length string | `VARCHAR(50)` |
| `TEXT`                                | String of any length | `'hello world'` |
| `DATE`                                | Date value in `YYYY-MM-DD` format | `'2024-01-15'` |
| `TIME`                                | Time value in `hh:mm:ss` format | `'14:30:00'` |
| `DATETIME`                            | Combined date and time in `YYYY-MM-DD hh:mm:ss` format | `'2024-01-15 14:30:00'` |
| `UUID`                                | Universally unique identifier | `'550e8400-e29b-41d4-a716-446655440000'` |

!!! note

    For performance sensitive use-cases, always specify `size` and `d` for `DECIMAL` types. Unspecified `DECIMAL` types use arbitrary precision which trades storage efficiency and compute speed for exactness.

## Structured Types

### `JSON`
Stores a freeform JSON blob as raw text. You can navigate into a JSON value at query-level using bracket notation, but individual keys are not indexable. Best suited for data where you need flexibility and do not need to query specific keys frequently

``` sql
-- Bracket notation navigates into the JSON at query time
SELECT users.profile_data['preferences']['theme'] AS preferred_theme
FROM users
WHERE profile_data['preferences']['theme'] = 'dark'
```

### `JSONB`
Stores JSON in a decomposed **binary** format. Individual keys are indexable and path queries are significantly more efficient than pure `JSON`. The bracket notation is identical and the differences are purely storage and performance related. The syntax does not change. Best suited for `JSON` data you intend to query or filter against regularly.

```sql
SELECT users::metadata['ip'] AS ip_address
FROM users
WHERE metadata['ip'] = '192.168.1.1'
```
!!! tip

    When in doubt, use `JSONB` over `JSON`. The performance benefits on read-heavy workloads outweigh the slightly higher write cost in most cases.

### `ARRAY`
Stores an ordered list of values of a single type. Nested arrays are valid but a **foreign key reference** cannot be defined on a nested array.

```sql
-- Single Type Array
tags ARRAY(VARCHAR(50))

-- Nested Array
matrix ARRAY(ARRAY(INT))

-- Foreign key on a top level array is valid
friend_ids ARRAY(INT) REFERENCES users(user_id)

-- Foreign key on a nexted array is not valid
nested_ids ARRAY(ARRAY(INT)) REFERENCES users(user_id) -- ERROR
```
