# Utility Commands
Administrative and diagnostic commands.

```grammar title="Grammar"
utility_stmt        ::= refresh_stmt
                      | explain_stmt
                      | info_stmt

refresh_stmt        ::= REFRESH MATERIALIZED VIEW identifier

explain_stmt        ::= EXPLAIN ( ANALYZE )? ( VERBOSE )? ( select_stmt | insert_stmt | update_stmt | delete_stmt )

info_stmt           ::= INFO ( TABLE | COLLECTION | VIEW | INDEX | DATABASE ) identifier?
```

## Description
Utility commands provide maintenance and inspection operations for LinkDB. They
include refreshing materialized views, analyzing query plans, and inspecting
metadata for tables, collections, views, indexes, and the database.

## REFRESH MATERIALIZED VIEW
Updates the contents of a materialized view.

```sql title="Refresh a materialized view"
REFRESH MATERIALIZED VIEW user_stats;
```

## EXPLAIN
Shows the execution plan for a query without running it. The plan output depends
on the level requested.

`EXPLAIN`
:   Returns the planned execution steps.

`EXPLAIN ANALYZE`
:   Executes the query and returns timing and row statistics.

`EXPLAIN VERBOSE`
:   Returns a more detailed plan.

`EXPLAIN ANALYZE VERBOSE`
:   Executes the query and returns detailed timing and row statistics.

```sql title="Explain a select statement"
EXPLAIN SELECT users.username FROM users WHERE users.status = 'active';
```

```sql title="Explain analyze a select statement"
EXPLAIN ANALYZE SELECT users.username FROM users WHERE users.status = 'active';
```

```sql title="Explain verbose a select statement"
EXPLAIN VERBOSE SELECT users.username FROM users;
```

```sql title="Explain analyze verbose a select statement"
EXPLAIN ANALYZE VERBOSE SELECT users.username FROM users;
```

```sql title="Explain an insert statement"
EXPLAIN INSERT INTO users (username) VALUES ('john');
```

```sql title="Explain an update statement"
EXPLAIN UPDATE users SET status = 'inactive' WHERE user_id = 1;
```

```sql title="Explain a delete statement"
EXPLAIN DELETE FROM users WHERE user_id = 1;
```

```sql title="Explain a select statement with a window function"
EXPLAIN ANALYZE
SELECT users.username, ROW_NUMBER() OVER (ORDER BY users.created_at) AS rn
FROM users;
```

## INFO
Returns metadata about a catalog object or the database as a whole.

`INFO TABLE identifier`
:   Metadata for a specific table.

`INFO COLLECTION identifier`
:   Metadata for a specific collection.

`INFO VIEW identifier`
:   Metadata for a specific view.

`INFO INDEX identifier`
:   Metadata for a specific index.

`INFO DATABASE`
:   Metadata for the current database.

```sql title="Info on a table"
INFO TABLE users;
```

```sql title="Info on a collection"
INFO COLLECTION posts;
```

```sql title="Info on a view"
INFO VIEW active_users;
```

```sql title="Info on an index"
INFO INDEX idx_username;
```

```sql title="Info on the database"
INFO DATABASE;
```

## See Also
[CREATE VIEW](ddl/create/create-view.md), [CREATE INDEX](ddl/create/create-index.md)

---
[← Back to Language Reference](index.md)
