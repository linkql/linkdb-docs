# CREATE DATABASE
Creates a new database.

```grammar title="Grammar"
CREATE DATABASE [ IF NOT EXISTS ] name
```

## Description
`CREATE DATABASE` creates a new database in the LinkDB system.

In order to create a database, you must be a **superuser**.

## Parameters

`IF NOT EXISTS`
:   If specified, does not throw an error if a database with the same name already exists.
    A notice is issued instead.

`name`
:   The name of the database to create. Database names must follow these rules:
    - Case insensitive, stored as lowercase
    - Must start with a letter or underscore
    - May only contain letters, numbers, and underscores
    - Maximum 63 characters
    - Cannot be a reserved word
    - Must be a valid directory name on the host OS

## Notes
- `CREATE DATABASE` cannot be used inside a transactional block.
- Use [`DROP DATABASE`](../drop/drop-database.md) to remove a database.

## Examples

```sql title="Create a new database"
CREATE DATABASE my_database;
```

```sql title="Create a database only if it does not already exist"
CREATE DATABASE IF NOT EXISTS my_database;
```

## See Also
[ALTER DATABASE](../alter/alter-database.md), [CREATE TABLE](create-table.md), [CREATE COLLECTION](create-collection.md)

---
[← Back to CREATE](../create.md)
