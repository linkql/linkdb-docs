# Set Operations
Combine the results of two or more `SELECT` statements.

```grammar title="Grammar"
select_stmt         ::= with_clause? intersect_stmt
                          ( ( UNION | EXCEPT ) ALL? intersect_stmt )*
                          ( ORDER BY order_item ( ',' order_item )* )?
                          limit_clause?
                          for_clause?

intersect_stmt      ::= select_core ( INTERSECT ALL? select_core )*
```

## Description
Set operators combine the rows returned by two `SELECT` statements. By default,
set operations remove duplicate rows. Use `ALL` to retain duplicates.

## Parameters

`UNION`
:   Returns rows that appear in either query.

`INTERSECT`
:   Returns rows that appear in both queries.

`EXCEPT`
:   Returns rows from the first query that do not appear in the second.

`ALL`
:   Keeps duplicate rows. Without `ALL`, duplicates are removed.

## Notes
- The two queries must return the same number of columns with compatible types. A
  different number of columns is an error. The result's column names come from the
  first query. (Compatibility of column types is not checked yet.)
- Multiple set operators can be chained in a single statement.
- `INTERSECT` binds more tightly than `UNION` and `EXCEPT`.
- `ORDER BY` and `LIMIT` apply to the combined result of all set operations.
- Either side can be wrapped in parentheses. A parenthesized query keeps its own
  `ORDER BY` and `LIMIT`, and a parenthesized set operation can be an operand of
  another.

## ORDER BY on a Set Operation
An `ORDER BY` after the last query sorts the combined result, so it can only refer to
that result's columns, by name. The names come from the first query, so an alias given
there works. It cannot use a table-qualified name or an expression, and a name that
appears twice in the result is ambiguous.

```sql
SELECT username AS name FROM users_a
UNION
SELECT username FROM users_b
ORDER BY name;
```

To sort or limit one side before combining, parenthesize it and give it its own
`ORDER BY` and `LIMIT`.

```sql
(SELECT username FROM users_a ORDER BY username LIMIT 5)
UNION ALL
(SELECT username FROM users_b ORDER BY username LIMIT 5);
```

## Examples

```sql title="UNION"
SELECT username FROM users_a
UNION
SELECT username FROM users_b;
```

```sql title="UNION ALL retaining duplicates"
SELECT username FROM users_a
UNION ALL
SELECT username FROM users_b;
```

```sql title="INTERSECT"
SELECT username FROM users_a
INTERSECT
SELECT username FROM users_b;
```

```sql title="INTERSECT ALL retaining duplicates"
SELECT username FROM users_a
INTERSECT ALL
SELECT username FROM users_b;
```

```sql title="EXCEPT"
SELECT username FROM users_a
EXCEPT
SELECT username FROM users_b;
```

```sql title="EXCEPT ALL retaining duplicates"
SELECT username FROM users_a
EXCEPT ALL
SELECT username FROM users_b;
```

## See Also
[SELECT](index.md), [ORDER BY and LIMIT](order-and-limit.md)

---
[← Back to SELECT](index.md)
