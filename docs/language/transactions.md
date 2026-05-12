# Transactions
# Transactions
Groups multiple statements into a single atomic operation.

```grammar title="Grammar"
transaction_stmt      ::= TRANSACTION ( transaction_mode )?
                          DO
                            transaction_statement+
                          END

transaction_mode      ::= READ ONLY
                        | READ WRITE

transaction_statement ::= insert_stmt ';'
                        | update_stmt ';'
                        | delete_stmt ';'
                        | select_stmt ';'
                        | for_stmt ';'
                        | savepoint_stmt ';'
                        | rollback_stmt ';'

savepoint_stmt        ::= SAVEPOINT identifier

rollback_stmt         ::= ROLLBACK ( TO identifier )?
```

## Description
A `TRANSACTION` block groups one or more statements into a single atomic operation.
Either all statements succeed and are committed together, or the entire block is rolled
back on failure. No partial commits occur within a transaction block.

LinkQL has three execution modes:

| Mode | Behavior |
|------|----------|
| **Transaction block** | Explicit `TRANSACTION DO...END`. All statements succeed or all roll back. |
| **[FOR](dml/for.md) loop** | Implicitly transactional. All iterations succeed or all roll back. |
| **Sequential statements** | Each statement commits independently. Execution stops on first failure, prior commits remain. |

## Parameters

`READ ONLY`
:   The transaction may only read data. Any attempt to modify data returns an error.
    Useful for reporting queries where you want to guarantee nothing gets changed.

`READ WRITE`
:   The transaction may read and write data. This is the default behavior if no mode
    is specified.

`SAVEPOINT identifier`
:   Creates a named checkpoint within the transaction. The transaction can be rolled
    back to this point without rolling back the entire block.

`ROLLBACK`
:   Rolls back the entire transaction. All changes made within the block are undone.

`ROLLBACK TO identifier`
:   Rolls back to a named savepoint. All changes made after the savepoint are undone.
    Changes made before the savepoint remain. The savepoint must have been defined
    earlier in the same transaction block — referencing an undefined savepoint returns
    an error detected at planning time.

## Execution Modes

### Transaction Block
All statements inside a `TRANSACTION DO...END` block are committed together or not
at all. If any statement fails, the entire block rolls back.

```sql
TRANSACTION
DO
    INSERT INTO users (username) VALUES ('john');
    UPDATE users SET status = 'active' WHERE username = 'john';
END;
```

### FOR Loop
[FOR](dml/for.md) loops are implicitly transactional. All iterations succeed as a whole or the
entire loop rolls back. If a `FOR` loop is inside a transaction block, its atomicity
is absorbed into the block — a loop failure rolls back the entire transaction.

```sql
TRANSACTION
DO
    FOR user IN inactive_users
    DO
        DELETE FROM sessions WHERE user_id = user.user_id;
        UPDATE users SET status = 'inactive' WHERE user_id = user.user_id;
    END;
END;
```

### Sequential Statements
Statements executed outside of a transaction block each commit independently. If one
fails, prior commits remain and execution stops.

```sql
INSERT INTO users (username) VALUES ('john');  -- commits independently
UPDATE users SET status = 'active' WHERE username = 'john';  -- commits independently
DELETE FROM sessions WHERE user_id = 1;  -- commits independently
```

## Savepoints
Savepoints create named checkpoints within a transaction that allow partial rollbacks.

```sql
TRANSACTION
DO
    INSERT INTO users (username) VALUES ('john');
    SAVEPOINT after_insert;
    UPDATE users SET status = 'active' WHERE username = 'john';
    ROLLBACK TO after_insert;  -- undoes the UPDATE, INSERT remains
END;
```

## Foreign Key Validation
Foreign key constraints are always validated at commit time, not inline. This means
a statement that would violate a foreign key constraint will not fail immediately —
the error is raised when the transaction commits.

## Notes
- DDL statements (`CREATE`, `ALTER`, `DROP`) are not permitted inside a transaction
  block. DDL auto-commits in most systems and would break the all-or-nothing guarantee.
- `ROLLBACK` with no `TO` rolls back the entire transaction.
- `ROLLBACK TO` an undefined savepoint returns an error detected at planning time
  before the transaction begins.
- `FOR` loops inside a transaction block have their atomicity absorbed into the block.
  A loop failure rolls back the entire transaction.
- Foreign key constraints are always deferred and validated at commit time.
- `READ ONLY` and `READ WRITE` modes are defined in the language spec but deferred
  to multi-user implementation.

## Examples

```sql title="Basic transaction"
TRANSACTION
DO
    INSERT INTO users (username) VALUES ('john');
    UPDATE users SET status = 'active' WHERE username = 'john';
END;
```

```sql title="Read only transaction"
TRANSACTION READ ONLY
DO
    SELECT users.username, users.status FROM users;
END;
```

```sql title="Transaction with savepoint"
TRANSACTION
DO
    INSERT INTO users (username) VALUES ('john');
    SAVEPOINT after_insert;
    UPDATE users SET status = 'active' WHERE username = 'john';
    ROLLBACK TO after_insert;
END;
```

```sql title="Manual rollback"
TRANSACTION
DO
    INSERT INTO users (username) VALUES ('john');
    ROLLBACK;
END;
```

```sql title="Transaction with FOR loop"
TRANSACTION
DO
    UPDATE users SET status = 'active' WHERE user_id = 1;
    FOR user IN inactive_users
    DO
        DELETE FROM sessions WHERE user_id = user.user_id;
        UPDATE users SET status = 'inactive' WHERE user_id = user.user_id;
    END;
END;
```

```sql title="Transaction across table and collection"
TRANSACTION
DO
    INSERT INTO users (username, first_name, last_name)
    VALUES ('john', 'John', 'Smith');
    INSERT INTO posts (user_id, content)
    VALUES (1, 'Hello world');
END;
```

## See Also
[FOR](dml/for.md), [INSERT](dml/insert.md), [UPDATE](dml/update.md), [DELETE](dml/delete.md)

---
[← Back to Language](index.md)
