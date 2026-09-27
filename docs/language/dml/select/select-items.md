# Select Items
Defines the columns, fields, and expressions to include in the result set.

```grammar title="Grammar"
select_item     ::= '*'
                  | table_star
                  | collection_star
                  | expr ( AS identifier )?
                  | function_call AS identifier

table_star      ::= identifier '.' '*'

collection_star ::= identifier '::' '*'
```

## Description
Select items define what data is returned by a `SELECT` statement. Each item in the
select list is either a column reference, a field reference, an expression, or a
function call. Multiple items are comma separated.

Tables are referenced with dot notation — `table.column`. Collections are referenced
with double colon notation — `collection::field`.

## Select Item Types

### Wildcard
Selects all columns and fields from all tables and collections in the query. Cannot
be combined with `DISTINCT`.

When the query uses a `NATURAL` join, a `USING` join, or a bare shorthand join, each
merged join column is returned once, ahead of the other columns. See
[Merged Columns](joins.md#merged-columns). A subquery, CTE, or view that selects `*`
exposes all of its columns to the query that uses it.

```sql
SELECT * FROM users;
```

### Table Wildcard
Selects all columns from a specific table. Unlike `*`, it never merges columns: every
column of the table is returned, including the ones a `NATURAL` or `USING` join merges.

```sql
SELECT users.* FROM users;
```

### Collection Wildcard
Selects all defined and free fields from a specific collection.

```sql
SELECT posts::* FROM posts;
```

### Column Reference
Selects a specific column from a table using dot notation.

```sql
SELECT users.username, users.first_name FROM users;
```

### Field Reference
Selects a specific field from a collection using double colon notation.

```sql
SELECT posts::content, posts::created_at FROM posts;
```

### Expression
Any valid expression can be used as a select item. If the result of the expression
is not a direct column or field reference, an alias is recommended for clarity.

```sql
SELECT users.first_name, users.last_name FROM users;
```

### Function Call
A function call used as a select item must always be aliased with `AS`. See
[Functions](../../functions.md) for available functions.

```sql
SELECT COUNT_ARRAY(comments::likes) AS num_likes FROM comments;
```

## Aliases
An alias renames the select item in the result set. The `AS` keyword is optional
for column references and expressions but required for function calls.

```sql
-- AS is optional for column references
SELECT users.username AS name FROM users;
SELECT users.username name FROM users;

-- AS is required for function calls
SELECT COUNT_ARRAY(comments::likes) AS num_likes FROM comments;
```

## DISTINCT
`DISTINCT` eliminates duplicate rows from the result set. It applies to the entire
result set, not individual columns. `DISTINCT` cannot be combined with `*`.

```sql
SELECT DISTINCT users.username FROM users;
```

## Notes
- `DISTINCT *` is not valid. Use `DISTINCT` with specific columns or fields only.
- Function calls must always be aliased with `AS`.
- Free fields on a collection that do not exist on a document return `NULL`.
- `collection::*` returns all defined and free fields found across documents. Documents
  that do not have a particular free field return `NULL` for that field.

## Examples

```sql title="Select specific columns from a table"
SELECT users.username, users.first_name, users.last_name
FROM users;
```

```sql title="Select all columns from a table"
SELECT users.* FROM users;
```

```sql title="Select fields from a collection"
SELECT posts::content, posts::created_at FROM posts;
```

```sql title="Mix table columns and collection fields"
SELECT
    users.username,
    posts::content,
    posts::created_at
FROM posts
LEFT JOIN users ON users.user_id = posts.user_id;
```

```sql title="Function call with alias"
SELECT
    users.username,
    COUNT_ARRAY(comments::likes) AS num_likes
FROM comments
LEFT JOIN users ON users.user_id = comments.user_id;
```

```sql title="DISTINCT on a column"
SELECT DISTINCT users.username FROM users;
```

## See Also
[FROM](from.md), [WHERE](where.md), [Functions](../../functions.md), [JSON Querying](../../json.md)

---
[← Back to SELECT](index.md)
