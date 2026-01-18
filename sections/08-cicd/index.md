---
title: CI/CD
has_children: false
nav_order: 9
---

# CI/CD

This section summarizes the CI/CD automation implemented with GitHub Actions, based on `.github/workflows/check.yml` and `.github/workflows/deploy.yml`. The purpose is to enforce quality gates on code changes and to automate releases in a repeatable manner.

## Overview

- **What is automated**: dependency setup, basic quality checks, tests, coverage reporting, and release publication.
- **Why**: to keep quality gates consistent across pull requests and to release from a repeatable, auditable pipeline.
- **How**: GitHub Actions workflows run on pushes, PRs, and manual dispatch.

## Continuous Integration (check.yml)

- **Triggers**: runs on pushes, pull requests, and manual dispatch, with ignores for routine metadata changes.
- **What it does**: installs dependencies, runs compile/type checks, executes tests, and produces a coverage report.
- **Where it runs**: tests are exercised across major OSes and multiple Python versions.

## Continuous Delivery (deploy.yml)

- **Scope**: there is no deployment of a hosted web app. The app is run locally, so “delivery” here means automated *release of artifacts* (PyPI + GitHub Releases) rather than shipping to a server.
- **When it runs**: after CI succeeds, a release workflow handles versioning and publishing.
- **Release behavior**: releases are automated on the main branch and are dry-run elsewhere.

## Secrets and environment variables

- **Required secrets** (in GitHub Actions):
    + `PYPI_USERNAME` and `PYPI_PASSWORD` for publishing to PyPI.
    + `RELEASE_TOKEN`, mapped to `GITHUB_TOKEN`, for creating GitHub Releases.
- **Derived environment flags**:
    + `RELEASE_DRY_RUN` toggles real publishing depending on branch and initial-commit checks.
    + `RELEASE_TEST_PYPI` enables TestPyPI publishing when running from a template repo.
