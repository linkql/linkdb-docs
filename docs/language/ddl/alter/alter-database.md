# ALTER DATABASE
Modifies an existing database.

```grammar title="Grammar"
alter_database_stmt ::= ALTER DATABASE identifier RENAME TO identifier
```

## Description
`ALTER DATABASE` modifies an existing database. In order to alter a database, you must be a **superuser**.

## Parameters

`identifier`
:   The name of the database to alter.

`RENAME TO identifier`
:   Renames the database to the new name. The new name must follow the same naming rules
    as [`CREATE DATABASE`](create-database.md) and must not already be in use by another
    database.

## Notes
- `ALTER DATABASE` cannot be used inside a transactional block.
- Renaming a database does not affect existing connections to that database.

## Examples

```sql title="Rename a database"
ALTER DATABASE my_database RENAME TO new_database;
```

## See Also
[CREATE DATABASE](create-database.md), [DROP DATABASE](../drop/drop-database.md)

---
[← Back to ALTER](../alter.md)
