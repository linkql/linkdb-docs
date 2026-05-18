# WHERE
Filters rows and documents based on a condition.

```grammar title="Grammar"
where_clause    ::= WHERE expr

expr            ::= or_expr

or_expr         ::= and_expr ( OR and_expr )*

and_expr        ::= not_expr ( AND not_expr )*

not_expr        ::= NOT not_expr | comparison

comparison      comparison ::= additive ( operator additive )?
                             | additive IS NOT? NULL
                             | additive IS NOT? MISSING
                             | additive IS NOT? NULL OR MISSING
                             | additive NOT? BETWEEN additive AND additive
                             | additive NOT? IN '(' expr (',' expr)* ')'
                             | additive NOT? IN '(' select_stmt ')'
                             | additive NOT? LIKE string_literal
                             | additive NOT? ILIKE string_literal
                             | EXISTS '(' select_stmt ')'

operator        ::= '=' | '!=' | '<' | '<=' | '>' | '>='
```

## Description
The `WHERE` clause filters rows and documents based on a condition. Only rows and
documents where the condition evaluates to `true` are included in the result set.

LinkQL distinguishes between two types of absence:

- **`NULL`** — the field exists but has no value. Follows standard SQL `NULL` comparison
  rules. Documents where a field is `NULL` are included in comparisons.
- **`MISSING`** — the field was never present on the document at all. Collection only and
  read-side only. Documents where a field is `MISSING` are excluded from comparisons
  silently rather than erroring.

## Operators

| Operator | Description |
|----------|-------------|
| `=` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `<=` | Less than or equal to |
| `>` | Greater than |
| `>=` | Greater than or equal to |

## Conditions

### IS NULL / IS MISSING
Tests whether a value is `NULL`, `MISSING`, or either. The `NOT` modifier inverts
the condition.

```sql
-- Field exists but has no value
WHERE posts::likes IS NULL

-- Field was never present on the document
WHERE posts::likes IS MISSING

-- Field is either null or missing
WHERE posts::likes IS NULL OR MISSING

-- Negated forms
WHERE posts::likes IS NOT NULL
WHERE posts::likes IS NOT MISSING
WHERE posts::likes IS NOT NULL OR MISSING
```

### BETWEEN
Tests whether a value falls within an inclusive range.

```sql
WHERE users.age BETWEEN 18 AND 65
WHERE users.age NOT BETWEEN 18 AND 65
```

### IN
Tests whether a value matches any value in a list or subquery.

```sql
WHERE users.status IN ('active', 'pending')
WHERE users.status NOT IN ('banned', 'deleted')

-- With subquery
WHERE users.user_id IN (
    SELECT orders.user_id FROM orders WHERE orders.total > 100
)
```

### LIKE / ILIKE
Tests whether a string matches a pattern. `%` matches any sequence of characters.
`_` matches any single character. `LIKE` is case sensitive. `ILIKE` is case
insensitive.

```sql
-- Case sensitive
WHERE users.username LIKE 'john%'

-- Case insensitive
WHERE users.username ILIKE 'john%'

-- Single character wildcard
WHERE users.username LIKE 'j_hn'

-- Negated
WHERE users.username NOT LIKE 'admin%'
```

### EXISTS
Tests whether a subquery returns any rows.

```sql
WHERE EXISTS (
    SELECT posts::post_id FROM posts WHERE posts::user_id = users.user_id
)
```

### Logical Operators
Conditions can be combined with `AND`, `OR`, and `NOT`.

```sql
WHERE users.age > 18 AND users.status = 'active'
WHERE users.age < 18 OR users.status = 'inactive'
WHERE NOT users.status = 'banned'
```

## NULL and MISSING Behavior

| Scenario | Behavior |
|----------|----------|
| Table column compared to value | Standard SQL NULL rules apply |
| Collection defined field is `NULL` | Document included in comparison |
| Collection free field is `MISSING` | Document excluded silently |
| Collection free field compared with `>`, `<`, etc. | Document excluded if field is `MISSING` |

## Notes
- `MISSING` is a collection only concept. Table columns can never be `MISSING`.
- `MISSING` is read-side only. You cannot insert or update a field to `MISSING`.
- Documents where a free field is `MISSING` are excluded from comparisons silently
  rather than returning an error.
- `NULL` is considered a value for the purposes of `IS NULL OR MISSING` — use
  `IS NULL` to specifically test for null values.
- `!=` is the preferred not-equal operator. `<>` is also accepted and converted
  to `!=` internally.
- `BETWEEN` is inclusive on both ends — `BETWEEN 1 AND 10` is equivalent to
  `>= 1 AND <= 10`.
- `LIKE` patterns use `%` for any sequence of characters and `_` for a single
  character.
- `ILIKE` is case insensitive. `LIKE` is case sensitive.

## Examples

```sql title="Basic filter"
SELECT users.username
FROM users
WHERE users.status = 'active';
```

```sql title="Multiple conditions"
SELECT users.username
FROM users
WHERE users.age > 18 AND users.status = 'active';
```

```sql title="Filter on collection free field"
SELECT posts::content
FROM posts
WHERE posts::likes > 10;
```

```sql title="NULL and MISSING checks"
SELECT posts::content
FROM posts
WHERE posts::likes IS NOT NULL OR MISSING;
```

```sql title="Pattern matching"
SELECT users.username
FROM users
WHERE users.username ILIKE 'john%';
```

```sql title="IN with subquery"
SELECT users.username
FROM users
WHERE users.user_id IN (
    SELECT orders.user_id
    FROM orders
    WHERE orders.total > 100
);
```

```sql title="EXISTS"
SELECT users.username
FROM users
WHERE EXISTS (
    SELECT posts::post_id
    FROM posts
    WHERE posts::user_id = users.user_id
);
```

## See Also
[SELECT Items](select-items.md), [FROM](from.md), [Joins](joins.md)

---
[← Back to SELECT](../select.md)
