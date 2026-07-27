# Subqueries
Queries nested inside another statement.

```grammar title="Grammar"
from_item           ::= LATERAL? identifier ( AS? identifier )?
                      | LATERAL? '(' select_stmt ')' AS? identifier
                      | UNNEST '(' expr ( ',' expr )* ')'
                          ( WITH ORDINALITY )? ( AS? identifier )?
```

## Description
A subquery is a `SELECT` statement used inside another query. Subqueries can appear
in the `FROM` clause as derived tables, in `WHERE` predicates with `IN`, `EXISTS`, or
comparison operators, and in expressions.

## Parameters

`LATERAL`
:   Allows a subquery in the `FROM` clause to reference columns from preceding
    `FROM` items.

`'(' select_stmt ')'`
:   A subquery used as a derived table or expression.

`AS? identifier`
:   An optional alias for a derived table subquery.

## Notes
- Subqueries in the `FROM` clause require an alias.
- Correlated subqueries reference columns from the outer query.
- `EXISTS` and `IN` subqueries are commonly used in `WHERE` clauses.
- Quantified comparisons (`ANY`, `SOME`, `ALL`) compare a value against all rows
  returned by a subquery.
- Row expressions can be compared with subqueries using `IN` and equality.

## Examples

```sql title="Subquery as source"
SELECT active.username
FROM (
    SELECT users.username
    FROM users
    WHERE users.last_login > '2024-01-01'
) AS active;
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

```sql title="Quantified comparison with ANY"
SELECT * FROM users WHERE age > ANY (SELECT age FROM minors);
```

```sql title="Quantified comparison with SOME"
SELECT * FROM users WHERE score = SOME (SELECT score FROM top_players);
```

```sql title="Quantified comparison with ALL"
SELECT * FROM products WHERE price <= ALL (SELECT price FROM discounts);
```

```sql title="Row expression in IN list"
SELECT * FROM users WHERE (first_name, last_name) IN (
    SELECT first_name, last_name FROM admins
);
```

## See Also
[SELECT](index.md), [FROM](from.md), [WHERE](where.md), [CTEs](cte.md)

---
[← Back to SELECT](index.md)
