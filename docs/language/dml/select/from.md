# FROM
Specifies the source tables and collections for a `SELECT` statement.

```grammar title="Grammar"
from_clause     ::= from_item ( ',' join_clause )*

from_item       ::= identifier ( AS? identifier )?
                  | '(' select_stmt ')' AS identifier

join_clause     ::= traditional_join
                  | shorthand_join
```

## Description
The `FROM` clause defines the source tables and collections for a `SELECT` statement.
It can reference a single table or collection, multiple sources joined together, or a
subquery. Joins are specified as part of the `FROM` clause — see [Joins](joins.md) for
full join syntax and behavior.

## Source Types

### Table or Collection
A direct reference to a table or collection by name. An optional alias can be assigned
with or without the `AS` keyword.

```sql
SELECT users.username FROM users;

-- With alias
SELECT u.username FROM users AS u;
SELECT u.username FROM users u;
```

### Subquery
A subquery can be used as a source in the `FROM` clause. Subqueries must always be
aliased with `AS`.

```sql
SELECT active.username
FROM (
    SELECT users.username
    FROM users
    WHERE users.last_login > '2024-01-01'
) AS active;
```

### Multiple Sources
Multiple sources are comma separated. When multiple sources are listed, joins are
resolved left to right with the first source as the base table. See [Joins](joins.md)
for full join behavior.

```sql
SELECT users.username, posts::content
FROM users, posts(user_id);
```

## Notes
- The first source listed in the `FROM` clause is always the base table for shorthand
  joins.
- Subqueries must always be aliased with `AS`.
- Tables and collections can be mixed freely in the same `FROM` clause.
- See [Joins](joins.md) for full join syntax including traditional and shorthand joins.

## Examples

```sql title="Single table"
SELECT users.username, users.first_name
FROM users;
```

```sql title="Single collection"
SELECT posts::content, posts::created_at
FROM posts;
```

```sql title="Table with alias"
SELECT u.username, u.first_name
FROM users AS u;
```

```sql title="Subquery as source"
SELECT active.username
FROM (
    SELECT users.username
    FROM users
    WHERE users.last_login > '2024-01-01'
) AS active;
```

```sql title="Multiple sources with a join"
SELECT users.username, posts::content
FROM users, posts(user_id);
```

## See Also
[Joins](joins.md), [WHERE](where.md), [SELECT Items](select-items.md)

---
[← Back to SELECT](../select.md)
