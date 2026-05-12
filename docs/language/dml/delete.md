# DELETE
Removes rows or documents from a table or collection.

```grammar title="Grammar"
delete_stmt     ::= DELETE FROM identifier
                    ( FROM from_clause )?
                    ( WHERE expr )?
                    ( CONFIRM string_literal )?

from_clause     ::= from_item ( ',' join_clause )*

from_item       ::= identifier ( AS? identifier )?
                  | '(' select_stmt ')' AS identifier
```

## Description
`DELETE FROM` removes one or more rows or documents from a table or collection. Rows
can be filtered with a `WHERE` clause. If no `WHERE` clause is provided, all rows or
documents in the table or collection are deleted.

A `FROM` clause can be used to join against other tables or collections when determining
which rows to delete.

For collections, deleting a document removes it entirely including all defined and free
fields.

## Parameters

`identifier`
:   The name of the table or collection to delete from.

`FROM from_clause`
:   Joins against one or more tables or collections when determining which rows to
    delete. Follows the same syntax as the [`FROM`](select/from.md) clause in `SELECT`.

`WHERE expr`
:   Filters which rows or documents are deleted. If omitted, all rows or documents in
    the table or collection are deleted.

`CONFIRM string_literal`
:   A confirmation string that must match the name of the table or collection being
    deleted from. Required when `CONFIRM_BULK_OPERATIONS` is enabled on the database
    and no `WHERE` clause is provided.

## Database Settings

`CONFIRM_BULK_OPERATIONS`
:   When enabled, requires a `CONFIRM` clause on any `DELETE` or `UPDATE` statement
    with no `WHERE` clause. Disabled by default.

## Notes
- If no `WHERE` clause is provided, all rows or documents are deleted.
- Deleting a document from a collection removes it entirely including all defined and
  free fields.
- `FROM` follows the same syntax as the `SELECT` `FROM` clause including shorthand
  and traditional join syntax.
- Foreign key constraints are validated at commit time. If a deleted row is referenced
  by another table or collection, the transaction will fail unless the reference action
  allows it.
- Use [`DROP TABLE`](../ddl/drop/drop-table.md) or
  [`DROP COLLECTION`](../ddl/drop/drop-collection.md) to remove the table or collection
  itself rather than just its data.
- Use [`TRUNCATE`](../ddl/drop/drop-table.md) to delete all rows without a `WHERE`
  clause more efficiently than `DELETE`.

## Examples

```sql title="Basic delete"
DELETE FROM users
WHERE users.last_login < '2020-01-01';
```

```sql title="Delete a single row"
DELETE FROM users
WHERE user_id = 1;
```

```sql title="Delete all rows"
DELETE FROM users;
```

```sql title="Delete all rows with confirmation"
DELETE FROM users
CONFIRM 'users';
```

```sql title="Delete from a collection"
DELETE FROM posts
WHERE posts::user_id = 1;
```

```sql title="Delete with FROM clause"
DELETE FROM sessions
FROM users
WHERE sessions.user_id = users.user_id
AND users.status = 'inactive';
```

```sql title="Delete with JOIN in FROM"
DELETE FROM sessions
FROM users, LEFT posts(user_id)
WHERE sessions.user_id = users.user_id
AND posts::status = 'deleted';
```

## See Also
[INSERT](insert.md), [UPDATE](update.md), [SELECT](select.md), [DROP TABLE](../ddl/drop/drop-table.md), [DROP COLLECTION](../ddl/drop/drop-collection.md)

---
[← Back to DML](../index.md)
