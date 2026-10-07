# ALTER VIEW
Modifies an existing view.

```ebnf title="Grammar"
alter_view_stmt     ::= ALTER ( MATERIALIZED )? VIEW identifier alter_view_cmd

alter_view_cmd      ::= RENAME TO identifier
                      | AS select_stmt
```

## Description
`ALTER VIEW` changes an existing view. It can rename the view or replace the query
that defines it.

## Parameters

`MATERIALIZED`
:   Accepted by the grammar, but `ALTER MATERIALIZED VIEW` itself is not yet
    implemented: it raises `ExecutorError: Materialized views not yet
    supported` at execution time instead of altering anything, regardless of
    which `alter_view_cmd` is given. This is narrower than materialized
    views' overall support — `CREATE MATERIALIZED VIEW` and
    [`REFRESH MATERIALIZED VIEW`](../../utility.md#refresh-materialized-view)
    do work; see [CREATE VIEW](../create/create-view.md)'s `MATERIALIZED`
    parameter for the full picture.

`identifier` (first)
:   The current name of the view.

`RENAME TO identifier`
:   Renames the view to the new name.

`AS select_stmt`
:   Replaces the query that defines the view with a new `SELECT` statement.

## Notes
- Renaming a view does not change its underlying query definition.
- The new name must not already be in use by another view in the database.
- `AS select_stmt` keeps the view's name and replaces its definition. The new query is
  checked in the same way as the query of a `CREATE VIEW` statement.

## Examples

```sql title="Rename a view"
ALTER VIEW active_users RENAME TO enabled_users;
```

```sql title="Replace a view's query"
ALTER VIEW active_users AS
SELECT users.username, users.email FROM users WHERE users.status = 'active';
```

## See Also
[CREATE VIEW](../../create/create-view.md), [DROP VIEW](../../drop/drop-view.md)

---
[← Back to ALTER INDEX / VIEW / TABLE / COLLECTION / DATABASE](../alter.md)
