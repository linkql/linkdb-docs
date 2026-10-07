# LinkDB

**LinkDB** is the database system that [LinkQL](../language/index.md) is the
query language for: an engine that manages relational tables and
schemaless/semi-schemaless collections in a single, unified store, plus the
`linkdb` command-line client used to talk to it.

LinkDB ships as a set of independently-versioned Python packages under
`src/linkdb/` — a lexer, parser, semantic analyzer, planner, executor,
catalog, and storage layer — wired together behind one `linkdb` CLI entry
point. There is no server process and no network protocol yet: the CLI
loads the engine in-process and talks directly to on-disk (or in-memory)
storage.

## Where to go next

- **[Installation](installation.md)** — install from source and get the
  `linkdb` command on your `PATH`.
- **[CLI](cli/index.md)** — launch the CLI, connect to a database, and learn
  the backslash-command/statement input model.
- **[Language](../language/index.md)** — the LinkQL reference: data types,
  DDL/DML syntax, transactions, functions, and more.

## Project status

LinkDB is MVP-stage software under active development. There is no public
Python API yet — the CLI is the only supported way to use it today (see
[Python API](api/index.md) for what's planned). Expect gaps and rough edges;
the [Language](../language/index.md) and [CLI](cli/index.md) reference pages
call out anywhere documented syntax isn't implemented yet.
