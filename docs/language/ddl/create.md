# CREATE

The `CREATE` statement defines new objects within Link. The following objects can be created:

- [CREATE DATABASE](create/create-database.md) — creates a new database
- [CREATE TABLE](create/create-table.md) — creates a new relational table with a predefined schema
- [CREATE COLLECTION](create/create-collection.md) — creates a new document collection

```sql
CREATE DATABASE my_database;
CREATE TABLE users( ... );
CREATE COLLECTION posts;
```

Just enough to orient the reader and show the basic shape of each statement, then link them off to the full reference.

Then update your `mkdocs.yml` nav to add the new pages under CREATE. Want to do that first before we write the individual pages?
