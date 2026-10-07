# DROP VIEW
Removes an existing view.

```ebnf title="Grammar"
drop_view_stmt      ::= DROP ( MATERIALIZED )? VIEW ( IF EXISTS )? identifier
```

## Description
`DROP VIEW` deletes the specified view from the database. Base tables,
collections, and their data are not affected.

## Parameters

`MATERIALIZED`
:   Accepted by the grammar, but `DROP MATERIALIZED VIEW` itself is not yet
    implemented: it raises `ExecutorError: Materialized views not yet
    supported` at execution time instead of dropping anything. This is
    narrower than materialized views' overall support — `CREATE MATERIALIZED
    VIEW` and
    [`REFRESH MATERIALIZED VIEW`](../../utility.md#refresh-materialized-view)
    do work; see [CREATE VIEW](../create/create-view.md)'s `MATERIALIZED`
    parameter for the full picture.

`IF EXISTS`
:   If specified, does not throw an error if the view does not exist. A notice is
    issued instead.

`identifier`
:   The name of the view to drop.

## Notes
- Dropping a view does not drop the tables or collections referenced by its query.

## Examples

```sql title="Drop a view"
DROP VIEW active_users;
```

```sql title="Drop a view if it exists"
DROP VIEW IF EXISTS active_users;
```

## See Also
[CREATE VIEW](../../create/create-view.md), [ALTER VIEW](../../alter/alter-view.md)

---
[← Back to DROP INDEX / VIEW / TABLE / COLLECTION / DATABASE](../drop.md)
