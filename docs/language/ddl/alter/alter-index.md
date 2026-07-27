# ALTER INDEX
Modifies an existing index.

```ebnf title="Grammar"
alter_index_stmt    ::= ALTER INDEX identifier RENAME TO identifier
```

## Description
`ALTER INDEX` changes the properties of an existing index. Currently the only
supported operation is renaming the index.

## Parameters

`identifier` (first)
:   The current name of the index.

`RENAME TO identifier`
:   Renames the index to the new name.

## Notes
- Renaming an index does not change its underlying columns, expression, or
  uniqueness constraint.
- The new name must not already be in use by another index in the database.

## Examples

```sql title="Rename an index"
ALTER INDEX idx_username RENAME TO idx_user_username;
```

## See Also
[CREATE INDEX](../../create/create-index.md), [DROP INDEX](../../drop/drop-index.md)

---
[← Back to ALTER INDEX / VIEW / TABLE / COLLECTION / DATABASE](../alter.md)
