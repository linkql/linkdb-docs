# Results

`Connection.execute()` returns a `Result` for the statement it ran;
`executescript()` returns a list of them. `Result` is a plain dataclass.

## Fields

Every field defaults to `None`. A field that doesn't apply to the statement
stays `None`.

| Field | Type | Meaning |
|-------|------|---------|
| `columns` | `list[str] \| None` | Column names, in order. Set for `SELECT`, and for `RETURNING` (where it names the returned columns). |
| `rows` | `list[dict] \| None` | The result rows, each a dict of column name to value. Set for `SELECT`. |
| `row_count` | `int \| None` | Rows read or affected. Set for `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. |
| `returning` | `list[dict] \| None` | Rows from a `RETURNING` clause. |
| `message` | `str \| None` | A status line, such as `CREATE TABLE users`, `BEGIN`, or `ROLLBACK`. |

## By statement

```python
conn.execute("CREATE TABLE users (id BIGINT PRIMARY KEY AUTOINCREMENT(1, 1), name VARCHAR(50))")
# Result(message='CREATE TABLE users')

conn.execute("INSERT INTO users (name) VALUES (?)", ["john"])
# Result(row_count=1)

conn.execute("INSERT INTO users (name) VALUES (?) RETURNING id", ["jane"])
# Result(columns=['id'], row_count=1, returning=[{'id': 2}])

conn.execute("SELECT id, name FROM users ORDER BY id")
# Result(columns=['id', 'name'],
#        rows=[{'id': 1, 'name': 'john'}, {'id': 2, 'name': 'jane'}],
#        row_count=2)

conn.execute("BEGIN")
# Result(message='BEGIN')
```

The repr omits fields that are `None` here for readability; the real repr
shows them all.

## Reading values

Rows are ordinary dicts, so index them by column name:

```python
result = conn.execute("SELECT id, name FROM users ORDER BY id")
for row in result.rows:
    print(row["id"], row["name"])

names = [row["name"] for row in result.rows]
```

Check `result.rows is not None` if you don't know whether a statement
produces rows. A `SELECT` that matches nothing returns `rows == []`, not
`None`.
