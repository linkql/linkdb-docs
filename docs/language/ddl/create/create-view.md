# CREATE VIEW
Creates a new view.

```ebnf title="Grammar"
create_view_stmt    ::= CREATE ( OR REPLACE )? ( MATERIALIZED )? VIEW ( IF NOT EXISTS )? identifier
                        AS select_stmt
```

## Description
`CREATE VIEW` defines a named query that can be referenced like a table or
collection. A view stores the query definition and computes its result each time
it is used.

## Parameters

`OR REPLACE`
:   If a view with the same name already exists, replace it with the new
    definition. Cannot be combined with `IF NOT EXISTS`.

`MATERIALIZED`
:   Accepted by the grammar, but not yet implemented. `CREATE MATERIALIZED
    VIEW` raises an error at execution time instead of creating anything; use
    plain `CREATE VIEW` instead.

`IF NOT EXISTS`
:   If specified, does not throw an error if a view with the same name already
    exists. A notice is issued instead. Cannot be combined with `OR REPLACE`.

`identifier`
:   The name of the view to create.

`select_stmt`
:   The `SELECT` statement whose result defines the view.

## Notes
- Views can query both tables and collections, including joins between them.
- `CREATE OR REPLACE VIEW` replaces an existing view with the same name.
- A view can be read anywhere a table can: in `FROM` and joins, in the `USING` clause of
  `DELETE` and the `FROM` clause of `UPDATE`, as a `MERGE` source, and as a `FOR` source.
  It is read-only: `INSERT`, `UPDATE`, `DELETE`, and `MERGE` cannot target a view.
- A view cannot refer to itself, directly or through other views. Using such a view is
  an error.

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

```sql title="Create a view over a table and collection join"
CREATE VIEW user_posts AS
SELECT users.username, posts::content
FROM users LEFT JOIN posts ON users.user_id = posts::user_id;
```

## See Also
[DROP VIEW](../drop/drop-view.md), [ALTER VIEW](../alter/alter-view.md)

---
[← Back to CREATE INDEX / VIEW / TABLE / COLLECTION / DATABASE](../create.md)
