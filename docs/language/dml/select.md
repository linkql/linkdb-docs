# SELECT
Retrieves data from one or more tables or collections.

```grammar title="Grammar"
select_stmt       ::= SELECT DISTINCT? select_item (',' select_item)*
                      FROM from_clause
                      ( WHERE expr )?
                      ( group_clause )?
                      ( ORDER BY order_item (',' order_item)* )?
                      ( LIMIT integer_literal ( OFFSET integer_literal )? )?

group_clause      ::= GROUP BY expr (',' expr)*
                       ( HAVING expr )?

select_item     ::= '*'
                  | table_star
                  | collection_star
                  | expr ( AS identifier )?
                  | function_call AS identifier

table_star      ::= identifier '.' '*'

collection_star ::= identifier '::' '*'

order_item        ::= expr ( ASC | DESC )?

from_clause       ::= from_item ( ',' join_clause )*

from_item         ::= identifier ( AS? identifier )?
                    | '(' select_stmt ')' AS identifier

join_clause       ::= traditional_join
                    | shorthand_join

traditional_join  ::= join_type? JOIN from_item join_condition

shorthand_join    ::= identifier
                    | join_type identifier
                    | join_type identifier '(' identifier (',' identifier)* ')'
                    | join_type identifier '(' identifier (',' identifier)* ')'
                          ON identifier '(' identifier ')'

join_condition    ::= ON expr
                    | USING '(' identifier ( ',' identifier )* ')'

join_type         ::= INNER | LEFT | RIGHT | FULL OUTER? | CROSS | OUTER
```

## Description
`SELECT` retrieves rows from one or more tables or collections. Results can be filtered,
grouped, ordered, and limited. LinkQL extends standard SQL `SELECT` with shorthand join
syntax, `LET` blocks for named subqueries, `FOR` loops for row level operations, and
support for querying both tables and collections in a single statement.

Tables are referenced with dot notation — `table.column`. Collections are referenced
with double colon notation — `collection::field`.

## Clauses
*Click for more info*

- [SELECT ITEMS](select/select-items.md) - specifies source columns to retrieve from your FROM clause
- [FROM](select/from.md) — specifies the source tables and collections, including join syntax
- [JOIN](select/joins.md) — joins tables and collections together
- [WHERE](select/where.md) — filters rows based on a condition
- [GROUP BY / HAVING](select/grouping.md) — groups rows and filters groups
- [ORDER BY / LIMIT](select/order-and-limit.md) — sorts and paginates results
- [LET](let.md) — defines named subqueries for use in the statement
- [FOR](for.md) — applies statements to each row in a result set
- [CASE](select/case.md) — conditional expressions, standard and shorthand

## Examples

```sql title="Basic select"
SELECT users.username, users.first_name
FROM users
WHERE users.last_name = 'Smith';
```

```sql title="Select across a table and collection"
SELECT
    users.username,
    posts::content,
    posts::created_at
FROM posts
LEFT JOIN users ON users.user_id = posts.user_id
WHERE posts::created_at > '2024-01-01';
```

```sql title="Full select with all clauses"
SELECT DISTINCT
    users.username,
    COUNT(posts::content) AS post_count
FROM posts
LEFT JOIN users ON users.user_id = posts.user_id
WHERE posts::created_at > '2024-01-01'
GROUP BY users.username
HAVING post_count > 5
ORDER BY post_count DESC
LIMIT 10 OFFSET 20;
```

## See Also
[INSERT](insert.md), [UPDATE](update.md), [DELETE](delete.md), [JSON Querying](../json.md)

---
[← Back to DML](../dml.md)
