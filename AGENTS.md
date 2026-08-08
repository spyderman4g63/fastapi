# AGENTS.md

## Cursor Cloud specific instructions

This repository is the **FastAPI** framework (a Python library), not a standalone deployable app. Dependencies are managed with [`uv`](https://docs.astral.sh/uv/); Python 3.11 is pinned in `.python-version` and installed/managed by `uv`. The startup update script runs `uv sync --extra all`, so the `.venv` is already populated when a session begins.

### Services / components

- The "product" is the `fastapi` package itself. There is no long-running backend service to run for development. To exercise it end to end, run any FastAPI app with the installed CLI (e.g. `fastapi dev <module>.py`).

### Running things (non-obvious notes)

- Always run project commands through `uv run ...` (or use the venv binaries in `.venv/bin/`) so the correct Python 3.11 environment and installed extras are used. The helper scripts (`scripts/lint.sh`, `scripts/test.sh`, `scripts/test-cov.sh`) assume this environment.
- Lint: `uv run bash scripts/lint.sh` (runs `mypy fastapi`, `ty check`, `ruff check`, and `ruff format --check`).
- Test: `uv run bash scripts/test.sh` (runs `pytest -n auto`; `scripts/test.sh` sets `PYTHONPATH=./docs_src`, which the `docs_src` tutorial tests require). Do not run `pytest` directly without that `PYTHONPATH` or many tests will fail to import.
- The `inline-snapshot` plugin is disabled under `pytest-xdist` (parallel run) — this is expected and prints an informational message; it does not indicate a failure.
- Coverage in CI is enforced at 100% by combining runs across many OS/Python matrix cells, so a single local `test-cov.sh` run on one interpreter will not reach 100% by itself.
- To demonstrate the framework, `fastapi dev /path/to/app.py` starts a hot-reloading Uvicorn server; interactive Swagger docs are served at `/docs`.
