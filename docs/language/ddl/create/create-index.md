# CREATE INDEX
Creates an index on a table or collection.

```ebnf title="Grammar"
create_index_stmt   ::= CREATE ( UNIQUE )? INDEX ( IF NOT EXISTS )? identifier
                        ON identifier '(' index_column ( ',' index_column )* ')'
                        ( WHERE expr )?

index_column        ::= expr ( ASC | DESC )?
```

## Description
`CREATE INDEX` builds a new index on the specified table or collection. Indexes
improve the performance of queries that filter, sort, or join on the indexed
columns and expressions.

An index can be created on one or more columns, on expressions, or on free fields
of a collection. A unique index enforces that no two indexed values are the same.

## Parameters

`UNIQUE`
:   Enforces that the indexed values must be unique across all rows in the table or
    collection.

`IF NOT EXISTS`
:   If specified, does not throw an error if an index with the same name already
    exists. A notice is issued instead.

`identifier` (first)
:   The name of the index to create.

`identifier` (second)
:   The name of the table or collection the index is created on.

`index_column`
:   A column, expression, or free field to index. Optionally followed by `ASC` or
    `DESC` to control sort order. If omitted, ascending order is used.

`WHERE expr`
:   Creates a partial index that only includes rows or documents satisfying the
    predicate. Can be combined with `UNIQUE` and expression indexes.

## Notes
- Indexes can be created on table columns, collection defined fields, and collection
  free fields using the `collection::field` notation.
- Expression indexes can use functions, such as `LOWER(email)`, or bracket notation
  such as `metadata['ip']`.
- Unique indexes enforce uniqueness on the computed indexed value, including for
  partial and expression indexes.
- Creating an index on a collection uses the same syntax as a table. The engine
  resolves column references against the collection schema and free fields.

## Examples

```sql title="Create a basic index"
CREATE INDEX idx_username ON users (username);
```

```sql title="Create a composite index"
CREATE INDEX idx_name ON users (first_name, last_name);
```

```sql title="Create a unique index"
CREATE UNIQUE INDEX idx_email ON users (email);
```

```sql title="Create an index if not exists"
CREATE INDEX IF NOT EXISTS idx_username ON users (username);
```

```sql title="Create an index with descending sort"
CREATE INDEX idx_created ON users (created_at DESC);
```

```sql title="Create an index with mixed sort directions"
CREATE INDEX idx_multi ON users (last_name ASC, first_name DESC);
```

```sql title="Create a partial index"
CREATE INDEX idx_active_email ON users (email) WHERE status = 'active';
```

```sql title="Create an expression index using a function"
CREATE INDEX idx_lower_email ON users (LOWER(email));
```

```sql title="Create an expression index using bracket notation"
CREATE INDEX idx_metadata_ip ON posts (metadata['ip']);
```

```sql title="Create an index on a collection free field"
CREATE INDEX idx_free_field ON posts (posts::mood);
```

```sql title="Create an index on a collection"
CREATE INDEX idx_post_user ON posts (user_id);
```

```sql title="Create a unique partial expression index"
CREATE UNIQUE INDEX idx_active_lower_email ON users (LOWER(email)) WHERE status = 'active';
```

## See Also
[DROP INDEX](../drop/drop-index.md), [ALTER INDEX](../alter/alter-index.md)

---
[← Back to CREATE INDEX / VIEW / TABLE / COLLECTION / DATABASE](../create.md)
