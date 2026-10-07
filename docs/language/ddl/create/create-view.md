# CREATE VIEW
Creates a new view.

```ebnf title="Grammar"
create_view_stmt    ::= CREATE ( OR REPLACE )? ( MATERIALIZED )? VIEW ( IF NOT EXISTS )? identifier
                        AS select_stmt
```

## Description
`CREATE VIEW` defines a named query that can be referenced like a table or
collection. A plain view stores only the query definition and recomputes its
result every time it is read. `CREATE MATERIALIZED VIEW` instead runs the
query immediately and stores its result as a snapshot — see the
`MATERIALIZED` parameter below for exactly what that does and doesn't
support today.

## Parameters

`OR REPLACE`
:   If a view with the same name already exists, replace it with the new
    definition. Cannot be combined with `IF NOT EXISTS`.

`MATERIALIZED`
:   Creates a materialized view. The defining query runs immediately and its
    result rows are stored under the view's name, instead of being
    recomputed on every read. Use
    [`REFRESH MATERIALIZED VIEW`](../../utility.md#refresh-materialized-view)
    to re-run the query later and overwrite the stored snapshot.

    The rest of materialized-view support is still partial: reading a
    materialized view back is not yet implemented (`SELECT ... FROM` it
    raises `AnalyzerError: Unsupported statement type:
    AnalyzedSelectStatement` instead of returning the stored rows), and so
    are [`ALTER MATERIALIZED VIEW`](../alter/alter-view.md) and
    [`DROP MATERIALIZED VIEW`](../drop/drop-view.md) (both raise
    `ExecutorError: Materialized views not yet supported`). In practice, a
    materialized view can be created and refreshed today, but not queried,
    redefined, renamed, or dropped.

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

```sql title="Create a materialized view"
CREATE MATERIALIZED VIEW user_stats AS
SELECT users.status, COUNT(*) AS total FROM users GROUP BY users.status;
```

## See Also
[DROP VIEW](../drop/drop-view.md), [ALTER VIEW](../alter/alter-view.md),
[REFRESH MATERIALIZED VIEW](../../utility.md#refresh-materialized-view)

---
[← Back to CREATE INDEX / VIEW / TABLE / COLLECTION / DATABASE](../create.md)
