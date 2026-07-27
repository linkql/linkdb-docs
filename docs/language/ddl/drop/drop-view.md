# DROP VIEW
Removes an existing view or materialized view.

```ebnf title="Grammar"
drop_view_stmt      ::= DROP ( MATERIALIZED )? VIEW ( IF EXISTS )? identifier
```

## Description
`DROP VIEW` deletes the specified view or materialized view from the database. For
a materialized view, the stored result data is also removed. Base tables,
collections, and their data are not affected.

## Parameters

`MATERIALIZED`
:   Specifies that the view being dropped is a materialized view.

`IF EXISTS`
:   If specified, does not throw an error if the view does not exist. A notice is
    issued instead.

`identifier`
:   The name of the view to drop.

## Notes
- Dropping a view does not drop the tables or collections referenced by its query.
- Dropping a materialized view removes the stored snapshot.

## Examples

```sql title="Drop a view"
DROP VIEW active_users;
```

```sql title="Drop a view if it exists"
DROP VIEW IF EXISTS active_users;
```

```sql title="Drop a materialized view"
DROP MATERIALIZED VIEW user_stats;
```

## See Also
[CREATE VIEW](../../create/create-view.md), [ALTER VIEW](../../alter/alter-view.md)

---
[← Back to DROP INDEX / VIEW / TABLE / COLLECTION / DATABASE](../drop.md)
