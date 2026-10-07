# Commands
Every backslash command the CLI understands, plus the `CREATE`/`ALTER`/
`DROP`/`TRUNCATE DATABASE` statements it executes itself. See [CLI](index.md)
for how backslash commands fit into the overall interaction model.

Bracketed letters mark the shortest accepted abbreviation, matching `psql`'s
own convention: `\c[onnect]` accepts both `\c` and `\connect`.

Everything below also appears, grouped the same way, when you type `\?` at
the prompt; `\? <command>` (e.g. `\? connect`) prints just one command's
entry.

## General

### `\q[uit]`
Quits the CLI immediately. No confirmation, no args — if a transaction is
open on the current connection it is simply abandoned, not rolled back or
committed.

### `\cls` / `\clear`
Clears the terminal screen (`cls` on Windows, `clear` elsewhere).

### `\s[tatus]`
Prints a table summarizing the current session:

| Property      | Value                                            |
|----------------|---------------------------------------------------|
| Database       | the connected database's name, or `None`          |
| Pipeline Mode  | the current [pipeline mode](#execution)           |
| Display Mode   | the current [display mode](#formatting)           |
| Version        | the CLI's version                                 |

## Help

### `\?`
Prints every command below, grouped by category, through a pager.

### `\? <command>`
Prints the entry for just one command — look it up by any of its accepted
names, with or without the leading backslash (`\? c` and `\? connect` are
equivalent). Unknown names raise an error rather than printing nothing.

## Connection

### `\c[onnect] <database_name>`
Connects to an already-existing database.

- `<database_name>` is required and is a logical name, never a path — it
  resolves to a fixed directory, `~/.linkdb/databases/<database_name>/`.
- **Never auto-creates.** If no directory exists under that name, this
  errors (`Database '<name>' does not exist.`) rather than creating one —
  use [`CREATE DATABASE`](#database-ddl-statements) first.
- Refuses to run if the *current* connection has an open transaction
  (`COMMIT`/`ROLLBACK` it first) — connecting swaps out the catalog, data
  store, planner, and executor entirely, so an in-flight transaction on the
  old connection would otherwise be silently abandoned.
- On success, the prompt changes from `linkdb=#` to `<database_name>=#`,
  and every statement from then on reads from and writes to that database's
  on-disk storage.

### `\disc[onnect]`
Disconnects from the current database and returns to the zero-config,
in-memory default (see [CLI](index.md#zero-config-default-database)).

- Errors if there is no active connection (`No active connection to
  disconnect.`).
- Errors if the current connection has an open transaction, for the same
  reason as `\c` above.

### `\l[ist]`
Lists every database under `~/.linkdb/databases/` (an absent root directory
means zero databases, not an error):

| Name     | Current |
|----------|---------|
| mydb     | `*`     |
| otherdb  |         |

The currently-connected database's row is marked `*` in the `Current`
column.

## Inspection

These mirror `psql`'s `\d`-family, with one LinkDB-specific addition
(`\dc`, for collections — LinkDB's schemaless analogue to tables). They
read from `state.catalog` directly and work identically against either the
zero-config in-memory catalog or a real connected database.

### `\d [name]`
Describes one table, collection, or view, or summarizes all of them if no
name is given.

- **Bare `\d`** — a `Name`/`Kind` table listing every table, collection, and
  view (sorted by name). Matches `psql`'s own bare `\d`, which likewise
  excludes indexes; use `\di` for those.
- **`\d <name>`** — resolves `<name>` against tables, collections, and
  views (whichever it is); errors if nothing matches.
    - For a **table or collection**: a `Column`/`Type`/`Nullable`/`Default`/
      `Identity` table, one row per column/field (`Identity` is
      `AUTOINCREMENT`, `AUTONOW`, `AUTO`, or blank). If the table/collection
      has any constraints, a second `Constraint`/`Kind`/`Columns` table
      follows.
    - For a **view**: a `Property`/`Value` table (`Materialized`: `YES`/`NO`,
      `Definition`: the view's defining query text), followed by a
      `Column`/`Type` table if the view has resolved columns.

### `\dt`
Lists every table: a `Name`/`Columns` table (column count), sorted by name.

### `\dc`
Lists every collection: a `Name`/`Fields` table (field count), sorted by
name.

### `\dv`
Lists every view: a `Name`/`Materialized` (`YES`/`NO`) table, sorted by
name.

### `\di`
Lists every index: a `Name`/`Owner`/`Unique` (`YES`/`NO`) table, sorted by
name.

## Execution

### `\p[ipeline] <mode>`
Changes which pipeline stage a dispatched statement stops at, or shows the
current mode if no `<mode>` is given. One of `lexer`, `parser`, `analyzer`,
`planner`, or `executor` (case-insensitive); anything else errors. The
session starts in `executor` mode — statements execute immediately unless
you deliberately switch to an earlier stage.

Every mode below `executor` is read-only by construction: it stops before
anything capable of a side effect runs, which makes `\p` useful for
inspecting how a statement gets lexed, parsed, analyzed, or planned
without actually running it.

| Mode       | What a dispatched statement does                                                    |
|------------|----------------------------------------------------------------------------------------|
| `lexer`    | Tokenized, then the token stream is printed as a table (`Line`/`Column`/`Token Type`/`Lexeme`/`Literal`). |
| `parser`   | Parsed, then the resulting syntax tree is printed.                                     |
| `analyzer` | Parsed and analyzed (names resolved, types checked against the catalog), then the analyzed tree is printed. |
| `planner`  | Parsed, analyzed, and planned, then the resulting plan tree is printed.                |
| `executor` | Parsed, analyzed, planned, and actually executed. **The default.** A `SELECT` (or a write with `RETURNING`) renders as a table; `EXPLAIN`/`EXPLAIN ANALYZE` prints a plan tree; a plain write prints an affected-row-count notice; DDL confirmations print as a notice. `CREATE`/`ALTER`/`DROP`/`TRUNCATE DATABASE` (see [below](#database-ddl-statements)) only take effect in this mode — lower modes show their analyzed/planned shape with no filesystem side effect. |

## Formatting

### `\dm <mode>`
Changes how tabular results are rendered: `table` (the default, a bordered
grid) or `expanded` (one row per screen, each column on its own line —
matches `psql`'s `\x`). Requires `<mode>`; omitting it is an error (use
`\s[tatus]` to see the current display mode instead).

!!! note "Only affects `lexer`- and `executor`-mode output"
    `\dm` changes how `lexer`-mode's token table and `executor`-mode's
    statement results (table/`RETURNING`/write-affected-rows) are rendered.
    It has no effect on `\d`/`\dt`/`\dc`/`\dv`/`\di`/`\l`/`\s`, or on the
    syntax/analyzed/plan trees `parser`/`analyzer`/`planner` mode print —
    those always render as a fixed-format table or tree regardless of the
    current display mode.

## Input/Output

### `\i <file_path>`
Reads `<file_path>` (conventionally `.sql` or `.lql`) and runs every
statement in it through the same dispatch path typed input uses, respecting
whatever [pipeline mode](#execution) is currently set — the `executor`
default, unless you've switched to an earlier stage, in which case `\i`
only prints that stage's output for the whole file instead of running it.

- Errors (file not found, unreadable, etc.) raise before anything runs.
- Statements are split the same way typed multi-line input is: lines
  accumulate until one ends with `;`; a trailing, unterminated accumulation
  at end-of-file is silently dropped rather than run as a partial statement.
- A single statement's error is displayed and that statement is skipped —
  the rest of the file still runs. Each statement that completes is
  recorded to history, same as typed input.

## Database DDL statements

`CREATE DATABASE`, `ALTER DATABASE ... RENAME TO`, `DROP DATABASE`, and
`TRUNCATE DATABASE` are LinkQL statements typed and terminated with `;`
exactly like any other — not backslash commands — but they're documented
here because of *where* they run: the CLI intercepts all four and executes
them itself, as plain filesystem operations against
`~/.linkdb/databases/`, before they would otherwise reach `Planner`/
`Executor`. `linkdb_executor` itself still refuses to execute these four
statement types (by design — see ADR 0007 in its own docs); only the CLI
knows how to act on them, and only once [pipeline mode](#execution) has
reached `executor`.

There is no privilege/role system anywhere in LinkDB (no `superuser`
concept, despite what some DDL reference pages describe) and no
`CONFIRM_DROP` setting to bypass confirmation — both are aspirational
language in those pages, not real behavior. Nothing here enforces who may
run these statements.

- **`CREATE DATABASE [IF NOT EXISTS] name`** creates
  `~/.linkdb/databases/<name>/` (just the directory — no marker file is
  needed; any directory under the root counts as a database for `\c`/`\l`).
  Errors if it already exists, unless `IF NOT EXISTS` is given, in which
  case a notice is issued instead.
- **`DROP DATABASE [IF EXISTS] name CONFIRM 'name'`** deletes the database
  directory and everything in it. The `CONFIRM` string must match `name`
  case-insensitively or this errors without deleting anything. Also refuses
  (regardless of `CONFIRM`) if `name` is the *currently connected* database
  — disconnect first. Missing directory errors unless `IF EXISTS` is given.
- **`TRUNCATE DATABASE [IF EXISTS] name CONFIRM 'name' [RESTART IDENTITY]`**
  removes everything under the database's `data/` directory but leaves its
  schema (`system/`) intact. Same `CONFIRM` and self-targeting checks as
  `DROP DATABASE`. `RESTART IDENTITY` is accepted but has no extra effect:
  autoincrement values are always recomputed live from existing data, so
  wiping `data/` already resets them regardless of whether the clause is
  present.
- **`ALTER DATABASE name RENAME TO new_name`** renames the directory.
  Errors if `new_name` already exists, or if `name` is the currently
  connected database (same self-targeting guard as above).

## Implicit transaction wrapping of multi-statement batches

More than one semicolon-terminated statement can end up dispatched
together as a single batch — most commonly several statements typed on one
physical line (`INSERT ...; INSERT ...;`), but the same rule applies to any
one line of an [`\i`](#i-file_path) file. In `executor`
[pipeline mode](#execution), a batch of **more than one** top-level
statement is automatically wrapped in an implicit transaction (`BEGIN`,
then every statement in order, then `COMMIT`; any failure triggers a full
`ROLLBACK` before the error is raised) — *unless* the batch disqualifies
itself by already managing its own transaction:

- it contains a `BEGIN`/`COMMIT`/`ROLLBACK`/`SAVEPOINT`/`RELEASE SAVEPOINT`
  statement, or a `TRANSACTION ... DO ... END` block, anywhere in it;
- a transaction from an *earlier* dispatched line is already open; or
- it contains a `CREATE`/`ALTER`/`DROP`/`TRUNCATE DATABASE` statement — a
  filesystem operation outside the transaction machinery's reach, so it
  can't be rolled back if a later statement in the same batch fails.

A disqualified batch instead runs each statement independently, exactly as
if they'd been typed on separate lines: earlier statements' effects stand
even if a later one in the same batch fails.

Either way, a failure's error message names the failing statement by
position and (regenerated) text: `statement 2 of 3 (INSERT INTO widgets
(id, name) VALUES (1, 'a')) failed: <original error>`.

See [Transactions](../../language/transactions.md) for what a transaction
guarantees once started, explicitly or implicitly.

## See Also
[CLI](index.md), [Transactions](../../language/transactions.md)

---
[← Back to CLI](index.md)
