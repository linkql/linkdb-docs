# Installation

LinkDB is MVP-stage software. It is **not** published to PyPI — there is an
unrelated package already named `linkdb` on PyPI (a CLI bookmark manager),
so `pip install linkdb` will install the wrong thing. Install a release
wheel, or install from source.

The distribution is named **`linkdbms`**, but you import it and run it as
`linkdb`.

## Requirements

- Python 3.12 or later (`requires-python = ">=3.12"`).
- Git, with submodule support (only for installing from source).

## Install a release wheel

A wheel is a single file that bundles the CLI, the Python API, and every
engine package, so there are no other packages or submodules to install.
Use a fresh virtual environment:

```sh
python3 -m venv .venv
source .venv/bin/activate        # .venv\Scripts\activate on Windows
pip install ./linkdbms-<version>-py3-none-any.whl
```

Releases are attached to GitHub releases on
[`linkql/linkdb`](https://github.com/linkql/linkdb). To build a wheel
yourself from a full checkout (submodules included):

```sh
python -m pip install build
python -m build --wheel
```

The wheel appears in `dist/`. To upgrade, install the newer wheel with
`pip install --upgrade`.

## Install from source

LinkDB's engine is split across several packages — `linkdb-core`,
`linkdb-lexer`, `linkdb-parser`, `linkdb-analyzer`, `linkdb-planner`,
`linkdb-executor`, `linkdb-catalog`, and `linkdb-storage` — each developed
in its own Git repository and pulled into the main [`linkdb`
repo](https://github.com/linkql/linkdb) as a submodule under `src/linkdb/`.
None of these are on PyPI either, so they need to be installed as editable
checkouts alongside the main package.

1. Clone the repository with its submodules:

    ```sh
    git clone --recurse-submodules https://github.com/linkql/linkdb.git
    cd linkdb
    ```

    If you already cloned without `--recurse-submodules`, fetch the
    submodules afterward:

    ```sh
    git submodule update --init --recursive
    ```

2. Create and activate a virtual environment:

    ```sh
    python3 -m venv .venv
    source .venv/bin/activate
    ```

3. Install the project's `requirements.txt`, which editable-installs every
   engine package plus the main `linkdb` package in one step:

    ```sh
    pip install -r requirements.txt
    ```

## Check the install

Either route puts a `linkdb` console script on your `PATH` — it's the
`[project.scripts]` entry `linkdb = "linkdb.cli.repl:main"` from
`pyproject.toml`. Run it to confirm the install:

```sh
linkdb
```

`\q` leaves the CLI. To check the Python API:

```sh
python -c "import linkdb; print(linkdb.connect().execute('SELECT 1').rows)"
```

See [CLI](cli/index.md) for what to do once the CLI is running, or [Python
API](api/index.md) to use LinkDB from your own code.
