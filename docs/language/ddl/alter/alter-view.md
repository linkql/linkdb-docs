# ALTER VIEW
Modifies an existing view or materialized view.

```ebnf title="Grammar"
alter_view_stmt     ::= ALTER ( MATERIALIZED )? VIEW identifier RENAME TO identifier
```

## Description
`ALTER VIEW` changes the properties of an existing view. Currently the only
supported operation is renaming the view.

## Parameters

`MATERIALIZED`
:   Specifies that the view being altered is a materialized view.

`identifier` (first)
:   The current name of the view.

`RENAME TO identifier`
:   Renames the view to the new name.

## Notes
- Renaming a view does not change its underlying query definition or, for
  materialized views, the stored data.
- The new name must not already be in use by another view in the database.

## Examples

```sql title="Rename a view"
ALTER VIEW active_users RENAME TO enabled_users;
```

```sql title="Rename a materialized view"
ALTER MATERIALIZED VIEW user_stats RENAME TO user_status_stats;
```

## See Also
[CREATE VIEW](../../create/create-view.md), [DROP VIEW](../../drop/drop-view.md)

---
[← Back to ALTER INDEX / VIEW / TABLE / COLLECTION / DATABASE](../alter.md)
