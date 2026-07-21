# Complete Grammar
Complete grammar reference for LinkQL including anything yet to be implemented.

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

statement       ::= ddl_stmt ';'
                  | dml_stmt ';'
                  | utility_stmt ';'
                  | transaction_stmt ';'
                  | for_stmt ';'
                  | let_block ';'

ddl_stmt        ::= create_stmt
                  | alter_stmt
                  | drop_stmt
                  | truncate_stmt

create_stmt     ::= create_database_stmt
                  | create_table_stmt
                  | create_collection_stmt
                  | create_view_stmt
                  | create_index_stmt

alter_stmt      ::= alter_database_stmt
                  | alter_table_stmt
                  | alter_collection_stmt
                  | alter_view_stmt
                  | alter_index_stmt

dml_stmt        ::= select_stmt
                  | insert_stmt
                  | update_stmt
                  | delete_stmt

utility_stmt    ::= refresh_view_stmt
                  | explain_stmt
                  | info_stmt

let_block       ::= LET let_binding (',' let_binding)* dml_stmt

let_binding     ::= identifier '=' '(' select_stmt ')'
                  | recursive_let

recursive_let   ::= RECURSIVE identifier '=' '(' select_stmt ')'
                      UNION ALL '(' select_stmt ')'
                      ( cycle_clause )*
                      ( SEARCH ( DEPTH | BREADTH ) FIRST BY identifier SET identifier )?

cycle_clause    ::= CYCLE ON identifier ( ',' identifier )* SET identifier
                  | MAX ITERATIONS integer_literal
```

---

## DDL — CREATE

```
create_database_stmt        ::= CREATE DATABASE ( IF NOT EXISTS )? identifier

create_table_stmt           ::= CREATE TABLE ( IF NOT EXISTS )? identifier '(' table_item (',' table_item)* ','? ')'

create_collection_stmt      ::= CREATE COLLECTION identifier ( IF NOT EXISTS )?
                                  ( '(' collection_item (',' collection_item)* ','? ')' )?

create_view_stmt            ::= CREATE ( OR REPLACE )? MATERIALIZED? VIEW
                                  ( IF NOT EXISTS )? identifier AS select_stmt

create_index_stmt           ::= CREATE UNIQUE? INDEX ( IF NOT EXISTS )? identifier
                                  ON identifier '(' order_item ( ',' order_item )* ')'
                                  ( WHERE expr )?

table_item                  ::= field_def
                              | table_constraint

collection_item             ::= field_def
                              | table_constraint

field_def                   ::= '_id'
                              | identifier data_type field_constraint*

field_constraint            ::= named_field_constraint
                              | field_property

named_field_constraint      ::= constraint_name? named_constraint_type

named_constraint_type       ::= UNIQUE
                              | PRIMARY KEY
                              | SECONDARY KEY
                              | CHECK '(' expr ')'
                              | REFERENCES reference_target

reference_target            ::= identifier ( '(' identifier ')' )?
                                  ( ON DELETE reference_action )?
                                  ( ON UPDATE reference_action )?

field_property             ::= AUTOINCREMENT ( '(' integer_literal ',' integer_literal ')' )?
                              | AUTONOW
                              | AUTO
                              | DEFAULT expr
                              | NOT NULL
                              | NULL

table_constraint            ::= constraint_name? table_constraint_type

table_constraint_type       ::= PRIMARY KEY '(' identifier (',' identifier)* ')'
                              | SECONDARY KEY '(' identifier (',' identifier)* ')'
                              | UNIQUE '(' identifier (',' identifier)* ')'
                              | CHECK '(' expr ')'
                              | FOREIGN KEY '(' identifier ')' REFERENCES identifier ( '(' identifier ')' )?
                                  ( ON DELETE reference_action )?
                                  ( ON UPDATE reference_action )?

constraint_name             ::= CONSTRAINT ( IF NOT EXISTS)? identifier

existing_constraint_name    ::= CONSTRAINT ( IF EXISTS )? identifier

reference_action            ::= CASCADE
                              | SET NULL
                              | SET DEFAULT
                              | RESTRICT
                              | NO ACTION
```

---

## DDL — ALTER

```
alter_database_stmt     ::= ALTER DATABASE identifier alter_database_cmd ( ',' alter_database_cmd )*

alter_database_cmd      ::= RENAME TO identifier

alter_table_stmt        ::= ALTER TABLE identifier alter_table_cmd ( ',' alter_table_cmd )*

alter_view_stmt         ::= ALTER MATERIALIZED? VIEW identifier alter_view_cmd

alter_index_stmt        ::= ALTER INDEX ( IF EXISTS )? identifier alter_index_cmd

