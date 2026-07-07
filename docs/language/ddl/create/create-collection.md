# CREATE COLLECTION
Creates a new document collection.

``` title="Grammar"
create_collection_stmt  ::= CREATE COLLECTION ( IF NOT EXISTS ) identifier
                              ( '(' collection_item (',' collection_item)* ','? ')' )?

collection_item         ::= field_def
                          | table_constraint

field_def               ::= '_id'
                          | identifier data_type field_constraint*

field_constraint        ::= named_field_constraint
                          | field_property

named_field_constraint  ::= constraint_name? named_constraint_type

named_constraint_type   ::= UNIQUE
                          | PRIMARY KEY
                          | SECONDARY KEY
                          | CHECK '(' expr ')'
                          | REFERENCES reference_target

reference_target        ::= identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?

column_property         ::= AUTOINCREMENT ( '(' integer_literal ',' integer_literal ')' )?
                          | AUTONOW
                          | AUTO
                          | DEFAULT expr
                          | NOT NULL
                          | NULL

table_constraint        ::= constraint_name? table_constraint_type

table_constraint_type   ::= PRIMARY KEY '(' identifier (',' identifier)* ')'
                          | SECONDARY KEY '(' identifier (',' identifier)* ')'
                          | UNIQUE '(' identifier (',' identifier)* ')'
                          | CHECK '(' expr ')'
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
`CREATE COLLECTION` creates a new document collection in the current database. Unlike
tables, collections do not require a predefined schema. Each document in a collection
can have its own shape.

Collections can optionally define **defined fields** — fields that every document in the
collection must contain. Any fields beyond those are **free fields** — schemaless and
flexible. A collection created with no fields at all is entirely schemaless.

To create a collection you must be a **superuser** or have the `CREATE` privilege on the
current database.

## Parameters
`IF NOT EXISTS`
:   If specified, does not throw an error if a collection with the same name already exists.
    A notice is issued instead.

`identifier`
:   The name of the collection to create. Must be unique within the database — no table
    or collection may share the same name.

`collection_item`
:   An optional field definition or table constraint. If no items are defined, the
    collection is entirely schemaless.

`field_def`
:   A defined field consisting of a name, a [data type](../data_types.md), and zero or
    more column constraints. All defined fields must be present on every document inserted
    into the collection.

`_id`
:   A shorthand field definition. Automatically creates a `UUID` primary key column named
    `_id` with `DEFAULT gen_uuid()`.

`NOT NULL`
:   The field may not contain a `NULL` value.

`UNIQUE`
:   All values in the field must be unique across the collection.

`PRIMARY KEY`
:   Marks the field as the primary key. Implies `NOT NULL` and `UNIQUE`. Defining an
    explicit `PRIMARY KEY` suppresses the auto-generation of `_id`. If no `PRIMARY KEY`
    is defined, it will instead define a `_id` field as indicated above.

`SECONDARY KEY`
:   Promotes the field to a `PRIMARY KEY` and forms a composite key with the
    auto-generated `_id` field. This is the only way to define a composite key that
    includes the auto-generated `_id`. Multiple `SECONDARY KEY` fields are all added to
    the same composite.

    `SECONDARY KEY` only exists in the parser — it is converted to a `PRIMARY KEY` at
    parse time. Beyond the `_id` composite behavior, it is identical to `PRIMARY KEY` in
    every way.

    | Scenario | Result |
    |----------|--------|
    | No primary key defined | `_id` auto-generated as sole primary key |
    | `PRIMARY KEY` defined explicitly | No `_id` generated, behaves like a table |
    | One `SECONDARY KEY` defined | `_id` auto-generated, composite of `_id` + field |
    | Multiple `SECONDARY KEY` fields | `_id` auto-generated, composite of `_id` + all fields |

`AUTOINCREMENT ( start, step )`
:   Automatically increments the field value on each insert. `start` defines the initial
    value and `step` defines the increment. Defaults to `AUTOINCREMENT(1, 1)` if no
    arguments are provided.

`AUTONOW`
:   Automatically sets the field to the current datetime on insert. Does not update on
    subsequent updates. For use with `DATE`, `TIME`, or `DATETIME` fields.

`AUTO`
:   Automatically sets the field to the current datetime on insert and on every subsequent
    update. For use with `DATE`, `TIME`, or `DATETIME` fields.

`DEFAULT expression`
:   Sets a default value for the field when no value is provided on insert.

`CHECK ( expression )`
:   Defines a condition that must be true for any value inserted into the field.

`REFERENCES identifier ( field )`
:   Defines a foreign key relationship to another table or collection. The engine resolves
    whether the target is a table or collection automatically via the catalog.

`ON DELETE reference_action`
:   Defines the behavior when the referenced row or document is deleted.

`ON UPDATE reference_action`
:   Defines the behavior when the referenced row or document is updated.

`CONSTRAINT identifier`
:   Assigns a name to a constraint. Named constraints can be referenced in
    [`ALTER COLLECTION`](../alter/alter-collection.md) statements.

### Reference Actions

`CASCADE`
:   Propagates the delete or update to dependent documents.

`SET NULL`
:   Sets the foreign key field to `NULL` when the referenced row or document is deleted
    or updated.

`SET DEFAULT`
:   Sets the foreign key field to its default value when the referenced row or document
    is deleted or updated.

`RESTRICT`
:   Prevents the delete or update if dependent documents exist. This is the default.

`NO ACTION`
:   Same as `RESTRICT` but deferred — the check is performed at the end of the
    transaction rather than immediately.

## Notes
- If there are no `PRIMARY KEY` definitions, it will create a schemaless collection with the auto-generated `_id` field.
- `CREATE COLLECTION` cannot be used inside a transactional block.
- Any fields defined at creation time become **defined fields**. Every document inserted
  into the collection must include all defined fields.
- Fields added to a document beyond the defined fields are **free fields** — they are not
  tracked in the catalog and are not required on any other document.
- Use [`DROP COLLECTION`](../drop/drop-collection.md) to remove a collection.
- Use [`ALTER COLLECTION`](../alter/alter-collection.md) to modify an existing collection.
- Foreign key references are validated at commit time, not inline.

## Examples

```sql title="Fully schemaless collection"
-- This will include the auto-generated _id field.
CREATE COLLECTION posts;
```

```sql title="Collection with a secondary key"
-- This will create a composite of _id + user_id
CREATE COLLECTION posts (
    user_id INT SECONDARY KEY
);
```

```sql title="Collection with defined fields"
-- No primary or secondary key's are defined, so this has an _id field.
CREATE COLLECTION posts (
    user_id INT REFERENCES users(user_id),
    content JSON,
    created_at DATETIME AUTONOW,
    last_updated DATETIME AUTO
);
```

```sql title="Collection with multiple secondary keys"
-- No PRIMARY KEY, so it will be _id + order_id + event_type
CREATE COLLECTION order_events (
    order_id SECONDARY KEY,
    event_type SECONDARY KEY,
    payload JSON
);
```

## See Also
[ALTER COLLECTION](../alter/alter-collection.md), [DROP COLLECTION](../drop/drop-collection.md), [CREATE TABLE](create-table.md), [Data Types](../data_types.md)

---
[← Back to CREATE](../create.md)
