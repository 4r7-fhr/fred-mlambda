# CLAUDE.md

How to set up this repository for development and run the same checks CI runs.
For what the package does and how to use it, see `README.md`.

## Layout

- `fred-mlambda/` — the Python package (`setup.py`, `requirements.txt`, `README.md`).
- `fred-mlambda/src/main/fred/mlambda` — package source (namespace package `fred.mlambda`).
- `fred-mlambda/src/test` — unit tests (pytest).
- Repository root — development tooling:
  - `requirements-develop.txt` — development dependencies (pytest, coverage, pycodestyle, mypy, …).
  - `tests-unit.sh`, `tests-coverage.sh`, `check-style.sh`, `check-types.sh` — check scripts.
  - `.github/workflows/` — CI (`check-tests.yaml`, `check-style.yaml`, `check-types.yaml`, `check-wheel.yaml`).

Every script takes the package directory as its first argument. Run them from the repository root
and pass `fred-mlambda`, exactly as CI does.

## Setup

CI uses Python 3.12 (`setup.py` requires `>=3.11`). Use a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

1. Install the development dependencies (same as the CI "Install Requirements" step):

   ```bash
   python -m pip install --upgrade pip
   python -m pip install -r requirements-develop.txt
   ```

   `requirements-develop.txt` pins `typed-ast==1.5.5`, which has no prebuilt wheel for Python 3.12
   and is compiled from source, so a C compiler (e.g. `gcc`) must be available.

2. Install the package in editable mode (same as the CI "Install Package" step):

   ```bash
   python -m pip install -e ./fred-mlambda
   ```

   The package's runtime dependencies come from `fred-mlambda/requirements.txt`, which `setup.py`
   reads as `install_requires`, so this step installs them too.

## Checks

Run from the repository root.

| Check | Command | What it runs | CI workflow |
|-------|---------|--------------|-------------|
| Unit tests | `bash tests-unit.sh fred-mlambda` | `cd fred-mlambda; coverage run -m pytest src/test` | `check-tests.yaml` |
| Coverage report | `bash tests-coverage.sh fred-mlambda` | `cd fred-mlambda; coverage report --omit="test_*","*_remote_module_non_scriptable.py"` | `check-tests.yaml` |
| Style | `bash check-style.sh fred-mlambda` | `pycodestyle fred-mlambda --max-line-length 120` | `check-style.yaml` |
| Types | `bash check-types.sh fred-mlambda` | `mypy fred-mlambda --ignore-missing-imports --install-types --non-interactive` | `check-types.yaml` |

`tests-coverage.sh` reports on the `.coverage` data written by `tests-unit.sh`, so run the unit
tests first. CI runs them in that order:

```bash
bash tests-unit.sh fred-mlambda
bash tests-coverage.sh fred-mlambda
bash check-style.sh fred-mlambda
bash check-types.sh fred-mlambda
```

Style allows lines up to 120 characters. `check-types.sh` may install missing type stubs
(`--install-types --non-interactive`).

CI also runs `check-wheel.yaml` on pull requests, which builds a wheel with
`(cd fred-mlambda; python setup.py bdist_wheel)` and imports `fred.mlambda.version`. It has no
root script.
