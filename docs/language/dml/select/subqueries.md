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

## Correlated Subqueries
A subquery can reference columns of the queries that enclose it, at any depth. A column
name is looked up in the subquery's own `FROM` clause first and only then in the
enclosing query, working outward. A name the subquery itself has therefore always refers
to the subquery's own column; qualify it with the outer table's name to reach the outer
one instead.

```sql
-- Correlated EXISTS
SELECT users.username
FROM users
WHERE EXISTS (
    SELECT orders.order_id FROM orders WHERE orders.user_id = users.user_id
);

-- Correlated scalar subquery in the select list
SELECT users.username,
       (SELECT MAX(orders.total) FROM orders WHERE orders.user_id = users.user_id)
           AS largest_order
FROM users;
```

## Subqueries in FROM
A subquery in the `FROM` clause is a derived table. It can reference columns of the
queries that enclose it, but not the other items in its own `FROM` clause. Prefix it with
`LATERAL` to let it reference the items listed before it.

```sql
-- The most recent order for each user
SELECT users.username, latest.total
FROM users
CROSS JOIN LATERAL (
    SELECT orders.total
    FROM orders
    WHERE orders.user_id = users.user_id
    ORDER BY orders.created_at DESC
    LIMIT 1
) AS latest;
```

Without `LATERAL`, `users` is not visible inside the subquery and referencing it is an
error.

## Column Count
`IN` and the quantified comparisons (`ANY`, `SOME`, `ALL`) compare a value against every
row the subquery returns, so the subquery must return as many columns as the value has:
one for an ordinary expression, or one per item for a row expression. A mismatch is an
error. `EXISTS` does not care how many columns its subquery returns, and a scalar
subquery may return several when it is compared with a row expression.

```sql
SELECT users.username
FROM users
WHERE (users.first_name, users.last_name) IN (
    SELECT admins.first_name, admins.last_name FROM admins
);
```

A subquery that selects a collection wildcard (`posts::*`) is not checked, because its
free fields are not known in advance.

## WITH Inside Subqueries
A subquery, derived table, or CTE body can start with its own `WITH` clause. CTEs
defined by the enclosing statement, and `LET` bindings, are visible inside subqueries at
any depth. A name defined by an inner `WITH` hides an outer one with the same name, and
is not visible outside its own subquery.

```sql
-- A CTE from the enclosing statement, used inside a subquery
WITH big AS (SELECT orders.user_id FROM orders WHERE orders.total > 1000)
SELECT users.username
FROM users
WHERE users.user_id IN (SELECT big.user_id FROM big);

-- A subquery with its own WITH clause
SELECT users.username
FROM users
WHERE users.user_id IN (
    WITH big AS (SELECT orders.user_id FROM orders WHERE orders.total > 1000)
    SELECT big.user_id FROM big
);
```

A `WITH` clause cannot appear inside the body of a recursive CTE.

## Notes
- Subqueries in the `FROM` clause require an alias.
- Correlated subqueries reference columns from the outer query. Names resolve from the
  innermost query outward.
- A derived table sees the enclosing queries; only a `LATERAL` derived table also sees
  the `FROM` items before it.
- `IN` and quantified comparisons require the subquery to return as many columns as the
  value being compared.
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
