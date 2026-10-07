# Python API

LinkDB does not yet have a public, embeddable Python API — there is no
`connect()`, `Connection`, or result object you can `import` and call from
your own code.

Today the only way to use LinkDB from Python is the [CLI](../cli/index.md),
an interactive REPL process. Internally the CLI drives the engine through
`LinkDBState` in `src/linkdb/cli/state.py`, but that class is a CLI
implementation detail, not a supported public interface: its shape can
change without notice and it is not exposed as part of the `linkdb` package.

A real public API — connect to a database, execute a statement, get results
back, matching the shape implied by this section's page titles ([Connection](connection.md),
[Query Execution](query_execution.md), [Results](results.md)) — is wanted
and planned for a future effort, but nothing has been designed or built
yet. These pages will get real content once that work exists.
