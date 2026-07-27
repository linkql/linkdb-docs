# MERGE
Conditionally inserts or updates rows in a table or collection based on a source dataset.

```grammar title="Grammar"
merge_stmt          ::= with_clause? MERGE INTO identifier ( AS? identifier )?
                        USING merge_source
                        ON expr
                        merge_when_clause+
                        ( RETURNING return_item ( ',' return_item )* )?

merge_source        ::= identifier ( AS? identifier )?
                      | '(' select_stmt ')' ( AS? identifier )?

merge_when_clause   ::= WHEN MATCHED ( AND expr )? THEN UPDATE SET set_values
                      | WHEN NOT MATCHED ( AND expr )? THEN INSERT
                        ( '(' identifier ( ',' identifier )* ')' )?
                        VALUES '(' insert_value ( ',' insert_value )* ')'
```

## Description
`MERGE` atomically inserts or updates rows in a target table or collection based on
whether rows from a source dataset match rows in the target. The source can be another
table, collection, or the result of a `SELECT` statement.

For each row in the source, the engine evaluates the `ON` condition against the target.
If a match is found, the `WHEN MATCHED` clause is executed. If no match is found,
the `WHEN NOT MATCHED` clause is executed. Multiple `merge_when_clause` entries can be
provided, but at minimum one must be present.

The `RETURNING` clause optionally returns values from the affected rows.

## Parameters

`identifier`
:   The name of the target table or collection to merge into.

`AS? identifier`
:   Optional alias for the target table or collection.

`with_clause`
:   Optional common table expression (`WITH` or `WITH RECURSIVE`) that provides
    named subqueries available within the `MERGE` statement.

`USING merge_source`
:   The source dataset to compare against the target. Can be a table, collection,
    or a subquery.

`ON expr`
:   The matching condition evaluated for each source row against the target.
    Typically an equality comparison on key columns.

`WHEN MATCHED THEN UPDATE SET set_values`
:   Updates the matching target row with the specified values. An optional `AND expr`
    condition can further restrict which matched rows are updated.

`WHEN NOT MATCHED THEN INSERT ... VALUES ...`
:   Inserts a new row into the target with the specified values. An optional column
    list can name specific target columns. An optional `AND expr` condition can further
    restrict which unmatched rows are inserted.

`RETURNING return_item`
:   Returns values from the merged rows. Can return `*` for all columns or
    specific expressions with optional aliases.

## Notes
- At least one `WHEN MATCHED` or `WHEN NOT MATCHED` clause must be provided.
- `MERGE` is not supported inside `FOR` loop bodies.
- `MERGE` can be used inside a `LET` block as a valid DML statement.
- The source dataset is evaluated once before any inserts or updates are applied.
- `RETURNING` returns values from rows that were inserted or updated.

## Examples

```sql title="Basic MERGE from a table"
MERGE INTO users AS target
USING staging_users AS source
ON target.user_id = source.user_id
WHEN MATCHED THEN UPDATE SET
    username = source.username,
    email = source.email
WHEN NOT MATCHED THEN INSERT
    (user_id, username, email)
    VALUES (source.user_id, source.username, source.email);
```

```sql title="MERGE from a SELECT subquery"
MERGE INTO users AS target
USING (
    SELECT new_users.user_id, new_users.username, new_users.email
    FROM new_users
    WHERE new_users.status = 'active'
) AS source
ON target.user_id = source.user_id
WHEN MATCHED THEN UPDATE SET
    username = source.username,
    email = source.email
WHEN NOT MATCHED THEN INSERT
    (user_id, username, email)
    VALUES (source.user_id, source.username, source.email);
```

```sql title="MERGE with conditional matched clause"
MERGE INTO users AS target
USING staging_users AS source
ON target.user_id = source.user_id
WHEN MATCHED AND source.last_modified > target.last_modified THEN UPDATE SET
    username = source.username,
    email = source.email,
    last_modified = source.last_modified
WHEN NOT MATCHED THEN INSERT
    (user_id, username, email, last_modified)
    VALUES (source.user_id, source.username, source.email, source.last_modified);
```

```sql title="MERGE with RETURNING"
MERGE INTO users AS target
USING staging_users AS source
ON target.user_id = source.user_id
WHEN MATCHED THEN UPDATE SET username = source.username
WHEN NOT MATCHED THEN INSERT (user_id, username)
    VALUES (source.user_id, source.username)
RETURNING *;
```

```sql title="MERGE into a collection"
MERGE INTO posts AS target
USING staging_posts AS source
ON target::post_id = source::post_id
WHEN MATCHED THEN UPDATE SET content = source::content
WHEN NOT MATCHED THEN INSERT (post_id, content)
    VALUES (source::post_id, source::content);
```

## See Also
[INSERT](insert.md), [UPDATE](update.md), [DELETE](delete.md), [SELECT](select.md), [LET](let.md)

---
[← Back to DML](../select.md)
