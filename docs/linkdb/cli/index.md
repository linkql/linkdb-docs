# CLI
An interactive, `psql`-like terminal for LinkDB.

## Launching

Installing the `linkdb` package puts a `linkdb` console script on your
`PATH` (it points at `linkdb.cli.repl:main`). Run it with no arguments:

```sh title="Launch the CLI"
linkdb
```

You'll see a welcome banner followed by a prompt:

```text
┌────────────────────────────┐
│      LinkDB CLI v0.1.0      │
└────────────────────────────┘
      Type \? for help, \q to quit.

linkdb=#
```

## Zero-config default database

The CLI never requires you to connect to anything before you start typing
statements. Before you run `\c` for the first time (or after `\disc[onnect]`),
the prompt reads `linkdb=#` and every statement you type runs against an
empty, **in-memory** database — no files are read or written, and nothing
you create survives after you quit.

This default stack exists purely so you can experiment (try a `SELECT`,
sketch out a `CREATE TABLE`) without first having to create or connect to a
real, on-disk database. To work with persistent data, create a database with
`CREATE DATABASE <name>;` and connect to it with `\c <name>` — see
[Commands](commands.md) for both. Once connected, the prompt changes to
`<name>=#` and every statement reads from and writes to that database's
on-disk storage under `~/.linkdb/databases/<name>/`.

!!! warning "Statements don't execute until you switch pipeline mode"
    The CLI starts in `parser` pipeline mode, not `executor` — see
    [Commands → Execution](commands.md#execution). Out of the box, typing a
    statement only prints its parsed syntax tree; nothing is planned,
    analyzed, or run against the default database. Run `\p executor` (or
    `\p[ipeline] executor`) once per session to make statements actually
    execute. This is current, unpolished first-run behavior, not a
    deliberate "safe mode."

## Interaction model

The CLI reads from standard input in two kinds of input:

- **Backslash commands** — a line whose first non-whitespace character is
  `\`, e.g. `\c mydb` or `\dt`. These run immediately, one line at a time;
  they're never split across lines and never need a trailing `;`. See
  [Commands](commands.md) for the full list, or type `\?` at the prompt.
- **LinkQL statements** — everything else. These are buffered line by line
  until a line ends with `;`, at which point everything buffered since the
  last statement boundary is dispatched together. This lets you write a
  statement across several lines:

```text
linkdb=# CREATE TABLE users (
-#     id INT AUTOINCREMENT,
-#     username VARCHAR(50)
-# );
```

The prompt itself reflects this: it reads `=#` while the input buffer is
empty and `-#` while a statement is still being accumulated, matching the
connection-name/`linkdb` prefix either way.

### Multi-statement batches

Typing more than one `;`-terminated statement on the **same** line (e.g.
`INSERT ...; INSERT ...;`) dispatches them together as one batch — unlike a
single statement spread *across* several lines (like the `CREATE TABLE`
above), which is still just one statement once its closing `;` arrives. A
batch of more than one top-level statement is automatically wrapped in an
implicit transaction — see
[Commands → Implicit transaction wrapping of multi-statement batches](commands.md#implicit-transaction-wrapping-of-multi-statement-batches)
for exactly when that wrap applies, and [Transactions](../../language/transactions.md)
for what a transaction guarantees.

## Help

`\?` prints every backslash command grouped by category, piped through a
pager. `\? <command>` (e.g. `\? connect`) prints help for just that one
command. See [Commands](commands.md) for the same list laid out as a
reference page.
