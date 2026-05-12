# GROUP BY / HAVING
Groups rows and documents and filters groups based on a condition.

```grammar title="Grammar"
group_clause    ::= GROUP BY expr (',' expr)*
                      ( HAVING expr )?
```

## Description
`GROUP BY` groups rows and documents that share the same values in the specified
expressions into summary rows. It is typically used with aggregate functions to compute
a value for each group.

`HAVING` filters groups after aggregation. It is equivalent to `WHERE` but operates
on grouped results rather than individual rows. Aggregate functions are valid in
`HAVING`.

## GROUP BY

Rows and documents are grouped by the specified expressions. All columns and fields
in the `SELECT` list that are not inside an aggregate function must appear in the
`GROUP BY` clause.

```sql
-- Group by a single column
SELECT users.status, COUNT(users.user_id) AS total
FROM users
GROUP BY users.status;

-- Group by multiple columns
SELECT users.status, users.country, COUNT(users.user_id) AS total
FROM users
GROUP BY users.status, users.country;

-- Group across table and collection
SELECT users.status, COUNT_ARRAY(posts::likes) AS total_likes
FROM users, LEFT posts(user_id)
GROUP BY users.status;
```

## HAVING

`HAVING` filters groups after aggregation. It supports the same expressions and
operators as `WHERE` including aggregate functions.

```sql
-- Filter groups by aggregate
SELECT users.status, COUNT(users.user_id) AS total
FROM users
GROUP BY users.status
HAVING COUNT(users.user_id) > 10;

-- Filter groups by field value
SELECT users.country, COUNT(users.user_id) AS total
FROM users
GROUP BY users.country
HAVING users.country != 'unknown';
```

## WHERE vs HAVING

`WHERE` filters individual rows before grouping. `HAVING` filters groups after
aggregation. They can be used together.

```sql
SELECT users.status, COUNT(users.user_id) AS total
FROM users
WHERE users.created_at > '2024-01-01'
GROUP BY users.status
HAVING COUNT(users.user_id) > 5;
```

In this example `WHERE` first excludes users created before 2024, then `GROUP BY`
groups the remaining users by status, then `HAVING` excludes groups with 5 or fewer
users.

## NULL and MISSING Behavior

- **`NULL`** values in a grouped field are grouped together into a single `NULL` group.
- **`MISSING`** values on collection free fields exclude the document from grouping
  silently. Documents where the grouped field is `MISSING` are not included in any
  group.

## Notes
- All non-aggregate expressions in the `SELECT` list must appear in the `GROUP BY`
  clause.
- `HAVING` filters after aggregation. `WHERE` filters before aggregation.
- Aggregate functions are valid in `HAVING` but not in `WHERE`.
- Grouping on a collection free field excludes documents where that field is `MISSING`.
- Grouping across tables and collections is fully supported.

## Examples

```sql title="Basic grouping"
SELECT users.status, COUNT(users.user_id) AS total
FROM users
GROUP BY users.status;
```

```sql title="Grouping with HAVING"
SELECT users.country, COUNT(users.user_id) AS total
FROM users
GROUP BY users.country
HAVING COUNT(users.user_id) > 100;
```

```sql title="WHERE and HAVING together"
SELECT users.status, COUNT(users.user_id) AS total
FROM users
WHERE users.created_at > '2024-01-01'
GROUP BY users.status
HAVING COUNT(users.user_id) > 5;
```

```sql title="Grouping across table and collection"
SELECT users.username, COUNT_ARRAY(posts::likes) AS total_likes
FROM users, LEFT posts(user_id)
GROUP BY users.username
HAVING COUNT_ARRAY(posts::likes) > 50;
```

```sql title="Grouping on collection free field"
SELECT posts::category, COUNT(posts::post_id) AS total
FROM posts
GROUP BY posts::category;
```

## See Also
[WHERE](where.md), [SELECT Items](select-items.md), [ORDER BY / LIMIT](order-and-limit.md)

---
[← Back to SELECT](../select.md)
