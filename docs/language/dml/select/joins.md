# Joins
Combines rows from multiple tables and collections in a single query.

```grammar title="Grammar"
traditional_join    ::= join_type? JOIN from_item join_condition

shorthand_join      ::= identifier
                      | join_type identifier
                      | join_type identifier '(' identifier (',' identifier)* ')'
                      | join_type identifier '(' identifier (',' identifier)* ')'
                            ON shorthand_condition

shorthand_condition ::= identifier '(' identifier ')'
                      | expr

join_condition      ::= ON expr
                      | USING '(' identifier ( ',' identifier )* ')'

join_type           ::= INNER | LEFT | RIGHT | FULL OUTER? | CROSS | OUTER
```

## Description
LinkQL supports two join syntaxes — traditional SQL joins and a shorthand join syntax
designed for brevity. Both can be mixed freely in the same query. Joins are specified
as part of the `FROM` clause.

Tables are joined using dot notation — `table.column`. Collections are joined using
double colon notation — `collection::field`. Joins work across both tables and
collections and the engine resolves whether a target is a table or collection
automatically via the catalog.

## Join Types

| Type | Description |
|------|-------------|
| `INNER` | Returns rows with matching values in both sources. Default when no type is specified. |
| `LEFT` | Returns all rows from the left source and matching rows from the right. |
| `RIGHT` | Returns all rows from the right source and matching rows from the left. |
| `FULL OUTER` | Returns all rows from both sources, with `NULL` where there is no match. |
| `CROSS` | Returns the cartesian product of both sources. |
| `OUTER` | Shorthand for `FULL OUTER`. |

## Traditional Joins
Traditional joins follow standard SQL syntax. The join type is optional and defaults
to `INNER`.

```sql
-- INNER join
SELECT users.username, orders.total
FROM users
JOIN orders ON users.user_id = orders.user_id;

-- LEFT join
SELECT users.username, posts::content
FROM users
LEFT JOIN posts ON users.user_id = posts::user_id;

-- USING when column names are shared
SELECT users.username, orders.total
FROM users
JOIN orders USING (user_id);
```

## Shorthand Joins
Shorthand joins are a LinkQL specific syntax designed to reduce verbosity. They are
specified as part of the comma separated source list in the `FROM` clause. The first
source listed is always the base table — shorthand joins always join against it.

### Basic Shorthand
A bare identifier after the base table defaults to an `INNER` join on any shared
column names. If no shared column exists, an error is returned.

```sql
-- INNER join on shared column name
SELECT users.username, orders.total
FROM users, orders;
```

### Shorthand with Join Type
A join type before the identifier specifies the type of join.

```sql
-- LEFT join on shared column name
SELECT users.username, posts::content
FROM users, LEFT posts;
```

### Shorthand with Columns
Columns in parentheses specify which columns to join on. All specified columns must
be shared between the base table and the joined source or an error is returned.

```sql
-- Join on specific shared columns
SELECT users.username, posts::content
FROM users, posts(user_id);

-- LEFT join on specific shared columns
SELECT users.username, friends.username
FROM users, LEFT friends(user_id);
```

### Shorthand with ON
The `ON` clause overrides the default column matching when column names differ between
sources. Both a shorthand form and a full expression form are valid.

```sql
-- Shorthand ON form
SELECT users.username, posts::content
FROM users, OUTER posts(user_id) ON users(id);

-- Full expression ON form
SELECT users.username, posts::content
FROM users, OUTER posts(user_id) ON users.id = posts::user_id;
```

## Mixing Traditional and Shorthand
Traditional and shorthand joins can be mixed freely in the same query.

```sql
SELECT
    users.username,
    posts::content,
    comments::content
FROM
    users,
    LEFT posts(user_id),
    JOIN comments ON comments.post_id = posts::post_id;
```

## Notes
- The first source in the `FROM` clause is always the base table for shorthand joins.
- Shorthand joins with no columns specified default to joining on shared column names.
  If no shared column exists, an error is returned.
- Shorthand join columns must be shared between the base table and the joined source.
  If a specified column does not exist on both sources, an error is returned.
- Traditional and shorthand joins can be mixed in the same query.
- Joins work across tables and collections. The engine resolves whether a target is a
  table or collection automatically.

## Examples

```sql title="Traditional INNER join"
SELECT users.username, orders.total
FROM users
JOIN orders ON users.user_id = orders.user_id;
```

```sql title="Traditional LEFT join across table and collection"
SELECT users.username, posts::content
FROM users
LEFT JOIN posts ON users.user_id = posts::user_id;
```

```sql title="Shorthand join on shared column"
SELECT users.username, orders.total
FROM users, orders;
```

```sql title="Shorthand LEFT join with specific columns"
SELECT users.username, posts::content
FROM users, LEFT posts(user_id);
```

```sql title="Shorthand join with ON override"
SELECT users.username, posts::content
FROM users, OUTER posts(user_id) ON users(id);
```

```sql title="Mixed traditional and shorthand joins"
SELECT
    users.username,
    posts::content,
    COUNT_ARRAY(comments::likes) AS num_likes
FROM
    users,
    LEFT posts(user_id)
    LEFT JOIN comments ON comments.post_id = posts::post_id;
```

## See Also
[FROM](from.md), [WHERE](where.md), [SELECT Items](select-items.md)

---
[← Back to SELECT](../select.md)
