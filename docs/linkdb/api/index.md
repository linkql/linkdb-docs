# Python API

LinkDB can be embedded in your own Python code. The `linkdb` package exports
a small, synchronous API: open a [connection](connection.md), [execute
statements](query_execution.md) on it, and read the [results](results.md).
The engine runs in-process — there is no server to start.

```python
import linkdb

linkdb.create_database("demo", if_not_exists=True)

with linkdb.connect("demo") as conn:
    conn.execute("""
        CREATE TABLE IF NOT EXISTS users (
            user_id  BIGINT PRIMARY KEY AUTOINCREMENT(1, 1),
            username VARCHAR(50) NOT NULL
        )
    """)
    conn.execute("INSERT INTO users (username) VALUES (?)", ["john"])
    conn.execute("INSERT INTO users (username) VALUES (:name)", {"name": "jane"})

    result = conn.execute("SELECT user_id, username FROM users ORDER BY user_id")
    print(result.columns)  # ['user_id', 'username']
    print(result.rows)     # [{'user_id': 1, 'username': 'john'}, {'user_id': 2, 'username': 'jane'}]

linkdb.drop_database("demo", confirm="demo")
```

Install the package first — see [Installation](../installation.md). The
distribution is named `linkdbms`, but you import it as `linkdb`.

## What's exported

| Name | Kind | Page |
|------|------|------|
| `connect()` | function | [Connection](connection.md#connect) |
| `Connection` | class | [Connection](connection.md#connection) |
| `create_database()` | function | [Connection](connection.md#managing-databases) |
| `drop_database()` | function | [Connection](connection.md#managing-databases) |
| `set_default_database_root()` | function | [Connection](connection.md#where-named-databases-live) |
| `Cursor` | class | [Query Execution](query_execution.md#cursors) |
| `Result` | dataclass | [Results](results.md) |
| `LinkDBError` | exception | [Errors](#errors) |
| `LinkDBConnectionError` | exception | [Errors](#errors) |
| `LinkQLSyntaxError` | exception | [Errors](#errors) |

The names load on first use rather than at `import linkdb`, so importing the
package is cheap.

## Errors

Every error the API raises is a `LinkDBError`, so one `except` clause covers
all of them:

| Exception | Raised when |
|-----------|-------------|
| `LinkDBConnectionError` | A database can't be opened, or a closed connection or cursor is used. |
| `LinkQLSyntaxError` | A statement can't be lexed or parsed. Carries `line`, `column`, `lexeme`, and `source`. |
| `LinkDBError` | Anything else — wrong number of statements for `execute()`, a parameter mismatch, or a failure in the analyzer, planner, executor, or storage layers (these raise subclasses such as `AnalyzerError`). |

```python
try:
    conn.execute("SELEKT 1")
except linkdb.LinkQLSyntaxError as error:
    print(error.line, error.column)  # 1 1
    print(error)
```

```text
Unexpected keyword SELEKT at line 1, column 1
 --> Line 1, Column 1
  |
1 | SELEKT 1
  | ^^^^^^
```

`error.message` is the bare text; `str(error)` adds the position and the
offending source line, the same detail the [CLI](../cli/index.md) prints. The
full exception hierarchy is on the [Errors](../../reference/error_codes.md)
reference page.

## Limitations

- The API is synchronous only.
- Rows are dicts. There are no tuple rows, streaming results, or cursor
  context managers.
- There are no `commit()` or `rollback()` methods; transactions are driven
  with the `BEGIN`, `COMMIT`, and `ROLLBACK` statements. See
  [Connection → Transactions](connection.md#transactions).
- Two connections to the same on-disk database in one process are not
  coordinated, so write from only one at a time.
