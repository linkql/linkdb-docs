# Keywords
Reserved words in LinkQL that cannot be used as identifiers.

All keywords in LinkQL are fully reserved. They cannot be used as table names,
collection names, column names, field names, or any other identifier. If you need
to use a reserved word as an identifier, wrap it in double quotes:

```sql
-- Using a reserved word as an identifier
CREATE TABLE "select" (id INT);
SELECT "from"."select" FROM "from";
```

[**A**](#a) &nbsp; [**B**](#b) &nbsp; [**C**](#c) &nbsp; [**D**](#d) &nbsp; [**E**](#e) &nbsp; [**F**](#f) &nbsp; [**G**](#g) &nbsp; [**H**](#h) &nbsp; [**I**](#i) &nbsp; [**J**](#j) &nbsp; [**K**](#k) &nbsp; [**L**](#l) &nbsp; [**M**](#m) &nbsp; [**N**](#n) &nbsp; [**O**](#o) &nbsp; [**P**](#p) &nbsp; [**R**](#r) &nbsp; [**S**](#s) &nbsp; [**T**](#t) &nbsp; [**U**](#u) &nbsp; [**V**](#v) &nbsp; [**W**](#w) &nbsp; [**Y**](#y)

<a id="a"></a>
**A**

`ABS` `ACTION` `ADD` `ALL` `AND` `ANY` `ARRAY` `AS` `ASC` `AUTO` `AUTOINCREMENT` `AUTONOW` `AVG`

<a id="b"></a>
**B**

`BEGIN` `BETWEEN` `BIGINT` `BLOB` `BOOLEAN` `BREADTH` `BY` `BYTEA`

<a id="c"></a>
**C**

`CASCADE` `CASE` `CAST` `CEIL` `COALESCE` `COLLECTION` `COLUMN` `COMMIT` `CONCAT` `CONFIRM`
`CONFLICT` `CONSTRAINT` `CONTAINS` `COUNT` `COUNT_ARRAY` `CREATE` `CROSS` `CUBE` `CURRENT` `CYCLE`

<a id="d"></a>
**D**

`DATABASE` `DATE` `DATE_ADD` `DATE_DIFF` `DAY` `DECIMAL` `DEFAULT` `DELETE` `DEPTH` `DESC`
`DISTINCT` `DO` `DOUBLE` `DROP`

<a id="e"></a>
**E**

`ELSE` `END` `ESCAPE` `EXCEPT` `EXCLUDE` `EXCLUDED` `EXISTS`

<a id="f"></a>
**F**

`FALSE` `FETCH` `FIELD` `FILTER` `FIRST` `FLOAT` `FLOOR` `FOLLOWING` `FOR` `FOREIGN` `FREE` `FROM` `FULL`

<a id="g"></a>
**G**

`GEN_UUID` `GROUP` `GROUPING` `GROUPS`

<a id="h"></a>
**H**

`HAS` `HAVING`

<a id="i"></a>
**I**

`IDENTITY` `IF` `ILIKE` `IN` `INFINITY` `INNER` `INSERT` `INT` `INTERSECT` `INTERVAL` `INTO` `IS` `ITERATIONS`

<a id="j"></a>
**J**

`JOIN` `JSON` `JSONB`

<a id="k"></a>
**K**

`KEEP` `KEY` `KEYS`

<a id="l"></a>
**L**

`LAST` `LATERAL` `LEFT` `LENGTH` `LET` `LIKE` `LIMIT` `LOCKED` `LOWER`

<a id="m"></a>
**M**

`MATCHED` `MAX` `MERGE` `MIN` `MISSING` `MODIFY` `MONTH`

<a id="n"></a>
**N**

`NATURAL` `NO` `NOT` `NOTHING` `NOWAIT` `NULL` `NULLIF` `NULLS` `NUMERIC`

<a id="o"></a>
**O**

`OFFSET` `ON` `ONLY` `OR` `ORDER` `ORDINALITY` `OTHERS` `OUTER` `OVER`

<a id="p"></a>
**P**

`PARTITION` `PRECEDING` `PRECISION` `PRIMARY`

<a id="r"></a>
**R**

`RANGE` `READ` `REAL` `RECURSIVE` `REFERENCES` `RELEASE` `RENAME` `REPLACE` `RESTART` `RESTRICT`
`RETURNING` `RIGHT` `ROLLBACK` `ROLLUP` `ROUND` `ROWS`

<a id="s"></a>
**S**

`SAVEPOINT` `SEARCH` `SECONDARY` `SELECT` `SET` `SHARE` `SKIP` `SMALLINT` `SOME` `SUM` `SYSDATE`

<a id="t"></a>
**T**

`TABLE` `TEXT` `THEN` `TIES` `TIME` `TIMESTAMP` `TINYINT` `TO` `TODAY` `TRANSACTION` `TRIM` `TRUE` `TRUNCATE`

<a id="u"></a>
**U**

`UNBOUNDED` `UNION` `UNIQUE` `UNNEST` `UNPACK` `UPDATE` `UPPER` `USING` `UUID`

<a id="v"></a>
**V**

`VALUES` `VARBINARY` `VARCHAR`

<a id="w"></a>
**W**

`WHEN` `WHERE` `WINDOW` `WITH` `WRITE`

<a id="y"></a>
**Y**

`YEAR`

---

## Keywords by Category

### DDL

| Keyword | Description |
|---------|-------------|
| `ADD` | Adds a column, field, or constraint in `ALTER` statements |
| `ACTION` | Used in `NO ACTION` reference action |
| `ALTER` | Modifies an existing table, collection, or database |
| `AUTO` | Automatically sets a datetime field on insert and update |
| `AUTOINCREMENT` | Automatically increments a field on insert |
| `AUTONOW` | Automatically sets a datetime field on insert only |
| `BLOB` | Binary large object data type |
| `BYTEA` | Binary data type (alias for BLOB) |
| `CASCADE` | Propagates a drop or delete to dependent objects |
| `CHECK` | Defines a condition constraint on a column or field |
| `COLLECTION` | Identifies a document collection |
| `COLUMN` | Identifies a table column in `ALTER` statements |
| `COMMIT` | Commits an explicit transaction |
| `CONFIRM` | Required confirmation string for destructive operations |
| `CONSTRAINT` | Assigns a name to a constraint |
| `CREATE` | Creates a new database, table, or collection |
| `DATABASE` | Identifies a database object |
| `DEFAULT` | Defines a default value for a column or field |
| `DELETE` | Used in `ON DELETE` reference action |
| `DROP` | Removes a database, table, collection, column, field, or constraint |
| `FIELD` | Identifies a collection field in `ALTER` statements |
| `FOREIGN` | Used in `FOREIGN KEY` constraint |
| `FREE` | Declares free fields in `ALTER COLLECTION` and `INSERT` |
| `IDENTITY` | Used in `RESTART IDENTITY` to reset autoincrement counters |
| `IF` | Used in `IF EXISTS` and `IF NOT EXISTS` |
| `INFINITY` | Used in `CASCADE INFINITY` to drop all dependencies |
| `INTERVAL` | Time interval data type and typed literal |
| `NUMERIC` | Arbitrary precision decimal data type |
| `EXISTS` | Used in `IF EXISTS` and `IF NOT EXISTS` |
| `KEEP` | Preserves data when dropping a defined field from a collection |
| `KEY` | Used in `PRIMARY KEY`, `SECONDARY KEY`, `FOREIGN KEY` |
| `MODIFY` | Modifies a column, field, or constraint in `ALTER` statements |
| `NO` | Used in `NO ACTION` reference action |
| `NOT` | Used in `NOT NULL` constraint and expressions |
| `NULL` | Absence of a value. Used in constraints and expressions |
| `ON` | Used in `ON DELETE`, `ON UPDATE`, and join conditions |
| `PRIMARY` | Marks a column or field as the primary key |
| `REAL` | Single precision floating point data type |
| `RECURSIVE` | Declares a recursive CTE in `LET` and `WITH` clauses |
| `REFERENCES` | Defines a foreign key reference |
| `RELEASE` | Releases a savepoint |
| `RENAME` | Renames a table, collection, column, or field |
| `RESTART` | Used in `RESTART IDENTITY` |
| `RESTRICT` | Prevents a drop or delete if dependencies exist. Default behavior |
| `RETURNING` | Returns values from an `INSERT`, `UPDATE`, `DELETE`, or `MERGE` |
| `SECONDARY` | Defines a secondary key that forms a composite with `_id` |
| `SET` | Used in `SET NULL`, `SET DEFAULT`, and `UPDATE SET` |
| `SMALLINT` | 16-bit integer data type |
| `TABLE` | Identifies a relational table |
| `TINYINT` | 8-bit integer data type |
| `TO` | Used in `RENAME TO` |
| `TRUNCATE` | Removes all data from a table, collection, or database |
| `UNIQUE` | Enforces uniqueness on a column or field |
| `UPDATE` | Used in `ON UPDATE` reference action |
| `USING` | Provides a cast expression in `MODIFY COLUMN` and `MODIFY FIELD` |
| `VARBINARY` | Variable-length binary data type |

### DML

| Keyword | Description |
|---------|-------------|
| `AS` | Assigns an alias to a select item, table, or subquery |
| `ASC` | Ascending sort order in `ORDER BY` |
| `BY` | Used in `GROUP BY` and `ORDER BY` |
| `CONFLICT` | Used in `ON CONFLICT` for upsert behavior |
| `DEFAULT` | Default values in `INSERT` and `UPDATE` |
| `DELETE` | Removes rows or documents from a table or collection |
| `DESC` | Descending sort order in `ORDER BY` |
| `DISTINCT` | Eliminates duplicate rows from a result set |
| `DO` | Used in `ON CONFLICT DO`, `FOR DO`, and `TRANSACTION DO` |
| `END` | Closes a `FOR` loop or `TRANSACTION` block |
| `ESCAPE` | Escapes special characters in `LIKE` and `ILIKE` |
| `EXCLUDED` | References values from the attempted insert in `ON CONFLICT` |
| `FETCH` | Used in `FETCH FIRST` for result set pagination |
| `FILTER` | Filters aggregate function inputs with a `WHERE` condition |
| `FIRST` | Used in `FETCH FIRST` and `NULLS FIRST` |
| `FOR` | Iterates over a result set applying DML statements to each row |
| `FROM` | Specifies the source tables and collections for a query |
| `GROUP` | Used in `GROUP BY` |
| `HAVING` | Filters groups after aggregation |
| `IN` | Used in `FOR ... IN` and `IN (...)` expressions |
| `INSERT` | Adds rows or documents to a table or collection |
| `INTO` | Used in `INSERT INTO` |
| `LAST` | Used in `NULLS LAST` ordering |
| `LET` | Defines named subqueries for use within a statement |
| `LIMIT` | Restricts the number of rows returned |
| `MERGE` | Merges rows into a table or collection |
| `NOTHING` | Used in `ON CONFLICT DO NOTHING` |
| `NULLS` | Controls `NULL` ordering position in `ORDER BY` |
| `OFFSET` | Skips a number of rows before returning results |
| `ONLY` | Used in `FETCH ... ONLY` and `READ ONLY` |
| `ORDER` | Used in `ORDER BY` |
| `RETURNING` | Returns values from an `INSERT`, `UPDATE`, `DELETE`, or `MERGE` |
| `SELECT` | Retrieves data from one or more tables or collections |
| `UPDATE` | Modifies rows or documents in a table or collection |
| `VALUES` | Provides values for an `INSERT` statement |
| `WHERE` | Filters rows based on a condition |
| `WITH` | Used in `WITH FREE` and CTE (`WITH` / `WITH RECURSIVE`) clauses |

### Set Operations

| Keyword | Description |
|---------|-------------|
| `EXCEPT` | Returns rows in the first query but not the second |
| `INTERSECT` | Returns rows common to both queries |
| `UNION` | Combines result sets from two queries |

### Joins

| Keyword | Description |
|---------|-------------|
| `CROSS` | Returns the cartesian product of two sources |
| `FULL` | Used in `FULL OUTER` join |
| `INNER` | Returns rows with matching values in both sources |
| `JOIN` | Joins two sources together |
| `LEFT` | Returns all rows from the left source and matching rows from the right |
| `NATURAL` | Declares a natural join |
| `OUTER` | Shorthand for `FULL OUTER` join |
| `RIGHT` | Returns all rows from the right source and matching rows from the left |
| `USING` | Specifies shared columns for a join condition |

### Advanced Joins

| Keyword | Description |
|---------|-------------|
| `LATERAL` | Allows subqueries in the `FROM` clause to reference previous items |
| `ORDINALITY` | Used in `WITH ORDINALITY` to add a row number column |
| `UNNEST` | Expands an array into rows in the `FROM` clause |

### Transactions

| Keyword | Description |
|---------|-------------|
| `BEGIN` | Starts an explicit transaction |
| `COMMIT` | Commits the current transaction |
| `ONLY` | Used in `READ ONLY` transaction mode |
| `READ` | Used in `READ ONLY` and `READ WRITE` transaction modes |
| `RELEASE` | Releases a savepoint |
| `ROLLBACK` | Rolls back a transaction or to a savepoint |
| `SAVEPOINT` | Creates a named checkpoint within a transaction |
| `TRANSACTION` | Defines a transaction block |
| `WRITE` | Used in `READ WRITE` transaction mode |

### Recursive CTE Control

| Keyword | Description |
|---------|-------------|
| `BREADTH` | Used in `SEARCH BREADTH FIRST` for recursive CTE ordering |
| `CYCLE` | Detects cycles in recursive CTEs |
| `DEPTH` | Used in `SEARCH DEPTH FIRST` for recursive CTE ordering |
| `ITERATIONS` | Limits the number of iterations in a recursive CTE |
| `MAX` | Used in `MAX ITERATIONS` for recursive CTEs |
| `SEARCH` | Controls recursive CTE result ordering |

### Expressions

| Keyword | Description |
|---------|-------------|
| `AND` | Logical AND operator |
| `ANY` | Quantified comparison operator (synonym for `SOME`) |
| `BETWEEN` | Tests whether a value falls within an inclusive range |
| `CASE` | Conditional expression |
| `ELSE` | Fallback value in a `CASE` expression |
| `EXISTS` | Tests whether a subquery returns any rows |
| `FALSE` | Boolean false literal |
| `ILIKE` | Case insensitive pattern matching |
| `IS` | Used in `IS NULL`, `IS MISSING`, `IS NOT NULL` |
| `LIKE` | Case sensitive pattern matching |
| `MISSING` | Indicates a field that was never present on a document |
| `NOT` | Logical NOT operator |
| `NULL` | Absence of a value |
| `OR` | Logical OR operator |
| `SOME` | Quantified comparison operator (synonym for `ANY`) |
| `THEN` | Used in `CASE WHEN ... THEN` |
| `TRUE` | Boolean true literal |
| `WHEN` | Used in `CASE WHEN` |

### Quantified Comparison

| Keyword | Description |
|---------|-------------|
| `ALL` | Compares against all rows returned by a subquery |
| `ANY` | Compares against any row returned by a subquery (synonym for `SOME`) |
| `SOME` | Compares against any row returned by a subquery (synonym for `ANY`) |

### Window Functions

| Keyword | Description |
|---------|-------------|
| `CURRENT` | Used in `CURRENT ROW` frame bound and `EXCLUDE CURRENT ROW` |
| `EXCLUDE` | Excludes rows from a window frame |
| `FOLLOWING` | Frame bound that includes rows after the current row |
| `GROUPS` | Window frame unit (number of peer groups) |
| `NO` | Used in `NO OTHERS` and `EXCLUDE NO OTHERS` |
| `OTHERS` | Used in `EXCLUDE NO OTHERS` |
| `OVER` | Introduces a window specification |
| `PARTITION` | Divides the result set into partitions in a window |
| `PRECEDING` | Frame bound that includes rows before the current row |
| `RANGE` | Window frame unit based on sort key values |
| `ROWS` | Window frame unit based on physical row count |
| `TIES` | Used in `EXCLUDE TIES` and `WITH TIES` |
| `UNBOUNDED` | Used in `UNBOUNDED PRECEDING` and `UNBOUNDED FOLLOWING` |
| `WINDOW` | Defines named window specifications in a `SELECT` |

### Locking

| Keyword | Description |
|---------|-------------|
| `FOR` | Introduces a locking clause |
| `KEY` | Used in `KEY SHARE` and `NO KEY UPDATE` |
| `LOCKED` | Used in `SKIP LOCKED` |
| `NOWAIT` | Fails immediately if a lock cannot be acquired |
| `SHARE` | Used in `SHARE` and `KEY SHARE` lock modes |
| `SKIP` | Used in `SKIP LOCKED` |
| `UPDATE` | Used in `FOR UPDATE` and `NO KEY UPDATE` lock modes |

### Ordering

| Keyword | Description |
|---------|-------------|
| `FIRST` | Sorts `NULL` values first in `ORDER BY` |
| `LAST` | Sorts `NULL` values last in `ORDER BY` |
| `NULLS` | Controls `NULL` ordering position in `ORDER BY` |

### Fetch / Limit

| Keyword | Description |
|---------|-------------|
| `FETCH` | Used in `FETCH FIRST` for result set pagination |
| `FIRST` | Used in `FETCH FIRST` |
| `LIMIT` | Restricts the number of rows returned |
| `OFFSET` | Skips a number of rows before returning results |
| `ONLY` | Used in `FETCH ... ONLY` |
| `TIES` | Used in `WITH TIES` to include tied rows |

### Pattern Matching

| Keyword | Description |
|---------|-------------|
| `ESCAPE` | Escapes special characters in `LIKE` and `ILIKE` |

### Data Types

| Keyword | Description |
|---------|-------------|
| `ARRAY` | Ordered list of values of a single type |
| `BIGINT` | 64-bit integer |
| `BLOB` | Binary large object |
| `BOOLEAN` | True or false value |
| `BYTEA` | Binary data (alias for BLOB) |
| `DATE` | Date value in `YYYY-MM-DD` format |
| `DATETIME` | Combined date and time value |
| `DECIMAL` | Fixed precision decimal number |
| `DOUBLE` | Part of `DOUBLE PRECISION` data type |
| `FLOAT` | Floating point number |
| `INT` | 32-bit integer |
| `INTERVAL` | Time interval data type and typed literal |
| `JSON` | Freeform JSON blob |
| `JSONB` | Decomposed binary JSON with indexable keys |
| `NUMERIC` | Arbitrary precision decimal number |
| `PRECISION` | Part of `DOUBLE PRECISION` data type |
| `REAL` | Single precision floating point |
| `SMALLINT` | 16-bit integer |
| `TEXT` | String of any length |
| `TIME` | Time value in `hh:mm:ss` format |
| `TIMESTAMP` | Typed literal for timestamp values |
| `TINYINT` | 8-bit integer |
| `UUID` | Universally unique identifier |
| `VARBINARY` | Variable-length binary data |
| `VARCHAR` | Fixed length string |

### Typed Literals

| Keyword | Description |
|---------|-------------|
| `DATE` | Typed literal prefix for date values |
| `INTERVAL` | Typed literal prefix for interval values |
| `TIME` | Typed literal prefix for time values |
| `TIMESTAMP` | Typed literal prefix for timestamp values |

### Functions

| Keyword | Description |
|---------|-------------|
| `ABS` | Returns the absolute value of a number |
| `AVG` | Returns the average of a set of values |
| `CAST` | Converts a value to a specified data type |
| `CEIL` | Returns the smallest integer greater than or equal to a number |
| `COALESCE` | Returns the first non-null value in a list |
| `CONCAT` | Concatenates two or more strings |
| `CONTAINS` | Returns true if a JSON array contains a value |
| `COUNT` | Counts rows in a result set |
| `COUNT_ARRAY` | Counts items within an array value |
| `DATE_ADD` | Adds an amount to a date |
| `DATE_DIFF` | Returns the difference between two dates |
| `DAY` | Returns the day of the month from a date value |
| `FLOOR` | Returns the largest integer less than or equal to a number |
| `GEN_UUID` | Generates a new UUID value |
| `HAS` | Returns true if a key path exists in a JSON value |
| `KEYS` | Returns all top level keys of a JSON object |
| `LENGTH` | Returns the number of characters in a string |
| `LIKE` | Case sensitive pattern matching |
| `LOWER` | Converts a string to lowercase |
| `MAX` | Returns the maximum value in a set |
| `MERGE` | Merges two JSON objects |
| `MIN` | Returns the minimum value in a set |
| `MONTH` | Returns the month from a date value |
| `NOW` | Returns the current date and time |
| `NULLIF` | Returns null if two values are equal |
| `REPLACE` | Replaces occurrences of a substring |
| `ROUND` | Rounds a number to a specified number of decimal places |
| `SUM` | Returns the sum of a set of values |
| `SYSDATE` | Keyword alias for `NOW()`, converted by the parser |
| `TODAY` | Returns the current date |
| `TRIM` | Removes leading and trailing whitespace from a string |
| `UNPACK` | Unpacks a JSON object into free fields |
| `UPPER` | Converts a string to uppercase |
| `VALUES` | Returns all top level values of a JSON object |
| `YEAR` | Returns the year from a date value |

---

## See Also
[Grammar](grammar.md), [Error Codes](error_codes.md), [Data Types](../data_types.md)
