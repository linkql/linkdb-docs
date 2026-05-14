# Functions
Built-in functions available in LinkQL.

## Aggregate

`COUNT(expr)`
:   Counts the number of rows where `expr` is not `NULL` or `MISSING`.

`COUNT(*)`
:   Counts all rows in the result set regardless of `NULL` or `MISSING` values.

`SUM(expr)`
:   Returns the sum of all non-`NULL`, non-`MISSING` values of `expr`.

`AVG(expr)`
:   Returns the average of all non-`NULL`, non-`MISSING` values of `expr`.

`MIN(expr)`
:   Returns the minimum value of `expr`, excluding `NULL` and `MISSING` values.

`MAX(expr)`
:   Returns the maximum value of `expr`, excluding `NULL` and `MISSING` values.

`COUNT_ARRAY(expr)`
:   Counts the number of items within an array value. Unlike `COUNT`, which counts
    rows, `COUNT_ARRAY` looks inside the array itself. `COUNT` cannot be used on an
    array value and `COUNT_ARRAY` cannot be used to count rows.

```sql
SELECT COUNT_ARRAY(posts::likes) AS num_likes FROM posts;
```

## JSON

`HAS(json, path)`
:   Returns `true` if the specified key path exists in the JSON value, `false`
    otherwise. Dot notation in the path checks the full depth. See
    [JSON Querying](json.md) for full details.

`KEYS(json)`
:   Returns all top level keys of a JSON object as an `ARRAY(TEXT)`. See
    [JSON Querying](json.md) for full details.

`VALUES(json)`
:   Returns all top level values of a JSON object as an `ARRAY(JSON)`. See
    [JSON Querying](json.md) for full details.

`MERGE(json1, json2, direction?)`
:   Merges two JSON objects. If both share a key, `direction` determines which value
    is kept. Defaults to `RIGHT`. See [JSON Querying](json.md) for full details.

`CONTAINS(json, value)`
:   Returns `true` if a JSON array contains the specified value. See
    [JSON Querying](json.md) for full details.

`UNPACK(expr)`
:   Unpacks a JSON object into free fields. Each key becomes a free field with a
    resolved type. See [JSON Querying](json.md) for full details.

## String

`UPPER(str)`
:   Returns `str` converted to uppercase.

```sql
SELECT UPPER(users.username) AS username FROM users;
```

`LOWER(str)`
:   Returns `str` converted to lowercase.

```sql
SELECT LOWER(users.username) AS username FROM users;
```

`TRIM(str)`
:   Returns `str` with leading and trailing whitespace removed.

```sql
SELECT TRIM(users.username) AS username FROM users;
```

`LENGTH(str)`
:   Returns the number of characters in `str`.

```sql
SELECT LENGTH(users.username) AS username_length FROM users;
```

`SUBSTRING(str, start, length?)`
:   Returns a substring of `str` starting at position `start` (1-based). If `length` is provided, returns at most `length` characters; otherwise, returns the rest of the string.

```sql
SELECT SUBSTRING(users.username, 2, 3) AS username_part FROM users;
```

`REPLACE(str, from, to)`
:   Returns `str` with all occurrences of `from` replaced with `to`.

```sql
SELECT REPLACE(users.phone, '-', '') AS phone FROM users;
```

`CONCAT(str, ...)`
:   Concatenates two or more strings together.

```sql
SELECT CONCAT(users.first_name, ' ', users.last_name) AS full_name FROM users;
```

## Math

`ROUND(num, decimals?)`
:   Rounds `num` to the specified number of decimal places. Defaults to 0 decimal
    places if not specified.

```sql
SELECT ROUND(orders.total, 2) AS total FROM orders;
```

`FLOOR(num)`
:   Returns the largest integer less than or equal to `num`.

```sql
SELECT FLOOR(orders.total) AS total FROM orders;
```

`CEIL(num)`
:   Returns the smallest integer greater than or equal to `num`.

```sql
SELECT CEIL(orders.total) AS total FROM orders;
```

`ABS(num)`
:   Returns the absolute value of `num`.

```sql
SELECT ABS(orders.balance) AS balance FROM orders;
```

## Date

`NOW()`
:   Returns the current date and time as a `DATETIME`. `SYSDATE` is a keyword alias
    for `NOW()` and is converted to `NOW()` by the parser.

```sql
SELECT NOW() AS current_datetime;
SELECT SYSDATE AS current_datetime;
```

`TODAY()`
:   Returns the current date as a `DATE`.

```sql
SELECT TODAY() AS current_date;
```

`DATE_DIFF(date1, date2, unit)`
:   Returns the difference between two dates in the specified unit. Valid units are
    `'year'`, `'month'`, `'day'`, `'hour'`, `'minute'`, `'second'`.

```sql
SELECT DATE_DIFF(users.created_at, NOW(), 'day') AS days_since_created FROM users;
```

`DATE_ADD(date, amount, unit)`
:   Adds the specified amount to a date. Valid units are `'year'`, `'month'`, `'day'`,
    `'hour'`, `'minute'`, `'second'`.

```sql
SELECT DATE_ADD(users.created_at, 30, 'day') AS expiry_date FROM users;
```

`DAY(date)`
:   Returns the day of the month from a `DATE` or `DATETIME` value as an `INT`.

```sql
SELECT DAY(users.created_at) AS day FROM users;
```

`MONTH(date)`
:   Returns the month from a `DATE` or `DATETIME` value as an `INT`.

```sql
SELECT MONTH(users.created_at) AS month FROM users;
```

`YEAR(date)`
:   Returns the year from a `DATE` or `DATETIME` value as an `INT`.

```sql
SELECT YEAR(users.created_at) AS year FROM users;
```

## Type

`CAST(value AS type)`
:   Converts `value` to the specified [data type](data_types.md). Returns an error
    if the value cannot be cast to the target type.

```sql
SELECT CAST(users.age AS FLOAT) AS age FROM users;

-- Common use in ALTER
ALTER TABLE users
MODIFY COLUMN phone INT USING CAST(REPLACE(phone, '-', '') AS INT);
```

`COALESCE(val1, val2, ...)`
:   Returns the first non-`NULL` value in the list. Useful for providing fallback
    values. Note that `MISSING` is not the same as `NULL` — a `MISSING` value is
    excluded before `COALESCE` evaluates.

```sql
SELECT COALESCE(users.phone, users.mobile, 'n/a') AS contact FROM users;
```

`NULLIF(val1, val2)`
:   Returns `NULL` if `val1` equals `val2`, otherwise returns `val1`. Useful for
    avoiding division by zero or replacing sentinel values with `NULL`.

```sql
SELECT NULLIF(orders.total, 0) AS total FROM orders;
```

`GEN_UUID()`
:   Generates a new UUID value. Used automatically as the default for `_id` fields.

```sql
SELECT GEN_UUID() AS new_id;

-- Used as a default
CREATE TABLE users (
    _id UUID DEFAULT GEN_UUID()
);
```

## See Also
[Data Types](data_types.md), [JSON Querying](json.md), [SELECT Items](dml/select/select-items.md)

---
[← Back to Language](index.md)
