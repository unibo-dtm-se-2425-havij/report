---
title: Home
layout: home
has_children: false
nav_order: 1
---

# Project title

### Authors

- [Farideh Tavakoli](mailto:farideh.tavakoli@studio.unibo.it)

## Abstract

Havij is a lightweight nutrition tracking app focused on barcode-based product lookup
and simple meal logging. The system integrates the Open Food Facts API to retrieve
product details and nutrition values per 100 g, then stores user-selected entries in a
local SQLite database for persistence. A Streamlit Cloud demo is available at
https://havij-nutrition.streamlit.app/; the hosted instance uses ephemeral storage, so
user data does not persist across restarts, while local runs keep the SQLite-based
persistence. The Streamlit interface includes user accounts
with sign up and login. In the `Log Meal` tab, users can look up a barcode to auto-fill
nutrition data or enter values manually, then log portions in grams. The `Today` tab
shows the day log with per-entry nutrient breakdowns, daily totals for calories,
protein, carbs, and fat, and a control to remove entries. The `Last 7 Days` tab provides
a summary table and a line chart to track trends over time, and the `Profile` tab shows
basic account metadata. The codebase follows a layered architecture with domain models and
rules, application services for business logic, infrastructure adapters for external
API and persistence, and a presentation layer for the UI. This separation keeps the
core logic testable and makes it straightforward to swap storage or data sources. The
project targets Python 3.10+ and is configured with Poetry for dependency management,
with automated checks for type safety and test coverage.

## Disclaimer (if needed)

During the preparation of this work, the author used OpenAI Codex to refine and edit
text content and to generate portions of the codebase. After using this tool, the
author reviewed and edited the content as needed and takes full responsibility for the
final report.

