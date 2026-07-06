# ALTER TABLE
Modifies an existing table.

```grammar title="Grammar"
alter_table_stmt        ::= ALTER TABLE identifier alter_table_cmd

alter_table_cmd         ::= ADD COLUMN field_def
                          | DROP COLUMN identifier
                          | MODIFY COLUMN field_def ( USING expr )?
                          | RENAME COLUMN identifier TO identifier
                          | RENAME TO identifier
                          | ADD table_constraint
                          | MODIFY table_constraint
                          | DROP CONSTRAINT identifier

field_def               ::= '_id'
                          | identifier data_type column_constraint*

column_constraint       ::= constraint_name? column_constraint_type

column_constraint_type  ::= NOT NULL
                          | NULL
                          | UNIQUE
                          | PRIMARY KEY
                          | AUTOINCREMENT ( '(' integer_literal ',' integer_literal ')' )?
                          | AUTONOW
                          | AUTO
                          | DEFAULT expression
                          | CHECK '(' expression ')'
                          | REFERENCES identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?

table_constraint        ::= constraint_name? table_constraint_type

table_constraint_type   ::= PRIMARY KEY '(' identifier (',' identifier)* ')'
                          | UNIQUE '(' identifier (',' identifier)* ')'
                          | CHECK '(' expression ')'
                          | FOREIGN KEY '(' identifier ')' REFERENCES identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?

constraint_name         ::= CONSTRAINT identifier

reference_action        ::= CASCADE
                          | SET NULL
                          | SET DEFAULT
                          | RESTRICT
                          | NO ACTION
```

## Description
`ALTER TABLE` modifies the structure of an existing table. This includes adding, dropping,
and modifying columns, renaming the table or its columns, and managing table constraints.

To alter a table you must be a **superuser** or have the `ALTER` privilege on the current
database.

## Parameters

`identifier`
:   The name of the table to alter.

`ADD COLUMN field_def`
:   Adds a new column to the table. The column definition follows the same rules as
    [`CREATE TABLE`](create-table.md). If the table already has rows, a `DEFAULT` value
    must be provided so existing rows can be populated.

`DROP COLUMN identifier`
:   Removes a column from the table. If the column has dependencies — such as constraints
    or foreign key references from other tables — the engine follows the same precedence
    as cascade on a table drop:
    - Named constraint behavior takes precedence
    - Inline definition behavior takes precedence over the default
    - Defaults to `RESTRICT` if no behavior is defined

`MODIFY COLUMN field_def`
:   Rewrites the full column definition including name, type, and all constraints. The
    entire column definition must be restated — partial updates are not supported.

`USING expr`
:   An optional expression used during column modification to cast or transform existing
    data to the new type. Any valid expression is accepted.

`RENAME COLUMN identifier TO identifier`
:   Renames an existing column. Does not affect the column's type or constraints.

`RENAME TO identifier`
:   Renames the table. The new name must follow the same naming rules as
    [`CREATE TABLE`](create-table.md) and must not already be in use by another table
    or collection.

`ADD table_constraint`
:   Adds a new table level constraint. See [`CREATE TABLE`](create-table.md) for
    constraint definitions.

`DROP CONSTRAINT identifier`
:   Removes a constraint by name. The constraint type does not need to be specified —
    only the name is required.

`MODIFY CONSTRAINT identifier table_constraint_type`
:   Sugar for dropping the named constraint and adding a new one in its place. The
    constraint name is retained.

### Reference Actions

`CASCADE`
:   Propagates the delete or update to dependent rows.

`SET NULL`
:   Sets the foreign key column to `NULL` when the referenced row is deleted or updated.

`SET DEFAULT`
:   Sets the foreign key column to its default value when the referenced row is deleted
    or updated.

`RESTRICT`
:   Prevents the delete or update if dependent rows exist. This is the default.

`NO ACTION`
:   Same as `RESTRICT` but deferred — the check is performed at the end of the
    transaction rather than immediately.

## Notes
- `MODIFY COLUMN` requires the full column definition to be restated. All constraints
  not included in the new definition will be dropped.
- `MODIFY CONSTRAINT` is syntactic sugar for `DROP CONSTRAINT` followed by `ADD` with
  the same constraint name.
- Adding a column to a table with existing rows requires a `DEFAULT` value. This is
  enforced by the engine, not the parser.
- Column constraints are modified via `MODIFY COLUMN`. Table level constraints are
  modified via `MODIFY CONSTRAINT`.
- Use [`DROP TABLE`](../drop/drop-table.md) to remove a table entirely.

## Examples

```sql title="Add a column"
ALTER TABLE users
ADD COLUMN phone_number VARCHAR(10) DEFAULT '000-000-0000';
```

```sql title="Drop a column"
ALTER TABLE users
DROP COLUMN email;
```

```sql title="Modify a column"
ALTER TABLE users
MODIFY COLUMN phone_number INT NOT NULL DEFAULT 0;
```

```sql title="Modify a column with a cast"
ALTER TABLE users
MODIFY COLUMN phone_number INT USING CAST(REPLACE(phone_number, '-', '') AS INT);
```

```sql title="Rename a column"
ALTER TABLE users
RENAME COLUMN phone_number TO phone;
```

```sql title="Rename a table"
ALTER TABLE users
RENAME TO user_data;
```

```sql title="Add a named constraint"
ALTER TABLE users
ADD CONSTRAINT CHK_username UNIQUE(username);
```

```sql title="Drop a constraint"
ALTER TABLE users
DROP CONSTRAINT CHK_username;
```

```sql title="Modify a constraint"
ALTER TABLE users
MODIFY CONSTRAINT CHK_username UNIQUE(id, username);
```

## See Also
[CREATE TABLE](create-table.md), [DROP TABLE](../drop/drop-table.md), [ALTER COLLECTION](alter-collection.md)

---
[← Back to ALTER](../alter.md)
