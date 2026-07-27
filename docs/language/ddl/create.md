# CREATE

The `CREATE` statement defines new objects within Link. The following objects can be created:

- [CREATE DATABASE](create/create-database.md) — creates a new database
- [CREATE TABLE](create/create-table.md) — creates a new relational table with a predefined schema
- [CREATE COLLECTION](create/create-collection.md) — creates a new document collection
- [CREATE INDEX](create/create-index.md) — creates an index on a table or collection
- [CREATE VIEW](create/create-view.md) — creates a view or materialized view

```sql title="Examples"
CREATE DATABASE my_database;
CREATE TABLE users( ... );
CREATE COLLECTION posts;
CREATE INDEX idx_username ON users (username);
CREATE VIEW active_users AS SELECT * FROM users WHERE status = 'active';
```
