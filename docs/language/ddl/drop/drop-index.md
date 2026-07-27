# DROP INDEX
Removes an existing index.

```ebnf title="Grammar"
drop_index_stmt     ::= DROP INDEX ( IF EXISTS )? identifier
```

## Description
`DROP INDEX` deletes the specified index from the database. Dropping an index does
not affect the underlying table or collection data; it only removes the index
structure used for query optimization and uniqueness enforcement.

## Parameters

`IF EXISTS`
:   If specified, does not throw an error if the index does not exist. A notice is
    issued instead.

`identifier`
:   The name of the index to drop.

## Notes
- Dropping a unique index removes the associated uniqueness constraint.
- Dropping an index is an irreversible operation; the index must be recreated with
  `CREATE INDEX` if needed again.

## Examples

```sql title="Drop an index"
DROP INDEX idx_username;
```

```sql title="Drop an index if it exists"
DROP INDEX IF EXISTS idx_username;
```

## See Also
[CREATE INDEX](../../create/create-index.md), [ALTER INDEX](../../alter/alter-index.md)

---
[← Back to DROP INDEX / VIEW / TABLE / COLLECTION / DATABASE](../drop.md)
