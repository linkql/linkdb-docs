# LinkDB / LinkQL — Notes Beyond the Parser

Pulled together from past conversations to prep for moving past the parser stage.

---

## 1. Pipeline Architecture

Modeled directly on Postgres's three-layer translation from SQL text to execution,
but made an explicit four-stage pipeline for LinkDB (partly for learning value, partly
to keep concerns clean):

```
Parser    → Parse Tree   (purely syntactic, no catalog access)
Analyzer  → Query Tree   (resolved against the catalog)
Planner   → Plan Tree    (physical execution strategy)
Executor  → walks the plan tree, reads/writes via storage
```

**Parser**
- Tokens → parse tree. Doesn't know or care if referenced tables/columns exist.
- No catalog access — stays clean and isolated (matches Postgres's `SelectStmt`/`RangeVar`
  layer, which is a literal structural translation of the SQL with no validation).

**Analyzer** (`linkdb-analyzer`, not yet built)
- Parse tree → query tree.
- This is where `linkdb-catalog` gets consulted for the first time.
- Resolves whether an identifier is a table or a collection.
- Validates that referenced columns/fields actually exist.
- Resolves types.
- Roughly Postgres's `Query`/`RangeTblEntry`/`TargetEntry` layer — a `RangeVar("bar")`
  becomes something that knows `bar` is a table with known columns and an OID; a
  `ColumnRef("foo")` becomes something that knows it's column 3, type VARCHAR.
- In Postgres this layer also does view rewriting and permission checks

**Planner** (`linkdb-planner`, not yet built)
- Query tree → plan tree.
- Decides *how* to execute (scan strategy, join strategy).

**Executor** (`linkdb-executor`, not yet built)
- Walks the plan tree, performs the actual reads/writes through `linkdb-storage`.

### Why parallel structures instead of one AST that grows annotations
Discussed and rejected the "single AST with optional resolved-fields" approach in favor
of genuinely separate node hierarchies per stage (parse tree nodes vs. query tree nodes
vs. plan tree nodes), which is what Postgres, Apache Calcite (`SqlNode` vs `RelNode`),
and MongoDB's internal query engine all do. The more sophisticated the system, the more
it tends toward parallel structures rather than a single mutating tree.

### Resulting dependency graph

```
linkdb-parser    → linkdb-core only
linkdb-analyzer  → linkdb-core + linkdb-parser + linkdb-catalog
linkdb-planner   → linkdb-core + linkdb-analyzer + linkdb-catalog
linkdb-executor  → linkdb-core + linkdb-planner + linkdb-catalog + linkdb-storage
```

---

## 2. Catalog's Role

The catalog is the thing that makes the relational/document bridge actually work
mechanically — it's consulted by the analyzer (and later planner/executor) any time
the engine needs to answer "is this a table or a collection, and what does it look
like":

- Resolves whether a `REFERENCES name(column)` target, a `FROM`/join source, or a
  bare identifier is a table or a collection.
- Validates that a referenced column/field exists before letting a query proceed.
- Will need to track defined fields vs. free fields per collection, primary/foreign
  key structure, and constraint names (since `DROP CONSTRAINT`/`MODIFY CONSTRAINT`
  address constraints by name only — the catalog needs to be the source of truth
  for name → constraint-type lookups).
- Backed by the per-database `system/` JSON files (`tables.json`, `collections.json`,
  `relationships.json`) at the storage layer, but the catalog itself should be the
  in-memory/query-facing abstraction over those files, not a pass-through.

---

## 3. Storage Layer (MVP)

- Flat JSON files, no external DB engine dependency for the MVP.
- Per database:
  - `/database_name/system/` — `tables.json`, `collections.json`, `relationships.json`
  - `/database_name/data/` — one file per table or collection
- Deliberately abstracted behind a storage interface so the backend can be swapped
  later (e.g. for a binary format, SQLite-backed storage, etc.) without touching the
  executor.
- Single-user for MVP — concurrency control is explicitly deferred, not designed yet.
- Foreign key validation is always deferred to commit time, never inline — this is a
  storage/transaction-layer concern as much as a parser one, so the executor's write
  path needs a commit-time validation pass baked in from the start rather than bolted
  on later.

---

## 4. Open Design Question: Variable-Depth / Schema-Agnostic Querying

Flagged early as a deferred topic and still not resolved:

- How does LinkQL handle querying into nested, variable-depth, or unknown-schema data
  (the core differentiator vs. plain SQL)?
- The current placeholder answer is `EXPAND(field, depth)` — a SELECT-level aggregate
  that flattens nested collection fields into tabular columns, with depth defaulting
  to 1, filtered via `HAVING`. This is documented in memory but:
  - **Not yet in `functions.md` or either grammar file** — it's a real gap, not just
    a docs sync issue.
  - Its exact grammar shape is still open: is it a plain two-arg function call
    (`identifier, integer_literal` with parser-supplied default, same pattern as
    `MERGE`'s defaulted `direction`), or does it need SELECT/HAVING-level grammar
    beyond an ordinary function call (more like how `COUNT` interacts specially with
    `GROUP BY`)? This needs to be decided before it can get a proper AST node.
  - This was called out as **the most important functional gap to close** before the
    grammar can be considered complete, since it's central to the relational/document
    bridging pitch — worth prioritizing in analyzer/planner design too, since
    flattening nested fields into tabular output has real implications for how the
    query tree and plan tree represent result shape.

---

## 5. Other Named Gaps from the Grammar Completeness Review

Not yet decided, surfaced during a grammar audit pass, relevant to scope planning
beyond the parser:

- **String functions**: have `CONCAT`, `UPPER`, `LOWER`, `TRIM`, `LENGTH`, `REPLACE`;
  missing `SUBSTRING`, `SPLIT`, `POSITION`/`INSTR`, `LPAD`/`RPAD`.
- **Array functions**: have `COUNT_ARRAY`, `CONTAINS`; missing Postgres-style
  `ARRAY_APPEND`, `ARRAY_LENGTH`, `UNNEST`, slicing. Flagged as likely expected by
  MongoDB-background users given `ARRAY` is first-class.
- **Comments**: no `--` or `/* */` syntax defined anywhere yet — lexer-level, but a
  near-certain day-one CLI usability issue.
---
