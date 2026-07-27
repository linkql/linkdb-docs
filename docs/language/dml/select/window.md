# Window Functions
Compute values across a window of rows related to the current row.

```grammar title="Grammar"
select_core         ::= '(' select_stmt ')'
                      | SELECT DISTINCT? select_item (',' select_item)*
                          ( FROM from_clause )?
                          ( WHERE expr )?
                          ( group_clause )?
                          ( WINDOW window_definition ( ',' window_definition)* )?

window_definition   ::= identifier AS '(' window_spec ')'

window_spec         ::= identifier
                      | ( PARTITION BY expr ( ',' expr )* )?
                        ( ORDER BY order_item ( ',' order_item )* )?
                        ( frame_clause )?

frame_clause        ::= ( ROWS | RANGE )
                        ( frame_start
                        | BETWEEN frame_boundary AND frame_boundary )

frame_start         ::= UNBOUNDED PRECEDING
                      | expr PRECEDING
                      | CURRENT ROW

frame_boundary      ::= UNBOUNDED PRECEDING
                      | UNBOUNDED FOLLOWING
                      | expr PRECEDING
                      | expr FOLLOWING
                      | CURRENT ROW

order_item          ::= expr ( ASC | DESC )? ( NULLS ( FIRST | LAST ) )?
```

## Description
Window functions perform calculations across a set of rows related to the current
row without collapsing the result set like aggregate functions do. The `OVER`
clause defines the window using partitions, ordering, and frame boundaries.

Named windows can be defined in the `WINDOW` clause and referenced by name in
`OVER` clauses.

## Parameters

`PARTITION BY expr`
:   Divides the result set into partitions. The window function is computed
    independently within each partition.

`ORDER BY order_item`
:   Defines the order of rows within each partition or the window. Required for
    ranking functions and frame-based calculations.

`frame_clause`
:   Limits the window to a subset of rows relative to the current row using
    `ROWS` or `RANGE`.

`WINDOW window_definition`
:   Defines named windows that can be referenced by `OVER` clauses.

## Notes
- Window functions can be combined with `GROUP BY`; the window is computed after
  grouping.
- Aggregate functions can also be used as window functions with `OVER`.
- `NULLS FIRST` and `NULLS LAST` control null ordering within `ORDER BY` in the
  window specification.

## Examples

```sql title="Window function with no partition, order, or frame"
SELECT ROW_NUMBER() OVER () AS rn FROM users;
```

```sql title="Window function with partition only"
SELECT users.status, COUNT(*) OVER (PARTITION BY users.status) AS status_count FROM users;
```

```sql title="Window function with order only"
SELECT users.username, RANK() OVER (ORDER BY users.created_at) AS signup_rank FROM users;
```

```sql title="Window function with partition and order"
SELECT
    users.status,
    users.username,
    ROW_NUMBER() OVER (PARTITION BY users.status ORDER BY users.created_at) AS rn
FROM users;
```

```sql title="Window functions using LAG and LEAD"
SELECT
    orders.order_id,
    orders.total,
    LAG(orders.total) OVER (ORDER BY orders.order_date) AS prev_total,
    LEAD(orders.total) OVER (ORDER BY orders.order_date) AS next_total
FROM orders;
```

```sql title="Running sum with single-bound frame clause"
SELECT
    orders.order_date,
    SUM(orders.total) OVER (ORDER BY orders.order_date ROWS UNBOUNDED PRECEDING) AS running_total
FROM orders;
```

```sql title="Running sum with explicit BETWEEN frame clause"
SELECT
    orders.order_date,
    SUM(orders.total) OVER (
        ORDER BY orders.order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders;
```

```sql title="Moving average with bounded frame clause"
SELECT
    orders.order_date,
    AVG(orders.total) OVER (
        ORDER BY orders.order_date
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
    ) AS moving_avg
FROM orders;
```

```sql title="Running sum using RANGE frame clause"
SELECT
    orders.order_date,
    SUM(orders.total) OVER (
        ORDER BY orders.order_date
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders;
```

```sql title="Window function referencing a named window"
SELECT
    users.status,
    users.username,
    ROW_NUMBER() OVER w AS rn
FROM users
WINDOW w AS (PARTITION BY users.status ORDER BY users.created_at);
```

```sql title="Multiple named windows"
SELECT
    users.username,
    ROW_NUMBER() OVER by_status AS rn1,
    RANK() OVER by_created AS rn2
FROM users
WINDOW
    by_status AS (PARTITION BY users.status),
    by_created AS (ORDER BY users.created_at);
```

```sql title="Window function alongside GROUP BY"
SELECT
    users.status,
    COUNT(users.user_id) AS total,
    SUM(COUNT(users.user_id)) OVER () AS grand_total
FROM users
GROUP BY users.status;
```

```sql title="Window function on a collection field"
SELECT
    posts::user_id,
    posts::content,
    ROW_NUMBER() OVER (PARTITION BY posts::user_id ORDER BY posts::created_at) AS post_num
FROM posts;
```

```sql title="Order by with NULLS FIRST"
SELECT users.username, users.last_login
FROM users
ORDER BY users.last_login DESC NULLS FIRST;
```

```sql title="Order by with NULLS LAST"
SELECT users.username, users.last_login
FROM users
ORDER BY users.last_login ASC NULLS LAST;
```

## See Also
[Functions in SELECT](../../functions.md), [SELECT](index.md), [ORDER BY and LIMIT](order-and-limit.md)

---
[← Back to SELECT](index.md)
