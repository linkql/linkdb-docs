# SELECT
Retrieves data from one or more tables or collections.

```grammar title="Grammar"
select_stmt         ::= with_clause? intersect_stmt ( ( UNION | EXCEPT ) ALL? intersect_stmt )*
                          ( ORDER BY order_item ( ',' order_item )* )?
                          limit_clause?
                          for_clause?

intersect_stmt      ::= select_core ( INTERSECT ALL? select_core )*

select_core         ::= '(' select_stmt ')'
                      | SELECT DISTINCT? select_item (',' select_item)*
                          ( FROM from_clause )?
                          ( WHERE expr )?
                          ( group_clause )?
                          ( WINDOW window_definition ( ',' window_definition)* )?

with_clause         ::= WITH ( RECURSIVE )? cte ( ',' cte )*

cte                 ::= identifier ( '(' identifier ( ',' identifier )* ')' )? AS '(' select_stmt ')'

select_item         ::= '*'
                      | table_star
                      | collection_star
                      | expr ( AS identifier )?
                      | function_call AS identifier

table_star          ::= identifier '.' '*'

collection_star     ::= identifier '::' '*'

group_clause        ::= GROUP BY grouping_element (',' grouping_element)*
                          ( HAVING expr )?

grouping_element    ::= expr
                      | '(' ')'
                      | '(' expr ( ',' expr )* ')'
                      | ROLLUP '(' grouping_element (  ',' grouping_element )* ')'
                      | CUBE '(' grouping_element (  ',' grouping_element )* ')'
                      | GROUPING SETS '(' grouping_element (  ',' grouping_element )* ')'

order_item          ::= expr ( ASC | DESC )? ( NULLS ( FIRST | LAST ) )?

from_clause         ::= from_item ( ',' shorthand_join )* ( traditional_join )*

from_item           ::= LATERAL? identifier ( AS? identifier )?
                      | LATERAL? '(' select_stmt ')' AS? identifier
                      | UNNEST '(' expr ( ',' expr )* ')'
                          ( WITH ORDINALITY )? ( AS? identifier )?

traditional_join    ::= NATURAL? join_type? JOIN from_item join_condition

shorthand_join      ::= identifier
                      | join_type identifier
                      | join_type identifier '(' identifier (',' identifier)* ')'
                      | join_type identifier '(' identifier (',' identifier)* ')' ON shorthand_condition

shorthand_condition ::= identifier '(' identifier ')'
                      | expr

join_condition      ::= ON expr
                      | USING '(' identifier ( ',' identifier )* ')'

join_type           ::= INNER | LEFT OUTER? | RIGHT OUTER? | FULL OUTER? | CROSS

window_definition   ::= identifier AS '(' window_spec ')'

limit_clause        ::= LIMIT expr ( OFFSET expr )?
                      | OFFSET expr ( ROW | ROWS )? ( FETCH FIRST expr ( ROW | ROWS ) ( ONLY | WITH TIES )? )?
                      | FETCH FIRST expr ( ROW | ROWS ) ( ONLY | WITH TIES )?

for_clause          ::= FOR ( UPDATE | NO KEY UPDATE | SHARE | KEY SHARE )
                          ( OF identifier ( ',' identifier )* )?
                          ( NOWAIT | SKIP LOCKED )?
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

- [SELECT ITEMS](select-items.md) - specifies source columns to retrieve from your FROM clause
- [FROM](from.md) — specifies the source tables and collections, including join syntax
- [JOIN](joins.md) — joins tables and collections together
- [WHERE](where.md) — filters rows based on a condition
- [GROUP BY / HAVING](grouping.md) — groups rows and filters groups
- [ORDER BY / LIMIT](order-and-limit.md) — sorts and paginates results
- [CASE](case.md) — conditional expressions, standard and shorthand
- [Common Table Expressions](cte.md) — named subqueries using `WITH`
- [Subqueries](subqueries.md) — queries nested inside another statement
- [Window Functions](window.md) — compute values across rows
- [Set Operations](set-ops.md) — combine results with `UNION`, `INTERSECT`, and `EXCEPT`
- [LET](../let.md) — defines named subqueries for use in the statement
- [FOR](../for.md) — applies statements to each row in a result set

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
LEFT JOIN users ON users.user_id = posts::user_id
WHERE posts::created_at > '2024-01-01';
```

```sql title="Full select with all clauses"
SELECT DISTINCT
    users.username,
    COUNT(posts::content) AS post_count
FROM posts
LEFT JOIN users ON users.user_id = posts::user_id
WHERE posts::created_at > '2024-01-01'
GROUP BY users.username
HAVING post_count > 5
ORDER BY post_count DESC
LIMIT 10 OFFSET 20;
```

```sql title="CTE"
WITH cte AS (SELECT id FROM users) SELECT * FROM cte;
```

```sql title="Recursive CTE"
WITH RECURSIVE nums AS (
    (SELECT 1) UNION ALL (SELECT n + 1 FROM nums WHERE n < 5)
) SELECT * FROM nums;
```

```sql title="Window function"
SELECT
    users.username,
    ROW_NUMBER() OVER (PARTITION BY users.status ORDER BY users.created_at) AS rn
FROM users;
```

```sql title="UNION"
SELECT username FROM users_a
UNION
SELECT username FROM users_b;
```

## See Also
[INSERT](../insert.md), [UPDATE](../update.md), [DELETE](../delete.md), [JSON Querying](../../json.md)

---
[← Back to Language Reference](../../index.md)
