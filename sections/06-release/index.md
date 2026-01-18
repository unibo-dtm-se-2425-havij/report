---
title: Release
has_children: false
nav_order: 7
---

# Release

## Artefacts and release targets

- This project is a local Streamlit application rather than a library meant for general distribution. Even so, it is packaged as a single Python artefact (`havij_nutrition`), and each release builds an sdist (`.tar.gz`) and a wheel (`.whl`) via Poetry in `dist/`.
- Automatic publishing to PyPI is configured to demonstrate a full release workflow (versioning, tagging, build, and publication). The same `dist/*` files are attached to GitHub Releases for traceability. A TestPyPI target (`pypi-test`) is available for test publications.

## Release automation and commands

- Releases are automated by GitHub Actions. `.github/workflows/check.yml` runs the quality gates and then calls `.github/workflows/deploy.yml`; the deploy job runs `npx semantic-release` and performs a real publish only on `main`/`master` (other branches are dry runs).
- `release.config.mjs` configures `semantic-release` to:
   + compute the next version from Conventional Commits,
   + run `poetry version -- <nextRelease.version>`,
   + run `poetry publish --build` (or `--repository pypi-test`),
   + update `CHANGELOG.md`, create the git tag, and publish a GitHub Release with `dist/*` assets.
- Required secrets: `PYPI_USERNAME` (usually `__token__`), `PYPI_PASSWORD` (PyPI API token), and `RELEASE_TOKEN` (exported as `GITHUB_TOKEN` for GitHub releases).

### Manual/CLI release (same steps as CI)

```bash
poetry install
npm install
PYPI_USERNAME=__token__ PYPI_PASSWORD=... GITHUB_TOKEN=... npx semantic-release --branches main
```

## Choice of the license

- Code and artefacts are released under the Apache 2.0 License (see `LICENSE` and `pyproject.toml`). It is permissive, allows commercial reuse and redistribution, and includes an explicit patent grant while requiring attribution and a license notice.

## Choice of the versioning schema

- The project follows SemVer (MAJOR.MINOR.PATCH) via `semantic-release`, using Conventional Commits to infer the version bump.
- `feat:` increments MINOR, `fix:` increments PATCH, and `BREAKING CHANGE:` or `!` in the type increments MAJOR. `semantic-release` writes the new version to `pyproject.toml`, updates `CHANGELOG.md`, and tags the release (e.g., `1.2.0`).
- All artefacts share the same version: the PyPI package version, the git tag, and the GitHub Release name are aligned by `semantic-release`.

### Creating a new version

1. Work on `feature/*` branches off `dev` (git-flow). When ready, optionally use a `release/*` branch for stabilization.
2. Ensure commits follow Conventional Commits and CI passes.
3. Merge into `main`/`master`. This triggers the deploy workflow, which tags the release, updates `CHANGELOG.md` and `pyproject.toml`, and publishes to PyPI.
