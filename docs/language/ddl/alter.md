# ALTER

The `ALTER` statement modifies existing objects within Link. The following objects can be altered:

- [ALTER DATABASE](alter/alter-database.md) — renames an existing database
- [ALTER TABLE](alter/alter-table.md) — modifies an existing table
- [ALTER COLLECTION](alter/alter-collection.md) — modifies an existing collection
- [ALTER INDEX](alter/alter-index.md) — renames an existing index
- [ALTER VIEW](alter/alter-view.md) — renames an existing view or materialized view

```sql title="Examples"
ALTER DATABASE my_database RENAME TO new_database;
ALTER TABLE users ADD COLUMN email VARCHAR(100);
ALTER COLLECTION posts ADD FIELD likes INT DEFAULT 0;
ALTER INDEX idx_username RENAME TO idx_user_name;
ALTER VIEW active_users RENAME TO active_user_list;
ALTER MATERIALIZED VIEW user_stats RENAME TO user_summary;
```
