# CREATE TABLE
Creates a new table for relational data.

```ebnf title="Grammar"
create_table_stmt       ::= CREATE TABLE identifier '(' table_item (',' table_item)* ')'

table_item              ::= field_def
                          | table_constraint

field_def               ::= '_id'
                          | identifier data_type column_constraint*

column_constraint       ::= NOT NULL
                          | NULL
                          | UNIQUE
                          | PRIMARY KEY
                          | SECONDARY KEY
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
`CREATE TABLE` creates a new relational table in the current database. All columns and their types must be defined at creation time. Tables are best suited for data with a consistent, well-known shape

## Parameters

`identifier`
:   The name of the table to create. Must be unique within the database — no table or
    collection may share the same name.

`field_def`
:   A column definition consisting of a name, a [data type](../data_types.md), and zero
    or more column constraints.

`_id`
:   A shorthand field definition. Automatically creates a `UUID` primary key column named
    `_id` with `DEFAULT gen_uuid()`.

`NOT NULL`
:   The column may not contain a `NULL` value.

`UNIQUE`
:   All values in the column must be unique across the table.

`PRIMARY KEY`
:   Marks the column as the primary key. Implies `NOT NULL` and `UNIQUE`. Only one primary
    key may be defined per table.

`SECONDARY KEY`
:   Syntactic sugar. Converts to a `PRIMARY KEY` and forms a composite key with `_id`.
    Multiple `SECONDARY KEY` fields are all added to the composite. Does not exist in the
    engine — parser only.

`AUTOINCREMENT ( start, step )`
:   Automatically increments the column value on each insert. `start` defines the initial
    value and `step` defines the increment. Defaults to `AUTOINCREMENT(1, 1)` if no
    arguments are provided.

`AUTONOW`
:   Automatically sets the column to the current datetime on insert. Does not update on
    subsequent updates. For use with `DATE`, `TIME`, or `DATETIME` columns.

`AUTO`
:   Automatically sets the column to the current datetime on insert and on every subsequent
    update. For use with `DATE`, `TIME`, or `DATETIME` columns.

`DEFAULT expression`
:   Sets a default value for the column when no value is provided on insert.

`CHECK ( expression )`
:   Defines a condition that must be true for any value inserted into the column.

`REFERENCES identifier ( column )`
:   Defines a foreign key relationship to another table or collection. The engine resolves
    whether the target is a table or collection automatically via the catalog.

`ON DELETE reference_action`
:   Defines the behavior when the referenced row is deleted. See reference actions below.

`ON UPDATE reference_action`
:   Defines the behavior when the referenced row is updated.

`CONSTRAINT identifier`
:   Assigns a name to a constraint. Named constraints can be referenced in
    [`ALTER TABLE`](../alter/alter-table.md) statements.

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
- Table names must be unique within the database. No table or collection may share the
  same name.
- `CREATE TABLE` cannot be used inside a transactional block.
- Use [`DROP TABLE`](../drop/drop-table.md) to remove a table.
- Use [`ALTER TABLE`](../alter/alter-table.md) to modify an existing table.
- Foreign key references are validated at commit time, not inline.

## Examples

```sql title="Basic table creation"
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY AUTOINCREMENT(1000, 1),
    username VARCHAR(50) UNIQUE,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    profile_data JSON,
    metadata JSONB
);
```

```sql title="Using the _id shorthand"
CREATE TABLE comments (
    _id,
    user_id BIGINT REFERENCES users(user_id),
    post_id BIGINT REFERENCES posts(post_id) ON DELETE SET NULL
);
```

```sql title="Named constraints and composite primary key"
CREATE TABLE order_items (
    order_id BIGINT,
    item_id BIGINT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, item_id),
    CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    CONSTRAINT fk_item FOREIGN KEY (item_id) REFERENCES items(item_id) ON DELETE RESTRICT
);
```

## See Also
[ALTER TABLE](../alter/alter-table.md), [DROP TABLE](../drop/drop-table.md), [CREATE COLLECTION](create-collection.md), [Data Types](../data_types.md)

---
[← Back to CREATE](../create.md)
