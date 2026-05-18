# CQL Grammar

## Notation
- `::=` Definition
- `|` Alternation
- `x?` Zero or one (Optional)
- `x*` Zero or more
- `x+` one or more
- `(x)` grouping
- `'x'` literal terminal
- `UPPER` keyword (case-insensitive)
- `lower` non-terminal

---

## PROGRAM

```
program     ::= statement+

statement   ::= select_stmt ';'
              | create_table_stmt ';'
              | create_collection_stmt ';'
              | drop_stmt ';'
              | alter_stmt ';'
              | insert_stmt ';'
              | update_stmt ';'
              | delete_stmt ';'
              | let_block ';'
              | for_stmt ';'

let_block   ::= let_binding+ select_stmt

let_binding ::= LET identifier '=' '(' select_stmt ')' ','
```

---

## FOR LOOP

```
for_stmt    ::= FOR identifier IN identifier
                DO
                  statement+
                END
```

---

## CREATE TABLE

```
create_table_stmt   ::= CREATE TABLE identifier '(' table_item (',' table_item)* ','? ')'

table_item          ::= field_def
                      | table_constraint
```

---

## CREATE COLLECTION

```
create_collection_stmt  ::= CREATE COLLECTION identifier ( '(' collection_item (',' collection_item)* ','? ')' )?

collection_item         ::= field_def
                          | table_constraint
```

---

## FIELD DEFINITION

```
field_def   ::= identifier data_type column_constraint*

column_constraint       ::= constraint_name? column_constraint_type

column_constraint_type  ::= NOT NULL
                          | NULL
                          | UNIQUE
                          | PRIMARY KEY
                          | SECONDARY KEY
                          | AUTOINCREMENT ( '(' integer_literal ',' integer_literal ')' )?
                          | AUTO
                          | AUTONOW
                          | DEFAULT expression
                          | CHECK '(' expression ')'
                          | REFERENCES identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?
```

---

## TABLE CONSTRAINT

```
table_constraint    ::= constraint_name? table_constraint_type

table_constraint_type   ::= PRIMARY KEY '(' identifier (',' identifier)* ')'
                          | SECONDARY KEY '(' identifier (',' identifier)* ')'
                          | UNIQUE '(' identifier (',' identifier)* ')'
                          | CHECK '(' expression ')'
                          | FOREIGN KEY '(' identifier ')' REFERENCES identifier ( '(' identifier ')' )?
                              ( ON DELETE reference_action )?
                              ( ON UPDATE reference_action )?

constraint_name     ::= CONSTRAINT identifier
```

---

## REFERENCE ACTION

```
reference_action    ::= CASCADE
                      | SET NULL
                      | SET DEFAULT
                      | RESTRICT
                      | NO ACTION
```

---

## DATA TYPES

```
data_type   ::= INT
              | BIGINT
              | FLOAT
              | DOUBLE
              | VARCHAR ( '(' integer_literal ')' )?
              | TEXT
              | BOOLEAN
              | TIMESTAMP
              | DATE
              | JSON
```

---

## ALTER STATEMENT
```
alter_stmt    ::= ALTER TABLE identifier alter_cmd

alter_cmd     ::= ADD COLUMN field_def
                | ADD table_constraint
                | RENAME TO identifier
                | DROP COLUMN identifier
                | RENAME COLUMN identifier TO identifier
                | MODIFY COLUMN field_def
```

---

## SELECT

```
select_stmt ::= SELECT DISTINCT? select_item (',' select_item)*
                FROM from_clause
                ( WHERE expr )?
                ( group_clause )?
                ( ORDER BY order_item (',' order_item)* )?
                ( LIMIT integer_literal ( OFFSET integer_literal )? )?

group_clause ::= GROUP BY expr (',' expr)*
                  ( HAVING expr )?

select_item ::= '*' | expr ( AS? identifier )?

order_item  ::= expr ( ASC | DESC )?
```

---

## FROM

```
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

---

## INSERT

```
insert_stmt     ::= INSERT INTO identifier ( '(' identifier ( ',' identifier )* ')' )?
                    insert_source

insert_source   ::= select_stmt
                  | VALUES '(' insert_value ( ',' insert_value )* ')'
                        ( ',' '(' insert_value ( ',' insert_value )* ')' )*

insert_value    ::= expr | DEFAULT
```

---

## UPDATE

```
update_stmt ::= UPDATE identifier
                SET set_values
                ( WHERE expr )?

set_values  ::= set_expr ( ',' set_expr )*

set_expr    ::= identifier '=' expr
```

---

## DELETE

```
delete_stmt ::= DELETE FROM identifier
                ( WHERE expr )?
```

---

## EXPRESSIONS

```
expr              ::= or_expr

or_expr           ::= and_expr ( OR and_expr )*

and_expr          ::= not_expr ( AND not_expr )*

not_expr          ::= NOT not_expr | comparison

comparison        comparison ::= additive ( operator additive )?
                               | additive IS NOT? NULL
                               | additive IS NOT? MISSING
                               | additive IS NOT? NULL OR MISSING
                               | additive NOT? BETWEEN additive AND additive
                               | additive NOT? IN '(' expr (',' expr)* ')'
                               | additive NOT? IN '(' select_stmt ')'
                               | additive NOT? LIKE string_literal
                               | additive NOT? ILIKE string_literal
                               | EXISTS '(' select_stmt ')'

operator          ::= '=' | '!=' | '<>' | '<' | '<=' | '>' | '>='

additive          ::= multiplicative ( ( '+' | '-' ) multiplicative )*

multiplicative    ::= unary ( ( '*' | '/' | '%' ) unary )*

unary             ::= ( '-' | '+' ) unary
                    | primary

primary           ::= literal
                    | function_call
                    | column_ref
                    | exists_expr
                    | '(' select_stmt ')'
                    | '(' expr ')'
                    | case_expr

case_expr         ::= standard_case | shorthand_case

standard_case     ::= CASE when_clause+ ( ELSE expr )? END

when_clause       ::= WHEN expr THEN expr

shorthand_case    ::= CASE shorthand_clause ( ',' shorthand_clause )* ( ELSE expr )? END

shorthand_clause  ::= expr '?' expr

exists_expr       ::= EXISTS '(' select_stmt ')'

function_call     ::= identifier '(' ( expr (',' expr)* )? ')'

column_ref        ::= identifier ( ( '.' | '::' ) identifier )*
                      ( '[' string_literal ']' )*
```

---

## LITERALS & PRIMITIVES

```
literal         ::= integer_literal
                  | float_literal
                  | string_literal
                  | boolean_literal
                  | NULL

integer_literal ::= digit+

float_literal   ::= digit+ '.' digit+

string_literal  ::= "'" any_char* "'"

boolean_literal ::= TRUE | FALSE

identifier      ::= letter ( letter | digit | '_' )*

letter          ::= 'a'..'z' | 'A'..'Z' | '_'

digit           ::= '0'..'9'
```
