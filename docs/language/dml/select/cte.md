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

## Examples

```sql title="SELECT with CTE"
WITH cte AS (SELECT id FROM users) SELECT * FROM cte;
```

```sql title="SELECT with recursive CTE"
WITH RECURSIVE nums AS (
    (SELECT 1) UNION ALL (SELECT n + 1 FROM nums WHERE n < 5)
) SELECT * FROM nums;
```

## See Also
[SELECT](index.md), [LET](../let.md), [Subqueries](subqueries.md)

---
[← Back to SELECT](index.md)
