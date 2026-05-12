# DROP TABLE
Removes an existing table or truncates all data within it.

```grammar title="Grammar"
drop_table_stmt     ::= DROP TABLE ( IF EXISTS )? identifier ( cascade_clause )?

truncate_table_stmt ::= TRUNCATE TABLE identifier ( RESTART IDENTITY )?

cascade_clause      ::= CASCADE ( cascade_depth )? ( cascade_override )?

cascade_depth       ::= integer_literal
                      | INFINITY

cascade_override    ::= SET NULL
                      | SET DEFAULT
                      | DROP
```

## Description
`DROP TABLE` permanently removes an existing table and all of its data. If any other
tables or collections hold foreign key references to this table, the drop will be blocked
by default. Use `CASCADE` to override this behavior.

`TRUNCATE TABLE` removes all data from the table but preserves the table definition and
its columns.

## Parameters

`IF EXISTS`
:   If specified, does not throw an error if the table does not exist. A notice is issued
    instead.

`identifier`
:   The name of the table to drop or truncate.

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

`RESTART IDENTITY`
:   Resets all autoincrement counters in the table back to their defined start values.
    Only valid on `TRUNCATE TABLE`.

## Notes
- `DROP TABLE` defaults to `RESTRICT` — if any dependencies exist the drop is blocked
  and an error is returned.
- `CASCADE` depth is a safety mechanism, not a selector. Specifying `CASCADE 2` means
  "I expect exactly 2 dependencies — error if you find more." It does not mean "affect
  only the first 2 dependencies."
- When no `cascade_override` is specified, each dependency is resolved individually
  following the precedence chain.
- `CASCADE INFINITY` will continue until no dependencies remain. Use with caution.
- `TRUNCATE TABLE` does not require confirmation and preserves the table definition.
- Foreign key references are validated at commit time, not inline.
- Use [`ALTER TABLE`](../alter/alter-table.md) to modify a table without removing it.

## Examples

```sql title="Drop a table"
DROP TABLE users;
```

```sql title="Drop a table if it exists"
DROP TABLE IF EXISTS users;
```

```sql title="Drop a table with one dependency"
DROP TABLE users CASCADE;
```

```sql title="Drop a table with a specific depth"
DROP TABLE users CASCADE 2;
```

```sql title="Drop a table and all dependencies regardless of depth"
DROP TABLE users CASCADE INFINITY;
```

```sql title="Drop a table and set all foreign key references to null"
DROP TABLE users CASCADE 2 SET NULL;
```

```sql title="Drop a table and set all foreign key references to their default"
DROP TABLE users CASCADE 2 SET DEFAULT;
```

```sql title="Drop a table and drop all dependent objects"
DROP TABLE users CASCADE 2 DROP;
```

```sql title="Truncate a table"
TRUNCATE TABLE users;
```

```sql title="Truncate a table and reset autoincrement counters"
TRUNCATE TABLE users RESTART IDENTITY;
```

## See Also
[CREATE TABLE](../create/create-table.md), [ALTER TABLE](../alter/alter-table.md), [DROP COLLECTION](drop-collection.md), [DROP DATABASE](drop-database.md)

---
[← Back to DROP](../drop.md)
