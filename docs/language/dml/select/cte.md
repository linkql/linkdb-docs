# Common Table Expressions (CTE)
Named subqueries defined with `WITH` or `WITH RECURSIVE`.

```grammar title="Grammar"
with_clause         ::= WITH ( RECURSIVE )? cte ( ',' cte )*

cte                 ::= identifier ( '(' identifier ( ',' identifier )* ')' )? AS '(' select_stmt ')'
```

## Description
A common table expression (CTE) gives a name to a subquery so it can be referenced
in the main `SELECT`. Recursive CTEs can reference themselves to build iterative
result sets.

## Parameters

`WITH`
:   Introduces one or more CTEs.

`RECURSIVE`
:   Allows a CTE to reference itself. Required for recursive CTEs.

`identifier`
:   The name of the CTE.

`'(' identifier (',' identifier)* ')'`
:   Optional explicit column list for the CTE. If omitted, column names are derived
    from the inner `SELECT`.

`AS '(' select_stmt ')'`
:   The query that defines the CTE.

## Notes
- CTEs can be used in `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and `MERGE` statements.
- Recursive CTEs must have an anchor query and a recursive query separated by a
  set operator such as `UNION ALL`.
- Column names for a recursive CTE can be declared explicitly in the CTE header.
- The anchor query and the recursive query of a recursive CTE must return the same
  number of columns.
- A CTE is visible in the whole statement, including inside its subqueries at any
  depth. See [Subqueries](subqueries.md#with-inside-subqueries).
- A CTE body can itself start with a `WITH` clause. Names defined there are visible only
  inside that body and hide outer names with the same name. A `WITH` clause cannot appear
  inside the body of a recursive CTE.
- A CTE defined with `SELECT *` exposes all of its columns to the query that uses it.

## Examples

```sql title="SELECT with CTE"
WITH cte AS (SELECT id FROM users) SELECT * FROM cte;
```

```sql title="SELECT with recursive CTE"
WITH RECURSIVE nums AS (
    (SELECT 1) UNION ALL (SELECT n + 1 FROM nums WHERE n < 5)
) SELECT * FROM nums;
```

```sql title="CTE used inside a subquery"
WITH big AS (SELECT orders.user_id FROM orders WHERE orders.total > 1000)
SELECT users.username
FROM users
WHERE users.user_id IN (SELECT big.user_id FROM big);
```

```sql title="CTE defined inside another CTE"
WITH active AS (
    WITH recent AS (SELECT orders.user_id FROM orders WHERE orders.total > 100)
    SELECT recent.user_id FROM recent
)
SELECT users.username FROM users WHERE users.user_id IN (SELECT active.user_id FROM active);
```

## See Also
[SELECT](index.md), [LET](../let.md), [Subqueries](subqueries.md)

---
[← Back to SELECT](index.md)
