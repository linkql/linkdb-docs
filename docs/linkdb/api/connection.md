# Connection

A `Connection` is one open database. Get one from `connect()`.

## `connect()`

```python
linkdb.connect(target=":memory:", *, create=False) -> Connection
```

| Parameter | Description |
|-----------|-------------|
| `target` | What to open (see below). Defaults to `":memory:"`. Accepts `str` or `os.PathLike`. |
| `create` | Create the database if it doesn't exist. Defaults to `False`. |

`target` can be:

- **A bare name** such as `"demo"` — the database stored at
  `~/.linkdb/databases/demo`. See [Where named databases
  live](#where-named-databases-live) to change that folder.
- **A path** — anything that starts with `.` or `~`, or contains a path
  separator (`"./demo"`, `"/data/demo"`, `"~/demo"`). The path is the database
  directory itself. The database's name is the last path component.
- **`":memory:"`** — a throwaway in-memory database. Nothing is written to
  disk, and each in-memory connection is independent of every other.

If the database doesn't exist, `connect()` raises `LinkDBConnectionError`
unless you pass `create=True`.

```python
conn = linkdb.connect("demo", create=True)   # ~/.linkdb/databases/demo
conn = linkdb.connect("./demo")              # a database in the current folder
conn = linkdb.connect()                      # in-memory
```

!!! note
    Unlike the [CLI](../cli/commands.md#connection), `connect()` does not
    search the current folder for a bare name. Write `"./demo"` to open a
    database there.

## `Connection`

A connection autocommits: each statement is applied and saved as soon as it
runs, unless you have opened a transaction with `BEGIN` (see
[Transactions](#transactions)). On-disk data and schema changes are written
after every statement.

| Member | Description |
|--------|-------------|
| `execute(sql, params=None)` | Run one statement. See [Query Execution](query_execution.md#execute). |
| `executescript(sql)` | Run several statements. See [Query Execution](query_execution.md#executescript). |
| `cursor()` | Return a [`Cursor`](query_execution.md#cursors). |
| `close()` | Close the connection. Safe to call more than once. |
| `name` | The database name: the bare name, the last path component, or `":memory:"`. |
| `closed` | `True` once the connection is closed. |
| `in_transaction` | `True` while a `BEGIN` is open. Raises `LinkDBConnectionError` on a closed connection. |

A `Connection` is a context manager. Leaving the `with` block calls
`close()`:

```python
with linkdb.connect("demo") as conn:
    conn.execute("SELECT 1")
# conn is closed here
```

Using a closed connection raises `LinkDBConnectionError`.

### Transactions

Send the transaction statements through `execute()`:

```python
conn.execute("BEGIN")
conn.execute("DELETE FROM users")
print(conn.in_transaction)   # True
conn.execute("ROLLBACK")     # the delete is undone
```

There are no `commit()` or `rollback()` methods. `close()` rolls back a
transaction that is still open. See [Transactions](../../language/transactions.md)
for what the language guarantees.

## Managing databases

These helpers create and delete on-disk databases without opening one. They
do the same work as the `CREATE DATABASE` and `DROP DATABASE` statements.

### `create_database()`

```python
linkdb.create_database(target, *, if_not_exists=False) -> None
```

Creates an empty database from a bare name or a directory path. Raises
`LinkDBError` if it already exists, unless `if_not_exists=True`, and if
`target` is `":memory:"`.

### `drop_database()`

```python
linkdb.drop_database(name, *, confirm, if_exists=False) -> None
```

Deletes the database `name` and all of its data. `name` must be a bare name,
not a path. `confirm` is required and must repeat the name
(case-insensitively), as the `CONFIRM` clause of `DROP DATABASE` does. Raises
`LinkDBError` if the confirmation doesn't match or the database doesn't exist,
unless `if_exists=True`.

```python
linkdb.drop_database("demo", confirm="demo")
```

Close any connection to the database before dropping it.

## Where named databases live

Bare-name databases live under `~/.linkdb/databases`. To use a different
folder for this process:

```python
linkdb.set_default_database_root("/data/linkdb")
linkdb.set_default_database_root(None)   # back to ~/.linkdb/databases
```

The setting applies to `connect()`, `create_database()`, `drop_database()`,
and to `CREATE DATABASE` / `DROP DATABASE` statements you run. It does not
affect paths. The CLI's `--root` option does the same thing; see
[CLI](../cli/index.md#choosing-where-databases-live).
