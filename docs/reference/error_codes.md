# Errors

LinkDB has no numeric error-code scheme. Errors are plain Python exception
classes rooted at `LinkDBError`, defined in
`linkdb_core.exceptions.error`. Every instance carries a human-readable
`message`; `LexerError` and `ParserError` additionally carry `line`,
`column`, and `lexeme` so the failure can be pointed at in the source text.

When the CLI catches a `LinkDBError`, it prints the exception's class name
(with the trailing `Error` stripped) and its message, e.g.
`ERROR<ObjectNotFound>: ...`. If the error carries `line`/`column`/`lexeme`,
the CLI also prints the offending source line with a caret under the
lexeme.

## Hierarchy

```
LinkDBError            base class; also raised directly for CLI-layer errors
                        (unknown REPL command, bad database admin arguments,
                        no active connection, etc.)
├── LexerError          tokenizing failure
├── ParserError         syntax error while parsing a statement
├── PlannerError        query planning failure
├── ExecutorError       statement execution failure
├── GeneratorError      SQL/plan generation failure
├── AnalyzerError       semantic analysis failure
├── StorageError        data store read/write failure
└── CatalogError        catalog operation failure
    ├── ObjectNotFoundError     referenced table/collection/view/index missing
    ├── DuplicateObjectError    object with that name already exists
    └── ConstraintNotFoundError referenced constraint missing
```

## Reference

### `LinkDBError`

Base class for every LinkDB error. Also raised directly by the CLI layer
(`linkdb.cli`) for problems that aren't tied to a specific pipeline stage —
an unknown `\`-command, a missing argument to a command, a `CREATE`/`ALTER`/
`DROP` `DATABASE` naming conflict, or no active connection to disconnect.

Carries: `message`.

### `LexerError`

Raised when the lexer can't tokenize the source — e.g. an unterminated
string or an unexpected character.

Carries: `message`, `line`, `column`, `lexeme`.

### `ParserError`

Raised when a statement doesn't match the grammar — an unexpected token, a
reserved keyword used unquoted as an identifier, or unexpected end of input.

Carries: `message`, `line`, `column`, `lexeme` (empty string if not
applicable).

### `PlannerError`

Raised when the planner is given a statement or construct it doesn't know
how to plan, e.g. an unsupported statement type or an unsupported statement
inside a `FOR` loop body.

Carries: `message` (stored as `"PlannerError: {message}"`).

### `ExecutorError`

Raised when a DDL or DML operation fails at execution time — an
unsupported `CASCADE` clause, dropping an object still referenced by
another, or an `ALTER` targeting a column, field, or constraint that
doesn't exist.

Carries: `message` (stored as `"ExecutorError: {message}"`).

### `GeneratorError`

Raised when code generation hits a node it can't translate — an
unsupported literal type, data type, or operator, or a malformed `IS NULL`
/ `IS MISSING` check.

Carries: `message` (stored as `"GeneratorError: {message}"`).

### `AnalyzerError`

Raised when semantic analysis fails — an unsupported statement type, a
`LET` binding missing its statement, or an unresolvable `INFO` target.

Carries: `message` (stored as `"AnalyzerError: {message}"`).

### `StorageError`

Raised when the data store can't read or write a table's or collection's
data — e.g. malformed JSON found on disk.

Carries: `message` (stored as `"StorageError: {message}"`).

### `CatalogError`

Base class for catalog-related errors, and raised directly for catalog
operations that are invalid independent of a missing or duplicate object —
no system store configured, creating an index on a view, adding a
constraint to a view, or a materialized view's stored statements being
missing or not a `SELECT`.

Carries: `message` (stored as `"CatalogError: {message}"`).

#### `ObjectNotFoundError`

Raised when a referenced table, collection, view, or index isn't in the
catalog.

Carries: `message`, optional — defaults to a generic "not found" message.
Because it calls into `CatalogError`'s constructor, the stored message is
double-prefixed: `"CatalogError: ObjectNotFoundError: {message}"`.

#### `DuplicateObjectError`

Raised when creating an object (index, constraint, table, or collection)
whose name is already taken.

Carries: `message`, optional — defaults to a generic "already exists"
message. Stored as `"CatalogError: DuplicateObjectError: {message}"`.

#### `ConstraintNotFoundError`

Raised when an operation (e.g. `DROP CONSTRAINT`) references a constraint
that doesn't exist in the catalog.

Carries: `message`. Stored as
`"CatalogError: ConstraintNotFoundError: {message}."` (note the appended
period).

---

## See Also
[Grammar](grammar.md), [Keywords](keywords.md), [Data Types](../data_types.md)
