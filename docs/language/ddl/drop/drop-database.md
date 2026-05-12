# DROP DATABASE
Removes an existing database or truncates all data within it.

```grammar title="Grammar"
drop_database_stmt      ::= DROP DATABASE ( IF EXISTS )? identifier CONFIRM string_literal

truncate_database_stmt  ::= TRUNCATE DATABASE ( IF EXISTS )? identifier CONFIRM string_literal
                              ( RESTART IDENTITY )?
```

## Description
`DROP DATABASE` permanently removes an existing database and all of its tables,
collections, and data. This operation cannot be undone.

`TRUNCATE DATABASE` removes all data from every table and collection in the database
but preserves the schema. All table and collection definitions remain intact.

Both operations require **superuser** privileges and confirmation.

## Parameters

`IF EXISTS`
:   If specified, does not throw an error if the database does not exist. A notice is
    issued instead.

`identifier`
:   The name of the database to drop or truncate.

`CONFIRM string_literal`
:   A confirmation string that must match the name of the database. This is required to
    prevent accidental data loss. Database names are case insensitive so the confirmation
    string is matched case insensitively.

`RESTART IDENTITY`
:   Resets all autoincrement counters in every table and collection back to their defined
    start values. Only valid on `TRUNCATE DATABASE`.

## Notes
- `DROP DATABASE` and `TRUNCATE DATABASE` cannot be used inside a transactional block.
- Both operations require **superuser** privileges.
- The confirmation string must match the database name. Since database names are stored
  as lowercase, the match is case insensitive.
- `TRUNCATE DATABASE` preserves all table and collection definitions. Only data is
  removed.
- A database level setting `CONFIRM_DROP` can be used to disable the confirmation
  requirement. It is highly recommended to leave this enabled.
- Use [`ALTER DATABASE`](alter/alter-database.md) to rename a database.

## Examples

```sql title="Drop a database"
DROP DATABASE my_database CONFIRM 'my_database';
```

```sql title="Drop a database if it exists"
DROP DATABASE IF EXISTS my_database CONFIRM 'my_database';
```

```sql title="Truncate a database"
TRUNCATE DATABASE my_database CONFIRM 'my_database';
```

```sql title="Truncate a database and reset all autoincrement counters"
TRUNCATE DATABASE my_database CONFIRM 'my_database' RESTART IDENTITY;
```

## See Also
[CREATE DATABASE](../create/create-database.md), [ALTER DATABASE](../alter/alter-database.md), [DROP TABLE](drop-table.md), [DROP COLLECTION](drop-collection.md)

---
[← Back to DROP](../drop.md)
