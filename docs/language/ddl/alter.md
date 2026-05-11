# ALTER

The `ALTER` statement modifies existing objects within Link. The following objects can be altered:

- [ALTER DATABASE](alter/alter-database.md) — renames an existing database
- [ALTER TABLE](alter/alter-table.md) — modifies an existing table
- [ALTER COLLECTION](alter/alter-collection.md) — modifies an existing collection

```sql title="Examples"
ALTER DATABASE my_database RENAME TO new_database;
ALTER TABLE users ADD COLUMN email VARCHAR(100);
ALTER COLLECTION posts ADD FIELD likes INT DEFAULT 0;
```