alter_table_cmd         ::= ADD COLUMN ( IF NOT EXISTS )? field_def
                          | DROP COLUMN ( IF EXISTS )? identifier
                          | MODIFY COLUMN ( IF EXISTS )? field_def ( USING expr )?
                          | RENAME COLUMN ( IF EXISTS )? identifier TO identifier
                          | RENAME TO identifier
                          | ADD table_constraint
                          | MODIFY CONSTRAINT existing_constraint_name table_constraint_type
                          | DROP CONSTRAINT existing_constraint_name

alter_collection_stmt   ::= ALTER COLLECTION identifier alter_collection_cmd ( ',' alter_collection_cmd )*

alter_collection_cmd    ::= ADD FIELD ( IF NOT EXISTS )? field_def
                          | DROP FIELD ( IF EXISTS )? identifier ( KEEP )?
                          | DROP FREE FIELD ( IF EXISTS )? identifier
                          | MODIFY FIELD ( IF EXISTS )? field_def ( USING expr )?
                          | RENAME FIELD ( IF EXISTS )? identifier TO identifier
                          | RENAME TO identifier
                          | ADD table_constraint
                          | MODIFY CONSTRAINT existing_constraint_name table_constraint_type
                          | DROP CONSTRAINT existing_constraint_name

alter_view_cmd          ::= RENAME TO identifier
                          | AS select_stmt

alter_index_cmd         ::= RENAME TO identifier
```

---

## DDL — DROP

```
drop_stmt                   ::= drop_database_stmt
                              | drop_table_stmt
                              | drop_collection_stmt
                              | drop_view_stmt
                              | drop_index_stmt

drop_database_stmt          ::= DROP DATABASE ( IF EXISTS )? identifier CONFIRM string_literal

drop_table_stmt             ::= DROP TABLE ( IF EXISTS )? identifier ( cascade_clause )?

drop_collection_stmt        ::= DROP COLLECTION ( IF EXISTS )? identifier ( cascade_clause )?

drop_view_stmt              ::= DROP MATERIALIZED? VIEW ( IF EXISTS )? identifier

drop_index_stmt             ::= DROP INDEX ( IF EXISTS )? identifier

cascade_clause              ::= CASCADE ( cascade_depth )? ( cascade_override )?

cascade_depth               ::= integer_literal
                              | INFINITY

cascade_override            ::= SET NULL
                              | SET DEFAULT
                              | DROP

truncate_stmt               ::= truncate_database_stmt
                              | truncate_table_stmt
                              | truncate_collection_stmt

truncate_database_stmt      ::= TRUNCATE DATABASE ( IF EXISTS )? identifier CONFIRM string_literal
                                  ( RESTART IDENTITY )?

truncate_table_stmt         ::= TRUNCATE TABLE identifier ( RESTART IDENTITY )?

truncate_collection_stmt    ::= TRUNCATE COLLECTION identifier ( RESTART IDENTITY )?
```
---

## Utility Statements

```
refresh_view_stmt       ::= REFRESH MATERIALIZED VIEW identifier

explain_stmt            ::= EXPLAIN ANALYZE? VERBOSE? explainable_stmt

explainable_stmt        ::= select_stmt
                          | insert_stmt
                          | update_stmt
                          | delete_stmt

info_stmt               ::= INFO info_target

info_target             ::= TABLE identifier
                         | COLLECTION identifier
                         | VIEW identifier
                         | INDEX identifier
                         | DATABASE
```

---

## DML — SELECT

```
select_stmt         ::= intersect_stmt ( ( UNION | EXCEPT ) ALL? intersect_stmt )*
                          ( ORDER BY order_item ( ',' order_item )* )?
                          limit_clause?

intersect_stmt      ::= select_core ( INTERSECT ALL? select_core )*

select_core         ::= '(' select_stmt ')'
                      | SELECT DISTINCT? select_item (',' select_item)*
                          FROM from_clause
                          ( WHERE expr )?
                          ( group_clause )?
                          ( WINDOW window_definition ( ',' window_definition)* )?

select_item         ::= '*'
                      | table_star
                      | collection_star
                      | expr ( AS identifier )?
                      | function_call AS identifier

table_star          ::= identifier '.' '*'

collection_star     ::= identifier '::' '*'

group_clause        ::= GROUP BY expr (',' expr)*
                          ( HAVING expr )?

order_item          ::= expr ( ASC | DESC )?

from_clause         ::= from_item ( ',' shorthand_join )* ( traditional_join )*

from_item           ::= identifier ( AS? identifier )?
                      | '(' select_stmt ')' AS? identifier

