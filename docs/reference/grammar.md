# Grammar
Complete grammar reference for LinkQL.

## Notation
- `::=` Definition
- `|` Alternation
- `x?` Zero or one (optional)
- `x*` Zero or more
- `x+` One or more
- `(x)` Grouping
- `'x'` Literal terminal
- `UPPER` Keyword (case-insensitive)
- `lower` Non-terminal

---

## Program

```
program         ::= statement+

statement       ::= select_stmt ';'
                  | insert_stmt ';'
                  | update_stmt ';'
                  | delete_stmt ';'
                  | create_table_stmt ';'
                  | create_collection_stmt ';'
                  | create_database_stmt ';'
                  | alter_table_stmt ';'
                  | alter_collection_stmt ';'
                  | alter_database_stmt ';'
                  | drop_stmt ';'
                  | truncate_stmt ';'
                  | transaction_stmt ';'
                  | for_stmt ';'
                  | let_block ';'

let_block       ::= let_binding+ dml_stmt

let_binding     ::= LET identifier '=' '(' select_stmt ')' ','

dml_stmt        ::= select_stmt
                  | insert_stmt
                  | update_stmt
                  | delete_stmt
```

---

## DDL — CREATE

```
create_database_stmt    ::= CREATE DATABASE ( IF NOT EXISTS )? identifier

create_table_stmt       ::= CREATE TABLE identifier '(' table_item (',' table_item)* ','? ')'

create_collection_stmt  ::= CREATE COLLECTION identifier
                              ( '(' collection_item (',' collection_item)* ','? ')' )?

table_item              ::= field_def
                          | table_constraint

collection_item         ::= field_def
                          | table_constraint

field_def               ::= '_id'
                          | identifier data_type column_constraint*

column_constraint       ::= constraint_name? column_constraint_type

column_constraint_type  ::= NOT NULL
                          | NULL
                          | UNIQUE
                          | PRIMARY KEY
                          | SECONDARY KEY
                          | AUTOINCREMENT ( '(' integer_literal ',' integer_literal ')' )?
                          | AUTONOW
                          | AUTO
                          | DEFAULT expr
                          | CHECK '(' expr ')'
                          | REFERENCES identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?

table_constraint        ::= constraint_name? table_constraint_type

table_constraint_type   ::= PRIMARY KEY '(' identifier (',' identifier)* ')'
                          | SECONDARY KEY '(' identifier (',' identifier)* ')'
                          | UNIQUE '(' identifier (',' identifier)* ')'
                          | CHECK '(' expr ')'
                          | FOREIGN KEY '(' identifier ')' REFERENCES identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?

constraint_name         ::= CONSTRAINT identifier

reference_action        ::= CASCADE
                          | SET NULL
                          | SET DEFAULT
                          | RESTRICT
                          | NO ACTION
```

---

## DDL — ALTER

```
alter_database_stmt     ::= ALTER DATABASE identifier RENAME TO identifier

alter_table_stmt        ::= ALTER TABLE identifier alter_table_cmd

alter_table_cmd         ::= ADD COLUMN field_def
                          | DROP COLUMN identifier
                          | MODIFY COLUMN field_def ( USING expr )?
                          | RENAME COLUMN identifier TO identifier
                          | RENAME TO identifier
                          | ADD table_constraint
                          | DROP CONSTRAINT identifier
                          | MODIFY CONSTRAINT identifier table_constraint_type

alter_collection_stmt   ::= ALTER COLLECTION identifier alter_collection_cmd

alter_collection_cmd    ::= ADD FIELD field_def
                          | DROP FIELD identifier ( KEEP )?
                          | DROP FREE FIELD identifier
                          | MODIFY FIELD field_def ( USING expr )?
                          | RENAME FIELD identifier TO identifier
                          | RENAME TO identifier
                          | ADD table_constraint
                          | DROP CONSTRAINT identifier
                          | MODIFY CONSTRAINT identifier table_constraint_type
```

---

## DDL — DROP

