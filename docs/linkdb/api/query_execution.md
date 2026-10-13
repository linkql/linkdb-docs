# Query Execution

Run [LinkQL](../../language/index.md) statements with `Connection.execute()`,
`Connection.executescript()`, or a [`Cursor`](#cursors).

## `execute()`

```python
conn.execute(sql, params=None) -> Result
```

Runs **exactly one** statement and returns its [`Result`](results.md). The
trailing `;` is optional. Passing zero or several statements raises
`LinkDBError`; use [`executescript()`](#executescript) for several.

```python
conn.execute("INSERT INTO users (username) VALUES ('john')")
result = conn.execute("SELECT username FROM users")
```

### Parameters

Pass values separately instead of building the SQL string yourself.

- `?` placeholders take a **sequence**, filled in order.
- `:name` placeholders take a **dict**.

```python
conn.execute("INSERT INTO users (username) VALUES (?)", ["john"])
conn.execute("INSERT INTO users (username) VALUES (:name)", {"name": "jane"})
conn.execute(
    "SELECT username FROM users WHERE user_id > :min AND active = :active",
    {"min": 0, "active": True},
)
```

Rules:

- Supported value types are `None` (becomes `NULL`), `bool`, `int`, `float`,
  `str`, and `uuid.UUID`. Anything else raises `LinkDBError`, as does a
  non-finite float (`nan`, `inf`). A `uuid.UUID` is bound as `CAST('...' AS UUID)`.
- Values are never pasted into the SQL text, so a string containing quotes or
  keywords is always treated as plain data.
- A placeholder stands for a value, not a name. You can't parameterize a
  table or column name.
- Placeholders inside string literals, quoted identifiers, and comments are
  left alone, as is the `::` operator.
- A `?` directly after an expression is the shorthand `CASE` operator, not a
  placeholder.
- These raise `LinkDBError`: placeholders with no parameters, a sequence
  whose length doesn't match the number of `?` placeholders, a dict used with
  `?` (or a sequence used with `:name`), and a `:name` missing from the dict.
  Unused keys in a dict are ignored.

## `executescript()`

```python
conn.executescript(sql) -> list[Result]
```

Runs several statements in order and returns one `Result` per statement. It
takes no parameters.

```python
results = conn.executescript("""
    CREATE TABLE tags (tag_id BIGINT PRIMARY KEY, label VARCHAR(20));
    INSERT INTO tags (tag_id, label) VALUES (1, 'a');
    SELECT label FROM tags;
""")
print(results[-1].rows)   # [{'label': 'a'}]
```

The statements are **not** wrapped in a transaction. If one fails, the
statements before it stay applied unless you opened a transaction yourself
with `BEGIN`. (The CLI behaves differently: it wraps a multi-statement batch
in an implicit transaction. See
[Commands](../cli/commands.md#implicit-transaction-wrapping-of-multi-statement-batches).)

## Cursors

`conn.cursor()` returns a `Cursor`, a [PEP 249](https://peps.python.org/pep-0249/)-style
wrapper that holds the rows of the last statement and hands them out
gradually. Rows are dicts, as everywhere else in the API.

```python
cur = conn.cursor()
cur.execute("SELECT username FROM users WHERE username = ?", ["john"])
print(cur.fetchone()["username"])   # john

cur.execute("SELECT username FROM users ORDER BY user_id")
for row in cur:
    print(row["username"])
```

| Member | Description |
|--------|-------------|
| `execute(sql, params=None)` | Run one statement, replacing any earlier rows. Returns `None`. |
| `fetchone()` | The next row, or `None` when there are no more. |
| `fetchmany(size=None)` | Up to `size` rows. Defaults to `arraysize`, which is `1`. |
| `fetchall()` | All remaining rows. |
| `description` | A tuple of 7-item tuples, one per column, with the column name first and `None` for the rest. `None` if the statement returned no columns. |
| `rowcount` | The statement's [`row_count`](results.md#fields), or `-1` if it has none. |
| `connection` | The `Connection` it came from. |
| `arraysize` | Default size for `fetchmany()`. |
| `close()` | Close the cursor. Further `execute()` calls raise `LinkDBConnectionError`. |

A cursor is also iterable, yielding the remaining rows. It runs one
statement at a time and has no `executemany()`.
