---
title: Development
has_children: false
nav_order: 5
---

# Development

## DVCS

Distributed Version Control System (Git and GitHub) was used throughout the project. Although the work was done by a single developer, branches, PRs, and issues were still used to document the workflow, and the local and remote repositories were kept aligned through frequent pull/push operations.

- Branching follows git-flow conventions: `dev` is the integration branch and `main` is the release branch; new work is developed on feature branches (e.g., `feature/manual-meal-entry`) and merged via PRs.
- Commit messages follow Conventional Commits (`feat`, `chore`, `refactor`, `feat(ui)`, etc.). Releases are automated with semantic-release, producing `chore(release)` commits and updating `CHANGELOG.md`.
- Pull requests are the standard integration path; examples include PR #6 (feature branch to `dev`) and multiple closed PRs such as #8-#12. CI checks are run in GitHub Actions on PRs.
- Issues are used to plan and track work (e.g., #1 "Add user to the app" and #2 "CI is failing"), and issue references appear in the changelog (e.g., #3-#5).
- Code reviews are lightweight. Given the single-developer setup, PR reviews were informal, with CI checks serving as the primary quality gate.

## Implementation details

- Network protocols: HTTPS is used for requests to the API via the `requests` library.

- In-transit data representation: JSON is used for API responses, matching Open Food Facts v2 and enabling straightforward parsing in Python.

- Database queries: SQLite is queried with SQL via `sqlite3`, which fits a local single-file store (`DB_PATH`, default `data/app.sqlite`).

- Authentication: local username/password with salted PBKDF2-HMAC-SHA256 hashing; credentials are stored in SQLite and session state is managed in Streamlit.

- Authorization: per-user access is enforced by filtering queries on `user_id` in the repository/service layer; no role-based model is needed for the current scope.

## Technological details

- Languages/tools: Python 3.10+ is used for the application, and Node.js is used only for release automation with semantic-release.

- Frameworks/libraries: Streamlit for the UI, `requests` for Open Food Facts API calls, `pandas` for tables/charts, and `sqlite3` for persistence. Dev tooling includes Poetry, mypy, coverage, and unittest.

- External services: Open Food Facts API for product data; GitHub Actions plus semantic-release for CI/CD and automated releases; PyPI as the publication target.