```
drop_stmt               ::= drop_database_stmt
                          | drop_table_stmt
                          | drop_collection_stmt

drop_database_stmt      ::= DROP DATABASE ( IF EXISTS )? identifier CONFIRM string_literal

drop_table_stmt         ::= DROP TABLE ( IF EXISTS )? identifier ( cascade_clause )?

drop_collection_stmt    ::= DROP COLLECTION ( IF EXISTS )? identifier ( cascade_clause )?

cascade_clause          ::= CASCADE ( cascade_depth )? ( cascade_override )?

cascade_depth           ::= integer_literal
                          | INFINITY

cascade_override        ::= SET NULL
                          | SET DEFAULT
                          | DROP

truncate_stmt           ::= truncate_database_stmt
                          | truncate_table_stmt
                          | truncate_collection_stmt

truncate_database_stmt  ::= TRUNCATE DATABASE ( IF EXISTS )? identifier CONFIRM string_literal
                              ( RESTART IDENTITY )?

truncate_table_stmt     ::= TRUNCATE TABLE identifier ( RESTART IDENTITY )?

truncate_collection_stmt ::= TRUNCATE COLLECTION identifier ( RESTART IDENTITY )?
```

---

## DML — SELECT

```
select_stmt     ::= SELECT DISTINCT? select_item (',' select_item)*
                    FROM from_clause
                    ( WHERE expr )?
                    ( group_clause )?
                    ( ORDER BY order_item (',' order_item)* )?
                    ( LIMIT integer_literal ( OFFSET integer_literal )? )?

select_item     ::= '*'
                  | table_star
                  | collection_star
                  | expr ( AS identifier )?
                  | function_call AS identifier

table_star      ::= identifier '.' '*'

collection_star ::= identifier '::' '*'

group_clause    ::= GROUP BY expr (',' expr)*
                      ( HAVING expr )?

order_item      ::= expr ( ASC | DESC )?

from_clause     ::= from_item ( ',' join_clause )*

from_item       ::= identifier ( AS? identifier )?
                  | '(' select_stmt ')' AS identifier

join_clause     ::= traditional_join
                  | shorthand_join

traditional_join    ::= join_type? JOIN from_item join_condition

shorthand_join      ::= identifier
                      | join_type identifier
                      | join_type identifier '(' identifier (',' identifier)* ')'
                      | join_type identifier '(' identifier (',' identifier)* ')'
                            ON shorthand_condition

shorthand_condition ::= identifier '(' identifier ')'
                      | expr

join_condition  ::= ON expr
                  | USING '(' identifier ( ',' identifier )* ')'

join_type       ::= INNER | LEFT | RIGHT | FULL OUTER? | CROSS | OUTER
```

---

## DML — INSERT

```
insert_stmt     ::= INSERT INTO identifier ( '(' identifier ( ',' identifier )* ')' )?
                      ( with_free )?
                    insert_source
                    ( ON CONFLICT conflict_target conflict_action )?

insert_source   ::= select_stmt
                  | VALUES '(' insert_value ( ',' insert_value )* ')'
                        ( ',' '(' insert_value ( ',' insert_value )* ')' )*

insert_value    ::= expr | DEFAULT

with_free       ::= WITH FREE free_source

free_source     ::= '(' identifier ( ',' identifier )* ')'
                  | UNPACK '(' expr ')'
                  | UNPACK '(' ')'

conflict_target ::= '(' identifier ( ',' identifier )* ')'

conflict_action ::= DO NOTHING
                  | DO UPDATE SET conflict_set ( ',' conflict_set )*

conflict_set    ::= identifier '=' expr
                  | identifier '=' EXCLUDED '.' identifier
```

---

## DML — UPDATE

```
update_stmt     ::= UPDATE identifier
                    ( with_free )?
                    SET set_values
                    ( FROM from_clause )?
                    ( WHERE expr )?
                    ( CONFIRM string_literal )?

set_values      ::= set_expr ( ',' set_expr )*

set_expr        ::= identifier '=' expr
                  | identifier '=' EXCLUDED '.' identifier
```

---

## DML — DELETE

```
delete_stmt     ::= DELETE FROM identifier
                    ( FROM from_clause )?
                    ( WHERE expr )?
                    ( CONFIRM string_literal )?
```

