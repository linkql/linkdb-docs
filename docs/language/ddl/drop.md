# DROP

The `DROP` statement removes existing objects from Link. The `TRUNCATE` statement removes
all data from an object without removing the object itself.

- [DROP DATABASE](drop/drop-database.md) — removes an existing database
- [DROP TABLE](drop/drop-table.md) — removes an existing table
- [DROP COLLECTION](drop/drop-collection.md) — removes an existing collection

```sql title="Examples"
DROP DATABASE my_database CONFIRM 'my_database';
DROP TABLE IF EXISTS users;
DROP COLLECTION IF EXISTS posts;

TRUNCATE TABLE users;
TRUNCATE COLLECTION posts;
```

## CASCADE

When dropping a table or collection that has dependencies, Link defaults to `RESTRICT` —
the drop is blocked if any foreign key references exist. To override this, use `CASCADE`.

See each individual page for full CASCADE behavior and syntax.
