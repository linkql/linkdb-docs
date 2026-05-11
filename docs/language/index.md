# Link Query Language
**LinkQL** (Link Query Language) is the query language for **Link**, a database system designed to manage **relational** and **non-relational** data in a single, unified system. Rather than maintaining separate database systems and writing glue code between them or using a third-party software, Link lets you define, query, and relate both structured tables and flexible document collections using one consistent syntax.

## Core Concepts
### Tables
Tables are structured, predefined data-stores -- identical to tables in any other relational database. Every column and its type must be defined before data can be inserted. Tables are best suited for data with a consistent, well-known shape.

### Collections
Collections are document stores where each document can have its own shape. Collections can optionally define **required fields**, which every document in that collection must contain. Any fields beyond those are termed **free fields** -- they can exists on some documents and not others with no penalty.

### Defined Fields vs Free Fields
When you create a collection with fields, those become **defined fields**. Every document in that collection must include them. Any additional fields you add to a document beyond the declared ones are **free fields** -- schemaless and flexible. A collection created with no defined fields is entirely schemaless.

### Querying Across Tables/Collections
LinkQL lets you join tables and collections in a single query. Foreign key references work across both, and the engine resolves whether a target is a table or collection automatically. This means your relational and document data can reference eath other directly without any application-level translations.

## Language Reference
- **[Data Types](data_types.md)** — All primitive and structured types available in LinkQL
- **[DDL](ddl/create.md)** — Creating, altering, and dropping tables, collections, and databases
- **[DML](dml/select.md)** — Querying and manipulating data
- **[Transactions](transactions.md)** — Transaction behavior and syntax
- **[Functions](functions.md)** — Built-in functions available in LinkQL