traditional_join    ::= NATURAL? join_type? JOIN from_item join_condition

shorthand_join      ::= identifier
                      | join_type identifier
                      | join_type identifier '(' identifier (',' identifier)* ')'
                      | join_type identifier '(' identifier (',' identifier)* ')' ON shorthand_condition

shorthand_condition ::= identifier '(' identifier ')'
                      | expr

join_condition      ::= ON expr
                      | USING '(' identifier ( ',' identifier )* ')'

join_type           ::= INNER | LEFT | RIGHT | FULL OUTER? | CROSS | OUTER

window_definition    ::= identifier AS '(' window_spec ')'

limit_clause        ::= LIMIT expr ( OFFSET expr )?
                      | OFFSET expr ( ROW | ROWS )? ( FETCH FIRST expr ( ROW | ROWS ) ( ONLY | WITH TIES )? )?
                      | FETCH FIRST expr ( ROW | ROWS ) ( ONLY | WITH TIES )?
```
---

## Window Functions
```
window_function_call    ::= function_call OVER ( identifier | '(' window_spec ')' )

window_spec             ::= ( PARTITION BY expr (',' expr)* )?
                          ( ORDER BY order_item (',' order_item)* )?
                          ( frame_clause )?

frame_clause           ::= ( ROWS | RANGE ) frame_bound
                         | ( ROWS | RANGE ) BETWEEN frame_bound AND frame_bound

frame_bound            ::= UNBOUNDED PRECEDING
                         | UNBOUNDED FOLLOWING
                         | CURRENT ROW
                         | integer_literal PRECEDING
                         | integer_literal FOLLOWING
```

---

## DML — INSERT

```
insert_stmt     ::= INSERT INTO identifier ( '(' identifier ( ',' identifier )* ')' )?
                      ( with_free )?
                    insert_source
                    ( ON CONFLICT conflict_target conflict_action )?
                    ( RETURNING return_item ( ',' return_item )* )?

insert_source   ::= select_stmt
                  | '(' select_stmt ')'
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

return_item     ::= '*'
                  | expr ( AS identifier )?
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
                    ( RETURNING return_item ( ',' return_item )* )?

set_values      ::= set_expr ( ',' set_expr )*

set_expr        ::= identifier '=' expr
```

---

## DML — DELETE

```
delete_stmt     ::= DELETE FROM identifier
                    ( FROM from_clause )?
                    ( WHERE expr )?
                    ( CONFIRM string_literal )?
                    ( RETURNING return_item ( ',' return_item )* )?
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
expr                ::= or_expr

or_expr             ::= and_expr ( OR and_expr )*

and_expr            ::= not_expr ( AND not_expr )*

not_expr            ::= NOT not_expr | comparison

comparison          ::= additive ( operator additive )?
                      | additive IS NOT? NULL
                      | additive IS NOT? MISSING
                      | additive IS NOT? NULL OR MISSING
                      | additive NOT? BETWEEN additive AND additive
                      | additive NOT? IN '(' expr (',' expr)* ')'
                      | additive NOT? IN '(' select_stmt ')'
                      | additive NOT? LIKE string_literal
                      | additive NOT? ILIKE string_literal
                      | exists_expr

operator            ::= '=' | '!=' | '<' | '<=' | '>' | '>='

additive            ::= multiplicative ( ( '+' | '-' ) multiplicative )*

multiplicative      ::= unary ( ( '*' | '/' | '%' ) unary )*

unary               ::= ( '-' | '+' ) unary
                      | primary

primary             ::= literal
                      | function_call
                      | window_function_call
                      | column_ref
                      | exists_expr
                      | '(' select_stmt ')'
                      | '(' expr ')'
                      | case_expr

case_expr           ::= standard_case | shorthand_case

standard_case       ::= CASE when_clause+ ( ELSE expr )? END

when_clause         ::= WHEN expr THEN expr

shorthand_case      ::= CASE shorthand_clause ( ',' shorthand_clause )* ( ELSE expr )? END

shorthand_clause    ::= expr '?' expr

exists_expr         ::= EXISTS '(' select_stmt ')'

function_call       ::= identifier '(' ( expr ( ',' expr )* )? ')'
                      | special_function

column_ref          ::= identifier ( ( '.' | '::' ) identifier )*
                        ( '[' string_literal ']' )*
```

---

## Functions
```
special_function    ::= cast_func
                      | merge_func

cast_func           ::= CAST '(' expr AS data_type ')'

merge_func          ::= MERGE '(' expr ',' expr ( ',' ( LEFT | RIGHT ) )? ')'

unpack_func         ::= UNPACK '(' expr? ')'
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
