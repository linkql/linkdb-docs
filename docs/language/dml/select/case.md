# CASE
Evaluates a series of conditions and returns a value based on the first match.

```grammar title="Grammar"
case_expr           ::= standard_case | shorthand_case

standard_case       ::= CASE when_clause+ ( ELSE expr )? END

when_clause         ::= WHEN expr THEN expr

shorthand_case      ::= CASE shorthand_clause ( ',' shorthand_clause )* ( ELSE expr )? END

shorthand_clause    ::= expr '?' expr
```

## Description
`CASE` is a conditional expression that evaluates a series of conditions and returns
the value of the first matching condition. If no condition matches and an `ELSE` clause
is provided, the `ELSE` value is returned. If no condition matches and there is no
`ELSE`, `NULL` is returned.

LinkQL supports two forms of `CASE` — standard and shorthand. Both forms are
equivalent in behavior. The shorthand form is designed for brevity.

`CASE` can be used anywhere an expression is valid — in `SELECT`, `WHERE`, `HAVING`,
`ORDER BY`, and as part of other expressions including nested `CASE` expressions.

## Standard CASE
The standard form uses `WHEN` and `THEN` keywords. Each clause evaluates a condition
and returns a value if the condition is true.

```sql
CASE
    WHEN users.age > 65 THEN 'senior'
    WHEN users.age > 18 THEN 'adult'
    ELSE 'minor'
END
```

## Shorthand CASE
The shorthand form uses `?` instead of `WHEN` and `THEN`. Conditions and values are
comma separated. No trailing commas allowed.

```sql
CASE
    users.age > 65 ? 'senior',
    users.age > 18 ? 'adult'
    ELSE 'minor'
END
```

## ELSE
Both forms support an optional `ELSE` clause that returns a value when no condition
matches. If `ELSE` is omitted and no condition matches, `NULL` is returned.

## Nested CASE
`CASE` expressions can be nested since `CASE` is a valid expression anywhere an
expression is accepted. Nesting is valid in both standard and shorthand forms.

```sql
CASE
    WHEN users.age > 18 THEN
        CASE
            WHEN users.age > 65 THEN 'senior'
            ELSE 'adult'
        END
    ELSE 'minor'
END
```

## Notes
- Conditions are evaluated left to right. The first matching condition is returned.
- If no condition matches and `ELSE` is omitted, `NULL` is returned.
- The trailing comma on the last shorthand clause is optional.
- Both forms are equivalent in behavior. Use whichever is more readable for the
  situation.
- `CASE` can be used anywhere an expression is valid.

## Examples

```sql title="Standard CASE in SELECT"
SELECT
    users.username,
    CASE
        WHEN users.age > 65 THEN 'senior'
        WHEN users.age > 18 THEN 'adult'
        ELSE 'minor'
    END AS age_group
FROM users;
```

```sql title="Shorthand CASE in SELECT"
SELECT
    users.username,
    CASE
        users.age > 65 ? 'senior',
        users.age > 18 ? 'adult',
        ELSE 'minor'
    END AS age_group
FROM users;
```

```sql title="CASE in WHERE"
SELECT users.username
FROM users
WHERE
    CASE
        WHEN users.status = 'active' THEN users.age > 18
        ELSE users.age > 21
    END;
```

```sql title="CASE in ORDER BY"
SELECT users.username, users.status
FROM users
ORDER BY
    CASE
        WHEN users.status = 'active' THEN 1
        WHEN users.status = 'pending' THEN 2
        ELSE 3
    END ASC;
```

```sql title="Nested CASE"
SELECT
    users.username,
    CASE
        WHEN users.age > 18 THEN
            CASE
                WHEN users.age > 65 THEN 'senior'
                ELSE 'adult'
            END
        ELSE 'minor'
    END AS age_group
FROM users;
```

```sql title="CASE with collection fields"
SELECT
    posts::content,
    CASE
        posts::status = 'published' ? 'live',
        posts::status = 'draft' ? 'in progress',
        ELSE 'unknown'
    END AS post_status
FROM posts;
```

## See Also
[SELECT Items](select-items.md), [WHERE](where.md), [Functions](../../functions.md)

---
[← Back to SELECT](../select.md)
