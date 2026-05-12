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

`ABS` `ACTION` `ADD` `AND` `AS` `ASC` `AUTO` `AUTOINCREMENT` `AUTONOW` `AVG`

<a id="b"></a>
**B**

`BETWEEN` `BIGINT` `BOOLEAN` `BY`

<a id="c"></a>
**C**

`CASCADE` `CASE` `CAST` `CEIL` `COALESCE` `COLLECTION` `COLUMN` `CONCAT` `CONFIRM`
`CONFLICT` `CONSTRAINT` `CONTAINS` `COUNT` `COUNT_ARRAY` `CREATE` `CROSS`

<a id="d"></a>
**D**

`DATABASE` `DATE` `DATE_ADD` `DATE_DIFF` `DAY` `DECIMAL` `DEFAULT` `DELETE` `DESC`
`DISTINCT` `DO` `DROP`

<a id="e"></a>
**E**

`ELSE` `END` `EXCLUDED` `EXISTS`

<a id="f"></a>
**F**

`FALSE` `FIELD` `FLOAT` `FLOOR` `FOR` `FOREIGN` `FREE` `FROM` `FULL`

<a id="g"></a>
**G**

`GEN_UUID` `GROUP`

<a id="h"></a>
**H**

`HAS` `HAVING`

<a id="i"></a>
**I**

`IDENTITY` `IF` `ILIKE` `IN` `INFINITY` `INNER` `INSERT` `INT` `INTO` `IS`

<a id="j"></a>
**J**

`JOIN` `JSON` `JSONB`

<a id="k"></a>
**K**

`KEEP` `KEY` `KEYS`

<a id="l"></a>
**L**

`LEFT` `LENGTH` `LET` `LIKE` `LIMIT` `LOWER`

<a id="m"></a>
**M**

`MAX` `MERGE` `MIN` `MISSING` `MODIFY` `MONTH`

<a id="n"></a>
**N**

`NO` `NOT` `NOTHING` `NULL` `NULLIF`

<a id="o"></a>
**O**

`OFFSET` `ON` `ONLY` `OR` `ORDER` `OUTER`

<a id="p"></a>
**P**

`PRIMARY`

<a id="r"></a>
**R**

`READ` `REFERENCES` `RENAME` `REPLACE` `RESTART` `RESTRICT` `RIGHT` `ROLLBACK` `ROUND`

<a id="s"></a>
**S**

`SAVEPOINT` `SECONDARY` `SELECT` `SET` `SUM` `SYSDATE`

<a id="t"></a>
**T**

`TABLE` `TEXT` `THEN` `TIME` `DATETIME` `TO` `TODAY` `TRANSACTION` `TRIM` `TRUE` `TRUNCATE`

<a id="u"></a>
**U**

`UNIQUE` `UNPACK` `UPDATE` `UPPER` `USING` `UUID`

<a id="v"></a>
**V**

`VALUES` `VARCHAR`

<a id="w"></a>
**W**

`WHEN` `WHERE` `WITH` `WRITE`

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
| `CASCADE` | Propagates a drop or delete to dependent objects |
| `CHECK` | Defines a condition constraint on a column or field |
| `COLLECTION` | Identifies a document collection |
| `COLUMN` | Identifies a table column in `ALTER` statements |
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
| `EXISTS` | Used in `IF EXISTS` and `IF NOT EXISTS` |
| `KEEP` | Preserves data when dropping a defined field from a collection |
| `KEY` | Used in `PRIMARY KEY`, `SECONDARY KEY`, `FOREIGN KEY` |
| `MODIFY` | Modifies a column, field, or constraint in `ALTER` statements |
| `NO` | Used in `NO ACTION` reference action |
| `NOT` | Used in `NOT NULL` constraint and expressions |
| `NULL` | Absence of a value. Used in constraints and expressions |
| `ON` | Used in `ON DELETE`, `ON UPDATE`, and join conditions |
| `PRIMARY` | Marks a column or field as the primary key |
| `REFERENCES` | Defines a foreign key reference |
| `RENAME` | Renames a table, collection, column, or field |
| `RESTART` | Used in `RESTART IDENTITY` |
| `RESTRICT` | Prevents a drop or delete if dependencies exist. Default behavior |
| `SECONDARY` | Defines a secondary key that forms a composite with `_id` |
| `SET` | Used in `SET NULL`, `SET DEFAULT`, and `UPDATE SET` |
| `TABLE` | Identifies a relational table |
| `TO` | Used in `RENAME TO` |
| `TRUNCATE` | Removes all data from a table, collection, or database |
| `UNIQUE` | Enforces uniqueness on a column or field |
| `UPDATE` | Used in `ON UPDATE` reference action |
| `USING` | Provides a cast expression in `MODIFY COLUMN` and `MODIFY FIELD` |

