---
title: Developer guide
has_children: false
nav_order: 11
---

# Developer Guide

This guide helps new contributors get productive in the Havij codebase and follow the team conventions.

## Contact and issue reporting

- Use GitHub Issues to report bugs and propose changes; link issues from your PR description when possible.
- For questions that do not fit an issue, contact the maintainer listed in `pyproject.toml`:
  **Farideh Tavakoli** (`farideh.tavakoli@studio.unibo.it`).

## Project structure

The app is organized as a small layered architecture:

- `havij/domain/`: core domain models (`meal`, `product`, `nutrients`, `user`) and validation rules in `havij/domain/rules.py`.
- `havij/application/`: service layer (meal, product, user) and ports.
- `havij/infrastructure/`: persistence (SQLite repositories) and Open Food Facts API adapter.
- `havij/presentation/`: Streamlit UI, with the entry point at `havij/presentation/streamlit_app.py`.
- `tests/`: unit tests organized by domain, application, and infrastructure layers.

## Development environment

Prerequisites:
- Python 3.10+
- Poetry

Setup (from the repo root):
```bash
poetry install
```

Run the app:
```bash
poetry run streamlit run havij/presentation/streamlit_app.py
```

Optional: override the SQLite path (default is `data/app.sqlite`):
```bash
DB_PATH="data/app.sqlite" poetry run streamlit run havij/presentation/streamlit_app.py
```

## Quality gates and tests

The CI pipeline (see `.github/workflows/check.yml`) enforces these commands:

- Compile check: `poetry run poe compile`
- Type checks: `poetry run poe mypy`
- Unit tests: `poetry run poe test`
- Coverage run/report: `poetry run poe coverage` and `poetry run poe coverage-report`

Run these locally before opening a PR to avoid CI failures.

## Workflow and conventions

- **Branching**: `dev` is the integration branch. Create `feature/*`, `release/*`, or `hotfix/*` branches off `dev`.
- **PRs**: open a pull request into `dev` and ensure CI checks pass.
- **Commit messages**: follow Conventional Commits (`feat`, `fix`, `chore`, etc.). Releases are automated via semantic-release when `main` is updated.
- **Coding style**: keep domain logic in `havij/domain/`, IO and persistence in `havij/infrastructure/`, and keep the Streamlit UI thin. Use type hints; `mypy` is part of the quality gates.

## Tooling notes

- Use `poetry run ...` for all commands to ensure the correct virtual environment.
- Tests use the Python `unittest` framework; add new tests under `tests/` and keep them focused on a single layer.
