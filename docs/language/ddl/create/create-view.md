# CREATE VIEW
Creates a new view or materialized view.

```ebnf title="Grammar"
create_view_stmt    ::= CREATE ( OR REPLACE )? ( MATERIALIZED )? VIEW ( IF NOT EXISTS )? identifier
                        AS select_stmt
```

## Description
`CREATE VIEW` defines a named query that can be referenced like a table or
collection. A view stores the query definition and computes its result each time
it is used. A materialized view stores the query result on creation and may be
refreshed separately.

## Parameters

`OR REPLACE`
:   If a view with the same name already exists, replace it with the new
    definition. Cannot be combined with `IF NOT EXISTS`.

`MATERIALIZED`
:   Creates a materialized view that persists the query result. Materialized views
    must be refreshed explicitly; see related DDL for refresh behavior.

`IF NOT EXISTS`
:   If specified, does not throw an error if a view with the same name already
    exists. A notice is issued instead. Cannot be combined with `OR REPLACE`.

`identifier`
:   The name of the view to create.

`select_stmt`
:   The `SELECT` statement whose result defines the view.

## Notes
- Views can query both tables and collections, including joins between them.
- `CREATE OR REPLACE VIEW` and `CREATE OR REPLACE MATERIALIZED VIEW` replace an
  existing view or materialized view with the same name.
- Materialized views store a snapshot of the query result at creation time.

## Examples

```sql title="Create a basic view"
CREATE VIEW active_users AS
SELECT users.username FROM users WHERE users.status = 'active';
```

```sql title="Create a view if not exists"
CREATE VIEW IF NOT EXISTS active_users AS
SELECT users.username FROM users WHERE users.status = 'active';
```

```sql title="Create or replace a view"
CREATE OR REPLACE VIEW active_users AS
SELECT users.username, users.email FROM users WHERE users.status = 'active';
```

```sql title="Create a materialized view"
CREATE MATERIALIZED VIEW user_stats AS
SELECT users.status, COUNT(users.user_id) AS total FROM users GROUP BY users.status;
```

```sql title="Create a view over a table and collection join"
CREATE VIEW user_posts AS
SELECT users.username, posts::content
FROM users LEFT JOIN posts ON users.user_id = posts::user_id;
```

```sql title="Create or replace a materialized view"
CREATE OR REPLACE MATERIALIZED VIEW user_stats AS
SELECT users.status, COUNT(users.user_id) AS total FROM users GROUP BY users.status;
```

## See Also
[DROP VIEW](../drop/drop-view.md), [ALTER VIEW](../alter/alter-view.md)

---
[← Back to CREATE INDEX / VIEW / TABLE / COLLECTION / DATABASE](../create.md)
