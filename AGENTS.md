# AutoGluon

AutoGluon is a monorepo of AutoML Python packages. The repo contains 7 sub-packages, each independently installable: `common`, `core`, `features`, `tabular`, `multimodal`, `timeseries`, `eda`.

## Build & Test Commands

### Install a package for development (install dependencies first in order)

```bash
# Minimal chain for tabular work
pip install -e "common/[tests]"
pip install -e "core/[all,tests]"
pip install -e "features/"
pip install -e "tabular/[all,tests]"

# For timeseries work
pip install -e "common/[tests]"
pip install -e "core/[all,tests]"
pip install -e "timeseries/[tests]"

# For multimodal work
pip install -e "common/[tests]"
pip install -e "core/[all,tests]"
pip install -e "multimodal/[tests]"
```

### Run unit tests locally (no external services required)

```bash
cd common     && python -m pytest tests/unittests/
cd core       && python -m pytest tests/unittests/
cd features   && python -m pytest tests/features/    # no unittests/ subdir — tests live here
cd tabular    && python -m pytest tests/unittests/
cd timeseries && python -m pytest tests/unittests/
cd multimodal && python -m pytest tests/unittests/
cd eda        && python -m pytest tests/unittests/
```

Do NOT run `tests/` directly for tabular (has `regressiontests/`) or timeseries (has `smoketests/`) — those need external infrastructure and long runtimes and are CI-only.

### Run a single test file or test

```bash
python -m pytest path/to/test_file.py
python -m pytest path/to/test_file.py::test_function_name
```

### Lint (check only — what CI runs)

```bash
ruff format --diff common/ core/ features/ tabular/ multimodal/ timeseries/
ruff check --select I common/ core/ features/ tabular/ multimodal/ timeseries/
ruff check timeseries/
```

### Lint (auto-fix — run before committing)

```bash
ruff format common/ core/ features/ tabular/ multimodal/ timeseries/
ruff check --fix --select I common/ core/ features/ tabular/ multimodal/ timeseries/
```

### Lint a single file

```bash
ruff format path/to/file.py
ruff check --fix --select I path/to/file.py
```

### Type check (timeseries only — pyright is configured for this package)

```bash
pyright timeseries/src/
```

### Pre-commit hooks (optional, run all checks at once)

```bash
pre-commit install   # one-time setup
pre-commit run       # run on staged files
```

## Architecture

### Package dependency order (bottom → top)

```
common → core → features → tabular
                         → multimodal
                         → timeseries
                         → eda
```

Each package uses the `autogluon.*` namespace (e.g. `autogluon.tabular`, `autogluon.timeseries`).

### Where to find things

| Concern | Location |
|---|---|
| Shared utilities | `common/src/autogluon/common/` |
| Model base classes, HPO, metrics, learner | `core/src/autogluon/core/` |
| Feature generators | `features/src/autogluon/features/` |
| Tabular predictors & models | `tabular/src/autogluon/tabular/` |
| Time series predictors & models | `timeseries/src/autogluon/timeseries/` |
| Multimodal predictors | `multimodal/src/autogluon/multimodal/` |
| Exploratory data analysis | `eda/src/autogluon/eda/` |
| Per-package tests | `<pkg>/tests/unittests/` (or `features/tests/features/`) |
| CI scripts | `.github/workflow_scripts/` |
| Dependency version ranges | `core/src/autogluon/core/_setup_utils.py` |

### Tests structure (per package)

| Tier | Directory | Locally safe? |
|---|---|---|
| Unit tests | `tests/unittests/` (or `tests/features/` for the `features` package) | Yes |
| Smoke tests | `tests/smoketests/` (timeseries only) | No — requires models and long runtimes |
| Regression tests | `tests/regressiontests/` (tabular only) | No — requires datasets and infra |
| Style check | `tests/test_check_style.py` | Yes — runs ruff checks |

Slow tests within unit test files are gated by `@pytest.mark.slow` and skipped by default. CI passes `--runslow` to enable them.

## Key Conventions

- **Lazy imports for optional deps**: use `autogluon.common.utils.try_import` instead of top-level imports for any dependency that is not universally installed.
- **NumPy-style docstrings** on all public APIs (see `numpydoc` format reference).
- **Minimize new dependencies**: prefer existing packages; discuss any new dependency in the PR.
- **Small, fast unit tests**: train on tiny data subsamples with minimal iterations. Tests that take more than a few seconds are too slow for unit tests.
- **`__init__.py` unused imports are allowed**: ruff rule F401 is ignored in `__init__.py` files.

## PR Conventions

- Reference the issue number in the PR description: `*Issue #123*`
- CI runs automatically on every PR — don't need to run the full suite locally
- Lint failures block merge — always run `ruff format` and `ruff check --fix` before pushing
- Commit messages: imperative, concise (`fix: ...`, `feat: ...`, `refactor: ...`)
