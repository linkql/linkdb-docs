# LET
Defines named subqueries for use within a statement.

```grammar title="Grammar"
let_block       ::= LET let_binding ( ',' let_binding )* dml_stmt

let_binding     ::= identifier '=' '(' select_stmt ')'
                  | recursive_let

recursive_let   ::= RECURSIVE identifier '=' '(' select_stmt ')'
                      UNION ALL '(' select_stmt ')'
                      ( cycle_clause )*
                      ( SEARCH ( DEPTH | BREADTH ) FIRST BY identifier SET identifier )?

cycle_clause    ::= CYCLE ON identifier ( ',' identifier )* SET identifier
                  | MAX ITERATIONS integer_literal

with_clause     ::= WITH ( RECURSIVE )? cte_binding ( ',' cte_binding )*

cte_binding     ::= identifier ( '(' identifier ( ',' identifier )* ')' )?
                      AS '(' select_stmt ')'
```

## Description
`LET` defines one or more named subqueries that can be referenced within a single
statement. It is LinkQL's equivalent of SQL's `WITH` clause (common table expressions).
`LET` bindings are scoped to the statement they are defined in and cannot be referenced
outside of it.

`LET` can be used with `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. Each binding is
defined as a named subquery and can be referenced by name in subsequent bindings or
in the final statement.

## Parameters

`identifier`
:   The name assigned to the subquery. This name is used to reference the result set
    in subsequent bindings or in the final statement.

`select_stmt`
:   The subquery that defines the named result set. Must be a valid `SELECT` statement.

`dml_stmt`
:   The statement that uses the defined bindings. Can be a `SELECT`, `INSERT`,
    `UPDATE`, `DELETE`, or `MERGE` statement.

`recursive_let`
:   Defines a recursive named subquery using the `RECURSIVE` keyword. The first
    `SELECT` provides the base result, and the second `SELECT` provides the recursive
    step. Results are combined with `UNION ALL`.

`cycle_clause`
:   Detects cycles in recursive CTEs. `CYCLE ON` marks columns to track and a
    column to set when a cycle is detected. `MAX ITERATIONS` limits the recursion depth.

`SEARCH (DEPTH | BREADTH) FIRST BY identifier SET identifier`
:   Controls the order of rows returned by a recursive CTE. `DEPTH FIRST` uses
    depth-first search ordering; `BREADTH FIRST` uses breadth-first search ordering.
    The `BY` column determines ordering, and the `SET` column stores the ordering value.

`with_clause`
:   SQL-standard common table expression syntax. `WITH RECURSIVE` introduces one
    or more named subqueries (`cte_binding`) that can reference each other. Each
    binding is an identifier (with optional column list) followed by `AS ( select_stmt )`.

`cte_binding`
:   A single CTE definition within a `WITH` clause. The optional column list defines
    names for the result columns. The `AS` clause wraps the defining `SELECT`.

## Chaining Bindings
Multiple `LET` bindings are comma separated. A later binding can reference any
earlier binding by name. Circular references — where binding `a` references `b`
and `b` references `a` — are detected by the planner and return an error.

```sql
LET inactive_users = (
    SELECT users.user_id, users.username
    FROM users
    WHERE users.last_login < '2020-01-01'
), inactive_sessions = (
    SELECT sessions.session_id
    FROM sessions, inactive_users(user_id)
)

SELECT
    inactive_users.username,
    inactive_sessions.session_id
FROM inactive_users, LEFT inactive_sessions(user_id);
```

## Notes
- `LET` bindings are scoped to the statement they are defined in. They cannot be
  referenced outside of that statement.
- A later binding can reference any earlier binding by name.
- Circular references between bindings are detected by the planner and return an
  error.
- `LET` bindings can be used with `SELECT`, `INSERT`, `UPDATE`, `DELETE`, and `MERGE`.
- `LET` is LinkQL's equivalent of SQL's `WITH` clause. Users familiar with CTEs
  will find `LET` behaves the same way.

## Examples

```sql title="Basic LET with SELECT"
LET inactive_users = (
    SELECT users.user_id, users.username
    FROM users
    WHERE users.last_login < '2020-01-01'
)

SELECT
    inactive_users.username,
    sessions.last_active
FROM inactive_users, LEFT sessions(user_id);
```

```sql title="Chained LET bindings"
LET inactive_users = (
    SELECT users.user_id, users.username
    FROM users
    WHERE users.last_login < '2020-01-01'
), inactive_sessions = (
    SELECT sessions.session_id, sessions.user_id
    FROM sessions, inactive_users(user_id)
)

SELECT
    inactive_users.username,
    inactive_sessions.session_id
FROM inactive_users, LEFT inactive_sessions(user_id);
```

```sql title="LET with DELETE"
LET inactive_users = (
    SELECT users.user_id
    FROM users
    WHERE users.last_login < '2020-01-01'
)

DELETE FROM sessions
WHERE user_id IN (
    SELECT inactive_users.user_id FROM inactive_users
);
```

```sql title="LET with UPDATE"
LET inactive_users = (
    SELECT users.user_id
    FROM users
    WHERE users.last_login < '2020-01-01'
)

UPDATE users
SET status = 'inactive'
WHERE user_id IN (
    SELECT inactive_users.user_id FROM inactive_users
);
```

## See Also
[FOR](for.md), [SELECT](select/index.md), [INSERT](insert.md), [UPDATE](update.md), [DELETE](delete.md)

---
[← Back to SELECT](../select/index.md)
