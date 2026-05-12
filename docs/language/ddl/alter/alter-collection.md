# ALTER COLLECTION
Modifies an existing collection.

```grammar title="Grammar"
alter_collection_stmt   ::= ALTER COLLECTION identifier alter_collection_cmd

alter_collection_cmd    ::= ADD FIELD field_def
                          | DROP FIELD identifier ( KEEP )?
                          | DROP FREE FIELD identifier
                          | MODIFY FIELD field_def ( USING expr )?
                          | RENAME FIELD identifier TO identifier
                          | RENAME TO identifier
                          | ADD table_constraint
                          | DROP CONSTRAINT identifier
                          | MODIFY CONSTRAINT identifier table_constraint_type

field_def               ::= '_id'
                          | identifier data_type column_constraint*

column_constraint       ::= constraint_name? column_constraint_type

column_constraint_type  ::= NOT NULL
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
                          | SECONDARY KEY '(' identifier (',' identifier)* ')'
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
`ALTER COLLECTION` modifies the structure of an existing collection. This includes adding,
dropping, and modifying defined fields, renaming the collection or its fields, and managing
constraints.

Only **defined fields** can be modified, renamed, or dropped via `DROP FIELD`. Free fields
must be removed using `DROP FREE FIELD`.

To alter a collection you must be a **superuser** or have the `ALTER` privilege on the
current database.

## Parameters

`identifier`
:   The name of the collection to alter.

`ADD FIELD field_def`
:   Adds a new defined field to the collection. The field definition follows the same
    rules as [`CREATE COLLECTION`](create-collection.md). If the collection already has
    documents, a `DEFAULT` value must be provided so existing documents can be populated.
    If a free field with the same name already exists on a document, that value is used
    instead of the default.

`DROP FIELD identifier`
:   Removes a defined field from the catalog and scrubs the data from all documents in
    the collection. The field must be defined or an error is returned.

`DROP FIELD identifier KEEP`
:   Removes a defined field from the catalog but does not scrub the data. Existing field
    values remain on documents as free fields. The field must be defined or an error is
    returned.

`DROP FREE FIELD identifier`
:   Removes a free field's data from all documents in the collection. Documents that do
    not contain the field are skipped. Returns the number of documents affected.

`MODIFY FIELD field_def`
:   Rewrites the full field definition including name, type, and all constraints. The
    field must be defined. The entire field definition must be restated — partial updates
    are not supported.

`USING expr`
:   An optional expression used during field modification to cast or transform existing
    data to the new type. Any valid expression is accepted.

`RENAME FIELD identifier TO identifier`
:   Renames an existing defined field. The field must be defined. Does not affect the
    field's type or constraints.

`RENAME TO identifier`
:   Renames the collection. The new name must follow the same naming rules as
    [`CREATE COLLECTION`](create-collection.md) and must not already be in use by another
    table or collection.

`ADD table_constraint`
:   Adds a new table level constraint to the collection. See
    [`CREATE COLLECTION`](create-collection.md) for constraint definitions.

`DROP CONSTRAINT identifier`
:   Removes a constraint by name. The constraint type does not need to be specified —
    only the name is required.

`MODIFY CONSTRAINT identifier table_constraint_type`
:   Sugar for dropping the named constraint and adding a new one in its place. The
    constraint name is retained.

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
- Only defined fields can be modified, renamed, or dropped via `DROP FIELD`. Attempting
  to use `DROP FIELD` on a free field returns an error.
- `DROP FREE FIELD` silently skips documents that do not contain the field and returns
  the total number of documents affected.
- `MODIFY FIELD` requires the full field definition to be restated. All constraints not
  included in the new definition will be dropped.
- `MODIFY CONSTRAINT` is syntactic sugar for `DROP CONSTRAINT` followed by `ADD` with
  the same constraint name.
- Adding a field to a collection with existing documents requires a `DEFAULT` value. This
  is enforced by the engine, not the parser.
- Use [`DROP COLLECTION`](../drop/drop-collection.md) to remove a collection entirely.

## Examples

```sql title="Add a defined field"
ALTER COLLECTION posts
ADD FIELD likes INT DEFAULT 0;
```

```sql title="Drop a defined field and scrub data"
ALTER COLLECTION posts
DROP FIELD user_id;
```

```sql title="Drop a defined field but keep data as free field"
ALTER COLLECTION posts
DROP FIELD user_id KEEP;
```

```sql title="Drop a free field"
ALTER COLLECTION posts
DROP FREE FIELD likes;
```

```sql title="Modify a defined field"
ALTER COLLECTION posts
MODIFY FIELD replies ARRAY(INT) REFERENCES users(user_id);
```

```sql title="Modify a field with a cast"
ALTER COLLECTION posts
MODIFY FIELD post_id INT USING CAST(post_id AS INT);
```

```sql title="Rename a defined field"
ALTER COLLECTION posts
RENAME FIELD replies TO user_replies;
```

```sql title="Rename a collection"
ALTER COLLECTION posts
RENAME TO user_posts;
```

```sql title="Add a constraint"
ALTER COLLECTION posts
ADD CONSTRAINT CHK_content CHECK (content IS NOT NULL);
```

```sql title="Drop a constraint"
ALTER COLLECTION posts
DROP CONSTRAINT CHK_content;
```

```sql title="Modify a constraint"
ALTER COLLECTION posts
MODIFY CONSTRAINT CHK_content CHECK (content IS NOT NULL AND content != '');
```

## See Also
[CREATE COLLECTION](create-collection.md), [DROP COLLECTION](../drop/drop-collection.md), [ALTER TABLE](alter-table.md)

---
[← Back to ALTER](../alter.md)