### DML

| Keyword | Description |
|---------|-------------|
| `AS` | Assigns an alias to a select item, table, or subquery |
| `ASC` | Ascending sort order in `ORDER BY` |
| `BY` | Used in `GROUP BY` and `ORDER BY` |
| `CONFLICT` | Used in `ON CONFLICT` for upsert behavior |
| `DELETE` | Removes rows or documents from a table or collection |
| `DESC` | Descending sort order in `ORDER BY` |
| `DISTINCT` | Eliminates duplicate rows from a result set |
| `DO` | Used in `ON CONFLICT DO`, `FOR DO`, and `TRANSACTION DO` |
| `END` | Closes a `FOR` loop or `TRANSACTION` block |
| `EXCLUDED` | References values from the attempted insert in `ON CONFLICT` |
| `FOR` | Iterates over a result set applying DML statements to each row |
| `FROM` | Specifies the source tables and collections for a query |
| `GROUP` | Used in `GROUP BY` |
| `HAVING` | Filters groups after aggregation |
| `IN` | Used in `FOR ... IN` and `IN (...)` expressions |
| `INSERT` | Adds rows or documents to a table or collection |
| `INTO` | Used in `INSERT INTO` |
| `LET` | Defines named subqueries for use within a statement |
| `LIMIT` | Restricts the number of rows returned |
| `NOTHING` | Used in `ON CONFLICT DO NOTHING` |
| `OFFSET` | Skips a number of rows before returning results |
| `ORDER` | Used in `ORDER BY` |
| `SELECT` | Retrieves data from one or more tables or collections |
| `UPDATE` | Modifies rows or documents in a table or collection |
| `VALUES` | Provides values for an `INSERT` statement |
| `WHERE` | Filters rows based on a condition |
| `WITH` | Used in `WITH FREE` for free field declarations |

### Joins

| Keyword | Description |
|---------|-------------|
| `CROSS` | Returns the cartesian product of two sources |
| `FULL` | Used in `FULL OUTER` join |
| `INNER` | Returns rows with matching values in both sources |
| `JOIN` | Joins two sources together |
| `LEFT` | Returns all rows from the left source and matching rows from the right |
| `OUTER` | Shorthand for `FULL OUTER` join |
| `RIGHT` | Returns all rows from the right source and matching rows from the left |
| `USING` | Specifies shared columns for a join condition |

### Transactions

| Keyword | Description |
|---------|-------------|
| `ONLY` | Used in `READ ONLY` transaction mode |
| `READ` | Used in `READ ONLY` and `READ WRITE` transaction modes |
| `ROLLBACK` | Rolls back a transaction or to a savepoint |
| `SAVEPOINT` | Creates a named checkpoint within a transaction |
| `TRANSACTION` | Defines a transaction block |
| `WRITE` | Used in `READ WRITE` transaction mode |

### Expressions

| Keyword | Description |
|---------|-------------|
| `AND` | Logical AND operator |
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
| `THEN` | Used in `CASE WHEN ... THEN` |
| `TRUE` | Boolean true literal |
| `WHEN` | Used in `CASE WHEN` |

### Data Types

| Keyword | Description |
|---------|-------------|
| `ARRAY` | Ordered list of values of a single type |
| `BIGINT` | 64-bit integer |
| `BOOLEAN` | True or false value |
| `DATE` | Date value in `YYYY-MM-DD` format |
| `DATETIME` | Combined date and time value |
| `DECIMAL` | Fixed precision decimal number |
| `FLOAT` | Floating point number |
| `INT` | 32-bit integer |
| `JSON` | Freeform JSON blob |
| `JSONB` | Decomposed binary JSON with indexable keys |
| `TEXT` | String of any length |
| `TIME` | Time value in `hh:mm:ss` format |
| `UUID` | Universally unique identifier |
| `VARCHAR` | Fixed length string |

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