---

## FOR

```
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

---

## Transactions

```
transaction_stmt        ::= TRANSACTION ( transaction_mode )?
                            DO
                              transaction_statement+
                            END

transaction_mode        ::= READ ONLY
                          | READ WRITE

transaction_statement   ::= insert_stmt ';'
                          | update_stmt ';'
                          | delete_stmt ';'
                          | select_stmt ';'
                          | for_stmt ';'
                          | savepoint_stmt ';'
                          | rollback_stmt ';'

savepoint_stmt          ::= SAVEPOINT identifier

rollback_stmt           ::= ROLLBACK ( TO identifier )?
```

---

## Expressions

```
expr            ::= or_expr

or_expr         ::= and_expr ( OR and_expr )*

and_expr        ::= not_expr ( AND not_expr )*

not_expr        ::= NOT not_expr | comparison

comparison      ::= additive ( operator additive )?
                  | additive IS NOT? NULL
                  | additive IS NOT? MISSING
                  | additive IS NOT? NULL OR MISSING
                  | additive NOT? BETWEEN additive AND additive
                  | additive NOT? IN '(' expr (',' expr)* ')'
                  | additive NOT? IN '(' select_stmt ')'
                  | additive NOT? LIKE string_literal
                  | additive NOT? ILIKE string_literal
                  | EXISTS '(' select_stmt ')'

operator        ::= '=' | '!=' | '<' | '<=' | '>' | '>='

additive        ::= multiplicative ( ( '+' | '-' ) multiplicative )*

multiplicative  ::= unary ( ( '*' | '/' | '%' ) unary )*

unary           ::= ( '-' | '+' ) unary
                  | primary

primary         ::= literal
                  | function_call
                  | column_ref
                  | exists_expr
                  | '(' select_stmt ')'
                  | '(' expr ')'
                  | case_expr

case_expr       ::= standard_case | shorthand_case

standard_case   ::= CASE when_clause+ ( ELSE expr )? END

when_clause     ::= WHEN expr THEN expr

shorthand_case  ::= CASE shorthand_clause+ ( ELSE expr )? END

shorthand_clause ::= expr '?' expr ','?

exists_expr     ::= EXISTS '(' select_stmt ')'

function_call   ::= identifier '(' ( expr (',' expr)* )? ')'

column_ref      ::= identifier ( ( '.' | '::' ) identifier )*
                    ( '[' string_literal ']' )*
```

---

## JSON Access

```
json_access     ::= expr '[' string_literal ']' ( '[' string_literal ']' )*

json_function   ::= HAS '(' expr ',' string_literal ')'
                  | KEYS '(' expr ')'
                  | VALUES '(' expr ')'
                  | MERGE '(' expr ',' expr ( ',' ( LEFT | RIGHT ) )? ')'
                  | CONTAINS '(' expr ',' expr ')'
                  | UNPACK '(' expr ')'
                  | UNPACK '(' ')'
```

---

## Data Types

```
data_type       ::= INT
                  | BIGINT
                  | FLOAT
                  | DECIMAL ( '(' integer_literal ',' integer_literal ')' )?
                  | BOOLEAN
                  | VARCHAR ( '(' integer_literal ')' )?
                  | TEXT
                  | DATE
                  | TIME
                  | DATETIME
                  | UUID
                  | JSON
                  | JSONB
                  | ARRAY '(' data_type ')'
```

---

## Literals and Primitives

```
literal         ::= integer_literal
                  | float_literal
                  | string_literal
                  | boolean_literal
                  | NULL
                  | MISSING

integer_literal ::= digit+

float_literal   ::= digit+ '.' digit+

string_literal  ::= "'" any_char* "'"

boolean_literal ::= TRUE | FALSE

identifier      ::= letter ( letter | digit | '_' )*

letter          ::= 'a'..'z' | 'A'..'Z' | '_'

digit           ::= '0'..'9'
```

---

## See Also
[Keywords](keywords.md), [Data Types](../data_types.md), [Functions](../functions.md)
