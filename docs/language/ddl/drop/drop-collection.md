# DROP COLLECTION
Removes an existing collection or truncates all data within it.

```grammar title="Grammar"
drop_collection_stmt        ::= DROP COLLECTION ( IF EXISTS )? identifier ( cascade_clause )?

truncate_collection_stmt    ::= TRUNCATE COLLECTION ( IF EXISTS )? identifier ( RESTART IDENTITY )?

cascade_clause              ::= CASCADE ( cascade_depth )? ( cascade_override )?

cascade_depth               ::= integer_literal
                              | INFINITY

cascade_override            ::= SET NULL
                              | SET DEFAULT
                              | DROP
```

## Description
`DROP COLLECTION` permanently removes an existing collection and all of its documents.
If any other tables or collections hold foreign key references to this collection, the
drop will be blocked by default. Use `CASCADE` to override this behavior.

When `CASCADE` affects a dependent collection, the engine has additional granularity
compared to tables. Rather than dropping the entire collection, it can instead drop only
the defined field that holds the reference, or null the field values while leaving the
collection and its documents intact.

`TRUNCATE COLLECTION` removes all documents from the collection but preserves the
collection definition and its defined fields.

## Parameters

`IF EXISTS`
:   If specified, does not throw an error if the collection does not exist. A notice is
    issued instead.

`identifier`
:   The name of the collection to drop or truncate.

`CASCADE`
:   Overrides the default `RESTRICT` behavior and allows the drop to proceed even if
    dependencies exist. Defaults to `CASCADE 1` if no depth is specified.

`cascade_depth`
:   The number of dependencies you expect to be affected by the drop. If the engine finds
    more dependencies than specified, it returns an error and the drop is aborted. Use
    `INFINITY` to drop all dependencies regardless of depth.

    Depth counts total dependencies, not chain length. For example:
    - `users` → `posts` → `comments` is a depth of 2
    - `users` → `posts` and `users` → `orders` is also a depth of 2

`cascade_override`
:   Overrides the default precedence behavior for all dependencies. If not specified,
    the engine follows the precedence chain for each dependency:

    `drop time > named constraint > inline definition > RESTRICT`

    Available overrides:

    - `DROP` — drops all dependent tables and collections entirely
    - `SET NULL` — sets all foreign key references to `NULL`
    - `SET DEFAULT` — sets all foreign key references to their defined default value

    When `CASCADE` affects a dependent collection specifically, the override applies
    at the field level rather than the collection level where possible:

    | Override | Effect on dependent table | Effect on dependent collection |
    |----------|--------------------------|-------------------------------|
    | `DROP` | Drops the table | Drops the collection |
    | `SET NULL` | Sets foreign key column to `NULL` | Sets foreign key field to `NULL` |
    | `SET DEFAULT` | Sets foreign key column to default | Sets foreign key field to default |

    If no override is specified and the precedence chain resolves to `DROP` for a
    collection dependency, the engine drops the defined field rather than the entire
    collection unless the collection has no other defined fields, in which case the
    collection itself is dropped.

`RESTART IDENTITY`
:   Resets all autoincrement counters in the collection back to their defined start
    values. Only valid on `TRUNCATE COLLECTION`.

## Notes
- `DROP COLLECTION` defaults to `RESTRICT` — if any dependencies exist the drop is
  blocked and an error is returned.
- `CASCADE` depth is a safety mechanism, not a selector. Specifying `CASCADE 2` means
  "I expect exactly 2 dependencies — error if you find more." It does not mean "affect
  only the first 2 dependencies."
- When no `cascade_override` is specified, each dependency is resolved individually
  following the precedence chain.
- `CASCADE INFINITY` will continue until no dependencies remain. Use with caution.
- `TRUNCATE COLLECTION` does not require confirmation and preserves the collection
  definition and all defined fields.
- Foreign key references are validated at commit time, not inline.
- Use [`ALTER COLLECTION`](../alter/alter-collection.md) to modify a collection without
  removing it.

## Examples

```sql title="Drop a collection"
DROP COLLECTION posts;
```

```sql title="Drop a collection if it exists"
DROP COLLECTION IF EXISTS posts;
```

```sql title="Drop a collection with one dependency"
DROP COLLECTION posts CASCADE;
```

```sql title="Drop a collection with a specific depth"
DROP COLLECTION posts CASCADE 2;
```

```sql title="Drop a collection and all dependencies regardless of depth"
DROP COLLECTION posts CASCADE INFINITY;
```

```sql title="Drop a collection and set all foreign key references to null"
DROP COLLECTION posts CASCADE 2 SET NULL;
```

```sql title="Drop a collection and set all foreign key references to their default"
DROP COLLECTION posts CASCADE 2 SET DEFAULT;
```

```sql title="Drop a collection and drop all dependent objects"
DROP COLLECTION posts CASCADE 2 DROP;
```

```sql title="Truncate a collection"
TRUNCATE COLLECTION posts;
```

```sql title="Truncate a collection and reset autoincrement counters"
TRUNCATE COLLECTION posts RESTART IDENTITY;
```

## See Also
[CREATE COLLECTION](../create/create-collection.md), [ALTER COLLECTION](../alter/alter-collection.md), [DROP TABLE](drop-table.md), [DROP DATABASE](drop-database.md)

---
[← Back to DROP](../drop.md)
