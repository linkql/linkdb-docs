# ORDER BY / LIMIT
Sorts and paginates the result set.

```grammar title="Grammar"
order_clause    ::= ORDER BY order_item (',' order_item)*

order_item      ::= expr ( ASC | DESC )?

limit_clause    ::= LIMIT integer_literal ( OFFSET integer_literal )?
```

## Description
`ORDER BY` sorts the result set by one or more expressions. `LIMIT` restricts the
number of rows returned. `OFFSET` skips a number of rows before returning results.
Both are commonly used together for pagination.

## ORDER BY

Rows are sorted by the specified expressions. The default sort order is `ASC`
(ascending) if not specified. Multiple expressions are sorted left to right — the
first expression is the primary sort, subsequent expressions are used to break ties.

```sql
-- Single column ascending (default)
SELECT users.username FROM users ORDER BY users.username;

-- Single column descending
SELECT users.username FROM users ORDER BY users.created_at DESC;

-- Multiple columns
SELECT users.username FROM users
ORDER BY users.country ASC, users.username ASC;
```

## LIMIT and OFFSET

`LIMIT` restricts the number of rows returned. `OFFSET` skips a number of rows
before returning results. Together they enable pagination.

```sql
-- Return first 10 rows
SELECT users.username FROM users LIMIT 10;

-- Return rows 11-20
SELECT users.username FROM users LIMIT 10 OFFSET 10;
```

## NULL and MISSING Behavior
- **`NULL`** values sort last in `ASC` order and first in `DESC` order.
- **`MISSING`** values on collection free fields are excluded from the result set
  before sorting. They do not appear in the sorted output.

## Notes
- `ASC` is the default sort order and can be omitted.
- Multiple `ORDER BY` expressions are evaluated left to right.
- `OFFSET` requires `LIMIT` to be specified.
- `NULL` values sort last in ascending order and first in descending order.
- `MISSING` values are excluded before sorting.
- `LIMIT` and `OFFSET` are applied after all other clauses.

## Examples

```sql title="Basic ordering"
SELECT users.username, users.created_at
FROM users
ORDER BY users.created_at DESC;
```

```sql title="Multiple sort expressions"
SELECT users.username, users.country
FROM users
ORDER BY users.country ASC, users.username ASC;
```

```sql title="Limit results"
SELECT users.username
FROM users
ORDER BY users.created_at DESC
LIMIT 10;
```

```sql title="Pagination"
SELECT users.username
FROM users
ORDER BY users.created_at DESC
LIMIT 10 OFFSET 20;
```

```sql title="Order by collection free field"
SELECT posts::content, posts::likes
FROM posts
ORDER BY posts::likes DESC
LIMIT 20;
```

## See Also
[WHERE](where.md), [GROUP BY / HAVING](grouping.md), [SELECT Items](select-items.md)

---
[← Back to SELECT](../select.md)
