# FOR
Iterates over a result set and applies DML statements to each row or document.

```grammar title="Grammar"
for_stmt        ::= FOR identifier IN for_source
                    DO
                      for_statement+
                    END

for_source      ::= identifier
                  | '(' select_stmt ')'

for_statement   ::= insert_stmt ';'
                  | update_stmt ';'
                  | delete_stmt ';'
```

## Description
`FOR` iterates over a result set and applies one or more DML statements to each row
or document. It is useful for performing row level operations that would otherwise
require multiple queries.

The `FOR` loop always iterates over a snapshot of the result set taken at the start
of the loop. Modifications made inside the loop do not affect the snapshot — subsequent
iterations always see the original data.

`FOR` loops are implicitly transactional. All iterations succeed as a whole or all
roll back. If a `FOR` loop is inside a block transaction, its atomicity is absorbed
into the block.

## Parameters

`identifier`
:   The loop variable. Represents the current row or document on each iteration.
    Table columns are accessed with dot notation — `user.column`. Collection fields
    are accessed with double colon notation — `user::field`.

`for_source`
:   The result set to iterate over. Can be a table or collection identifier, a LET
    binding name, or a subquery.

`for_statement`
:   A DML statement to apply on each iteration. Each statement must end with a
    semicolon. Only `INSERT`, `UPDATE`, and `DELETE` are valid inside a `FOR` loop.
    DDL statements and nested `FOR` loops are not permitted.

## Transactions
`FOR` loops are implicitly transactional — all iterations are treated as a single
unit of work. If any iteration fails, all changes made by the loop are rolled back.

If a `FOR` loop is used inside a block transaction, the loop's atomicity is absorbed
into the block. A failure inside the loop causes the entire block to roll back.

## Notes
- The loop always iterates over a snapshot. Modifications inside the loop do not
  affect subsequent iterations.
- Only `INSERT`, `UPDATE`, and `DELETE` are valid inside a `FOR` loop. `SELECT`
  and DDL statements are not permitted.
- Nested `FOR` loops are not supported.
- `FOR` loops are implicitly transactional. All iterations succeed or all roll back.
- A `FOR` loop inside a block transaction is absorbed into the block's atomicity.
- Use `LET` to define a named subquery before the loop and reference it by name
  as the source.

## Examples

```sql title="Basic FOR loop"
FOR user IN inactive_users
DO
    DELETE FROM sessions WHERE user_id = user.user_id;
    UPDATE users SET status = 'inactive' WHERE user_id = user.user_id;
END
```

```sql title="FOR loop with a subquery source"
FOR user IN (
    SELECT users.user_id, users.username
    FROM users
    WHERE users.last_login < '2020-01-01'
)
DO
    UPDATE users SET status = 'inactive' WHERE user_id = user.user_id;
    DELETE FROM sessions WHERE user_id = user.user_id;
END
```

```sql title="FOR loop with a LET binding"
LET inactive_users = (
    SELECT users.user_id
    FROM users
    WHERE users.last_login < '2020-01-01'
),
FOR user IN inactive_users
DO
    UPDATE users SET status = 'inactive' WHERE user_id = user.user_id;
    DELETE FROM sessions WHERE user_id = user.user_id;
END
```

```sql title="FOR loop over a collection"
FOR post IN (
    SELECT posts::post_id, posts::user_id
    FROM posts
    WHERE posts::created_at < '2020-01-01'
)
DO
    DELETE FROM comments WHERE post_id = post::post_id;
    DELETE FROM posts WHERE post_id = post::post_id;
END
```

## See Also
[LET](let.md), [UPDATE](update.md), [DELETE](delete.md), [INSERT](insert.md), [Transactions](../transactions.md)

---
[← Back to SELECT](../select.md)
